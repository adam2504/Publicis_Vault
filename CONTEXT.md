---
type: context
Dernière mise à jour: 2026-08-03
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

**Statut (23/07)** : Engine **6241** en prod depuis le 23/07 (`MMM_Agent_v3_live_ui_scope`). Feature B (scope écran live) livrée : l'agent connaît désormais le KPI, la période, l'onglet et la langue affichés à l'écran, et répond dans ce scope par défaut (surchargeable dimension par dimension, annonce le scope sur sa 1re réponse). Défaut critique corrigé en revue de branche : les bornes de période passent en dates réelles (`WHERE date BETWEEN`) — les colonnes `year`/`week` n'existent pas dans `tb_model_contributions`. Guardrails whitelist durcis (`[`, `]`, `;`, `=`, newlines exclus), security review validée. PRs #1688 (`develop`) → #1689 (`main`) mergées. Rollback disponible sur engine `2659` sans redéploiement.

**Prochaine action** : surveiller le monitoring (`no_answer` + `uiScopeRejected` en prod) ; mettre au backlog le chip de contexte chatbot (`Contexte : ROAS · juin 2025 – mai 2026`) ; relancer le conseil DE Stellantis (Zenith Media — Marit, Janina, Virginia silencieux depuis le mail deck) pour l'initiation agent ; suite du backlog : enrichissement `BUSINESS_CONTEXT` avec les DS (Dan/Hajar), `cf-budget-allocator-prod` en tool, knowledge par client.

**Blocker** : cadrage "recommandation" à trancher avec Baptiste — option A (deux niveaux interne/client : reco pleine en interne, faits seulement côté client) vs option B (assistant pur, aucune reco ni interne ni client). Décision structurante : conditionne le tool d'allocation budget (`cf-budget-allocator-prod`). Conseil DE Stellantis ne répond plus depuis le mail deck.

---

### 2. AMC Analytics — MINE × L'ORÉAL
**implication : lead** | discipline : dev | client : LOREAL

Module ConnectedHub centralisant les analyses Amazon Marketing Cloud (Full Funnel depuis BigQuery) et les dashboards Looker (en iframe). Deux types d'utilisateurs : équipes data (Full Funnel) et équipes conseil/traders (dashboards).

**Statut (09/07 — stable)** : Full Funnel branché sur BigQuery — PR **#1653** (`develop`) et **#1654** (`main`) mergées. Architecture data : projet `amira-test` (EU), dataset `AMC_ConnectedHub_7cR1jE` par client, tables `<étude>__<marque>__<période>` + registre. Données chargées : Mugler (298 l.) + Azzaro (1 366 l.). Matrice d'accès révisée : L'Oréal = Audiences Insights seul ; Publicis commerce = les 3 dashboards. Accès Looker de Nicolas Vivies (PMO Retail Media L'Oréal) débloqué (partage "unlisted" côté Looker avec Khadija). Aucune session depuis le 09/07 (23 jours).

**Prochaine action** : automatiser la sync du registre BQ à chaque import (manuel aujourd'hui) ; configurer Firestore + comptes utilisateurs Publicis via Settings ; cadrer le calendrier de migration dashboards Looker → React natif avec Khadija.

**Blocker** : ingestion BQ et mise à jour du registre sont manuelles aujourd'hui (aucune Cloud Function dédiée). Migration Looker → React dépend du calendrier Khadija.

---

### 3. Brand Store v2 — L'ORÉAL
**implication : participant** | discipline : data-analyst | client : LOREAL

Étude L'Oréal Brand Store. Adam est participant (suivi, pas de livrable porté).

**Statut** : aucune session dans le vault, aucun TODO actif côté Adam. Sujet porté par d'autres. Pas d'action prévue de la part d'Adam.

---

## Blockers transverses

| Blocker | Projet(s) | Qui débloque |
|---|---|---|
| Cadrage "recommandation" agent — option A (deux niveaux interne/client) vs option B (assistant pur, aucune reco) | MMM AI Agent | Baptiste |
| Conseil DE Stellantis silencieux — Zenith Media ne répond plus depuis le mail deck | MMM AI Agent | Katia + Zenith Media DE |
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
| Hajar | Data Scientist — testing MMM, AMC modeled audiences, Creative Insights | MMM AI Agent, AMC Analytics |
| Manu | AI Office / manager direct DA — Copilot audiences | (transverse) |
| Inès | Data Strat Stellantis — contact ouverture conseil MMM | MMM AI Agent |
| Katia | Data Strat Stellantis — contact ouverture conseil MMM | MMM AI Agent |
| Pierre | Data Strat L'Oréal — référent AMC, Brand Store | AMC Analytics |
| Brieg | Lead DS (parti 12/06/2026) — a initié MMM Agent avec Adam et Hajar | MMM AI Agent (historique) |

---

## Contexte récent important

| Date | Fait |
|---|---|
| 2026-07-23 | Feature B (scope écran live) livrée en prod : engine **6241**. L'agent connaît KPI, période, onglet et langue de l'écran. Défaut critique corrigé : bornes de période en dates réelles (colonnes `year`/`week` absentes de BQ). PRs #1688/#1689 mergées. |
| 2026-07-23 | Basma quitte Publicis — knowledge audiences LiveRamp non rempli à son départ (schéma scaffoldé, `[À COMPLÉTER]` partout). Fenêtre de capture définitivement close. |
| 2026-07-22 | Engine MMM Agent **2659** en prod : `CTX_MODULE_VIZ` (aide lecture graphiques bilingue EN/FR), retry backend `no_answer`, polish chatbot. PRs #1685/#1686 mergées. |
| 2026-07-20 | `BUSINESS_CONTEXT` décomposé en blocs `CTX_*` scopés par sous-agent — hallucination `execute_sql` fixée. LiveRamp (Julien Guého) : création d'audience = UI only, API fermée. |
| 2026-07-09 | Full Funnel AMC branché sur BigQuery (PR #1653/#1654 mergées sur main). Accès Looker Nicolas Vivies débloqué (partage "unlisted" avec Khadija). |

---

## Ce qui n'est PAS un projet actif d'Adam

| Projet | Statut vault | Qui porte | Note |
|---|---|---|---|
| Viral Beauty (L'ORÉAL) | `4.Archive/` — archivé | Jules (a repris en relais depuis le 06/07) | Adam était participant, maintenant backup passif. Départ Basma = fin de ce sujet côté Adam. |
| Passation Basma (PUBLICIS) | `4.Archive/` — archivé le 2026-08-03 | — | Fenêtre de capture close au départ de Basma (23/07), knowledge resté `[À COMPLÉTER]`. Volet Copilot / audiences **abandonné** côté Adam (cohérent avec le positionnement DS/IA : pas de rôle de transition). |
| FeedGen Catégorisation (MINE) | `4.Archive/` — archivé le 2026-08-03 | Dan (solution LLM en intégration) | Exploration RAG d'Adam arrivée après coup. Acquis techniques valides (BQ natif, métrique hiérarchique, V3 rerank 88,8 %). Réactivable si Manu fournit le jeu d'éval ou si Dan ouvre le RAG en amont. |
| EVBB Scoring (STELLANTIS) | `3.Resources/Onboardings/` — `implication: onboardé` | Dan | Adam onboardé uniquement. La note documente le travail de Dan (pipeline BQML Logistic Regression, bid values VBB). |
| Creative Insights Verisure (VERISURE) | `3.Resources/Onboardings/` — `implication: onboardé` | Hajar + Léonie | Adam onboardé uniquement. Aucune contribution au livrable. |
