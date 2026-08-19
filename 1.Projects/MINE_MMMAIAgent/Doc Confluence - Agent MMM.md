---
type: note
projet: MMM AI Agent
---

> Note de travail : contenu destiné à **Confluence**. Copier à partir du titre ci-dessous, sans cette frontmatter ni ce bloc. Aucun wikilink dedans, la page est autonome hors du vault.

# MMM AI Agent — documentation technique

> **Périmètre de cette page** : l'**agent** seul, c'est à dire le pipeline multi-agents Google ADK déployé sur Vertex AI Agent Engine dans le projet `med-dtam-prd-mg`.
> L'**intégration dans ConnectedHub** (route backend, frontend React, feature flag, gestion des sessions) est documentée à part et suivie par l'équipe ConnectedHub.
> Rédigée le 19/08/2026 dans le cadre de la passation. Référent à partir du 22/08 : Dan.

---

## 1. En une page

L'agent est un assistant conversationnel branché dans le module MMM de ConnectedHub. Il répond en langage naturel à des questions sur les données MMM d'un client, ROAS et contributions par média, comparaisons de périodes, saturation, sans que l'utilisateur ait à passer par la console GCP ou par un data strat.

| | |
| --- | --- |
| Projet GCP | `med-dtam-prd-mg` |
| Région | `europe-west1` (contrainte RGPD) |
| Framework | Google ADK, pinné `google-adk==1.26.0` |
| Runtime | Vertex AI Agent Engine (Reasoning Engine) |
| Service account | `mmm-agent-sa@med-dtam-prd-mg.iam.gserviceaccount.com` |
| Table source | `med-dtam-prd-mg.mine.tb_model_contributions` |
| Repo | `github.com/Publicis-Media-France-FR5140/MMM_AI_Agent` |
| Engine en prod | `6241382153116975104`, display name `MMM_Agent_v3_live_ui_scope` |
| Engine de rollback | `2659…`, version du 22/07 |

Coût : environ **$35/mois de runtime fixe** (facturé même sans usage, 24h/24) plus environ **$0,05 par question**. La structure est à 75% fixe, ce qui veut dire qu'un engine oublié coûte autant qu'un engine utilisé.

---

## 2. Le contrat d'entrée

C'est la frontière entre l'agent et ConnectedHub. Tout message reçu par l'agent a cette forme :

```
[MINE_CLIENT_ID: opel_fr]
[MINE_UI_CONTEXT: kpi=…; kpi_label=…; period=2025-06-02..2026-05-11; tab=results; lang=fr]
Quel est mon meilleur média en ROAS ?
```

- **Ligne 1, obligatoire** : le `client_id`. Il est dérivé côté serveur à partir de la session authentifiée, **jamais** fourni par l'utilisateur. C'est le pivot de toute l'isolation inter-client.
- **Ligne 2, optionnelle** : le scope de l'écran au moment de l'envoi (KPI, période, onglet, langue). L'objet entier peut être absent, et ça arrive en trafic normal, pas seulement en cas limite.
- **Le reste** : la question.

Toute évolution de ce format se décide **conjointement** avec l'équipe ConnectedHub. Un changement unilatéral d'un côté casse l'autre.

---

## 3. Architecture du pipeline

L'agent n'est pas un agent unique avec un gros prompt. C'est une chaîne de **9 `LlmAgent`**, dont chacun a un rôle étroit et un toolset restreint.

```
MMM_agent (Flash) — gatekeeper et routeur
│   Évalue si la question est assez précise. Sinon, demande une clarification.
│   Sinon, transfère à data_pipeline.
│
└── data_pipeline (SequentialAgent)
      ├── context_setter_agent (Flash) → extract_and_set_client_context
      │     Regex Python : extrait le client_id vers le session state
      ├── schema_agent (Flash) → get_table_info
      │     Schéma live de la table BigQuery
      ├── discovery_agent (Flash) → run_discovery
      │     SQL hardcodé. Vérités terrain : KPIs disponibles, training_date
      │     par KPI, canaux par KPI
      ├── refinement_loop (LoopAgent, max 3 itérations)
      │     ├── query_writer_agent (Pro) — écrit le SQL, aucun outil
      │     ├── sql_guard (SequentialAgent)
      │     │     ├── filter_checker_agent (Flash) → validate_client_filter
      │     │     └── executor_agent (Flash) → execute_sql (WriteMode.BLOCKED)
      │     └── validator_agent (Flash) → exit_loop
      └── answer_agent (Pro) — synthèse en langage naturel
```

**9 à 17 appels LLM par question**, dont 2 en `gemini-2.5-pro` (`query_writer_agent` et `answer_agent`). Environ 47 secondes pour une réponse normale. Ce sont ces deux appels Pro et le contexte accumulé qui dominent le coût.

### Pourquoi cette découpe

Trois choix structurants, à connaître avant de modifier quoi que ce soit :

1. **`context_setter_agent` est le premier maillon d'un `SequentialAgent`**, et pas un outil que le root agent doit penser à appeler. L'ordre est donc **structurel**, pas dépendant d'une instruction que le LLM peut ignorer. C'est ce qui garantit que le `client_id` est toujours en session state avant la moindre requête.
2. **L'écriture du SQL est séparée de son exécution** (`query_writer_agent` n'a aucun outil, il produit du texte). C'est ce qui permet à `filter_checker_agent` de valider la requête **avant** qu'elle atteigne BigQuery.
3. **`sql_guard` est un `SequentialAgent`**, ce qui garantit que le checker tourne avant l'executor. Là encore, structurel plutôt qu'instructionnel.

---

## 4. Le contexte métier, scopé par sous-agent

Le contexte métier n'est pas un bloc monolithique. Il est découpé en blocs composables (`CTX_INTRO`, `CTX_DISPLAY`, `CTX_DISCOVERY_NOTE`, `CTX_SQL_RULES`, `CTX_FIELDS`, `CTX_METRICS`, `CTX_KPI_SELECTION`), assemblés en trois profils et injectés sélectivement.

| Agent | Contexte injecté |
| --- | --- |
| `schema_agent` | **aucun** (`CTX_NONE`), plus « exactly one tool: get_table_info » |
| `discovery_agent` | `CTX_NO_SQL`, plus « exactly one tool: run_discovery » |
| `query_writer_agent` | `CTX_FULL` **+ `CTX_UI_SCOPE`** |
| `validator_agent` | `CTX_NO_SQL` |
| `answer_agent` | `CTX_NO_SQL` **+ `CTX_MODULE_VIZ` + `CTX_UI_SCOPE`** |
| `root_agent` | `CTX_NO_SQL` **+ `CTX_MODULE_VIZ` + `CTX_UI_SCOPE`** |

**La règle à respecter** : `CTX_SQL_RULES` et la syntaxe SQL ne vont **qu'au** `query_writer_agent`. Donner des règles SQL à un agent qui n'a pas d'outil d'exécution lui donne l'amorce pour halluciner du SQL et des résultats. Ce découpage est le correctif d'un vrai problème d'hallucination constaté en juillet, ce n'est pas une élégance.

### Les blocs particuliers

- **`CTX_MODULE_VIZ`** : décrit les graphiques du module et leurs contrôles, onglet par onglet, pour que l'agent aide à **lire** les graphiques. Bilingue EN et FR, les libellés étant sourcés des fichiers i18n de l'application. Générique, aucune variable client. Injecté dans `answer_agent` et `root_agent` seulement.
- **`CTX_UI_SCOPE`** : le scope écran, avec ses 9 règles de comportement. Les deux qui comptent le plus : le scope est un **défaut, pas une contrainte** (ce que l'utilisateur formule explicitement gagne toujours, dimension par dimension), et seule la ligne du **message courant** compte, les tours précédents étant de l'historique et jamais une source de défaut.
- **`run_discovery` reste volontairement non scopé** par le scope écran. Il dit ce qui **existe** (KPIs, dates, canaux), donc le scoper casserait les réponses à « qu'est ce que je peux regarder d'autre ? ».
- **Le routage du root est durci** : dès qu'un chiffre est nécessaire, il délègue à `data_pipeline`, même si la question nomme aussi un graphique. Il ne répond seul que sur des questions d'interface pures. Sans ça, il répondait « chart only » à des questions data.

---

## 5. Isolation inter-client

**C'est le point le plus sensible du système.** La table BigQuery contient tous les clients. L'isolation ne repose pas sur des tables séparées mais sur l'injection du `client_id` côté serveur, plus plusieurs couches Python que le LLM ne peut pas contourner.

| Couche | Mécanisme | Force |
| --- | --- | --- |
| Backend ConnectedHub | Préfixe `[MINE_CLIENT_ID: <id>]`, **toujours en première ligne** | Dure, côté serveur |
| `extract_and_set_client_context` | Regex Python, **leftmost match** | Dure |
| `run_discovery` | SQL hardcodé `WHERE client_id = @client_id` | Dure |
| `validate_client_filter` | Vérification Python avant toute exécution BigQuery | Dure |
| `sql_guard` | `SequentialAgent` qui garantit checker avant executor | Dure |
| `query_writer_agent` | Instruction d'inclure `client_id` dans le SQL | Souple |

Validé en test par Dan le 07/07/2026 : depuis Opel DE, une requête sur Peugeot a été refusée proprement.

### Ce qui donne réellement la garantie

**C'est le leftmost match, pas l'ancrage de la regex.** Le backend émet toujours sa ligne en premier, donc la première occurrence trouvée dans le message est toujours la valeur dérivée du serveur. Un marqueur forgé plus loin dans le texte utilisateur ne peut pas détourner l'extraction.

Le code contient deux motifs :

```python
CLIENT_ID_RE = re.compile(r'^\[MINE_CLIENT_ID:\s*([^\]\n]+)\]\s*$', re.MULTILINE)
CLIENT_ID_RE_FALLBACK = re.compile(r'\[MINE_CLIENT_ID:\s*([^\]]+)\]')
```

> ### Avertissement, à lire avant toute modification de cette regex
>
> **Supprimer le repli `CLIENT_ID_RE_FALLBACK` est une régression, pas un durcissement.**
>
> Le `raw_message` n'est pas lu directement sur le réseau, il est transmis par `context_setter_agent`, c'est à dire **par un LLM** à qui on demande de recopier verbatim le texte du premier message utilisateur. Un modèle peut ajouter un préambule, encadrer la ligne de guillemets, ou laisser une espace en tête. Le motif ancré ne tolère rien de tout ça.
>
> Sans le repli, l'extraction renvoie alors une erreur fatale et **toutes** les questions data cassent, pas seulement celles qui portent un scope.
>
> L'ancrage est de la défense en profondeur par dessus le leftmost match, il n'est pas la source de la garantie. Le test `tests/test_ui_scope.py` verrouille ce comportement, ne pas le contourner.

### Le scope écran est du contenu non fiable

Le `ui_context` vient du navigateur, et il atterrit juste à côté du garde-fou d'isolation. La protection est côté backend ConnectedHub, en **whitelist stricte avec rejet** et non nettoyage :

- Libellés : `^[\p{L}\p{N} _\-./&'(),%:+|]{1,64}$`. Dates : `^\d{4}-\d{2}-\d{2}$`. Énumérations fermées pour `tab` et `lang`.
- Exclure `[`, `]`, `;`, `=` et les retours à la ligne rend un faux marqueur **structurellement inexprimable**. **C'est toute la propriété de sécurité, ne jamais la relâcher.**
- Rejet champ par champ : un `tab` invalide est ignoré, il ne casse jamais la conversation. Les champs rejetés sont loggés, sinon un rejet systématique serait invisible.

---

## 6. Le repo

`github.com/Publicis-Media-France-FR5140/MMM_AI_Agent`, branche `main`.

```
MMM_Agent/
  agent.py                     tout le pipeline, les blocs CTX_* et les tools
  agent_tester.py              script d'interrogation en local
  requirements.txt             c'est CELUI-CI que lit adk deploy
  .env                         projet, région, flags de télémétrie (aucun secret)
  .agent_engine_config.json    display_name, description, service account
tests/                         35 tests, pytest
requirements.txt               dépendances racine
requirements-dev.txt           outillage de test
```

Lancer les tests :

```bash
python -m pytest tests -q
```

Les tests ne sont pas décoratifs : ils verrouillent les propriétés qu'une modification de prompt casserait sans erreur visible. `test_context_scoping.py` vérifie que chaque interprète reçoit bien son profil de contexte, `test_ui_scope.py` verrouille le leftmost match et les 9 règles du scope, `test_module_viz.py` vérifie que le root délègue toujours les questions data malgré le bloc viz.

> Les docstrings de `extract_and_set_client_context`, `run_discovery` et `validate_client_filter` **sont lues par l'ADK** et envoyées au modèle comme description des outils. Les modifier change le comportement de l'agent. Ce ne sont pas de simples commentaires.

---

## 7. Déployer

### Prérequis

- `gcloud auth application-default login`. **L'ADC expire d'un jour à l'autre**, c'est la première chose à vérifier quand une commande échoue sans raison apparente.
- Les rôles listés au §10.

### Commandes

Créer un nouvel engine :

```bash
PYTHONUTF8=1 adk deploy agent_engine MMM_Agent --validate-agent-import
```

Mettre à jour un engine existant, sans re-pointer le backend :

```bash
adk deploy … --agent_engine_id <id> --project med-dtam-prd-mg
```

### Les pièges

Tous constatés en production entre le 20 et le 23 juillet 2026.

| Piège | Ce qui se passe si on l'oublie |
| --- | --- |
| `google-adk` doit être pinné `==1.26.0` dans **`MMM_Agent/requirements.txt`**, pas celui de la racine | `adk deploy` lit le requirements du dossier agent. Sans pin, `google-adk 2.x` s'installe et l'engine **crash au démarrage** sur `google.cloud.dataplex_v1` manquant |
| `--validate-agent-import` | Valide l'import **avant** de créer l'engine. Sans ce flag, on crée un engine mort qu'il faut ensuite retrouver et supprimer |
| `PYTHONUTF8=1` | Sur console Windows en cp1252, le caractère de fin plante l'affichage et la commande annonce « Deploy failed » **alors que l'engine a bien été créé**. Piège coûteux, on redéploie inutilement |
| `--project` explicite sur un update en place | Sans lui, 404 sur le mauvais projet |
| `--display_name` n'est appliqué **que** quand il est passé | Un update sans le flag retombe sur le `display_name` de `.agent_engine_config.json` et **écrase silencieusement** le nom versionné. Le nom versionné vit donc dans ce fichier, **à bumper à chaque nouvelle version** |

### Nouvel engine ou update en place

**Créer un nouvel engine** dès que le changement touche **tout le trafic** et pas seulement la fonctionnalité ajoutée, et que la vérification ne peut se faire qu'après déploiement.

Le cas d'école est le correctif de la regex `client_id` : un update en place de la production aurait exposé tous les clients avant toute vérification possible. La bonne séquence est alors : nouvel engine, test en local pointé dessus, puis bascule par PR côté ConnectedHub.

À l'inverse, un changement **inerte** tant que la production ne pointe pas dessus peut être poussé en place sans risque.

### Gestion des engines

**Règle : exactement deux engines vivants**, la production et son rollback. On ne supprime l'ancien qu'une fois le nouveau prouvé en monitoring.

Ce n'est pas de l'hygiène gratuite : chaque engine déployé facture son runtime en continu, environ $35 par mois **même inutilisé**. Les engines `6073…` et `7899…` ont été supprimés le 23/07 pour cette raison.

Historique : `6073…` (avril), puis `7899…` (scoping du contexte, 20/07), puis `2659…` (aide graphiques et correctifs, 22/07, rollback actuel), puis `6241…` (scope écran live, 23/07, production actuelle).

---

## 8. Diagnostiquer

### Les quatre sources

| Source | Contenu | Rétroactif |
| --- | --- | --- |
| **Logs backend** (`pmed-portal-prd-mg`) | `authors`, `lastChunk`, `streamErrors`, `adkSessionId`, `status` | oui |
| **GCS** `mmm-agent-chat-logs` (`med-dtam-prd-mg`) | Un JSON par échange, avec `adkSessionId` comme clé de jointure | oui |
| **Sessions API** de l'Agent Engine | Events internes : author par sous-agent, appels d'outils, SQL, réponses | **oui, et gratuit** |
| **Trace Explorer** | Timing, waterfall, structure. Environ 48 spans par question | non, seulement depuis le grant du 08/07 |

Attention, **les logs backend sont dans un autre projet GCP** que l'agent. C'est la raison pour laquelle l'accès aux deux projets est nécessaire.

### Recette pour un `no_answer`

1. **Repérer**, via l'alerte mail ou la requête de triage :
   ```
   jsonPayload.metric="mmm_agent_exchange" AND jsonPayload.status="no_answer"
   ```
   Récupérer `authors` et `adkSessionId`.
2. **Localiser** : le champ `authors` (par exemple `MMM_agent>context_setter_agent>schema_agent`) donne directement le dernier sous-agent atteint, donc l'endroit où le pipeline meurt.
3. **Creuser** : Sessions API sur l'`adkSessionId`, qui donne les events internes même si aucune trace n'existe.
   ```
   GET https://europe-west1-aiplatform.googleapis.com/v1beta1/projects/med-dtam-prd-mg/locations/europe-west1/reasoningEngines/6241382153116975104/sessions/<adkSessionId>/events
   ```
4. **Performance seulement** : Trace Explorer. Une durée courte, environ 10 secondes contre 47 normalement, avec un waterfall tronqué, signe un raté amont.

> Deux avertissements. Le **span status est toujours `UNSET`**, c'est le comportement normal en OpenTelemetry et l'ADK ne stampe pas `ERROR` sur un échec : ne jamais diagnostiquer par le status, se fier à `lastChunk` et `streamErrors`. Et le filtre du Trace Explorer est `service.name = 6241382153116975104`, **à mettre à jour à chaque bascule d'engine**, sinon on lit le bruit de la Cloud Function d'allocation budget.

### L'alerte

Policy log-based `alertPolicies/527257283050504576`, nommée *MMM agent — no_answer (log-based)*. Elle se déclenche sur chaque ligne de log `no_answer` et **extrait `authors`, `adkSessionId` et `customerId` dans la notification**, ce qui permet de trier directement depuis le mail. Rate limit de 5 minutes.

La métrique log-based `mmm_agent_exchange` (DELTA/INT64, labels `status` et `customerId`) reste disponible pour construire des dashboards.

---

## 9. Coût

Structure à environ 75% fixe.

| Poste | Nature | Montant |
| --- | --- | --- |
| Agent Engine runtime | **Fixe**, 24h/24, même sans usage | environ $35/mois |
| Tokens Gemini | Variable, par question | environ $0,05, jusqu'à $0,10 avec retries |
| Artifact Registry, GCS, Scheduler, Secret Manager | Continu | environ $6/mois |

Par client, un seul client portant toute la base fixe : environ $85/mois à 1 000 questions, $185 à 3 000, $335 et plus à 6 000.

Leviers si le coût devient un sujet : passer `answer_agent` de Pro à Flash, ou réduire `max_query_result_rows=200` avant l'appel de synthèse.

> **Écarté explicitement** : alléger le contexte métier pour économiser des tokens. La décision est inverse, on l'**enrichit** pour la qualité de réponse. La qualité prime sur ce micro gain.

Observabilité : environ 2,5 millions de spans gratuits par mois, soit environ 52 000 questions. À tous les volumes envisagés aujourd'hui, le coût de tracing est nul.

---

## 10. Accès et rôles

Deux projets GCP, c'est le point à ne pas rater.

### `med-dtam-prd-mg`

| Rôle | Pour quoi faire |
| --- | --- |
| `roles/aiplatform.user` | Créer, mettre à jour, supprimer les engines. Appeler `streamQuery`. Lire la Sessions API |
| `roles/iam.serviceAccountUser` **sur `mmm-agent-sa`** | Déployer un engine qui **tourne sous** ce SA. **Piège le plus courant** : sans ce rôle, `adk deploy` échoue en `PERMISSION_DENIED` alors même que `aiplatform.user` est accordé |
| `roles/storage.objectAdmin` sur le bucket de staging | `adk deploy` y dépose le package |
| `roles/storage.objectViewer` sur `mmm-agent-chat-logs` | Relire un échange complet |
| `roles/bigquery.dataViewer` et `roles/bigquery.jobUser` | Rejouer les requêtes à la main |
| `roles/cloudtrace.user` | Lire le Trace Explorer |

### `pmed-portal-prd-mg`

| Rôle | Pour quoi faire |
| --- | --- |
| `roles/logging.viewer` | Lire les logs `mmm_agent_exchange`. **Sans lui, la recette de diagnostic ne démarre pas** |
| `roles/monitoring.viewer` | Métrique log-based et incidents |
| `roles/monitoring.editor` | Gérer la policy d'alerte et son destinataire |

### Le service account, à ne pas confondre

`mmm-agent-sa@med-dtam-prd-mg.iam.gserviceaccount.com` porte les 7 rôles dont l'agent a besoin **à l'exécution** : `aiplatform.user`, `bigquery.dataViewer`, `bigquery.jobUser`, `cloudtrace.agent`, `logging.logWriter`, `monitoring.metricWriter`, `telemetry.tracesWriter`. Rien à y changer. Un humain a besoin de droits pour **piloter** l'agent, le SA a besoin de droits pour le **faire tourner**.

---

## 11. Bugs et limites connus

| Sujet | Gravité | Détail |
| --- | --- | --- |
| Sessions backend en mémoire | Limite connue | La `Map` en mémoire du backend ConnectedHub perd les sessions au redémarrage. La mémoire persistante par utilisateur est un chantier ouvert. **Côté intégration, pas côté agent** |
| `context_setter_agent` dit « premier message » | Fragilité de formulation | Son instruction demande le texte du **premier** message utilisateur, alors que la règle de récence du scope exige le message **courant**. En pratique ça fonctionne, vérifié le 23/07, parce que les agents lisent la ligne dans le transcript. Mais si un jour une réponse porte sur un KPI périmé, c'est là qu'il faut regarder. La correction tient en une ligne |
| `ui_scope` en session state jamais lu | Code mort | `extract_and_set_client_context` écrit `tool_context.state["ui_scope"]`, mais aucun agent ne lit cette clé. Le scope circule en réalité par le **transcript**. Inoffensif, mais ne pas croire que modifier cette clé change le comportement |
| Colonne marché | Imprécision documentée | `level` **est** la clé marché dans tout le système : la Cloud Function d'allocation fait `level = request_json["market"]`, et le frontend groupe ses scopes par `d.level`. La description dans `CTX_FIELDS` (« granularité de l'analyse ») est au mieux incomplète. Retirer `market` des règles SQL de l'agent reste le bon choix, sa table étant `tb_model_contributions` et aucun client multi marché n'existant aujourd'hui, mais à reprendre le jour où un client multi marché arrive |

---

## 12. Ce qui reste ouvert

### Backlog agent

- **Enrichir le contexte métier** avec les définitions des variables et des données du modèle. C'est le point de reprise le plus naturel côté data science, et le besoin le plus remonté en phase de testing.
- **Brancher `cf-budget-allocator-prod` comme outil**. Rien n'implémenté, cadrage complet disponible. Trois pièges qui produisent des chiffres faux sans lever d'erreur : tout est **positionnel** (bornes, inflations et réponse sont des tableaux sans nom de média), `inflations` est en **pourcents** et non en ratio, et `t` est un **indice de départ** et non une durée. L'ordre des médias se dérive de `tb_model_configuration`, il ne se devine pas, et le LLM ne doit jamais composer ces tableaux. Prérequis : `roles/run.invoker` sur ce seul service, à ne demander que le jour où le sujet démarre.
- **Knowledge par client** : contexte secteur, concurrents, cadrage par `client_id`. Chantier v2, à traiter d'un bloc avec la couche pédagogique et l'expertise média, pas en silos.

### Décisions produit non tranchées

- **Jusqu'où l'agent recommande.** Deux options sur la table : deux niveaux selon le compte (recommandation pleine en interne, faits seuls côté client), ou assistant pur sans aucune recommandation. Arbitrage attendu de Baptiste. C'est **structurant** : en option assistant pur, l'outil d'allocation budget ne pourrait que lire et expliquer une allocation, jamais en proposer une.
- **Ouverture au conseil sur un MMM live**, en commençant par Stellantis (Opel DE et Peugeot DE). Jalon terrain, pas technique.

---

## 13. Sources signalées à d'autres équipes

À rappeler aux data scientists propriétaires de `cf-budget-allocator-prod` : la fonction **interpole le `client_id` directement dans le SQL**, sans paramètre, et s'exécute avec le service account `internal@med-dtam-prd-mg`.

```python
WHERE client_id = '{client_id}' AND level = '{level}' AND kpi = '{kpi}'
```

Aujourd'hui sans risque, la valeur venant du backend ConnectedHub. Mais le jour où une valeur issue d'un LLM atteindrait ce payload, ce serait une injection SQL exploitable. C'est ce qui fait passer la règle « injecter le `client_id` depuis le session state » du statut de bonne pratique à celui d'obligation.
