---
type: project
statut: en cours
discipline: dev
implication: lead
client: LOREAL
---

Plateforme : [[Mine Platform]] · Client : [[LOREAL]]

## Objectif du module

Centraliser **tout ce qui touche à Amazon Marketing Cloud** sur Mine : analyses, dashboards et (à terme) extraction de données directement depuis la plateforme AMC.

Le module est pensé pour plusieurs types d'utilisateurs :

- **Équipes data** : création d'analyses, traitement des données AMC
- **Traders** : exploration des résultats, insights campagnes
- **Équipes conseil** : lecture des dashboards, fourniture d'insights aux clients

## Structure du module

Le module est organisé en **sections accessibles via un hub central** :

### 1. Analyses

Le **Full Funnel** lit ses données **directement depuis BigQuery** (sélection d'une table étude × marque × période). Voir **Architecture data (BigQuery)** plus bas.

Depuis août 2026, ce n'est plus un deck de slides figées mais un **workspace pivot** : moteur d'agrégation paramétrable, cas d'usage pré-réglés, filtres, slicers de page et export Excel. Le deck et la sortie PowerPoint ont été supprimés.

- **Full Funnel** — workspace pivot : parcours, place des leviers, synergie, conversions assistées, recrutement NTB
- **Campaign Focus** — (à venir)

#### Workspace pivot — principes

Le parcours est `sélection de l'analyse → regroupement des leviers → workspace`.

- **Moteur pur et testé**, en trois étages : normalisation (une fois par analyse) → dérivation (au changement de mapping ou de niveau) → agrégation (à chaque réglage). Calcul côté client, les tables font moins de 50 000 lignes.
- **Cas d'usage** = un `PivotView` littéral, jamais une impasse : la barre de réglages édite le même objet.
- **La vue vit dans l'URL** — un tableau filtré est un lien partageable, le bouton retour est un annuler.
- **Slicers de page** hors de la vue pivot : changer de cas d'usage ne les efface pas, comme un slicer Excel tient à travers tous les TCD d'une feuille.

#### Règles de justesse encodées

Elles viennent des notes de méthode du classeur de Jules et de l'exploration de la donnée. Les enfreindre produit des chiffres faux **sans rien casser à l'écran** — d'où leur présence dans le moteur plutôt que dans la vigilance de l'utilisateur.

| Règle                                                                     | Pourquoi                                                                                                                       |
| ------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| Les ratios se calculent après agrégation                                  | La moyenne des taux n'est pas le taux du total                                                                                 |
| Le dénominateur du taux de conversion est réglable (reach ou impressions) | « selon ce qui est pertinent »                                                                                                 |
| Les clics ne sont jamais dénominateur d'un taux                           | Les formats vidéo sont optimisés à la vue, pas au clic                                                                         |
| Un seul `analysis_level` à la fois                                        | Une table empile sept analyses ; `Path to conversion` et `Media Mix` portent exactement le même reach et les mêmes conversions |
| Une seule `granularity` à la fois                                         | `Format` et `Channel` sont la même population à deux mailles (reach identique à moins de 1,5 %)                                |
| Avertissement dès que le reach est sommé sur plusieurs parcours           | C'est un dédoublonné d'utilisateurs, la somme est théoriquement fausse                                                         |

### 2. Dashboards

Migration des dashboards Looker (Khadija) vers React. Embarqués en iframe dans un premier temps, puis migrés progressivement.

**Client L'Oréal :**

- **Audiences Insights** — seul dashboard accessible à L'Oréal (MM & P2C ne contient pas de data L'Oréal)

**Client Publicis (équipes commerce internes) :**

- **Audiences Insights**
- **MM & P2C**
- **Etude Efficacite Video Amazon**

> Matrice d'accès révisée le 2026-07-09 (voir Décisions clés). Les données sont isolées par dataset BQ dédié au client (voir Architecture data ci-dessous).

### 3. Data Querying _(futur)_

Requêtage et extraction de données directement depuis la plateforme AMC, sans passer par un export manuel. Premier test effectué par Brieg avec un agent ayant accès à un MCP avec APIs Amazon Ads, peu concluant pour l'instant.

## Gestion des accès

Système à deux niveaux :

**Niveau client** (`availableDashboards`) — stocké dans `customers/{id}/products/amazon-marketing-cloud-analytics`. Définit quels dashboards existent pour ce client. Configurable depuis la page Settings (admins plateforme) ou directement en Firestore.

**Niveau utilisateur** (`allowedDashboards`) — stocké dans `users/{id}/products/amazon-marketing-cloud-analytics`. Définit, parmi les dashboards disponibles pour le client, lesquels chaque utilisateur voit. Remplace l'ancien `dashboards: boolean`.

La page **Settings** (admins module + admins plateforme) permet de gérer les deux niveaux. Les checkboxes utilisateur sont automatiquement filtrées par le périmètre client — impossible d'accorder un dashboard hors périmètre.

Les équipes conseil n'ont pas accès aux sous-modules d'analyses.

**Partage des dashboards Looker (iframe).** Les dashboards sont embarqués via un lien de partage Looker. Un lien à partage **restreint** n'est accessible qu'aux personnes ayant _déjà_ l'accès au Looker **et un compte Google lié à leur adresse Publicis** — ce qui exclut les utilisateurs sans compte Google (cas rencontré avec Nicolas Vivies, PMO Retail Media). Pour que l'iframe soit accessible à **toute personne ayant accès au sous-module** dans ConnectedHub, le dashboard doit être partagé en **« unlisted »** côté Looker (toute personne avec le lien y accède). C'est le paramètre à appliquer sur les dashboards embarqués. _(Résolu avec Khadija le 2026-07-09.)_

## Architecture data (BigQuery)

Le module lit ses données dans **BigQuery** (projet `amira-test`, région **EU**), via des **routes backend** — le client ne touche jamais BQ.

- **Isolation par client** : **un dataset par locataire ConnectedHub**, dérivé du customerId → `AMC_ConnectedHub_<customerId>`. L'Oréal = `AMC_ConnectedHub_7cR1jE`. Un client ne peut atteindre la data d'un autre (frontière dataset/IAM). _(Le split dev/prod initial a été abandonné le 2026-07-09 : un seul dataset par client.)_
- **Nomenclature des tables** :
  - Analyses (volatiles) : `<étude>__<marque>__<période>` — ex. `full_funnel__mugler__2025_q4` (`__` entre dimensions, `_` simple à l'intérieur).
  - Dashboards natifs (futurs, fixes) : préfixe `dash_` — ex. `dash_audiences_insights`.
- **Registre** : table `registry` dans chaque dataset, une ligne par table analyse (libellés UI, dates réelles, `row_count`, `is_active`). Le module la lit pour bâtir la sélection ; l'UI affiche les libellés, pas le nom BQ brut. La date « dernière modif » vient de `__TABLES__.last_modified_time`.
- **Routes** : `GET /analyses` (catalogue via registre) et `GET /analyses/:tableId` (lecture d'une table, `tableId` validé contre le registre, valeurs sérialisées en string → réutilise le pipeline CSV existant côté client).
- **Ingestion** : CSV AMC transformés (`;`, décimales virgule→point, dates ISO, `path` intact) puis chargés non partitionnés. Données actuelles : Mugler (298 l.) + Azzaro (1366 l.). ⚠️ le registre doit être mis à jour à chaque nouvel import (manuel aujourd'hui).

## Stack technique

- **Client** : React 19 + TypeScript, React Router 7
- **Routing** : `client/src/app/routes/app/amazon-marketing-cloud-analytics/`
- **Feature** : `client/src/features/amazon-marketing-cloud-analytics/`
- **Backend** : Express — `server/src/features/amazon-marketing-cloud-analytics/`
- **Data** : BigQuery, projet `amira-test` (EU), dataset `AMC_ConnectedHub_7cR1jE` — voir Architecture data
- **Branches** : `feature/AMC-Analytics` (accès dashboards, mergée) ; `feat/amc-full-funnel-bigquery` (Full Funnel × BQ, mergée sur `main` — PR #1653/#1654) ; `feat/amc-pivot-workspace` (workspace pivot — PR #1719 à #1722 sur `develop`, PR #1723 sur `main`, toutes mergées le 2026-08-10)
- **Moteur pivot** : `client/src/features/amazon-marketing-cloud-analytics/pivot/` — couche pure, sans React, ~390 tests

## Décisions clés

| Date       | Décision                                                                                                                                                                    | Raison                                                                                                                                                               |
| ---------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 2026-06-19 | Renommage "Amazon Marketing Cloud analyses" → **AMC Analytics**                                                                                                             | Nom trop long, pas cohérent entre les pages                                                                                                                          |
| 2026-06-19 | Icône **Layers** (couches empilées)                                                                                                                                         | Représente le caractère multi-couches de la plateforme (analyses + dashboards + data) — plus fidèle que l'ancien icône camembert                                     |
| 2026-06-19 | Nouvelle description hub                                                                                                                                                    | Reflète l'élargissement du module au-delà de la seule analyse full funnel                                                                                            |
| 2026-06-25 | Accès dashboards : `allowedDashboards: string[]` par utilisateur                                                                                                            | Remplace le boolean `dashboards` — permet un contrôle granulaire par dashboard                                                                                       |
| 2026-06-25 | Périmètre client : `availableDashboards` dans le doc Firestore client                                                                                                       | Empêche d'accorder à un user un dashboard hors périmètre de son client                                                                                               |
| 2026-06-25 | Module ouvert à Publicis (1 dashboard) + L'Oréal limité à 2 dashboards                                                                                                      | Séparation commerce (Publicis) / data-trading (L'Oréal)                                                                                                              |
| 2026-07-09 | **Matrice d'accès révisée** : L'Oréal = Audience Insights **seul** ; Publicis (commerce) = **les 3** (Audience Insights + MM & P2C + Étude Vidéo)                           | MM & P2C ne contient pas de data L'Oréal (accès demandé par curiosité, non accordé) ; les études commerce sont côté Publicis. Remplace la répartition du 25/06       |
| 2026-07-09 | **Data BQ** : un dataset par client (`AMC_ConnectedHub_<customerId>`, EU), tables analyse `<étude>__<marque>__<période>` + registre ; Full Funnel lit BQ via routes backend | Isolation par client au niveau dataset ; découpler nom BQ / libellé UI ; remplacer l'upload CSV                                                                      |
| 2026-07-09 | Abandon du split dev/prod (`_dev`) → un seul dataset par client                                                                                                             | Simplicité (un client, faible volume) ; contrepartie : plus d'isolation staging                                                                                      |
| 2026-08-06 | **Le deck de slides est supprimé**, remplacé par un workspace pivot                                                                                                         | Retour de Jules : « on ne peut pas faire ce qu'on veut ». Les calculs vivaient dans les composants de slide — rien n'était paramétrable parce que rien n'était isolé |
| 2026-08-06 | Mapping des leviers **éditable dans l'app**, persisté par analyse au niveau customer                                                                                        | C'est une convention client que l'équipe partage, pas une préférence personnelle ; Jules l'ajuste par campagne                                                       |
| 2026-08-06 | Export Excel plutôt que vues sauvegardées                                                                                                                                   | Pierre met à jour les tableaux du PowerPoint depuis Excel — c'est le maillon de production                                                                           |
| 2026-08-07 | **La vue vit dans l'URL** (`nuqs`), pas dans des vues nommées                                                                                                               | Partageable par lien, rechargeable, bouton retour = annuler ; zéro backend, l'adaptateur est déjà monté globalement                                                  |
| 2026-08-07 | Colonnes nommées par leur **source brute**, ratios en `numérateur / dénominateur`                                                                                           | Retour de Jules : un libellé traduit rompt le lien avec la donnée, et avec lui la certitude sur le KPI utilisé                                                       |
| 2026-08-07 | `analysis_level` et `granularity` en **mono-sélection**, chaque cas d'usage lié à son niveau                                                                                | Une table empile sept analyses ; `Path to conversion` et `Media Mix` portent exactement les mêmes chiffres. Les additionner produisait des chiffres faux en silence  |
| 2026-08-07 | Niveaux de regroupement ramenés de six à quatre : `1 famille · 2 type · 3 funnel · 4 overlap`                                                                               | Les niveaux 4 et 5 dupliquaient ce que `granularity = Channel` fournit déjà pré-agrégé côté serveur                                                                  |
| 2026-08-07 | « Place des leviers » lit les lignes `Place of channel` du trading au lieu de dériver la décomposition                                                                      | Leurs chiffres plutôt que des chiffres à rapprocher des leurs ; ils ont une catégorie `Solo` pour les parcours mono-touche                                           |
| 2026-08-07 | Le module reste **en français**, i18n plus tard                                                                                                                             | Pas prioritaire — à ne pas oublier                                                                                                                                   |

## Personnes clés

| Rôle dans le projet                         | Personne    |
| ------------------------------------------- | ----------- |
| Dev — module Mine                           | Adam, Eddie |
| Data / dashboards Looker, pipeline AMC      | Khadija     |
| Data Strat — référent L'Oréal               | Pierre      |
| AMC Modeled Audiences, backup data querying | Hajar       |

## Questions ouvertes

- [ ] Calendrier de migration des dashboards Looker → React (dépend de Khadija)
- [ ] Périmètre exact du Campaign Focus
- [ ] Faisabilité et accès API pour le data querying direct AMC
- [x] Créer module client Publicis avec dashboard "Etude Efficacite Video Amazon" → système d'accès implémenté, reste à configurer Firestore + utilisateurs Publicis
- [x] Clarifier accès aux dashboards L'Oréal → **révisé 2026-07-09** : L'Oréal = Audience Insights **seul** (MM & P2C sans data L'Oréal). ~~Ancienne réponse (25/06) : audiences-insights + mm-p2c.~~
- [x] Quelles équipes auront accès à quoi ? → **révisé 2026-07-09** : Publicis (commerce) = les 3 dashboards ; L'Oréal = Audience Insights seul. ~~Ancienne réponse (25/06) : Publicis 1 dashboard / L'Oréal 2.~~

## Évolutions envisagées

- **Traçabilité de version des études** _(idée, écartée pour l'instant)_ : à chaque import, la Cloud Function horodate la table (`loaded_at`) dans le registre, et le module **estampille l'export Excel** (« données extraites le X · chargées le Y · N lignes »). But : savoir sur quelle version de la donnée repose une étude et détecter les ré-imports. On a déjà ~80% (la tuile « Last Updated » lit `last_modified_time`). Version lourde — **régénérer** une ancienne étude à l'identique — = conserver des **snapshots BQ** par import (registre → `snapshot_ref`). À rattacher au [[Contrat d'ingestion BQ]] (nommage, transfo CSV, MAJ du registre par la CF). _(Depuis le passage au workspace pivot, l'URL porte déjà la vue — il ne manque que la version de la donnée.)_
- **Conversions assistées depuis `Place of channel`** : attribuées = `Finisher` + `Solo`, assistées = `Beginner` + `Intermediate`. Même raisonnement que « Place des leviers » — utiliser la donnée du trading plutôt que de la dériver des parcours. Non tranché.
- **i18n du module** : tout le workspace est en français codé en dur, alors que le module a ses fichiers `fr`/`en` et que Jules travaille en anglais sur ConnectedHub. Pas prioritaire, à ne pas oublier.

## Réunions

- [[2026-06-25 Point AMC Khadija]]
- [[2026-07-07 Onboarding AMC Khadija]]
- [[2026-08-10 Retours Jules Workspace Pivot]]

## Ressources

- Notions sur l'outil AMC (comptes, instances, tables, API) : [[AMC — Notions]]
- Procédure d'ajout d'un extract (nommage, transfo CSV, MERGE du registre) : [[Contrat d'ingestion BQ]]
- Documentation API AMC : https://advertising.amazon.com/API/docs/en-us/reference/api-overview

## Dernières sessions

- **2026-08-10** — **Couche graphiques** du workspace pivot. Un graphique est décrit et non écrit (`Preset.charts`), en ajouter un est un objet de plus dans `PRESETS` — six cas d'usage en portent. Quatre formes dont un **Sankey** des parcours et un **nuage volume × ROAS**. Le double axe barres + ligne de taux a été **refusé** (l'alignement des deux échelles fabrique une corrélation absente) : volume et taux voyagent en deux graphiques. Couleurs passées par un validateur d'accessibilité, d'où deux contraintes : une série = une couleur, et une rampe ordinale ne sépare que quatre pas. Puis simplification : graphiques **figés sur la vue du preset**, réglages descendus sous eux et limités au tableau, tiroir « Avancé » supprimé, **niveau de regroupement en slicer à la place de `granularity`** (qui reste filtrée et épinglée). Un plantage corrigé : le tooltip du nuage vidait la page. PR #1724 sur `develop`.
- **2026-08-07** — Alignement sur les motifs `max-metrics` / `feed-manager` / `spm` : vue dans l'URL via `nuqs`, contrôles `Combobox`, scorecards sur `Card`, rail repliable. Slicers de page repris de son onglet `output`. Surtout : **trois double-comptes corrigés** — `analysis_level` (sept analyses empilées dans une table, `Path to conversion` et `Media Mix` portent les mêmes chiffres), `granularity` (deux mailles de la même population), et un slicer mono-sélection qui affichait sa valeur sans jamais l'appliquer. Colonnes nommées par leur source brute. « Place des leviers » lit désormais les lignes `Place of channel` du trading. PR #1720/#1721 mergées, #1722 ouverte.
- **2026-08-06** — **Refonte du Full Funnel en workspace pivot**, en remplacement du deck de slides figées, suite au retour de Jules (« on ne peut pas faire ce qu'on veut »). Moteur pur en trois étages avec les règles de méthode encodées (ratios après agrégation, clics jamais dénominateur, avertissement sur la somme des reach), mapping des leviers éditable, cas d'usage pré-réglés, filtres, export Excel. Validé contre son classeur : 0 divergence sur 200 lignes. Suppression du deck, de la route PowerPoint et de 7 composants morts — 10 803 lignes. PR #1719 mergée. Correction d'un cadrage erroné de ma part : les conversions assistées étaient calculables, contrairement à ce que j'avais conclu.
- **2026-07-09** — Full Funnel branché sur BQ (dataset par client, nomenclature `<étude>__<marque>__<période>`, registre, routes backend) ; PR #1653/#1654 mergées sur main. Matrice d'accès révisée (L'Oréal = Audience Insights seul, Publicis commerce = les 3 dashboards). Incident Looker résolu : partage basculé en « unlisted » avec Khadija après blocage de Nicolas Vivies (PMO L'Oréal, pas de compte Google). Split dev/prod abandonné.
- **2026-06-25** — Système d'accès à deux niveaux : `availableDashboards` par client + `allowedDashboards` par utilisateur (remplace le boolean). Settings UI mise à jour, 18 tests serveur. L'Oréal → audiences-insights + mm-p2c ; Publicis → etude-efficacite-video-amazon.
- **2026-06-22** — Intégration des vrais dashboards Looker (URLs fournies par Khadija) : Audiences Insights, MM & P2C, Étude Efficacité Vidéo Amazon. Composant `LookerDashboard` avec navigation multi-pages par onglets. Masquage du bandeau natif Looker par clip CSS.
- **2026-06-19** — Migration `AutomaticPageHeaderBeta`, rebranding complet en « AMC Analytics », nouvelle icône Layers, premier dashboard fonctionnel (`dashboardRegistry` opérationnel), résolution de 56 erreurs ESLint, merge PR sur develop.
