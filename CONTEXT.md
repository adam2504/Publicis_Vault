---
type: context
Dernière mise à jour: 2026-08-12
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

**Statut (23/07 — aucune session depuis, 20 jours)** : Engine **6241** en prod depuis le 23/07 (`MMM_Agent_v3_live_ui_scope`). Feature B (scope écran live) livrée : l'agent connaît désormais le KPI, la période, l'onglet et la langue affichés à l'écran, et répond dans ce scope par défaut (surchargeable dimension par dimension, annonce le scope sur sa 1re réponse). Défaut critique corrigé en revue de branche : les bornes de période passent en dates réelles (`WHERE date BETWEEN`) — les colonnes `year`/`week` n'existent pas dans `tb_model_contributions`. Guardrails whitelist durcis (`[`, `]`, `;`, `=`, newlines exclus), security review validée. PRs #1688 (`develop`) → #1689 (`main`) mergées. Rollback disponible sur engine `2659` sans redéploiement.

**Prochaine action** : surveiller le monitoring (`no_answer` + `uiScopeRejected` en prod) ; mettre au backlog le chip de contexte chatbot (`Contexte : ROAS · juin 2025 – mai 2026`) ; relancer le conseil DE Stellantis (Zenith Media — Marit, Janina, Virginia silencieux depuis le mail deck) pour l'initiation agent ; suite du backlog : enrichissement `BUSINESS_CONTEXT` avec les DS (Dan/Hajar), `cf-budget-allocator-prod` en tool, knowledge par client.

**Blocker** : cadrage "recommandation" à trancher avec Baptiste — option A (deux niveaux interne/client : reco pleine en interne, faits seulement côté client) vs option B (assistant pur, aucune reco ni interne ni client). Décision structurante : conditionne le tool d'allocation budget (`cf-budget-allocator-prod`). Conseil DE Stellantis ne répond plus depuis le mail deck.

---

### 2. AMC Analytics — MINE × L'ORÉAL
**implication : lead** | discipline : dev | client : LOREAL

Module ConnectedHub centralisant les analyses Amazon Marketing Cloud (Full Funnel depuis BigQuery) et les dashboards Looker (en iframe). Deux types d'utilisateurs : équipes data (Full Funnel) et équipes conseil/traders (dashboards).

**Statut (11/08 — actif)** : Module **complet en production sur `develop` et `main` (branches identiques)**. Chaîne complète de PRs mergées : workspace pivot (#1719-#1723, 10/08), couche graphiques (#1724/#1725 — 4 formes : barres, empilement 100%, nuage de points, Sankey), retours Jules (#1726/#1728 — niveau d'analyse imposé par cas d'usage, entrée exploration libre, deux nouveaux cas d'usage Vue d'ensemble et Débuteur/finisseur, catégorie sans nom corrigée à la racine), analyses enregistrées (#1733/#1736 — Firestore, `?a=<id>` dans l'URL, module qui ouvre désormais sur l'historique des analyses). 413 tests (356 client + 57 serveur), typage propre, build OK. Un commit local (`76dbb7212` sur `feat/amc-saved-analyses`) non poussé — bloqué par l'incident git du 11/08. Dernier retour Jules pendant : les `analysis_level` **`time of`**.

**Prochaine action** : résoudre l'incident git et pousser le commit AMC en attente ; traiter le dernier retour Jules (`time of`) ; vérifier le module à l'écran (aucune validation visuelle formelle malgré les tests) ; configurer Firestore + comptes utilisateurs Publicis via Settings ; cadrer le calendrier de migration dashboards Looker → React natif avec Khadija.

**Blocker** : incident git du 11/08 (remote détaché, attente James). Ingestion BQ et mise à jour du registre sont manuelles. Migration Looker → React dépend du calendrier Khadija.

---

### 3. Brand Store v2 — L'ORÉAL
**implication : participant** | discipline : data-analyst | client : LOREAL

Étude L'Oréal Brand Store. Adam est participant (suivi, pas de livrable porté).

**Statut** : aucune session dans le vault, aucun TODO actif côté Adam. Sujet porté par d'autres. Pas d'action prévue de la part d'Adam.

---

### 4. Creative Insights — MINE
**implication : lead** (à confirmer) | discipline : dev + data-science | client : **MINE** — module plateforme transverse

Module ConnectedHub d'automatisation des Creative Insights. Objectif : rendre self-service la chaîne aujourd'hui exécutée à la main par les DS, de la récupération des assets créa (Meta, TikTok, Snapchat) jusqu'à la restitution au Data Analyst.

**Statut (12/08 — dev actif)** : Première session de dev le 12/08. **Étapes 1 et 2 livrées localement** : projets persistés en BigQuery (dataset `connectedhub` créé dans `zen-creativeinsights-dev-mg`), déclenchement Meta branché et **validé de bout en bout** (35 créas récupérées en 1 min 35, fichiers GCS horodatés du jour). Bibliothèque de créas (étape 3) fonctionnelle mais non commitée — à relire à l'écran avant commit. Deux commits locaux (`575304be6`, `5062ebf88`) — rien poussé, remote détaché depuis l'incident du 11/08. Ticket IT envoyé pour les droits du service account `interface@` sur `zen-creativeinsights-dev-mg` (sans ça : marche en local, pas déployé). Découverte structurante : **un projet = un compte annonceur** (la Cloud Function n'écrit aucun `project_id`, ce qui interdit deux projets sur le même compte). TikTok et Snapchat ne remontent pas `ad_id` : jointure performance impossible sur ces plateformes aujourd'hui.

**Prochaine action** : relire et commiter l'étape 3 (bibliothèque de créas) ; attendre réponse Hajar sur le `project_id` dans les CF et la table d'assets (débloque tout le reste) ; suivre ticket IT ; localiser les prompts Gemini existants et l'infrastructure d'exécution pour l'étape 4 (non trouvés dans le projet GCP exploré).

**Blocker** : incident git du 11/08 (remote détaché, commits non poussés, déploiement bloqué). Ticket IT en attente (service account sans droits). `project_id` absent des Cloud Functions et tables GCS — bloque la granularité par projet et tout le dev aval. TikTok et Snapchat sans `ad_id` : prérequis DS avant tout dev multi-plateformes. Répartition du dev non tranchée (couche ConnectedHub vs backend DS — Abhishek démarre le dev initial selon le CR).

---

## Blockers transverses

| Blocker | Projet(s) | Qui débloque |
|---|---|---|
| **Incident git 11/08** — compte GitHub compromis, remote détaché, tout push bloqué | Tous (AMC commit non poussé, Creative Insights commits non poussés) | Adam + James |
| Cadrage "recommandation" agent — option A (deux niveaux interne/client) vs option B (assistant pur, aucune reco) | MMM AI Agent | Baptiste |
| Conseil DE Stellantis silencieux — Zenith Media ne répond plus depuis le mail deck | MMM AI Agent | Katia + Zenith Media DE |
| `project_id` absent des Cloud Functions et tables GCS — bloque multi-projets sur un même compte annonceur | Creative Insights | Hajar |
| TikTok et Snapchat ne remontent pas `ad_id` — jointure performance impossible | Creative Insights | Hajar + Abhishek |
| Ticket IT en attente (droits service account `interface@` sur `zen-creativeinsights-dev-mg`) | Creative Insights | IT / James |
| Prompts Gemini et notebooks d'exécution non localisés (étape 4 bloquée) | Creative Insights | Hajar |
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
| 2026-08-12 | **Creative Insights — première session de dev.** Étapes 1 et 2 livrées localement : projets en BigQuery, déclenchement Meta validé de bout en bout (35 créas, 1 min 35). Étape 3 (bibliothèque de créas) fonctionnelle, non commitée. Deux commits locaux non poussés (remote détaché). Découverte structurante : un projet = un compte annonceur (pas de `project_id` dans les CF). |
| 2026-08-11 | **Incident git — compte GitHub compromis.** Remote détaché, tout push bloqué. Attente réponse de James (remote assaini ? `main` réécrit ?). Commits Creative Insights et commit AMC restent locaux. |
| 2026-08-11 | Session AMC Analytics : **retours Jules (10/11 actions)** — niveau d'analyse imposé par cas d'usage, entrée exploration libre, Vue d'ensemble, Débuteur/finisseur. **Analyses enregistrées** (Firestore, `?a=<id>` dans l'URL, lien partageable fonctionnel). PRs #1726/#1728 et #1733/#1736 mergées — module identique sur `develop` et `main`, en production. |
| 2026-08-10 | Session AMC Analytics : **couche graphiques** (PR #1724/#1725 — Sankey, nuage de points volume×ROAS, barres, empilement 100%). Double axe refusé par principe. Graphiques figés sur la vue du preset, réglages descendus sous eux. |
| 2026-07-23 | **Feature B MMM livrée en prod** : engine **6241**. L'agent connaît KPI, période, onglet et langue de l'écran. Défaut critique corrigé : bornes de période en dates réelles (colonnes `year`/`week` absentes de BQ). PRs #1688/#1689 mergées. |

---

## Ce qui n'est PAS un projet actif d'Adam

| Projet | Statut vault | Qui porte | Note |
|---|---|---|---|
| Viral Beauty (L'ORÉAL) | `4.Archive/` — archivé | Jules (a repris en relais depuis le 06/07) | Adam était participant, maintenant backup passif. Départ Basma = fin de ce sujet côté Adam. |
| Passation Basma (PUBLICIS) | `4.Archive/` — archivé le 2026-08-03 | — | Fenêtre de capture close au départ de Basma (23/07), knowledge resté `[À COMPLÉTER]`. Volet Copilot / audiences **abandonné** côté Adam (cohérent avec le positionnement DS/IA : pas de rôle de transition). |
| FeedGen Catégorisation (MINE) | `4.Archive/` — archivé le 2026-08-03 | Dan (solution LLM en intégration) | Exploration RAG d'Adam arrivée après coup. Acquis techniques valides (BQ natif, métrique hiérarchique, V3 rerank 88,8 %). Réactivable si Manu fournit le jeu d'éval ou si Dan ouvre le RAG en amont. |
| EVBB Scoring (STELLANTIS) | `3.Resources/Onboardings/` — `implication: onboardé` | Dan | Adam onboardé uniquement. La note documente le travail de Dan (pipeline BQML Logistic Regression, bid values VBB). |
| Creative Insights Verisure (VERISURE) | Note supprimée le 2026-08-10 | Hajar + Léonie | POC client jamais travaillé par Adam. Le sujet est devenu le module plateforme [[1.Projects/MINE_CreativeInsights/Creative Insights]], qui lui **est** un projet actif d'Adam (lead). |
