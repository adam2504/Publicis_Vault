---
type: note
projet: Creative Insights
---
Projet : [[Creative Insights]]
Plateforme : [[Mine Platform]]

# Plan de modifications côté DS

Document destiné à Hajar et Abhishek. Il décrit les évolutions attendues sur les Cloud Functions de collecte et sur les tables `tb_api_asset_urls`, pour que le module ConnectedHub puisse s'appuyer dessus.

Tout ce qui suit est basé sur la **source réellement déployée** de `cf-gather-meta-assets`, récupérée depuis `gs://run-sources-zen-creativeinsights-dev-mg-europe-west1/services/cf-gather-meta-assets/`, et non sur le dépôt GitHub qui peut avoir divergé.

---

## Résumé des demandes

| # | Demande | Priorité | Effort estimé |
| --- | --- | --- | --- |
| 1 | `project_id` en paramètre et en colonne | **P0** | Faible |
| 2 | Corriger le `DELETE` d'idempotence | **P0** | Très faible |
| 3 | Statut écrit par la fonction elle-même | **P1** | Moyen |
| 4 | Robustesse face aux erreurs d'API Meta | **P1** | Très faible |
| 5 | Filtre de période et multi-campagnes | **P2** | Faible |

Les points 1 et 2 sont indissociables : livrer le 1 sans le 2 introduit une perte de données silencieuse, expliquée plus bas.

---

## 1. `project_id` en paramètre et en colonne

### Pourquoi

Aujourd'hui la fonction n'écrit aucune notion de projet ni de client. Les créas d'un projet sont donc, côté module, simplement les lignes portant son `ad_account_id`. Cela impose **un projet par compte annonceur**, et rend impossible le cas que tu as toi-même identifié, plusieurs scopes pour un même client.

### Ce qui change

**a. Accepter le paramètre.** Dans `main()` :

```python
project_id = request_json.get('project_id')   # optionnel, chaîne
```

Le module fournit son propre identifiant de projet, déjà existant dans `connectedhub.tb_projects`. Rester optionnel garde la fonction appelable à la main comme aujourd'hui.

**b. Ajouter la colonne.** Sur `meta.tb_api_asset_urls`, et de façon symétrique sur les tables TikTok et Snapchat pour ne pas créer une divergence de plus entre les trois :

```sql
ALTER TABLE `zen-creativeinsights-dev-mg.meta.tb_api_asset_urls`
ADD COLUMN IF NOT EXISTS project_id STRING;
```

Nullable, pour que les lignes existantes restent valides.

**c. L'écrire.** Dans `archive_asset_urls()`, ajouter la colonne au DataFrame et au schéma de chargement :

```python
asset_urls_df['project_id'] = project_id
```

```python
bigquery.SchemaField(name="project_id", field_type="STRING"),
```

> Le chargement actuel utilise `autodetect=False` avec un schéma explicite et la disposition par défaut `WRITE_APPEND`. Ajouter la colonne dans la table **sans** l'ajouter au schéma de chargement est donc sans effet de bord : elle reste simplement à NULL. Les deux étapes peuvent être découplées dans le temps.

---

## 2. Corriger le `DELETE` d'idempotence

### Le code actuel

```python
query = f"""
DELETE FROM `{table_id}`
WHERE asset_id IN UNNEST(SPLIT('{';'.join(asset_urls_df['asset_id'])}', ';'))
"""
```

### Trois problèmes, dont un déjà actif aujourd'hui

**Il n'y a aucun filtre sur `ad_account_id`.** C'est un bug **déjà présent**, indépendamment du `project_id` : si une même créa est utilisée sur deux comptes annonceurs, ce qui arrive dans une même business manager, un run sur le compte A supprime la ligne du compte B. Le run suivant sur B la recrée, et les deux comptes se marchent dessus indéfiniment.

**Avec le `project_id`, le problème devient structurel.** Deux projets sur le même compte partageront forcément des créas. Le run du projet B supprimerait les lignes du projet A puis les réinsérerait avec son propre identifiant. **Le projet A perdrait ses créas sans aucun message d'erreur.** C'est la raison pour laquelle les demandes 1 et 2 ne doivent pas être livrées séparément.

**La requête est construite par concaténation de chaîne.** Sur 1500 créas, cela produit une requête de plusieurs dizaines de milliers de caractères. Les identifiants Meta sont numériques donc le risque d'injection est faible, mais la fragilité reste.

### Correction proposée

```python
query = f"""
DELETE FROM `{table_id}`
WHERE ad_account_id = @ad_account_id
  AND (@project_id IS NULL OR project_id = @project_id)
  AND asset_id IN UNNEST(@asset_ids)
"""

delete_config = bigquery.QueryJobConfig(
    query_parameters=[
        bigquery.ScalarQueryParameter("ad_account_id", "STRING", ad_account_id),
        bigquery.ScalarQueryParameter("project_id", "STRING", project_id),
        bigquery.ArrayQueryParameter("asset_ids", "STRING", list(asset_urls_df['asset_id'])),
    ]
)
bq_client.query(query, job_config=delete_config).result()
```

`archive_asset_urls()` doit donc recevoir `ad_account_id` et `project_id` en paramètres. Aujourd'hui elle reçoit `bucket_folder`, qui vaut déjà `ad_account_id`, mais mieux vaut un paramètre explicite.

---

## 3. Statut écrit par la fonction elle-même

### Le problème constaté

La fonction est **synchrone de bout en bout** et peut tourner jusqu'à une heure, ce qui est le plafond d'un service Cloud Run et non un réglage. L'appelant doit donc tenir la connexion HTTP pendant toute la durée du traitement.

Le 13/08, un run de 606 secondes s'est terminé par une erreur, mais **l'appelant n'était plus là pour l'apprendre**. Le projet est resté indéfiniment en état « récupération en cours » côté module, alors que plus rien ne tournait. Allonger le délai ne réglerait rien : le problème n'est pas la durée, c'est que l'issue n'existe que dans une réponse HTTP que personne ne garantit de recevoir.

### Correction proposée, version minimale

Une table de suivi des exécutions, dans laquelle la fonction écrit deux fois :

```sql
CREATE TABLE IF NOT EXISTS `zen-creativeinsights-dev-mg.meta.tb_gathering_runs` (
  run_id STRING NOT NULL,
  project_id STRING,
  ad_account_id STRING,
  campaign_id STRING,
  status STRING NOT NULL,        -- running | succeeded | failed
  started_at TIMESTAMP NOT NULL,
  ended_at TIMESTAMP,
  asset_count INT64,
  error_message STRING
);
```

- au démarrage, une ligne `running` avec le `run_id` fourni par l'appelant
- à la fin, mise à jour en `succeeded` ou `failed`, avec le nombre de créas ou le message d'erreur

Le module n'a alors plus besoin de tenir la connexion : il interroge la table. Cela résout d'un coup le plafond horaire, la perte d'issue et la question du suivi de progression.

Une version plus ambitieuse consisterait à basculer sur un **Cloud Run Job**, qui monte à 24 h par tâche. C'est la bonne cible à terme, mais la table de suivi apporte l'essentiel du bénéfice pour une fraction de l'effort.

### En complément : un retour HTTP honnête

La fonction renvoie aujourd'hui **HTTP 200 dans tous les cas**, y compris en échec, avec le vrai résultat dans le corps :

```python
return {"status": "500", "message": "No asset found"}, 200
```

Tester `response.ok` côté appelant ne détecte donc rien. Renvoyer le vrai code HTTP serait plus sain. Le module gère déjà le cas, mais tout autre consommateur tombera dans le piège.

---

## 4. Robustesse face aux erreurs d'API Meta

### Le `KeyError` qui fait tomber la fonction

Dans `asset_scrappers/MetaAssetScrapper.py`, `_get_next_pages()` accède directement à `response['data']` sans vérifier la présence d'une clé `error`. Dès que Meta répond une erreur au lieu d'une page, la fonction lève un `KeyError: 'data'` et **fait tomber toute la Cloud Function en HTTP 500**.

Constaté le 13/08 : un incident parfaitement lisible, un dépassement de quota, s'est transformé en crash muet. Deux lignes de garde suffisent :

```python
payload = response.json()
if 'error' in payload:
    print(f"Meta API error: {payload['error']}")
    return []          # ou lever une exception métier explicite
```

### Le quota sur les gros comptes

Sans `campaign_id`, `get_ad_account_assets()` énumère **toutes** les campagnes du compte puis fait un appel par campagne. Sur `1349117765200572`, cela représente **1545 campagnes**, ce qui dépasse le quota horaire du compte. Meta coupe en cours de route, puis throttle le compte entier pendant environ une heure.

Conséquence : **un run sans `campaign_id` n'est pas exécutable sur un gros compte.** Deux pistes, non exclusives :

- une temporisation avec relance progressive sur les codes 17 et 80004
- une remontée explicite du dépassement de quota à l'appelant, plutôt que le message actuel « No asset found », qui laisse croire à un compte vide

---

## 5. Filtre de période et multi-campagnes

Deux demandes fonctionnelles, moins urgentes.

**Période.** La fonction n'accepte aucun paramètre de date. Le formulaire du module propose déjà une période, aujourd'hui utilisée seulement en aval au moment de la jointure avec la performance. Accepter `date_from` et `date_to` permettrait de filtrer à la source, et réduirait mécaniquement le volume d'appels API.

**Multi-campagnes.** Le filtre campagne est appliqué en Python après récupération de la liste des campagnes :

```python
ad_account_campaigns = [c for c in ad_account_campaigns if c['id'] == campaign_id]
```

Passer d'un identifiant unique à une liste est trivial et le module envoie déjà plusieurs identifiants dans son formulaire.

---

## Migration des données existantes

Les créas déjà collectées auront `project_id` à NULL. Trois options :

| Option | Verdict |
| --- | --- |
| Backfill par `ad_account_id` | **Recommandé** |
| Repli sur `ad_account_id` dans les requêtes | Fonctionne, mais redevient ambigu dès le deuxième projet sur un compte |
| Tout re-collecter | À écarter, coût en quota API prohibitif |

> ⚠️ **Le backfill a une fenêtre.** Il n'est possible que tant que la correspondance projet vers compte est unique, ce qui est le cas aujourd'hui. Dès qu'un utilisateur crée un second projet sur un compte déjà collecté, plus personne ne peut dire à qui appartenaient les anciennes lignes. **La migration doit être faite en même temps que la bascule, pas après.**

---

## Ce qui est déjà prêt côté ConnectedHub

Pour que le chiffrage soit juste, voici ce qui n'est pas à faire de votre côté.

- Le module possède déjà un identifiant de projet stable, dans `connectedhub.tb_projects`
- La table `connectedhub.tb_project_asset_exclusions` est déjà clé sur `project_id`, elle traverse le changement sans migration
- Le périmètre des requêtes passe par un point unique, la bascule vers `project_id` représente deux fonctions à modifier
- L'appel est déjà authentifié et non bloquant côté navigateur, avec un état `is_stalled` qui rattrape les runs perdus
- Le module gère déjà le HTTP 200 systématique en analysant le corps de la réponse

---

## Tests d'acceptation proposés

Pour valider la livraison sans ambiguïté.

1. Deux projets distincts sur le **même** `ad_account_id`, chacun sur une campagne différente. Après deux runs, chacun voit ses propres créas et **aucun des deux n'a perdu de lignes**.
2. Un run relancé deux fois de suite sur le même projet ne crée pas de doublon.
3. Un run sur un compte dont le quota est épuisé renvoie un message mentionnant le quota, et non « No asset found ».
4. Un run interrompu côté appelant laisse tout de même une ligne `failed` ou `succeeded` dans la table de suivi.
5. Les lignes antérieures à la migration restent visibles depuis leur projet d'origine.
