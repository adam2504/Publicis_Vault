---
type: project
statut: archivé
discipline: data-science
implication: lead
client: MINE
---
Plateforme : [[Mine Platform]]

## Statut — archivé le 2026-08-03

Projet parqué. Dernière session le **06/07** (28 jours). L'exploration RAG est arrivée après coup : la solution LLM de Dan était déjà en intégration côté Feed Manager, ce qui laisse peu de place au RAG en amont. Le déblocage restant (vrai jeu d'éval multi-flux, GPC et titres bruts) dépend de Manu et n'a pas avancé.

Les acquis techniques restent valides et réutilisables : architecture BQ natif (`VECTOR_SEARCH`) actée contre Vertex AI Vector Search pour raison de coût idle, métrique hiérarchique plutôt qu'exact-match, et V3 retrieve→rerank benchmarké (LLM Gemini Flash : hiérarchique 75,9 % → **88,8 %**). Repo `feedgen-categorisation-rag` propre au commit `25073d1`.

Réactivation possible si Manu fournit le jeu d'éval ou si Dan ouvre le sujet du RAG en amont. Tâches retirées du [[TODO]].

---

## Objectif du module

Catégorisation automatique des produits d'un flux e-commerce dans la taxonomie Google Product Categories (GPC), en se basant sur les descriptions générées par FeedGen (module amont). Fait partie du module "Gestionnaire de Flux" de Mine.

## Contexte

Suite logique de FeedGen (génération IA de descriptions et d'attributs produits). Dan a un scoring existant ; l'étape actuelle est de faire correspondre chaque produit à la bonne catégorie GPC (jusqu'à 7 niveaux de profondeur).

Onboarding initié le 2026-06-11 avec Dan. Code source partagé sur GitHub par Dan.

## Stack technique

- **LLM** : Gemini 2.5 Flash (classification), Gemini 2.5 Flash-Lite (agent ADK)
- **Embeddings** : text-embedding-004 (768 dims) via Vertex AI
- **BQ dataset** : `pmed-portal-prd-mg.feed_manager_categorization`
- **GCP** : Cloud Functions, Pub/Sub, Workflows, Vertex AI Vector Search
- **Framework** : Google ADK (agent level-by-level)

## Architecture — approche Dan (production)

Classification progressive niveau par niveau dans l'arbre GPC :

1. Le LLM reçoit les candidats d'un seul niveau (ex. toutes les category_0)
2. Il score et choisit le meilleur candidat
3. On descend dans l'arbre avec ce choix, jusqu'à une feuille ou une abstention
4. Règles déterministes post-LLM : seuil de confiance (0.5), gap ambiguïté (0.1)

Deux modes : single-product (via ADK `LlmAgent`) et batch (jusqu'à 200 produits/appel LLM). Industrialisé via deux Cloud Functions déclenchées par Pub/Sub.

**Limitation connue** : pas de backtracking — si la category_0 est mal choisie, on reste bloqué dans la mauvaise branche.

## Tables BQ clés

| Table | Rôle |
|---|---|
| `tmp_tb_products` | Produits à classifier (dataset de test, 830 lignes) |
| `tb_consolidated_taxonomy_with_ids` | Taxonomie GPC complète (EN + FR, ~5600 lignes) |
| `tb_products_state` | État de classification par produit (category_0..6, score, is_leaf, abstain) |
| `tb_batch_status` | Suivi des batches (run_id, batch_id, status) |

## Flux produit — connexion avec le Feed Manager

FeedGen Catégorisation est une **feature avale** du Feed Manager (module ConnectedHub). Les produits sont les mêmes entités vues à deux étapes successives du pipeline :

```
Client upload son catalogue → Feed Manager importe le flux
         ↓
Feedgen (enrichissement IA) : génère de nouvelles colonnes sur les produits
(ex. colonne "feedgen" = "Oui/Non", descriptions enrichies, attributs…)
         ↓
FeedGen Catégorisation : prend la description produit et attribue
un google_product_category_id (jusqu'à 6 niveaux dans la taxonomie GPC)
         ↓
Le champ google_product_category revient dans le flux enrichi,
prêt pour l'export vers Google Shopping / Meta Ads
```

**Ce que chaque module voit du produit :**

| Module | Vision du produit |
|---|---|
| Feed Manager | Ligne du flux client — toutes les colonnes (titre, prix, image URL, etc.) |
| FeedGen Catégorisation | Seulement `id` + `description` — le reste ne compte pas pour classifier |

**Le signe révélateur dans les données de test** : le dataset `tb_test_set.csv` contient une colonne `feedgen = "Oui"` — c'est littéralement la colonne produite en amont par un job feedgen de ConnectedHub. Les produits arrivent ici après avoir déjà été enrichis par le module Feedgen.

**Pourquoi `google_product_category` est critique** : les clients renseignent souvent ce champ de façon incomplète (trop superficiel) ou incorrecte (mauvaise branche). Des catégories fausses = audiences mal ciblées = baisse de performance des campagnes Shopping/Méta.

**Surface d'intégration dans ConnectedHub** :
- Les jobs feedgen existants écrivent dans `pmed-portal-prd-mg.feed_manager.{customer_id}_{feed_id}_ready`
- C'est cette table qui doit alimenter `tb_products_state` (scope à paramétrer par customer/feed)
- Le résultat (`google_product_category_id` + chemin complet) remonte comme nouvelle colonne dans le flux

## Exploration Adam — recherche sémantique (testé, conclusions)

Approche alternative explorée dans `tmp_adam.ipynb` : embedder toute la taxonomie et trouver directement la catégorie la plus proche par similarité cosine, sans traversée niveau par niveau.

**Infra mise en place (opérationnelle)** :
- Embeddings EN+FR générés (5595 vecteurs, 768 dims) avec `text-embedding-004`
- Uploadés sur `gs://pmed-portal-feedgen-embeddings/feedgen/taxonomy-embeddings/`
- Index Vertex AI Vector Search (Brute Force, europe-west1) — READY
- Endpoint déployé : `projects/17959324466/locations/europe-west1/indexEndpoints/84464483245752320`, deployed index `feedgen_taxonomy_2`

**Résultats des tests — limitation critique identifiée** :

Pour "Table guéridon ronde en aluminium", les top 5 résultats sont tous dans `Équipements sportifs > Jeux d'intérieur` (billard, ping-pong, air hockey) et `Quincaillerie`. La bonne catégorie (Ameublement) n'apparaît pas.

Cause : le mot "table" est très présent dans les catégories de sport (table de ping-pong, table de billard). L'embedding fait du matching lexical sur ce mot plutôt que du raisonnement sémantique sur le contexte produit.

**Conclusion provisoire** : l'approche embedding seule est insuffisante sur des descriptions longues et des noms ambigus. L'approche LLM de Dan est supérieure sur ce cas.

## Répartition des approches — retour de Dan (2026-06-25)

Dan confirme : son approche LLM **a besoin de la description enrichie** pour bien fonctionner. Sans feedgen, la classification est dégradée.

Pour l'embedding (notre approche RAG), c'est l'inverse : on a besoin de texte **court et focalisé** → les titres sont plus adaptés que les descriptions.

**Split naturel des deux approches :**

| Cas | Input disponible | Approche recommandée |
|---|---|---|
| Produit avec description feedgen | Description riche (~500 mots) | Dan (LLM level-by-level) |
| Produit sans description feedgen | Titre brut client | Embedding RAG (à valider) |
| Produit avec titre feedgen | Titre enrichi | Embedding RAG (à tester) |

Les deux approches sont **complémentaires**, pas concurrentes — elles couvrent des cas différents selon ce que le flux client contient.

## Versions RAG à tester

| #   | Input                                          | Embedding                            | Sélection finale            | Statut                                                      |
| --- | ---------------------------------------------- | ------------------------------------ | --------------------------- | ----------------------------------------------------------- |
| V1  | Raw title                                      | default (symétrique)                 | cosine top-1                | ✅ Testé — résultats pauvres                                 |
| V2  | Raw title                                      | RETRIEVAL_QUERY / RETRIEVAL_DOCUMENT | cosine top-1                | ❌ Abandonné — `cos(default, RETRIEVAL_QUERY)=1.0` sur text-embedding-004 : task_type sans effet, levier inexistant (2026-07-05) |
| V3  | Raw title                                      | default                              | cosine top-5 → LLM reranker | ✅ Testé 2026-07-06 sur jeu gold Kérastase — LLM rerank : hiérarchique 75,9 %→88,8 %. Cross-encoder aussi comparé (voir [[Session 2026-07-06]]) |
| V4  | FeedGen title                                  | default                              | cosine top-1                | À faire                                                     |
| V5  | Raw title nettoyé (sans marque/couleur/taille) | default                              | cosine top-1                | En dernier                                                  |

## Architecture de serving en prod — décision (2026-07-02)

**Retenu : recherche vectorielle en BQ natif** (`VECTOR_SEARCH()` / `ML.DISTANCE(..., 'COSINE')`), pas Vertex AI Vector Search.

Raison : Vector Search facture des nœuds de serving **en continu 24/7 tant que l'index est déployé** (coût de disponibilité, pas d'usage). Pour 5600 vecteurs, ce plancher (~$500-1000/mois pour 1 index) est disproportionné — il ne se justifie qu'à des millions de vecteurs + QPS soutenu. Incident du 2026-07-02 : 2 index laissés déployés → ~$70/jour week-ends compris (voir [[Session 2026-07-02]]).

| | Vector Search | **BQ natif (retenu)** | Cloud Run |
|---|---|---|---|
| Coût idle | ~$500-1000/mois | **~$0** | ~$0 (scale to zero) |
| Coût usage | inclus (nœuds) | scan BQ (~centimes) | requête-seconde |
| Infra | endpoint permanent | **aucune (SQL)** | 1 conteneur |
| Fit flux | batch : surdimensionné | **batch : idéal** | online : idéal |

Les produits sont déjà dans BQ (`feed_manager_categorization`) → aucune donnée à déplacer. À 5600 vecteurs, brute-force suffit (pas de `VECTOR INDEX`). Cloud Run seulement si serving online depuis l'UI ConnectedHub un jour ; Vector Search jamais sauf explosion du volume.

**Conséquence exploration** : les index Vertex ont été undeployés le 2026-07-02. Pour V3+, expérimenter en local (FAISS/numpy, 17 Mo en mémoire) — plus de dépendance à un index Vertex déployé.

## Repo Adam — `C:\dev\DS\feedgen-categorisation-rag`

Repo git propre. Stack `rag/` : `embed` (embedding partagé + cache) · `local_search` (FAISS, exploration) · `bq_search` (BQ `VECTOR_SEARCH`, prod) · `classify`. Une seule interface `search_taxonomy_batch` sur deux backends. Migré hors Vertex le 2026-07-05, cluster Vertex supprimé, commit `c03ac82` (voir [[Session 2026-07-05]]).

**Tables BQ** (`feed_manager_categorization`) :
- `tb_taxonomy_embeddings` — 11 190 embeddings taxonomie (FR+EN) pour `VECTOR_SEARCH`
- `tb_casal_raw_titles` — 29 226 produits, titre brut (`Title`)
- `tb_casal_feedgen_titles` — 29 226 produits, titre FeedGen (`_1a6c_Title`)
- `tb_kerastase_eval` — 294 produits gold (titre opti main + GPC corrigé), jeu d'éval (voir [[Session 2026-07-06]])

**V1 sur la nouvelle archi** : le baseline stocké de juin s'est révélé non fiable ; V1 propre à rejouer via BQ (voir [[Session 2026-07-05]]).

## Jeu gold + méthode d'éval (2026-07-06)

Concrétisation du jeu gold cherché depuis le 07-05. `tb_kerastase_eval` : 294 produits Kérastase, **titre optimisé à la main + GPC corrigé** (vérité terrain). Script `scripts/setup_kerastase_eval.py`. Réserve : titres opti = mesure un **plafond**, pas la perf sur titres bruts.

**Métrique clé — hiérarchique, pas exact-match.** La vérité terrain met 78 % des produits dans le générique `486 = Soin des cheveux` ; l'exact-match ID est donc plafonné et trompeur. La bonne métrique : la prédiction est-elle dans la **bonne branche** (égale, ancêtre ou descendante du vrai chemin) ? Elle distingue une vraie erreur d'une simple différence de granularité.

## V3 reranking — résultats (2026-07-06)

Pattern **retrieve → rerank** : stage 1 (bi-encoder, top-10) inchangé, 2ᵉ étage re-score. Benchmark 3 bras sur le jeu gold. Code repo : `rag/rerank.py`, `rag/rerank_eval.py`, `experiments/v3_llm_rerank.py`, `experiments/v4_cross_encoder.py` (commit `25073d1`).

| Métrique | Arm 0 bi-encoder | Arm A LLM (Gemini Flash) | Arm B cross-encoder (bge) |
|---|---|---|---|
| Exact-ID | 3,1 % | 15,6 % | 24,1 % |
| Hiérarchique | 75,9 % | **88,8 %** | 60,5 % |

- **LLM rerank (arm A)** = meilleure sécurité de branche (+13 pts), rapide, sans infra → **retenu pour la prod**. Corrige les noms-tête ambigus (Huile, Masque…) via le contexte marque.
- **Cross-encoder (arm B)** = meilleur exact-ID mais pire hiérarchique (*sous* le baseline) : aveugle au domaine capillaire. Potentiel réel **seulement fine-tuné** (levier futur, coût GPU).

## Dernières sessions

- **2026-07-06** — Jeu gold `tb_kerastase_eval` construit (294 produits, GPC corrigé). Métrique hiérarchique définie (vs exact-match trompeur). V3 livré : benchmark retrieve→rerank sur 3 bras — LLM rerank (Gemini Flash) retenu : hiérarchique 75,9 %→**88,8 %**. Cross-encoder (bge) meilleur exact-ID mais pire hiérarchique → futur fine-tuning seulement.
- **2026-07-05** — Archi BQ finalisée : `classify.py` branché sur `bq_search`, cache d'embeddings (`title_embeddings_cache.parquet`), cluster Vertex supprimé du repo. Découverte : baseline juin non fiable (dérive embeddings). `task_type` enterré (cos default/RETRIEVAL_QUERY = 1.0 → aucun effet). Commit `c03ac82`.
- **2026-07-02** — Incident cost Vertex AI : 2 index laissés déployés → ~$70/jour. Undeploy des index. Décision archi prod : **BQ natif** (VECTOR_SEARCH) remplace Vertex. Table `tb_taxonomy_embeddings` + VECTOR_SEARCH validés de bout en bout.
- **2026-06-26** — V2 RETRIEVAL_DOCUMENT testée → pire que V1. Repo extrait (`feedgen-categorisation-rag`), refacto propre. Tables BQ de test créées (`tb_test_raw_titles`, `tb_test_feedgen_titles`). V1 lancée sur raw titles. Décision : V3 = LLM reranker sur top-k.
- **2026-06-25** — Endpoint Vertex déployé, tests embedding sur vrais produits : limitation critique identifiée (matching lexical, « table » → sport). Approches Adam/Dan = complémentaires (titres vs descriptions).
- **2026-06-24** — Lecture code Dan, embeddings EN+FR générés (5595 vecteurs), index Vertex AI Brute Force créé. Bucket dédié `pmed-portal-feedgen-embeddings` en europe-west1.

## Prochaines étapes

- [ ] Établir un V1 propre reproductible (raw + feedgen) via le chemin BQ
- [ ] Comparer raw vs feedgen → quantifier l'apport des titres enrichis (nouvelle méthode de comparaison à définir)
- [x] Implémenter V3 : LLM reranker sur top-k — fait le 2026-07-06 (Gemini Flash + cross-encoder comparé), voir [[Session 2026-07-06]]
- [ ] **Vrai jeu d'éval avec Manu** : GPC vérifié à la main + titre brut si possible (sortir du plafond « titres opti » et du mono-flux Kérastase)
- [ ] **Cross-encoder en prod** : archi de serving batch (pas d'endpoint continu) + comment le fine-tuner sur les labels corrigés
- [ ] **Intégration avec Dan** : brancher le RAG en amont de sa solution LLM (RAG dégrossit → LLM tranche)
- [ ] (dette) Valider la parité d'embedding Python vs `ML.GENERATE_EMBEDDING` avant tout passage full-SQL
