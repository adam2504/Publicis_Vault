---
type: context
Dernière mise à jour: 2026-08-15
---

# CONTEXT — Vault d'Adam

> Fichier de synthèse régénéré automatiquement. Source de vérité pour reprendre le contexte d'Adam sans rouvrir chaque note.

---

## Qui est Adam

Adam Jouini, apprenti Data & Dev chez Publicis Media (alternance), rattaché à la plateforme **ConnectedHub** (produit interne Mine). Il travaille au croisement du développement (React/Node.js/backend), de la data science (Vertex AI, embeddings, agents) et de l'analyse de données (AMC, MMM, audiences LiveRamp). Orientation data science exprimée lors du point évolution de juin 2026. Jules a dit explicitement vouloir proposer un poste à la sortie de l'alternance, en citant l'agent MMM et AMC Analytics comme réalisations commercialement visibles.

---

## Projets actifs

### 1. MMM AI Agent — MINE
**implication : lead** | discipline : dev + data-science | client : **MINE** — produit plateforme, **pas un projet L'Oréal**

Agent IA conversationnel dans le module MMM de ConnectedHub (Vertex AI Agent Engine). Répond aux questions ROI/média des clients sur leurs données MMM. C'est une **feature de la plateforme**, vendue en add-on à tout client ayant un MMM signé — la cible de déploiement est **Stellantis** (Opel DE / Peugeot DE), puis Longchamp.

**Statut (23/07 — aucune session depuis, 23 jours — stalled)** : Engine **6241** en prod depuis le 23/07 (`MMM_Agent_v3_live_ui_scope`). Feature B (scope écran live) livrée : l'agent connaît désormais le KPI, la période, l'onglet et la langue affichés à l'écran, et répond dans ce scope par défaut (surchargeable dimension par dimension, annonce le scope sur sa 1re réponse). Défaut critique corrigé en revue de branche : les bornes de période passent en dates réelles (`WHERE date BETWEEN`) — les colonnes `year`/`week` n'existent pas dans `tb_model_contributions`. Guardrails whitelist durcis (`[`, `]`, `;`, `=`, newlines exclus), security review validée. PRs #1688 (`develop`) → #1689 (`main`) mergées. Rollback disponible sur engine `2659` sans redéploiement.

**Prochaine action** : surveiller le monitoring (`no_answer` + `uiScopeRejected` en prod) ; mettre au backlog le chip de contexte chatbot (`Contexte : ROAS · juin 2025 – mai 2026`) ; relancer le conseil DE Stellantis (Zenith Media — Marit, Janina, Virginia silencieux depuis le mail deck) pour l'initiation agent ; suite du backlog : enrichissement `BUSINESS_CONTEXT` avec les DS (Dan/Hajar), `cf-budget-allocator-prod` en tool, knowledge par client.

**Blocker** : cadrage "recommandation" à trancher avec Baptiste — option A (deux niveaux interne/client : reco pleine en interne, faits seulement côté client) vs option B (assistant pur, aucune reco ni interne ni client). Décision structurante : conditionne le tool d'allocation budget (`cf-budget-allocator-prod`). Conseil DE Stellantis ne répond plus depuis le mail deck.

---

### 2. AMC Analytics — MINE × L'ORÉAL
**implication : lead** | discipline : dev | client : LOREAL

Module ConnectedHub centralisant les analyses Amazon Marketing Cloud (Full Funnel depuis BigQuery) et les dashboards Looker (en iframe). Deux types d'utilisateurs : équipes data (Full Funnel) et équipes conseil/traders (dashboards).

**Statut (11/08 — actif)** : Module **complet en production sur `develop` et `main` (branches identiques)**. Chaîne complète de PRs mergées : workspace pivot (#1719-#1723, 10/08), couche graphiques (#1724/#1725 — 4 formes : barres, empilement 100%, nuage de points, Sankey), retours Jules (#1726/#1728 — niveau d'analyse imposé par cas d'usage, entrée exploration libre, deux nouveaux cas d'usage Vue d'ensemble et Débuteur/finisseur, catégorie sans nom corrigée à la racine), analyses enregistrées (#1733/#1736 — Firestore, `?a=<id>` dans l'URL, module qui ouvre désormais sur l'historique des analyses). 413 tests (356 client + 57 serveur), typage propre, build OK. L'incident git du 11/08 est résolu depuis le 13/08 — statut du commit `76dbb7212` (`feat/amc-saved-analyses`) à vérifier. Derniers retours Jules en attente : feedbacks PowerPoint et feedbacks mail (dont les `analysis_level` **`time of`**).

**Prochaine action** : traiter les feedbacks PowerPoint et mail de Jules (`time of`) ; vérifier le commit `76dbb7212` (poussé ou non) ; réaliser un tableau des granularités des leviers AMC possibles et l'intégrer dans la logique des groupements ; vérifier le module à l'écran (aucune validation visuelle formelle malgré les tests) ; configurer Firestore + comptes utilisateurs Publicis via Settings ; cadrer le calendrier de migration dashboards Looker → React natif avec Khadija.

**Blocker** : ingestion BQ et mise à jour du registre sont manuelles. Migration Looker → React dépend du calendrier Khadija.

---

### 3. Brand Store v2 — L'ORÉAL
**implication : participant** | discipline : data-analyst | client : LOREAL

Étude L'Oréal Brand Store. Adam est participant (suivi, pas de livrable porté).

**Statut** : aucune session dans le vault, aucun TODO actif côté Adam. Sujet porté par d'autres. Pas d'action prévue de la part d'Adam.

---

### 4. Creative Insights — MINE
**implication : lead** (à confirmer avec Hajar et Jules) | discipline : dev + data-science | client : **MINE** — module plateforme transverse

Module ConnectedHub d'automatisation des Creative Insights. Objectif : rendre self-service la chaîne aujourd'hui exécutée à la main par les DS, de la récupération des assets créa (Meta, TikTok, Snapchat) jusqu'à la restitution au Data Analyst.

**Statut (13/08 — dev actif)** : Incident git résolu — tout ce qui était local est désormais sur GitHub. **Lots 1 et 2 livrés et mergés** : rôles éditeur/administrateur avec circuit de validation (mail SendGrid aux admins, carte d'activité dans la ligne BQ), sélecteur de rôle dans l'admin panel, icône du module (#1738-#1741). Bug de fraîcheur du rôle corrigé : hook `useRole` supprimé, `viewer_role` désormais calculé à la requête serveur — les boutons ne peuvent plus être en désaccord avec les autorisations réelles. **Découverte produit critique** : sans `campaign_id`, le scrapper énumère les 1545 campagnes du compte et se fait throttler par Meta (code 17/80004) — `campaign_id` n'est plus un filtre optionnel, c'est ce qui rend le run exécutable sur un gros compte. **`project_id` validé par Hajar** (13/08). **Lot 3 poussé** (`feat/creative-insights-project-id`, pas encore de PR) : colonne `project_id` créée sur `meta.tb_api_asset_urls`, requêtes avec repli `(project_id = @project_id OR (project_id IS NULL AND ad_account_id = @account_id))` — comportement actuel préservé à l'identique. Plan modifications DS rédigé ([[Plan modifications DS]]), pas encore envoyé à Hajar et Abhishek. Ticket IT toujours en attente (staging non fonctionnel — aucun rôle sur le projet, aucune ACL BQ).

**Prochaine action** : envoyer le [[Plan modifications DS]] à Hajar et Abhishek (5 évolutions attendues côté Cloud Functions) ; valider à l'écran le lot 2 (boutons admin sans rechargement, nouvelle icône) et le lot 3 (repli SQL) ; ouvrir une PR pour `feat/creative-insights-project-id` ; suivre le ticket IT ; localiser les prompts Gemini et l'infrastructure d'exécution pour l'étape 4 (non trouvés dans le projet GCP exploré).

**Blocker** : Plan DS non encore envoyé (Hajar et Abhishek ne connaissent pas les 5 évolutions attendues). Ticket IT en attente (service account sans droits, staging inaccessible). Fragilité de l'observateur asynchrone : si le process Node redémarre pendant un run long, l'issue n'est pas écrite (correctif durable : Cloud Tasks ou Cloud Run Job). TikTok et Snapchat sans `ad_id` : jointure performance impossible. Répartition du dev non tranchée (couche ConnectedHub vs backend DS). Prompts Gemini et notebooks d'exécution non localisés (étape 4 bloquée).

---

## Blockers transverses

| Blocker | Projet(s) | Qui débloque |
|---|---|---|
| Cadrage "recommandation" agent — option A (deux niveaux interne/client) vs option B (assistant pur, aucune reco) | MMM AI Agent | Baptiste |
| Conseil DE Stellantis silencieux — Zenith Media ne répond plus depuis le mail deck | MMM AI Agent | Katia + Zenith Media DE |
| Plan modifications DS non envoyé — Hajar et Abhishek ne connaissent pas les 5 évolutions Cloud Function attendues | Creative Insights | Adam (action immédiate) |
| Ticket IT en attente — droits service account `interface@` sur `zen-creativeinsights-dev-mg`, staging non fonctionnel | Creative Insights | IT / James |
| Fragilité observateur async — issue du run non écrite si le process Node meurt avant la fin ; correctif durable = Cloud Tasks ou Cloud Run Job | Creative Insights | Hajar + Abhishek (conception) |
| TikTok et Snapchat ne remontent pas `ad_id` — jointure performance impossible | Creative Insights | Hajar + Abhishek |
| `project_id` — concept validé par Hajar (13/08), forme exacte (paramètre d'entrée + colonne) à cadrer formellement dans le Plan DS | Creative Insights | Hajar |
| Prompts Gemini et notebooks d'exécution non localisés — étape 4 (extraction features) bloquée | Creative Insights | Hajar |
| Répartition du dev non tranchée (couche ConnectedHub vs backend DS) | Creative Insights | Hajar + Jules |
| Ingestion BQ AMC manuelle (registre non automatisé) | AMC Analytics | Khadija (ingestion) |

---

## Personnes clés transverses

| Personne | Rôle | Projets |
|---|---|---|
| Jules | Manager direct — arbitrages scope, évolution, recrutement | Tous |
| Baptiste | Head of Data (futur pôle études & mesure) — go agent, cadrage reco | MMM AI Agent |
| Eddie | Lead dev ConnectedHub — archi frontend/backend, décisions infra | MMM AI Agent, AMC Analytics |
| Khadija | Data Analyst — dashboards Looker AMC, ingestion BQ | AMC Analytics |
| Dan | Data Scientist — testing MMM Agent, EVBB Stellantis | MMM AI Agent, EVBB Scoring |
| Hajar | Data Scientist — testing MMM, AMC modeled audiences, porte le sujet Creative Insights | MMM AI Agent, AMC Analytics, Creative Insights |
| Abhishek | Data Scientist PGD (Inde) — cloud functions d'extraction d'assets, notebooks SIMBA, antécédent Leapmotor | Creative Insights |
| Thomas | Data Analyst Social + L'Oréal — consommateur final des Creative Insights, auteur du dashboard alimenté par CSV | Creative Insights |
| James | IT / infra — référent incident git du 11/08, remote repository | Tous |
| Manu | AI Office / manager direct DA — Copilot audiences | (transverse) |
| Inès | Data Strat Stellantis — contact ouverture conseil MMM | MMM AI Agent |
| Katia | Data Strat Stellantis — contact ouverture conseil MMM | MMM AI Agent |
| Pierre | Data Strat L'Oréal — référent AMC, Brand Store | AMC Analytics |
| Brieg | Lead DS (parti 12/06/2026) — a initié MMM Agent avec Adam et Hajar | MMM AI Agent (historique) |

---

## Contexte récent important

| Date | Fait |
|---|---|
| 2026-08-13 | **Creative Insights — deuxième session de dev.** Lots 1 et 2 mergés : rôles éditeur/administrateur, circuit de validation, mail admins, icône module (#1738-#1741). Bug de fraîcheur du rôle corrigé (`viewer_role` calculé côté serveur). Découverte critique : sans `campaign_id`, Meta throttle le compte entier (1545 campagnes énumérées, code 17/80004) — `campaign_id` obligatoire. `project_id` validé par Hajar. Lot 3 poussé : colonne `project_id` + repli SQL, comportement actuel préservé. Incident git résolu. |
| 2026-08-12 | **Creative Insights — première session de dev.** Étapes 1 et 2 livrées localement : projets en BigQuery, déclenchement Meta validé de bout en bout (35 créas, 1 min 35). Étape 3 (bibliothèque de créas) fonctionnelle, non commitée. Découverte structurante : un projet = un compte annonceur (pas de `project_id` dans les CF). |
| 2026-08-11 | **Incident git — compte GitHub compromis.** Remote détaché, tout push bloqué. Incident résolu le 13/08 (remote de nouveau accessible). |
| 2026-08-11 | Session AMC Analytics : **retours Jules (10/11 actions)** — niveau d'analyse imposé par cas d'usage, entrée exploration libre, Vue d'ensemble, Débuteur/finisseur. **Analyses enregistrées** (Firestore, `?a=<id>` dans l'URL, lien partageable fonctionnel). PRs #1726/#1728 et #1733/#1736 mergées — module identique sur `develop` et `main`, en production. |
| 2026-08-10 | Session AMC Analytics : **couche graphiques** (PR #1724/#1725 — Sankey, nuage de points volume×ROAS, barres, empilement 100%). Double axe refusé par principe. Graphiques figés sur la vue du preset, réglages descendus sous eux. |

---

## Ce qui n'est PAS un projet actif d'Adam

| Projet | Statut vault | Qui porte | Note |
|---|---|---|---|
| Viral Beauty (L'ORÉAL) | `4.Archive/` — archivé | Jules (a repris en relais depuis le 06/07) | Adam était participant, maintenant backup passif. Départ Basma = fin de ce sujet côté Adam. |
| Passation Basma (PUBLICIS) | `4.Archive/` — archivé le 2026-08-03 | — | Fenêtre de capture close au départ de Basma (23/07), knowledge resté `[À COMPLÉTER]`. Volet Copilot / audiences **abandonné** côté Adam (cohérent avec le positionnement DS/IA : pas de rôle de transition). |
| FeedGen Catégorisation (MINE) | `4.Archive/` — archivé le 2026-08-03 | Dan (solution LLM en intégration) | Exploration RAG d'Adam arrivée après coup. Acquis techniques valides (BQ natif, métrique hiérarchique, V3 rerank 88,8 %). Réactivable si Manu fournit le jeu d'éval ou si Dan ouvre le RAG en amont. |
| EVBB Scoring (STELLANTIS) | `3.Resources/Onboardings/` — `implication: onboardé` | Dan | Adam onboardé uniquement. La note documente le travail de Dan (pipeline BQML Logistic Regression, bid values VBB). |
| Creative Insights Verisure (VERISURE) | Note supprimée le 2026-08-10 | Hajar + Léonie | POC client jamais travaillé par Adam. Le sujet est devenu le module plateforme [[1.Projects/MINE_CreativeInsights/Creative Insights]], qui lui **est** un projet actif d'Adam (lead). |
