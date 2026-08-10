---
type: context
Dernière mise à jour: 2026-08-08
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

**Statut (23/07 — aucune session depuis)** : Engine **6241** en prod depuis le 23/07 (`MMM_Agent_v3_live_ui_scope`). Feature B (scope écran live) livrée : l'agent connaît désormais le KPI, la période, l'onglet et la langue affichés à l'écran, et répond dans ce scope par défaut (surchargeable dimension par dimension, annonce le scope sur sa 1re réponse). Défaut critique corrigé en revue de branche : les bornes de période passent en dates réelles (`WHERE date BETWEEN`) — les colonnes `year`/`week` n'existent pas dans `tb_model_contributions`. Guardrails whitelist durcis (`[`, `]`, `;`, `=`, newlines exclus), security review validée. PRs #1688 (`develop`) → #1689 (`main`) mergées. Rollback disponible sur engine `2659` sans redéploiement. Aucune session depuis le 23/07 (16 jours).

**Prochaine action** : surveiller le monitoring (`no_answer` + `uiScopeRejected` en prod) ; mettre au backlog le chip de contexte chatbot (`Contexte : ROAS · juin 2025 – mai 2026`) ; relancer le conseil DE Stellantis (Zenith Media — Marit, Janina, Virginia silencieux depuis le mail deck) pour l'initiation agent ; suite du backlog : enrichissement `BUSINESS_CONTEXT` avec les DS (Dan/Hajar), `cf-budget-allocator-prod` en tool, knowledge par client.

**Blocker** : cadrage "recommandation" à trancher avec Baptiste — option A (deux niveaux interne/client : reco pleine en interne, faits seulement côté client) vs option B (assistant pur, aucune reco ni interne ni client). Décision structurante : conditionne le tool d'allocation budget (`cf-budget-allocator-prod`). Conseil DE Stellantis ne répond plus depuis le mail deck.

---

### 2. AMC Analytics — MINE × L'ORÉAL
**implication : lead** | discipline : dev | client : LOREAL

Module ConnectedHub centralisant les analyses Amazon Marketing Cloud (Full Funnel depuis BigQuery) et les dashboards Looker (en iframe). Deux types d'utilisateurs : équipes data (Full Funnel) et équipes conseil/traders (dashboards).

**Statut (07/08 — actif)** : **Refonte majeure du Full Funnel** en **workspace pivot** suite au retour de Jules (« on ne peut pas faire ce qu'on veut »). Deck de 5 slides figées supprimé. Nouveau moteur pur en trois étages (normalisation → dérivation → agrégation), règles de méthode encodées (ratios après agrégation, clics jamais dénominateur, avertissement sur somme de reach), mapping des leviers éditable dans l'app (Firestore, niveau customer). Trois double-comptes corrigés en session du 07/08 : `analysis_level` (7 analyses empilées dans une table — `Path to conversion` et `Media Mix` portent les mêmes chiffres), `granularity` (2 mailles de la même population), slicer mono-sélection qui affichait sa valeur sans l'appliquer. Vue dans l'URL via `nuqs` (partageable, bouton retour = annuler). « Place des leviers » lit désormais les lignes `Place of channel` du trading (données source, pas des chiffres dérivés). PRs #1719 (pivot workspace), #1720 (noms de colonnes bruts), #1721 (slicers, correctifs double-comptes) mergées sur `develop`. PR **#1722** ouverte (cohérence slicers entre niveaux). PR sur `main` jamais faite pour ce chantier (`develop` accuse ~59 commits d'avance). Aucune vérification visuelle faite — tout validé par tests, typage et build uniquement.

**Prochaine action** : merger PR #1722 ; ouvrir la PR `develop → main` ; **vérifier le module à l'écran** (personne ne l'a encore ouvert) ; prévenir Jules que les totaux affichés sont plus bas (les correctifs arrêtent d'additionner ce qui ne devait pas l'être) ; configurer Firestore + comptes utilisateurs Publicis via Settings ; cadrer le calendrier de migration dashboards Looker → React natif avec Khadija.

**Blocker** : PR #1722 ouverte, aucune vérification visuelle faite. Migration Looker → React dépend du calendrier Khadija. Ingestion BQ et mise à jour du registre sont manuelles (aucune Cloud Function dédiée).

---

### 3. Brand Store v2 — L'ORÉAL
**implication : participant** | discipline : data-analyst | client : LOREAL

Étude L'Oréal Brand Store. Adam est participant (suivi, pas de livrable porté).

**Statut** : aucune session dans le vault, aucun TODO actif côté Adam. Sujet porté par d'autres. Pas d'action prévue de la part d'Adam.

---

### 4. Creative Insights — MINE
**implication : lead** (à confirmer) | discipline : dev + data-science | client : **MINE** — module plateforme transverse

Module ConnectedHub d'automatisation des Creative Insights. Objectif : rendre self-service la chaîne aujourd'hui exécutée à la main par les DS, de la récupération des assets créa (Meta, TikTok, Snapchat) jusqu'à la restitution au Data Analyst.

**Statut (10/08 — cadrage)** : projet ouvert le 10/08 suite à un point de briefing où Hajar et Abhishek ont introduit Adam aux besoins et aux idées de dev. Chaîne actuelle manuelle en 6 étapes (Cloud Functions → Data Platform → features via prompt Gemini → modèle de corrélation → export CSV → analyse par le DA). Chaîne cible cadrée en 7 étapes verrouillées séquentiellement, avec page de suivi. **Les cloud functions existent déjà et ont servi** (analyse Leapmotor d'Abhishek), avec des assets dans des buckets GCS. Le module orchestre de l'existant, il ne part pas de zéro. Aucune ligne de code écrite.

**Prochaine action** : obtenir de Hajar les pointeurs GCP (cloud functions, buckets) et explorer l'existant ; clarifier la répartition du dev avec Hajar et Jules ; trancher où le module s'arrête (export CSV branché sur le dashboard de Thomas, ou restitution native).

**Blocker** : répartition du dev non tranchée — le CR indique qu'Abhishek démarre le développement initial alors que le point servait à briefer Adam sur la couche ConnectedHub. Risque de deux chantiers parallèles. Aucun sponsor ni calendrier : Jules est informé, pas sollicité comme arbitre, et la capacité d'Adam (déjà lead sur deux modules) n'a pas été posée.

---

## Blockers transverses

| Blocker | Projet(s) | Qui débloque |
|---|---|---|
| Cadrage "recommandation" agent — option A (deux niveaux interne/client) vs option B (assistant pur, aucune reco) | MMM AI Agent | Baptiste |
| Conseil DE Stellantis silencieux — Zenith Media ne répond plus depuis le mail deck | MMM AI Agent | Katia + Zenith Media DE |
| PR #1722 + PR develop→main AMC jamais faite (~59 commits d'avance) | AMC Analytics | Adam |
| Aucune vérification visuelle du workspace pivot (tests verts, mais module non ouvert) | AMC Analytics | Adam |
| Ingestion BQ AMC manuelle (registre non automatisé) | AMC Analytics | Khadija (ingestion) |
| Répartition du dev non tranchée (couche ConnectedHub vs backend DS) | Creative Insights | Hajar + Jules |
| Pointeurs GCP de l'existant (cloud functions, buckets GCS) non transmis | Creative Insights | Hajar |

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
| Manu | AI Office / manager direct DA — Copilot audiences | (transverse) |
| Inès | Data Strat Stellantis — contact ouverture conseil MMM | MMM AI Agent |
| Katia | Data Strat Stellantis — contact ouverture conseil MMM | MMM AI Agent |
| Pierre | Data Strat L'Oréal — référent AMC, Brand Store | AMC Analytics |
| Brieg | Lead DS (parti 12/06/2026) — a initié MMM Agent avec Adam et Hajar | MMM AI Agent (historique) |

---

## Contexte récent important

| Date | Fait |
|---|---|
| 2026-08-10 | **Ouverture du projet Creative Insights** (module ConnectedHub, client MINE). Briefing dev par Hajar et Abhishek. Le POC Verisure et la piste Leapmotor sont absorbés : note Verisure supprimée, idée Leapmotor passée en `repris`. |
| 2026-08-07 | Session AMC Analytics : **trois double-comptes corrigés** (`analysis_level`, `granularity`, slicer sans effet). Vue dans l'URL via `nuqs`. « Place des leviers » lit désormais `Place of channel` du trading. PRs #1720/#1721 mergées, #1722 ouverte. |
| 2026-08-06 | **Refonte Full Funnel AMC en workspace pivot** suite retour de Jules (« on ne peut pas faire ce qu'on veut »). Deck de slides supprimé. Moteur pur 3 étages avec règles de méthode encodées. 0 divergence vs classeur Jules sur 200 lignes. PR #1719 mergée sur develop. |
| 2026-07-23 | Feature B (scope écran live) livrée en prod : engine **6241**. L'agent connaît KPI, période, onglet et langue de l'écran. Défaut critique corrigé : bornes de période en dates réelles (colonnes `year`/`week` absentes de BQ). PRs #1688/#1689 mergées. |
| 2026-07-23 | Basma quitte Publicis — knowledge audiences LiveRamp non rempli à son départ (schéma scaffoldé, `[À COMPLÉTER]` partout). Fenêtre de capture définitivement close. |
| 2026-07-22 | Engine MMM Agent **2659** en prod : `CTX_MODULE_VIZ` (aide lecture graphiques bilingue EN/FR), retry backend `no_answer`, polish chatbot. PRs #1685/#1686 mergées. |

---

## Ce qui n'est PAS un projet actif d'Adam

| Projet | Statut vault | Qui porte | Note |
|---|---|---|---|
| Viral Beauty (L'ORÉAL) | `4.Archive/` — archivé | Jules (a repris en relais depuis le 06/07) | Adam était participant, maintenant backup passif. Départ Basma = fin de ce sujet côté Adam. |
| Passation Basma (PUBLICIS) | `4.Archive/` — archivé le 2026-08-03 | — | Fenêtre de capture close au départ de Basma (23/07), knowledge resté `[À COMPLÉTER]`. Volet Copilot / audiences **abandonné** côté Adam (cohérent avec le positionnement DS/IA : pas de rôle de transition). |
| FeedGen Catégorisation (MINE) | `4.Archive/` — archivé le 2026-08-03 | Dan (solution LLM en intégration) | Exploration RAG d'Adam arrivée après coup. Acquis techniques valides (BQ natif, métrique hiérarchique, V3 rerank 88,8 %). Réactivable si Manu fournit le jeu d'éval ou si Dan ouvre le RAG en amont. |
| EVBB Scoring (STELLANTIS) | `3.Resources/Onboardings/` — `implication: onboardé` | Dan | Adam onboardé uniquement. La note documente le travail de Dan (pipeline BQML Logistic Regression, bid values VBB). |
| Creative Insights Verisure (VERISURE) | Note supprimée le 2026-08-10 | Hajar + Léonie | POC client jamais travaillé par Adam. Le sujet est devenu le module plateforme [[1.Projects/MINE_CreativeInsights/Creative Insights]], qui lui **est** un projet actif d'Adam (lead). |
