---
type: note
projet: Creative Insights
---

# Existant technique GCP

Projet : [[Creative Insights]]

Exploration du projet `zen-creativeinsights-dev-mg` le 10/08/2026, en CLI. Complète le briefing du 10/08 et [[Patterns ConnectedHub réutilisables]].

**Conclusion en une phrase** : l'amont existe et fonctionne, mais uniquement sur Meta, uniquement en prototype, et le modèle de données est fabriqué à la main par client. En l'état il ne peut pas porter un module générique.

---

## Inventaire

**Cloud Run** (region `europe-west1`, ce sont des functions gen2, invisibles via `gcloud functions list`) :

| Service | Rôle |
| --- | --- |
| `cf-gather-meta-assets` | Scraping des assets Meta |
| `cf-gather-snapchat-assets` | Scraping des assets Snapchat |
| `cf-gather-tiktok-assets` | Scraping des assets TikTok |

Service account : `internal@zen-creativeinsights-dev-mg.iam.gserviceaccount.com`. Timeout : **3600 s**. Aucun Cloud Scheduler dans le projet, tout est déclenché à la main.

**Buckets** : `creative_assets_meta`, `creative_assets_snapchat`, `creative_assets_tiktok`, `macro_analysis_results`.
Arborescence : `gs://creative_assets_<plateforme>/<ad_account_id>/<asset_id>.<asset_format>`.

**Datasets BigQuery** : trois par plateforme (`meta`, `snapchat`, `tiktok`), chacun avec une unique table `tb_api_asset_urls` partitionnée sur `ingestion_date`. Deux par client (`verisure`, `stellantis`) qui portent la couche analyse.

---

## Contrat d'appel des Cloud Run

Source récupérée depuis `gs://run-sources-zen-creativeinsights-dev-mg-europe-west1/`. Pour Meta :

```json
POST https://cf-gather-meta-assets-4dftyteusa-ew.a.run.app
{
  "ad_account_id": "1349117765200572",   // requis
  "page_id": "272877608226",             // requis
  "campaign_id": "120243694736540617"    // optionnel
}
```

Le traitement est **synchrone de bout en bout** : scraping API, téléchargement local, upload GCS, puis chargement BigQuery. D'où le timeout d'une heure.

Trois pièges pour le module :

1. **La fonction renvoie toujours HTTP 200**, y compris en erreur : le corps contient `{"status": "500", "message": "..."}`. Tester `response.ok` ne détecte rien, il faut parser le corps.
2. **Impossible à appeler depuis un handler Express.** Une heure de traitement synchrone impose un dispatch asynchrone plus polling de statut, exactement le pattern `feed-manager`.
3. **Aucun paramètre de période.** Le cadrage du 10/08 prévoit un formulaire avec « advertiser IDs, plateformes et timeframes ». Le timeframe n'existe pas dans la fonction, seul `campaign_id` filtre. Soit il faut le faire ajouter, soit le formulaire ment.

Autres points : la destination est **codée en dur** (table `meta.tb_api_asset_urls`, bucket `creative_assets_meta`, dossier = `ad_account_id`), il n'y a **aucune notion de projet ni de client**. L'idempotence est gérée par `DELETE FROM ... WHERE asset_id IN UNNEST(...)` suivi d'un load. Les credentials passent par Secret Manager, via un package interne `simba_platform_authentication` et un package `asset_scrappers` (filiation SIMBA confirmée).

---

## Le problème numéro un : les trois tables ne sont pas les mêmes

`tb_api_asset_urls` porte le même nom dans les trois datasets, mais pas les mêmes colonnes.

| Colonnes | meta | tiktok | snapchat |
| --- | --- | --- | --- |
| Hiérarchie campagne (`campaign_id`, `adset_id`, **`ad_id`**, `ad_creative_id`) | ✅ complète | ❌ absente | ❌ absente |
| Identifiant compte | `ad_account_id` | `advertiser_id` | `ad_account_id` |
| Spécifique | `ad_creative_object_type` | `asset_material_id` | `asset_width/height`, `asset_duration_seconds`, `asset_usages` |

Communes aux trois : `asset_id`, `asset_type`, `asset_format`, `asset_url`, `asset_creation_time`, `downloaded`, `uploaded_to_gcs`, `gcs_file_name`, `ingestion_date`.

**Conséquence directe** : seul Meta porte `ad_id`. Or c'est la seule clé qui permet de raccrocher un asset aux données de performance. **Sur TikTok et Snapchat, la jointure créa vers performance est aujourd'hui impossible.** C'était mon risque numéro un après la réunion, il est confirmé et il est pire que prévu : ce n'est pas une clé mal choisie, c'est une clé absente.

Cela se vérifie côté client : `verisure` n'a de vue Data Platform que pour Meta, et `stellantis` a dû passer par une source totalement différente pour TikTok (`tb_adverity_tiktok_engagement`, donc Adverity).

---

## Le lien avec la Data Platform

`verisure.mvw_dataplatform_meta_data` est une vue matérialisée sur `zen-dataplatform-pm-prd-amg.pds__meta__all.p_meta_standard_report_fr`, avec un `WHERE account_id = 1349117765200572` **codé en dur**, agrégée par `day, campaign_id, adset_id, ad_id`. Elle sort impressions, coûts et clics, ventilés par objectif de campagne, plus les quartiles vidéo et les landing page views.

Deux implications :

- **La maille de performance est l'`ad_id` par jour**, alors que la maille créa est l'`asset_id`. Un même ad peut porter plusieurs assets, donc attribuer la performance à un asset donné est ambigu. C'est exactement le piège de double-compte rencontré sur [[1.Projects/MINE_AMCAnalytics/AMC Analytics]]. La règle de répartition doit être décidée explicitement, pas subie.
- **Un compte client codé en dur dans une vue** signifie que chaque nouveau client demande une vue écrite à la main. C'est précisément ce que le module doit générer.

---

## Le modèle de données est fabriqué par client

`verisure.tb_gemini_features_data` (partitionnée sur `ingestion_date`, **clusterisée sur `asset_id`**) contient environ 70 colonnes physiques de features **propres à Verisure** : `app_myverisure_shown`, `burglary_or_risk_scenario`, `verisure_expert_present`, `police_or_emergency_call_mentioned`, `anti_jamming_or_multichannel_connectivity_mentioned`, `brand_red_dominant`, `product_alarm_panel_keypad_visible`... Les KPIs aussi sont spécifiques : `total_lb`, `total_lv`, `total_lj`, `total_rdv_bruts`, `total_rdv_nets`, `total_ventes`.

`stellantis` a une forme entièrement différente : `tb_extracted_features`, `tb_extracted_features_rules_applied`, `vw_consolidated_metrics_features`, `tb_adverity_tiktok_engagement`.

**Un nouveau client égale un nouveau dataset écrit à la main, avec un schéma différent.** C'est en contradiction frontale avec le principe de « sorties standardisées » acté en réunion, et c'est ce qui rend un module générique impossible en l'état.

**La sortie existe déjà à moitié** : la table contient `analysis_json` et `analysis_raw_text`, donc la sortie brute du LLM est conservée en JSON et les 70 colonnes n'en sont qu'un aplatissement. Le chemin propre est donc de faire de `analysis_json` le stockage canonique et de dériver les colonnes par client en vues. Ce n'est pas une réécriture, c'est un renversement de ce qui fait autorité.

### Ce qui est déjà bon et réutilisable tel quel

- **Fiabilité LLM** : `analysis_status`, `llm_success_runs`, `analysis_run_errors` tracent déjà les échecs et les reprises.
- **Modèle de corrélation** : `kpi_used`, `kpi_used_label`, `expected_rdv`, `kpi_lift`, `kpi_rate_per_1k`, `avg_kpi_used_all`, `avg_kpi_used_type`, `kpi_ratio_vs_all`, `kpi_ratio_vs_type`. C'est un modèle de lift par rapport à la moyenne, déjà construit.
- **Interprétation** : `performance_level`, `main_takeaway`, `best_features`, `worst_features`. Le LLM ne produit pas que des features, il produit déjà la lecture.
- **Aveu de fragilité utile** : la colonne `gcs_match_status` existe parce que la correspondance entre lignes BQ et fichiers GCS échoue parfois. Le problème est connu et déjà instrumenté.

---

## Volumétrie : c'est un prototype à l'arrêt

| Plateforme | Lignes | Assets distincts | Première ingestion | Dernière ingestion |
| --- | --- | --- | --- | --- |
| meta | 1 438 | 1 422 | 2026-05-12 | **2026-06-01** |
| tiktok | 1 266 | 1 266 | 2026-05-14 | **2026-05-15** |
| snapchat | 77 | 77 | 2026-05-12 | **2026-05-12** |

Snapchat n'a tourné qu'une seule journée. **Plus rien n'a été ingéré depuis le 1er juin, soit plus de deux mois.** Ce n'est pas un système en production qu'on industrialise, c'est un prototype arrêté qu'on relance. Cela change la nature du chantier et l'estimation de charge.

Le taux d'upload GCS est en revanche excellent : 1438/1438 sur Meta, 1266/1266 sur TikTok, 76/77 sur Snapchat.

---

## Ce que ça implique pour le module

1. **Le périmètre d'entrée réaliste est Meta seul.** TikTok et Snapchat ne peuvent pas être analysés en performance tant que leurs Cloud Run ne remontent pas la hiérarchie de campagne. C'est un prérequis à faire porter par Hajar et Abhishek, pas un sujet ConnectedHub.
2. **Faire évoluer les Cloud Run avant de brancher quoi que ce soit** : ajouter un paramètre de période, une notion de projet ou de client pour ne plus coder la destination en dur, et un retour HTTP honnête.
3. **Renverser le modèle de données** : `analysis_json` canonique, colonnes par client en vues dérivées. Sans ça, « sorties standardisées » reste un slogan.
4. **Trancher la règle d'attribution** performance ad_id vers asset_id avant d'afficher le moindre chiffre.
5. L'exécution asynchrone plus le suivi de statut est obligatoire côté ConnectedHub, ce n'est pas un confort.
