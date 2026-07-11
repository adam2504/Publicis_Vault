---
type: project
statut: en cours
discipline: data-science
implication: onboardé
client: STELLANTIS
---
# EVBB Scoring

**EVBB** = Enhanced Value-Based Bidding. Stellantis utilise le VBB pour définir une valeur de conversion sur plusieurs micro-conversions. L'objectif est de construire un modèle prédictif pour définir la valeur de chaque micro-conversion et modèle de voiture, afin d'optimiser le custom bidding entre pages (pas versus concurrents).

Dragonfly (équipe interne UK) gérait les POC EVBB en France. Désormais le marché européen est divisé : Publicis prend en charge certains pays avec sa propre méthodologie.

[Confluence — Stellantis Enhanced VBB](https://confluence.publicismedia.com/spaces/PMTFR/pages/892863970/Stellantis+Enhanced+VBB#StellantisEnhancedVBB-Context)

---

## Infrastructure GCP

- **Projet de travail** : `gbl-psa-analytics-publicis` — dataset : `enhanced_vbb_pmf`
- **Ne pas créer de tables/views/modèles en dehors de ce projet**
- **Données GA4** (read only) : `gbl-psa-analytics-cdf-prod` — dataset `analytics_ga4_eu`
- Accès via navigateur séparé ou navigation privée (comptes Stellantis distincts)

### Procédure d'accès

1. Créer un compte Stellantis (contact : Thibaut VOULOIR) via Smartsheet Form
2. Se connecter à `useraccount.fcagroup.com` et changer le mot de passe Stellantis
3. Demander accès BigQuery admin à `gbl-psa-analytics-publicis` + read sur `analytics_ga4_eu` du projet `gbl-psa-analytics-cdf-prod` (contact technique : Iker ZABALA IBANEZ)
4. Définir le mot de passe Google sur `useraccount.fcagroup.com` → section "Manage your account password"
5. Se connecter à GCP : `console.cloud.google.com/bigquery?project=gbl-psa-analytics-publicis`
6. Mettre en favoris le projet `gbl-psa-analytics-cdf-prod`

---

## Périmètre géographique

| Marque | Pays | Brand ID | Country ID | Modèles | Statut |
| ------ | ---- | -------- | ---------- | ------- | ------ |
| Peugeot | France | `3` / `AP` | `1` | 208, 308, 408 (+ 2008, 3008, 5008) | En production |
| Citroën | Espagne | `1` / `AC` | `2` | C5 Aircross (+ C3, C4, C3 Aircross, C4 X, C5 X) | En production |
| Opel | Allemagne | — | — | — | Début juillet |
| Fiat | Italie | — | — | — | Début juillet |
| Opel | Grande-Bretagne | — | — | — | Début juillet |

---

## Data Catalog

Toutes les tables GA4 sont partitionnées par `dt` et clusterisées par `brand` et `country` — toujours filtrer sur ces colonnes pour limiter les coûts.

### Tables GA4 — `gbl-psa-analytics-cdf-prod.analytics_ga4_eu`

| Table | Description |
| ----- | ----------- |
| `partitioned_events` | 1 ligne par event — 155 champs (event, user, device, geo, trafic, ecommerce, campaigns) |
| `partitioned_event_params` | 1 ligne par event + paramètre — clés utiles : `id_event`, `user_pseudo_id`, `event_name`, `key`, `string_value` |
| `partitioned_items` | 1 ligne par event + item — events `add_to_cart`, `purchase`, `select_item` |
| `partitioned_user_properties` | 1 ligne par event + propriété utilisateur |

### Tables CRM — `gbl-psa-analytics-publicis.dcr_cdp_landing_zone`

| Table | Description |
| ----- | ----------- |
| `{country}_matching_tdid_cid` | Mapping GA4 `user_pseudo_id` (cid) → Treasure Data `td_id` |
| `{country}_lead_2025_EVBB` | Leads — champs utiles : `timestamp`, `td_id`, `calc_brand_name`, `calc_model_name`, `brand`, `leadtype__c` |
| `{country}_order_2025_EVBB` | Orders — champs utiles : `brand`, `calc_brand_name`, `orderid__c`, `td_id`, `calc_model_name`, `ordercreationdate__c` |

Pays disponibles : AT, BE, DE, ES, FR, GB, IT, LU, NL, PL.

### Tables de référence — `gbl-psa-e2e-prod.referential_eu`

| Table | Description |
| ----- | ----------- |
| `GA4_E2EQTCCOC` | Mapping code pays Stellantis → ISO 2 lettres |
| `GA4_e2EQTFBRA` | Mapping code marque → nom |

`ID_BRAND` est INTEGER dans `GA4_e2EQTFBRA` et STRING dans les autres tables → `CAST(brd.ID_BRAND AS STRING)` nécessaire pour les jointures.

### Tables de sortie — `gbl-psa-analytics-publicis.dcr_cdp_landing_zone`

| Table / Objet | Description |
| ------------- | ----------- |
| `tb_evbb_training_data_fr_es_2024_2025` | Dataset d'entraînement — 134 M lignes |
| `tb_evbb_scores` | Matrice de valeurs : 1 ligne par micro-conversion × modèle |
| `sproc_evbb_compute_evbb_scores` | Stored procedure — calcule les bid values par event GA4 |

---

## Pipeline

### 1. Construction du dataset d'entraînement

**Unité d'observation** : un utilisateur déclenche un event GA4 à l'instant T. La séquence d'events autour de ce point est agrégée en features. Label : a-t-il commandé le même modèle de voiture dans les 90 jours ?

**CTEs BigQuery** :

| CTE | Rôle |
| --- | ---- |
| `base` | Nettoyage events GA4, standardisation pays/marque, extraction modèles (regex sur `string_value` où `key = 'vehicle_name'`) |
| `id_mapping` | Jointure `user_pseudo_id` → `td_id` par pays |
| `orders` | Nettoyage orders par pays/marque, extraction modèles |
| `events_with_td_id` | Events liés aux vrais utilisateurs CRM |
| `event_order_matched` | Matching event → order (même user, même modèle, dans les 90j post-event) |
| `event_sequence_order_matched` | Séquence d'events autour du starting event (−7j à +90j) |
| `event_sequence_filtered` | Suppression séquences avec activité antérieure au starting event |
| `features` | Agrégation en features binaires (45 flags) + label `converted` |

**Note** : `event_timestamp` GA4 est en microsecondes (16 chiffres) ; `timestamp` CRM en secondes unix (10 chiffres) → multiplier par 1 000 000 avant comparaison.

**Coût** : ~3.18 TB par exécution → ~17 USD/query.

**Taux de conversion observés** :
- FR (Peugeot) : 1 209 orders / 52 641 leads = **2%**
- ES (Citroën) : 234 orders / 7 935 leads = **3%** (mapping ids partiels)

### 2. Feature engineering

- **45 features binaires** (0/1) au départ — seules les features binaires sont conservées car coefficients interprétables et intégrables dans GA4
- Après suppression des features corrélées (Pearson r > 0.9), **10 features retenues** :

| Feature | Catégorie |
| ------- | --------- |
| `event_model_name_clean` | Dummy modèle voiture |
| `flag_view_item` | Browsing |
| `flag_tradein_request` | Trade-in |
| `flag_file_download` | Engagement |
| `flag_config_finished` | Configuration |
| `flag_testdrive_request` | Lead haute intention |
| `flag_add_to_cart` | Checkout |
| `flag_form_complete` | Lead |
| `flag_dealerlocator_resultlist` | Dealer research |
| `flag_offer_request` | Lead |

**Temporal leakage prevention** : seuls les events AVANT la soumission du lead sont utilisés comme features.

**Déséquilibre de classes** : ~3% de positifs — géré au niveau du modèle (class weighting).

**Prix moyens** saisis manuellement (`totalamount` toujours null dans les orders) :

| Modèle | Prix moyen |
| ------ | ---------- |
| 208 | 20 000 € |
| 308 | 27 000 € |
| 408 | 33 000 € |
| C5 Aircross | 35 000 € |

### 3. Modèle

**BigQuery ML — Logistic Regression** (coefficients interprétables, probabilités directement utilisables pour les bid values).

Évaluation par **Time Series split** : 20 premiers mois → train, 4 derniers mois → test.

| Marché | Precision | Recall | ROC-AUC | PR-AUC |
| ------ | --------- | ------ | ------- | ------ |
| FR (Peugeot) | 0.21 | 0.71 | 0.66 | 0.28 |
| ES (Citroën) | 0.25 | 0.55 | 0.63 | 0.27 |

Limite : hypothèse de linéarité → pas d'interactions entre micro-conversions capturées.

### 4. Du modèle aux bid values

Pour chaque micro-conversion *j* et modèle de voiture *m* :

- **p_baseline(m)** = σ(β₀ + βₘ) — probabilité sans micro-conversion
- **p_fired(m,j)** = σ(β₀ + βₘ + βⱼ) — probabilité si j déclenché
- **Calibration** : α(m) = cr(m) / p_baseline_raw(m) — ramène les probas brutes (~0.5) au taux de conversion réel (~0.03)
- **Δp(m,j)** = p_fired_calibrated(m,j) − p_baseline(m)
- **bid_value(m,j)** = Δp(m,j) × avg_price(m) si Δp > 0, sinon 0

Résultat sauvegardé dans `tb_evbb_scores` (1 ligne par micro-conversion × modèle, avec `p_fired`, `p_baseline`, `delta_p`, `bid_value`).

---

## Personnes clés

| Rôle                       | Personne |
| -------------------------- | -------- |
| Trader / Business — pilote | Emilie   |
| Data Strategist            | Pierre   |
| Data Scientist             | Dan      |

