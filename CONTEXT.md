---
type: context
Dernière mise à jour: 2026-08-19
---

# CONTEXT — Vault d'Adam

> Fichier de synthèse régénéré automatiquement. Source de vérité pour reprendre le contexte d'Adam sans rouvrir chaque note.

---

## Qui est Adam

Adam Jouini, apprenti Data & Dev chez Publicis Media (alternance), rattaché à la plateforme **ConnectedHub** (produit interne Mine). Il travaille au croisement du développement (React/Node.js/backend), de la data science (Vertex AI, embeddings, agents) et de l'analyse de données (AMC, MMM, audiences LiveRamp). Orientation data science exprimée lors du point évolution de juin 2026. Jules a dit explicitement vouloir proposer un poste à la sortie de l'alternance, en citant l'agent MMM et AMC Analytics comme réalisations commercialement visibles. **Dernier jour de travail : 21/08/2026.**

---

## Projets actifs

### 1. MMM AI Agent — MINE
**implication : lead** | discipline : dev + data-science | client : **MINE** — produit plateforme, **pas un projet L'Oréal**

Agent IA conversationnel dans le module MMM de ConnectedHub (Vertex AI Agent Engine). Feature de la plateforme vendue en add-on à tout client ayant un MMM signé — cible de déploiement : Stellantis (Opel DE / Peugeot DE), puis Longchamp.

**Statut (19/08 — passation active, dernier jour Adam le 21/08)** : Engine `6241` en prod (`MMM_Agent_v3_live_ui_scope`). Feature B (scope écran live) livrée et en prod depuis le 23/07. **Passation à Dan en cours (côté agent — pipeline ADK, Vertex AI)** : point tenu le 18/08. Actions complétées le 19/08 : code poussé dans le repo d'équipe `Publicis-Media-France-FR5140/MMM_AI_Agent` (29 commits, `main` remis à niveau sur `feat/live-ui-scope`) ; commentaires Python retirés (95 lignes, AST identique, 35 tests verts) ; alerte `no_answer` redirigée vers Dan en parallèle (channel `8248908768259935024` créé sur `danphan2@publicisgroupe.net` — ancien channel `5134431245708235828` à retirer avant le 04/09 après confirmation de réception) ; doc Confluence rédigée (reste à coller). **Côté intégration ConnectedHub** (backend `agent.ts`, SSE, feature flag, sessions) : second point à caler avec Eddie — pas encore fait.

**Prochaine action** : Confirmer la réception de l'alerte `no_answer` avec Dan avant le 21/08. Retirer l'ancien channel d'alerte avant le 04/09. Caler le point de passation intégration avec Eddie avant le 21/08.

**Blocker** : Passation intégration ConnectedHub (Eddie) non encore calée. Cadrage "recommandation" (option A/B) toujours non tranché — remis à Baptiste post-passation. Conseil DE Stellantis (Zenith Media) silencieux depuis le mail deck — remis à Katia/Inès.

---

### 2. AMC Analytics — MINE × L'ORÉAL
**implication : lead** | discipline : dev | client : LOREAL

Module ConnectedHub centralisant les analyses Amazon Marketing Cloud (workspace pivot depuis BigQuery) et les dashboards Looker (en iframe).

**Statut (18/08 — actif, PRs en attente de merge)** : Sessions des 17 et 18/08 très denses. **PRs #1749 (→ main) et #1750 (→ develop) ouvertes le 18/08**, pas encore mergées — contenu strictement AMC, 40 fichiers chacune. Travaux du 17-18/08 : tableau réécrit sur TanStack (en-tête à deux étages, redimensionnement, tri, lignes TOTAL / TOTAL FILTRÉ collées en tête, virtualisation au-delà de 150 lignes) ; filtres unifiés (seuils et coupures dans le même panneau) ; scorecards propres à chaque cas d'usage (répondent à la question posée, pas aux mesures de la vue — découverte : « Beginner » et « Finisher » donnaient toujours le même chiffre, remplacés par part de parcours solo et levier ouvrant/fermant le plus) ; entonnoir Vue d'ensemble à 4 niveaux (largeurs sans encodage quantitatif, même raisonnement que le Venn) ; Venn de synergie à géométrie fixe (proportionnel rejeté) ; noms KPI lisibles à l'écran, colonne source au survol. Découverte structurante via Khadija (17/08) : **un `path` vide = suppression de confidentialité AMC** (cohorte trop petite — AMC masque la dimension mais renvoie les métriques) ; la somme des parcours n'égalera jamais le total de l'étude — encodé dans le moteur (`Cut { count, measures }`). Bug latent corrigé : `readSum` lisait uniquement `impressions_cost`, `cost` valait 0 en silence sur toute extraction nommant sa colonne `spend`. **433 tests client au vert** (depuis le 17/08).

**Prochaine action** : Merger #1750 → develop en premier, vérifier en staging, puis #1749 → main. Signaler à Eddie : prop `modal` manquante sur `Combobox` partagé (bloquant pour les filtres en Dialog), `Button variant="secondary"` typé sans style, `onRowClick.logic` mal typé. Validation Jules non encore faite. Vérifier si « Av. CVR » de Jules = taux du total ou moyenne des taux (écart visible sur slide). Traiter les feedbacks mail restants (`time of`).

**Blocker** : PRs #1749/#1750 non mergées (validation Jules manquante). Ingestion BQ et registre toujours manuels. Migration Looker → React dépend du calendrier Khadija.

---

### 3. Brand Store v2 — L'ORÉAL
**implication : participant** | discipline : data-analyst | client : LOREAL

Étude L'Oréal Brand Store. Adam est participant (suivi, pas de livrable porté).

**Statut** : aucune session dans le vault, aucun TODO actif côté Adam. Sujet porté par d'autres. Pas d'action prévue de la part d'Adam.

---

### 4. Creative Insights — MINE
**implication : lead** (à confirmer avec Hajar et Jules) | discipline : dev + data-science | client : **MINE** — module plateforme transverse

Module ConnectedHub d'automatisation des Creative Insights. Objectif : self-service de la chaîne DS manuelle (récupération assets créa Meta/TikTok/Snapchat → extraction features Gemini → restitution DA).

**Statut (13/08 — stalled depuis 6 jours)** : Lots 1 et 2 mergés (#1738-#1741) : rôles éditeur/administrateur, circuit de validation mail, icône module, `viewer_role` calculé côté serveur. Lot 3 poussé (`feat/creative-insights-project-id`, pas encore de PR) : colonne `project_id` créée sur `meta.tb_api_asset_urls`, requêtes avec repli SQL — comportement actuel préservé. Plan modifications DS rédigé, **pas encore envoyé à Hajar et Abhishek**. Ticket IT toujours sans réponse, staging non fonctionnel. Découverte critique (13/08) : sans `campaign_id`, Meta throttle sur les 1545 campagnes du compte — `campaign_id` obligatoire. Adam part le 21/08 : continuité du projet à organiser.

**Prochaine action** : Envoyer le [[Plan modifications DS]] à Hajar et Abhishek (bloquant pour tout le reste côté DS). Ouvrir une PR pour `feat/creative-insights-project-id`. Suivre le ticket IT. Localiser les prompts Gemini et notebooks d'exécution (étape 4).

**Blocker** : Plan DS non envoyé (Hajar et Abhishek ne connaissent pas les 5 évolutions Cloud Function attendues). Ticket IT en attente (service account sans droits, staging inaccessible). Prompts Gemini et notebooks non localisés (étape 4 bloquée). TikTok et Snapchat sans `ad_id` — jointure performance impossible. Fragilité observateur async (correctif durable = Cloud Tasks). Répartition dev non tranchée (couche ConnectedHub vs backend DS).

---

## Blockers transverses

| Blocker | Projet(s) | Qui débloque |
|---|---|---|
| Passation intégration ConnectedHub non calée — backend `agent.ts`, SSE, feature flag, sessions | MMM AI Agent | Eddie (point à caler avant le 21/08) |
| Ancien channel alerte `no_answer` (`5134431245708235828`) à retirer avant le 04/09 | MMM AI Agent | Adam (après confirmation réception par Dan) |
| Cadrage "recommandation" agent — option A (deux niveaux interne/client) vs option B (assistant pur, aucune reco) | MMM AI Agent | Baptiste (post-passation Adam) |
| Conseil DE Stellantis silencieux — Zenith Media ne répond plus depuis le mail deck | MMM AI Agent | Katia + Zenith Media DE |
| PRs #1749/#1750 non mergées — validation Jules en attente | AMC Analytics | Jules |
| Prop `modal` manquante sur `Combobox` partagé — filtres incliquables dans Dialog | AMC Analytics | Eddie |
| Ingestion BQ AMC manuelle (registre non automatisé) | AMC Analytics | Khadija (ingestion) |
| Plan modifications DS non envoyé — Hajar et Abhishek ne connaissent pas les 5 évolutions Cloud Function attendues | Creative Insights | Adam (action immédiate avant le 21/08) |
| Ticket IT en attente — droits service account `interface@` sur `zen-creativeinsights-dev-mg`, staging non fonctionnel | Creative Insights | IT / James |
| TikTok et Snapchat ne remontent pas `ad_id` — jointure performance impossible | Creative Insights | Hajar + Abhishek |
| Prompts Gemini et notebooks d'exécution non localisés — étape 4 (extraction features) bloquée | Creative Insights | Hajar |
| Répartition du dev non tranchée (couche ConnectedHub vs backend DS) | Creative Insights | Hajar + Jules |

---

## Personnes clés transverses

| Personne | Rôle | Projets |
|---|---|---|
| Jules | Manager direct — arbitrages scope, évolution, recrutement | Tous |
| Baptiste | Head of Data (futur pôle études & mesure) — go agent, cadrage reco | MMM AI Agent |
| Eddie | Lead dev ConnectedHub — archi frontend/backend, décisions infra | MMM AI Agent, AMC Analytics |
| Khadija | Data Analyst — dashboards Looker AMC, ingestion BQ | AMC Analytics |
| Dan | Data Scientist — reprend le MMM Agent (passation en cours depuis le 18/08) | MMM AI Agent |
| Hajar | Data Scientist — testing MMM, AMC modeled audiences, porte le sujet Creative Insights | MMM AI Agent, AMC Analytics, Creative Insights |
| Abhishek | Data Scientist PGD (Inde) — cloud functions d'extraction d'assets, notebooks SIMBA | Creative Insights |
| Thomas | Data Analyst Social + L'Oréal — consommateur final des Creative Insights, auteur du dashboard CSV | Creative Insights |
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
| 2026-08-19 | **Passation MMM Agent — actions du jour.** Code poussé dans `Publicis-Media-France-FR5140/MMM_AI_Agent` (29 commits, main remis à niveau). Commentaires Python retirés (95 lignes, AST identique, 35 tests verts). Alerte `no_answer` redirigée vers Dan (channel additionnel créé — ancien à retirer avant le 04/09 après confirmation). Doc Confluence rédigée (reste à coller). |
| 2026-08-18 | **Point passation Dan (MMM Agent) + session AMC longue.** Passation : frontière agent / intégration posée, backlog transmis à Dan (`BUSINESS_CONTEXT`, `cf-budget-allocator-prod`, knowledge client), intégration ConnectedHub remise à Eddie (second point à caler). AMC : tableau TanStack, filtres unifiés, scorecards par cas d'usage, virtualisation, Venn synergie (géométrie fixe), noms KPI lisibles → PRs #1749 (→ main) et #1750 (→ develop) ouvertes, 40 fichiers chacune. |
| 2026-08-17 | **Session AMC — retours PowerPoint Jules.** Entonnoir Vue d'ensemble (4 niveaux, largeurs sans encodage quantitatif). Double axe `barsWithLine` opt-in par preset (borné — révision partielle de la règle du 10/08). Colonnes façon ConnectedFeed (`MeasureManager`, glisser-déposer, barre de contenu au-dessus du tableau). Découverte structurante via Khadija : un `path` vide = seuil de confidentialité AMC (masque dimension, conserve métriques) — la somme des parcours n'égale jamais le total de l'étude, encodé dans le moteur. Bug latent corrigé : `readSum` ne lisait pas `spend` (alias de `cost`). |
| 2026-08-13 | **Creative Insights — deuxième session de dev.** Lots 1 et 2 mergés : rôles éditeur/administrateur, circuit de validation, mail admins, icône module (#1738-#1741). Bug de fraîcheur du rôle corrigé (`viewer_role` calculé côté serveur). Découverte critique : sans `campaign_id`, Meta throttle le compte entier (1545 campagnes, code 17/80004) — `campaign_id` obligatoire. Lot 3 poussé : colonne `project_id` + repli SQL, comportement actuel préservé. |
| 2026-08-11 | **Session AMC Analytics** : retours Jules (10/11 actions), analyses enregistrées (Firestore, `?a=<id>` dans l'URL). PRs #1726/#1728 et #1733/#1736 mergées — module identique sur `develop` et `main`, en production. |

---

## Ce qui n'est PAS un projet actif d'Adam

| Projet | Statut vault | Qui porte | Note |
|---|---|---|---|
| Viral Beauty (L'ORÉAL) | `4.Archive/` — archivé | Jules (a repris en relais depuis le 06/07) | Adam était participant, maintenant backup passif. Départ Basma = fin de ce sujet côté Adam. |
| Passation Basma (PUBLICIS) | `4.Archive/` — archivé le 2026-08-03 | — | Fenêtre de capture close au départ de Basma (23/07), knowledge resté `[À COMPLÉTER]`. Volet Copilot / audiences **abandonné** côté Adam (cohérent avec le positionnement DS/IA : pas de rôle de transition). |
| FeedGen Catégorisation (MINE) | `4.Archive/` — archivé le 2026-08-03 | Dan (solution LLM en intégration) | Exploration RAG d'Adam arrivée après coup. Acquis techniques valides (BQ natif, métrique hiérarchique, V3 rerank 88,8 %). Réactivable si Manu fournit le jeu d'éval ou si Dan ouvre le RAG en amont. |
| EVBB Scoring (STELLANTIS) | `3.Resources/Onboardings/` — `implication: onboardé` | Dan | Adam onboardé uniquement. La note documente le travail de Dan (pipeline BQML Logistic Regression, bid values VBB). |
| Creative Insights Verisure (VERISURE) | Note supprimée le 2026-08-10 | Hajar + Léonie | POC client jamais travaillé par Adam. Le sujet est devenu le module plateforme [[1.Projects/MINE_CreativeInsights/Creative Insights]], qui lui **est** un projet actif d'Adam (lead). |
