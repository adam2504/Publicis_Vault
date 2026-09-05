---
type: context
Dernière mise à jour: 2026-09-05
---

# CONTEXT — Vault d'Adam

> Fichier de synthèse régénéré automatiquement. Source de vérité pour reprendre le contexte d'Adam sans rouvrir chaque note.

---

## Qui est Adam

Adam Jouini, apprenti Data & Dev chez Publicis Media (alternance), rattaché à la plateforme **ConnectedHub** (produit interne Mine). Il travaille au croisement du développement (React/Node.js/backend), de la data science (Vertex AI, embeddings, agents) et de l'analyse de données (AMC, MMM, audiences LiveRamp). Orientation data science exprimée lors du point évolution de juin 2026. Jules a dit explicitement vouloir proposer un poste à la sortie de l'alternance, en citant l'agent MMM et AMC Analytics comme réalisations commercialement visibles. **Dernier jour de travail : 21/08/2026. Adam n'est plus en poste depuis le 22/08/2026.**

---

## Projets actifs

### 1. MMM AI Agent — MINE
**implication : lead** | discipline : dev + data-science | client : **MINE** — produit plateforme, **pas un projet L'Oréal**

Agent IA conversationnel dans le module MMM de ConnectedHub (Vertex AI Agent Engine). Feature de la plateforme vendue en add-on à tout client ayant un MMM signé — cible de déploiement : Stellantis (Opel DE / Peugeot DE), puis Longchamp.

**Statut (21/08 — passation terminée, Adam parti)** : Engine `6241` en prod (`MMM_Agent_v3_live_ui_scope`). Feature B (scope écran live) en prod depuis le 23/07. Passation côté agent terminée : code poussé dans `Publicis-Media-France-FR5140/MMM_AI_Agent` (29 commits, `main` remis à niveau), commentaires Python retirés, alerte `no_answer` redirigée vers Dan (channel `8248908768259935024` créé — ancien channel `5134431245708235828` à retirer **avant le 04/09** après confirmation de réception — date dans 2 jours au 02/09), doc Confluence rédigée (reste à coller). **Passation intégration ConnectedHub (backend `agent.ts`, SSE, feature flag, sessions) non finalisée** avant le départ d'Adam — point avec Eddie non tenu avant le 21/08.

**Prochaine action (pour l'équipe)** : Vérifier si l'ancien channel d'alerte (`5134431245708235828`) a bien été retiré — délai 04/09 passé, statut non confirmé dans le vault. Coller la doc Confluence (Dan). Caler le point passation intégration avec Eddie (frontière agent / ConnectedHub, backend `agent.ts`, SSE, feature flag, sessions).

**Blocker** : Passation intégration ConnectedHub (Eddie) non calée avant le départ d'Adam. Cadrage "recommandation" (option A/B) remis à Baptiste post-passation. Conseil DE Stellantis (Zenith Media) silencieux depuis le mail deck — à relancer par Katia/Inès.

---

### 2. AMC Analytics — MINE × L'ORÉAL
**implication : lead** | discipline : dev | client : LOREAL

Module ConnectedHub centralisant les analyses Amazon Marketing Cloud (workspace pivot depuis BigQuery) et les dashboards Looker (en iframe).

**Statut (18/08 — PRs en attente, Adam parti le 21/08)** : **PRs #1749 (→ main) et #1750 (→ develop) ouvertes le 18/08**, pas encore mergées — contenu strictement AMC, 40 fichiers chacune. Travaux des 17-18/08 : tableau réécrit sur TanStack (en-tête à deux étages, redimensionnement, tri, lignes TOTAL / TOTAL FILTRÉ collées en tête, virtualisation au-delà de 150 lignes) ; filtres unifiés (seuils et coupures dans le même panneau) ; scorecards propres à chaque cas d'usage ; entonnoir Vue d'ensemble à 4 niveaux (largeurs sans encodage quantitatif) ; Venn de synergie à géométrie fixe ; noms KPI lisibles à l'écran, colonne source au survol. 433 tests client au vert.

**Prochaine action (pour l'équipe)** : Merger #1750 → develop en premier, vérifier en staging, puis #1749 → main. Explorer les feedbacks mail de Jules (ouverts dans le TODO au départ d'Adam). Signaler à Eddie : prop `modal` manquante sur `Combobox` partagé (bloquant pour les filtres en Dialog), `Button variant="secondary"` typé sans style, `onRowClick.logic` mal typé. Validation Jules non encore faite.

**Blocker** : PRs #1749/#1750 non mergées (validation Jules manquante). Ingestion BQ et registre toujours manuels. Migration Looker → React dépend du calendrier Khadija.

---

### 3. Brand Store v2 — L'ORÉAL
**implication : participant** | discipline : data-analyst | client : LOREAL

Étude L'Oréal Brand Store. Adam est participant (suivi, pas de livrable porté).

**Statut** : aucune session dans le vault, aucun TODO actif côté Adam. Sujet porté par d'autres. Pas d'action prévue de la part d'Adam. Stalled côté vault — Adam parti depuis le 21/08.

---

### 4. Creative Insights — MINE
**implication : lead** (à confirmer avec Hajar et Jules) | discipline : dev + data-science | client : **MINE** — module plateforme transverse

Module ConnectedHub d'automatisation des Creative Insights. Objectif : self-service de la chaîne DS manuelle (récupération assets créa Meta/TikTok/Snapchat → extraction features Gemini → restitution DA).

**Statut (21/08 — point Hajar/Abhishek, dernier jour Adam)** : **Étapes 1 à 4 livrées et démontrées le 21/08.** Étapes 5 à 7 non commencées. Démo complète avec Hajar et Abhishek le 21/08 : bonne réception, architecture des écrans et modèle de rôles validés sans objection. Annonce de la copie des Cloud Functions passée sans friction. **Architecture étape 5 débloquée** : sur proposition d'Abhishek, le projet gagnera un identifiant de table de performance (BQ ou Adverity), ce qui remplace l'attente d'une Cloud Function de Hajar et permet de prototyper. Hajar et Abhishek travaillent sur l'approche de sélection automatique et de rattachement à la performance. **14 commits locaux non poussés** sur `feat/creative-insights-scoping` (66 fichiers, 4 359 insertions), aucune PR ouverte. Droits DEV en place depuis le 19/08. **3 droits PROD non encore demandés** : `roles/aiplatform.user` sur `zen-creativeinsights-dev-mg`, `Storage Object Viewer` sur `creative_assets_tiktok` et `creative_assets_snapchat`, pour `interface@pmed-portal-prd-mg.iam.gserviceaccount.com`. Nouvelles demandes issues de la réunion du 21/08 : filtrer la bibliothèque de créas par valeur de feature (besoin Hajar) ; import fichier de prompt en plus de l'éditeur JSON.

**Prochaine action** : Pousser les 14 commits sur une branche basée sur `origin/develop` et ouvrir une PR. Réécrire le Plan modifications DS avant tout envoi (il présente encore comme demandes des choses livrées entre-temps). Ouvrir un ticket IT pour les 3 droits PROD manquants. Récupérer la liste de features de Léonie (jamais formalisée).

**Blocker** : 14 commits non poussés (risque de divergence / perte). Plan modifications DS à réécrire avant envoi — Hajar et Abhishek ne savent pas encore ce qui a été livré. Règle d'attribution performance (`ad_id` → `asset_id`) non tranchée — bloque l'étape 5. TikTok et Snapchat sans clé de jointure performance (`ad_id` absent) — évolution du scrapper nécessaire, pas un simple correctif. 3 droits PROD non encore demandés (ticket à ouvrir).

---

## Blockers transverses

| Blocker | Projet(s) | Qui débloque |
|---|---|---|
| ~~⚠️ Ancien channel alerte `no_answer` (`5134431245708235828`) à retirer avant le 04/09~~ — **délai passé (05/09), statut non confirmé dans le vault** | MMM AI Agent | Dan (à vérifier) |
| Passation intégration ConnectedHub non finalisée — backend `agent.ts`, SSE, feature flag, sessions | MMM AI Agent | Eddie (à contacter post-départ Adam) |
| Cadrage "recommandation" agent — option A (deux niveaux interne/client) vs option B (assistant pur, aucune reco) | MMM AI Agent | Baptiste (post-passation Adam) |
| Conseil DE Stellantis silencieux — Zenith Media ne répond plus depuis le mail deck | MMM AI Agent | Katia + Zenith Media DE |
| PRs #1749/#1750 non mergées — validation Jules en attente | AMC Analytics | Jules |
| Feedbacks mail de Jules non encore explorés | AMC Analytics | (à prendre en charge post-Adam) |
| Prop `modal` manquante sur `Combobox` partagé — filtres incliquables dans Dialog | AMC Analytics | Eddie |
| Ingestion BQ AMC manuelle (registre non automatisé) | AMC Analytics | Khadija (ingestion) |
| 14 commits non poussés sur `feat/creative-insights-scoping`, aucune PR ouverte | Creative Insights | (action immédiate — risque de divergence) |
| Plan modifications DS à réécrire avant envoi — présente comme demandes des choses livrées entre-temps | Creative Insights | (à réécrire avant envoi à Hajar/Abhishek) |
| 3 droits PROD non encore demandés : `roles/aiplatform.user` + `Storage Object Viewer` TikTok/Snapchat | Creative Insights | IT / James (nouveau ticket) |
| Règle d'attribution performance (`ad_id` → `asset_id`, un ad porte plusieurs assets) non tranchée | Creative Insights | Hajar + Abhishek |
| TikTok et Snapchat sans clé de jointure performance — évolution du scrapper nécessaire, pas un correctif | Creative Insights | Hajar + Abhishek |

---

## Personnes clés transverses

| Personne | Rôle | Projets |
|---|---|---|
| Jules | Manager direct — arbitrages scope, évolution, recrutement | Tous |
| Baptiste | Head of Data (futur pôle études & mesure) — go agent, cadrage reco | MMM AI Agent |
| Eddie | Lead dev ConnectedHub — archi frontend/backend, décisions infra | MMM AI Agent, AMC Analytics |
| Khadija | Data Analyst — dashboards Looker AMC, ingestion BQ | AMC Analytics |
| Dan | Data Scientist — reprend le MMM Agent (passation terminée au 21/08) | MMM AI Agent |
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
| 2026-08-21 | **Dernier jour d'Adam + Point Creative Insights Hajar/Abhishek.** Démo complète du module déroulée (accueil, création de projet, collecte, bibliothèque, prompts). Architecture des écrans et modèle de rôles validés sans objection. Hajar recentre le sujet : la sélection manuelle ne passe pas à l'échelle, elle veut une sélection pilotée par la performance. Abhishek propose que le projet désigne lui-même une table de performance (BQ ou Adverity) — débloque le prototype sans attendre leur Cloud Function. Annonce GCP (copie des CF, dataset `connectedhub`, correctifs) accueillie sans friction. 3 droits PROD non demandés en séance. Hajar et Abhishek travaillent sur l'approche. |
| 2026-08-20 | **Creative Insights — grosse session (17-20/08), étape 4 livrée.** Bibliothèque de prompts par industrie, extraction Vertex AI gemini-2.5-flash avec schéma typé dérivé du dictionnaire de 54 features, worker à concurrence 16 et reprenable. Cloud Function dupliquée en `cf-gather-meta-assets-ci` (4 correctifs). 14 commits locaux non poussés, aucune PR ouverte. Droits IT enfin posés sur le bon bénéficiaire (19/08, après deux erreurs). Job nocturne découvert : retire chaque nuit tout outil absent du registre prod. |
| 2026-08-19 | **Passation MMM Agent — actions du jour.** Code poussé dans `Publicis-Media-France-FR5140/MMM_AI_Agent` (29 commits, main remis à niveau). Commentaires Python retirés (95 lignes, AST identique, 35 tests verts). Alerte `no_answer` redirigée vers Dan (channel additionnel créé — ancien à retirer **avant le 04/09** après confirmation). Doc Confluence rédigée (reste à coller). |
| 2026-08-18 | **Point passation Dan (MMM Agent) + session AMC longue.** Passation : frontière agent / intégration posée, backlog transmis à Dan, intégration ConnectedHub remise à Eddie (point non tenu avant le 21/08). AMC : tableau TanStack, filtres unifiés, scorecards par cas d'usage, virtualisation, Venn synergie (géométrie fixe) → PRs #1749 (→ main) et #1750 (→ develop) ouvertes, 40 fichiers chacune. |
| 2026-08-17 | **Session AMC — retours PowerPoint Jules.** Entonnoir Vue d'ensemble (4 niveaux, largeurs sans encodage quantitatif). Double axe `barsWithLine` opt-in par preset. Colonnes façon ConnectedFeed (`MeasureManager`, glisser-déposer). Découverte structurante via Khadija : un `path` vide = seuil de confidentialité AMC — somme des parcours ≠ total de l'étude, encodé dans le moteur. Bug latent corrigé : `readSum` ne lisait pas `spend` (alias de `cost`). |

---

## Ce qui n'est PAS un projet actif d'Adam

| Projet | Statut vault | Qui porte | Note |
|---|---|---|---|
| Viral Beauty (L'ORÉAL) | `4.Archive/` — archivé | Jules (a repris en relais depuis le 06/07) | Adam était participant, maintenant backup passif. Départ Basma = fin de ce sujet côté Adam. |
| Passation Basma (PUBLICIS) | `4.Archive/` — archivé le 2026-08-03 | — | Fenêtre de capture close au départ de Basma (23/07), knowledge resté `[À COMPLÉTER]`. Volet Copilot / audiences **abandonné** côté Adam (cohérent avec le positionnement DS/IA : pas de rôle de transition). |
| FeedGen Catégorisation (MINE) | `4.Archive/` — archivé le 2026-08-03 | Dan (solution LLM en intégration) | Exploration RAG d'Adam arrivée après coup. Acquis techniques valides (BQ natif, métrique hiérarchique, V3 rerank 88,8 %). Réactivable si Manu fournit le jeu d'éval ou si Dan ouvre le RAG en amont. |
| EVBB Scoring (STELLANTIS) | `3.Resources/Onboardings/` — `implication: onboardé` | Dan | Adam onboardé uniquement. La note documente le travail de Dan (pipeline BQML Logistic Regression, bid values VBB). |
| Creative Insights Verisure (VERISURE) | Note supprimée le 2026-08-10 | Hajar + Léonie | POC client jamais travaillé par Adam. Le sujet est devenu le module plateforme Creative Insights, qui lui **est** un projet actif d'Adam (lead). |
