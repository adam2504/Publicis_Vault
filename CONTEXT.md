---
type: context
Dernière mise à jour: 2026-07-22
---

# CONTEXT — Vault d'Adam

> Fichier de synthèse régénéré automatiquement. Source de vérité pour reprendre le contexte d'Adam sans rouvrir chaque note.

---

## Qui est Adam

Adam Jouini, apprenti Data & Dev chez Publicis Media (alternance), rattaché à la plateforme **ConnectedHub** (produit interne Mine). Il travaille au croisement du développement (React/Node.js/backend), de la data science (Vertex AI, embeddings, agents) et de l'analyse de données (AMC, MMM, audiences LiveRamp). Orientation data science exprimée lors du point évolution de juin 2026. Jules a dit explicitement vouloir proposer un poste à la sortie de l'alternance, en citant l'agent MMM et AMC Analytics comme réalisations commercialement visibles.

---

## Projets actifs

### 1. MMM AI Agent — MINE
**implication : lead** | discipline : dev + data-science | client : MINE (L'Oréal et autres)

Agent IA conversationnel dans le module MMM de ConnectedHub (Vertex AI Agent Engine). Répond aux questions ROI/média des clients sur leurs données MMM.

**Statut (22/07)** : Engine **2659** en prod depuis le 22/07. Livraisons du cycle 20-22/07 : aide à la lecture des graphiques (`CTX_MODULE_VIZ`, bilingue EN/FR, sourcé des fichiers i18n), retry backend sur `no_answer` non-déterministe (1 retry, logs `attempt`/`retried`), fix hallucination `execute_sql` via décomposition du `BUSINESS_CONTEXT` en blocs scopés par sous-agent, polish chatbot (questions recommandées data/interprétation/navigation, fix portail z-index chatbot flottant, repositionnement si panneau droit ouvert). PRs #1682→#1683 (retry + eng 2659) et #1685→#1686 (polish) mergées sur main.

**Prochaine action** : surveiller le monitoring (`no_answer` + `retried:true` en prod) ; supprimer les anciens engines (6073 supprimable, 7899 rollback jusqu'à preuve de 2659) ; démarrer le backlog priorisé dans cet ordre : live-suivi UI (feature B), enrichissement `BUSINESS_CONTEXT` avec les DS, `cf-budget-allocator-prod` en tool, knowledge par client.

**Blocker** : cadrage "recommandation" (jusqu'où l'agent recommande vs se limite aux faits côté client) à trancher avec Baptiste + data strats. Validation formelle complète pas encore signée avant mise en avant client.

---

### 2. FeedGen Catégorisation — MINE
**implication : lead** | discipline : data-science | client : MINE

Catégorisation automatique des produits e-commerce dans la taxonomie GPC (Google Product Categories) via un pipeline RAG + LLM reranker, en aval de FeedGen.

**Statut (06/07)** : V3 LLM reranking livré et benchmarké sur jeu gold Kérastase (294 produits, titres optimisés main + GPC corrigé). Résultat : **88,8 % hiérarchique** (Gemini Flash reranker, arm A) vs 75,9 % bi-encoder seul. Cross-encoder (arm B) : meilleur exact-ID (24,1 %) mais pire hiérarchique (60,5 %), levier réel uniquement fine-tuné. Architecture stabilisée sur **BQ natif** (hors Vertex AI Vector Search, trop coûteux à idle). Repo `feedgen-categorisation-rag` propre, commit `25073d1`. V1 propre à rejouer via BQ (ancien baseline non fiable).

**Prochaine action** : construire un vrai jeu d'éval avec Manu (GPC vérifié à la main + titre brut — sortir du plafond "titres opti" et du mono-flux Kérastase) ; explorer le cross-encoder en prod (serving batch + fine-tuning) ; approcher Dan pour brancher le RAG en amont de sa solution LLM.

**Blocker** : dépendance à Manu pour le jeu d'éval multi-flux et en titres bruts. Aucune session depuis le 06/07 (16 jours).

---

### 3. AMC Analytics — MINE × L'ORÉAL
**implication : lead** | discipline : dev | client : LOREAL

Module ConnectedHub centralisant les analyses Amazon Marketing Cloud (Full Funnel depuis BigQuery) et les dashboards Looker (en iframe). Deux types d'utilisateurs : équipes data (Full Funnel) et équipes conseil/traders (dashboards).

**Statut (09/07)** : Full Funnel branché sur BigQuery — PR **#1653** (`develop`) et **#1654** (`main`) mergées. Architecture data : projet `amira-test` (EU), dataset `AMC_ConnectedHub_7cR1jE` par client, tables `<étude>__<marque>__<période>` + registre. Données chargées : Mugler (298 l.) + Azzaro (1 366 l.). Matrice d'accès révisée : L'Oréal = Audiences Insights seul ; Publicis commerce = les 3 dashboards. Accès Looker de Nicolas Vivies (PMO Retail Media L'Oréal) débloqué (partage "unlisted" côté Looker avec Khadija).

**Prochaine action** : automatiser la sync du registre BQ à chaque import (manuel aujourd'hui) ; configurer Firestore + comptes utilisateurs Publicis via Settings ; cadrer le calendrier de migration dashboards Looker → React natif avec Khadija.

**Blocker** : ingestion BQ et mise à jour du registre sont manuelles aujourd'hui (aucune Cloud Function dédiée). Migration Looker → React dépend du calendrier Khadija.

---

### 4. Passation Basma — PUBLICIS
**implication : lead** | discipline : data-analyst + data-science | client : PUBLICIS

Double objectif : capturer le knowledge audiences LiveRamp de Basma avant son départ le **23/07**, puis brancher le Copilot de Manu sur ce knowledge pour assister la remplaçante à la création d'audiences.

**Statut (20/07)** : Basma part **demain (23/07)**. Réponse LiveRamp (Julien Guého, Head CS Continental Europe, 20/07) : création d'audience Safe Haven = **UI only**, distribution API = réservée aux plateformes tierces. Piste ConnectedHub via API directe **fermée**. Seule voie programmatique partielle : Analytics Environment → Customer Profiles (à instruire avec Lydia/Hajar DS). Knowledge : 5 fichiers `.md` avec schéma excellent mais données réelles absentes (`[À COMPLÉTER]` partout). Copilot de Manu retenu (plus avancé) ; autorisation ajout fichiers bloquée avec Bradley.

**Prochaine action** : exploiter les dernières heures de passation avant le 23/07 pour remplir le knowledge ; nettoyer/segmenter ensuite ; brancher le Copilot de Manu ; instruire la piste Analytics Environment → Customer Profiles avec les DS.

**Blocker** : temps écoulé — Basma part le 23/07. Autorisation ajout fichier Copilot bloquée par Bradley.

---

### 5. Brand Store v2 — L'ORÉAL
**implication : participant** | discipline : data-analyst | client : LOREAL

Étude L'Oréal Brand Store. Adam est participant (suivi, pas de livrable porté).

**Statut** : aucune session dans le vault, aucun TODO actif côté Adam. Sujet porté par d'autres (Basma historiquement, désormais en transition avec son départ). Pas d'action prévue de la part d'Adam.

---

## Blockers transverses

| Blocker | Projet(s) | Qui débloque |
|---|---|---|
| Cadrage "recommandation" agent (scope côté client) | MMM AI Agent | Baptiste + data strats |
| Validation formelle agent avant go client | MMM AI Agent | Baptiste, Hajar, Dan |
| Autorisation ajout fichiers Copilot | Passation Basma | Bradley |
| Départ Basma le 23/07 — knowledge pas encore rempli | Passation Basma | Basma (urgence) |
| Jeu d'éval multi-flux FeedGen (GPC brut + titre brut) | FeedGen Catégorisation | Manu |
| Ingestion BQ AMC manuelle (registre non automatisé) | AMC Analytics | Khadija (ingestion) |
| API LiveRamp fermée — seule piste DS : Analytics Env → Customer Profiles | Passation Basma | Lydia / Hajar |

---

## Personnes clés transverses

| Personne | Rôle | Projets |
|---|---|---|
| Jules | Manager direct — arbitrages scope, évolution, recrutement | Tous |
| Baptiste | Head of Data (futur pôle études & mesure) — go agent, cadrage reco | MMM AI Agent |
| Eddie | Lead dev ConnectedHub — archi frontend/backend, décisions infra | MMM AI Agent, AMC Analytics |
| Khadija | Data Analyst — dashboards Looker AMC, ingestion BQ | AMC Analytics |
| Dan | Data Scientist — testing MMM Agent, FeedGen scoring existant | MMM AI Agent, FeedGen |
| Hajar | Data Scientist — testing MMM, AMC modeled audiences, EVBB | MMM AI Agent, AMC Analytics |
| Manu | AI Office — Copilot audiences, arbitrage scope Passation | Passation Basma, FeedGen (jeu d'éval) |
| Basma | Data Analyst L'Oréal — départ 23/07 | Passation Basma |
| Pierre | Data Strat L'Oréal — référent AMC, Brand Store | AMC Analytics |
| Bradley | ? — autorisation ajout fichiers Copilot | Passation Basma |
| Julien Guého | Head CS Continental Europe, LiveRamp — a confirmé no API | Passation Basma |
| Brieg | Lead DS (parti 12/06/2026) — a initié MMM Agent avec Adam et Hajar | MMM AI Agent (historique) |

---

## Contexte récent important

| Date | Fait |
|---|---|
| 2026-07-22 | Engine MMM Agent **2659** en prod : `CTX_MODULE_VIZ` (aide lecture graphiques bilingue), retry backend, polish chatbot (suggestions thématiques + fix portail z-index flottant). PRs #1685/#1686 mergées. |
| 2026-07-20 | MMM Agent v2 : `BUSINESS_CONTEXT` décomposé en blocs scopés par sous-agent — hallucination `execute_sql` fixée, validée en prod (engine 7899 puis cutover vers 2659). |
| 2026-07-20 | LiveRamp (Julien Guého) confirme : création d'audience Safe Haven = UI only, Activation API réservée aux plateformes tierces. Piste API ConnectedHub fermée ; veille MCP LiveRamp agents en cours. |
| 2026-07-09 | Full Funnel AMC branché sur BigQuery (PR #1653/#1654 mergées sur main). Accès Looker Nicolas Vivies débloqué (partage "unlisted" avec Khadija). |
| 2026-07-07 | Testing MMM Agent avec Dan : isolation inter-client validée (Opel vs Peugeot refusé proprement). Démo Fabien Bourrely (DG Starcom) → intérêt confirmé au-delà de l'équipe. |
| 2026-07-06 | FeedGen V3 LLM reranking : 88,8 % hiérarchique (Gemini Flash) vs 75,9 % baseline. Jeu gold Kérastase construit (`tb_kerastase_eval`, 294 produits). |

---

## Ce qui n'est PAS un projet actif d'Adam

| Projet | Statut vault | Qui porte | Note |
|---|---|---|---|
| Viral Beauty (L'ORÉAL) | `4.Archive/` — archivé | Jules (a repris en relais depuis le 06/07) | Adam était participant, maintenant backup passif. Départ Basma = fin de ce sujet côté Adam. |
| EVBB Scoring (STELLANTIS) | `3.Resources/Onboardings/` — `implication: onboardé` | Dan | Adam onboardé uniquement. La note documente le travail de Dan (pipeline BQML Logistic Regression, bid values VBB). |
| Creative Insights Verisure (VERISURE) | `3.Resources/Onboardings/` — `implication: onboardé` | Hajar + Léonie | Adam onboardé uniquement. Aucune contribution au livrable. |
