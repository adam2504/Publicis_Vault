# Architecture technique

Projet : [[MMM AI Agent]]

Doc technique de référence de l'agent MMM : l'**agent** (pipeline ADK sur Vertex), son **intégration dans ConnectedHub** (backend + frontend), et l'**observabilité / debugging** en prod. C'est la source de vérité côté vault (le repo `Agent MMM` et son README étaient un scratch perso). **À maintenir quand le pipeline ou l'intégration évoluent.**

Le suivi produit & business (contexte, roadmap, pricing, personnes, décisions produit, jalons) reste dans le Project Brief [[MMM AI Agent]].

---

## 1. Vue d'ensemble

Assistant conversationnel dans le module MMM de ConnectedHub. Répond en langage naturel sur les données MMM d'un client (ROI/ROAS par média, comparaisons de périodes, contributions, allocation budget), sans passer par la console GCP.

Deux moitiés :

- **L'agent** — pipeline multi-agents Google ADK déployé sur **Vertex AI Agent Engine** (projet `med-dtam-prd-mg`, région `europe-west1`).
- **L'intégration ConnectedHub** — backend Node/Express + frontend React dans le projet `pmed-portal-prd-mg`, qui expose l'agent aux utilisateurs de Mine.

**Données** : BigQuery `med-dtam-prd-mg.mine.tb_model_contributions` (contribution hebdo / spend par variable marketing, par client et par semaine).

**Flux global** :

```
Utilisateur → module MMM (frontend React)
   → { message, session_id, ui_context } — le scope écran est joint à la question
   → backend Mine (agent.ts, stream SSE)
      → validation whitelist du ui_context, composition des lignes
        [MINE_CLIENT_ID] + [MINE_UI_CONTEXT]
      → Vertex Agent Engine (streamQuery, pipeline ADK)
      ← chunks (un par étape de sous-agent)
   ← SSE : status (progression) / done (réponse) / error
← réponse rendue en markdown dans le chatbot
```

> Le `ui_context` est capturé **à l'envoi**, pas en continu : une réponse prend ~47 s, l'utilisateur peut changer de KPI entre-temps. C'est pourquoi l'agent annonce le scope qu'il a utilisé, et sait l'expliquer si l'utilisateur signale un décalage.

---

## 2. Architecture de l'agent (ADK / Vertex Agent Engine)

### Pipeline multi-agents

```
MMM_agent — LlmAgent (Gemini 2.5 Flash) — gatekeeper
│   Évalue si la question est assez précise. Sinon demande une clarification.
│   Sinon transfert (transfer_to_agent) vers data_pipeline.
│
└── data_pipeline — SequentialAgent
      ├── context_setter_agent (Flash) → extract_and_set_client_context
      │     Regex Python extrait le client_id du préfixe [MINE_CLIENT_ID: …] → session state
      ├── schema_agent (Flash) → get_table_info  (schéma live de la table BQ)
      ├── discovery_agent (Flash) → run_discovery (SQL hardcodé, client_id paramétré)
      │     Vérités terrain : KPIs dispo, training_date par KPI, channels par KPI
      ├── refinement_loop — LoopAgent (max 3 itérations)
      │     ├── query_writer_agent (Gemini 2.5 Pro) — écrit le SQL (texte, aucun outil)
      │     ├── sql_guard — SequentialAgent
      │     │     ├── filter_checker_agent (Flash) → validate_client_filter
      │     │     │     Vérif Python : client_id = '<id>' présent (égalité stricte)
      │     │     └── executor_agent (Flash) → execute_sql (BigQueryToolset, WriteMode.BLOCKED)
      │     └── validator_agent (Flash) → exit_loop
      │           Résultat suffisant → exit_loop() ; sinon explique le manque → retry
      └── answer_agent (Gemini 2.5 Pro)
            Interprète les résultats validés, réponse NL, précise KPI + période
```

### Les sous-agents

| Agent | Modèle | Outil(s) | Rôle |
| --- | --- | --- | --- |
| `MMM_agent` | Flash | aucun | Gatekeeper : clarifie ou transfère |
| `context_setter_agent` | Flash | `extract_and_set_client_context` | Extrait le `client_id` (Python, pas le LLM) |
| `schema_agent` | Flash | `get_table_info` | Schéma live de la table |
| `discovery_agent` | Flash | `run_discovery` | Vérités terrain (KPIs, dates, channels) |
| `query_writer_agent` | **Pro** | aucun | Écrit le SQL (texte) |
| `filter_checker_agent` | Flash | `validate_client_filter` | Garde-fou isolation (Python) |
| `executor_agent` | Flash | `execute_sql` | Exécute le SQL validé (SELECT only) |
| `validator_agent` | Flash | `exit_loop` | Valide le résultat ou relance le loop |
| `answer_agent` | **Pro** | aucun | Synthèse en langage naturel |

**~9 à 17 appels LLM par question**, dont 2 en Gemini 2.5 Pro (`query_writer` + `answer`). Voir le détail coût en **§5 Coût technique** ci-dessous.

### Contexte métier scopé par sous-agent (20-22/07)

Le `BUSINESS_CONTEXT` monolithique a été **décomposé en blocs composables** (`CTX_INTRO`, `CTX_DISPLAY`, `CTX_DISCOVERY_NOTE`, `CTX_SQL_RULES`, `CTX_FIELDS`, `CTX_METRICS`, `CTX_KPI_SELECTION`), injectés **sélectivement** :

| Agent | Contexte injecté |
| --- | --- |
| `schema_agent` | **aucun** (`CTX_NONE`) + « exactly one tool: `get_table_info` » |
| `discovery_agent` | `CTX_NO_SQL` + « exactly one tool: `run_discovery` » |
| `query_writer_agent` | `CTX_FULL` (le **seul** avec `CTX_SQL_RULES`) |
| `validator_agent` | `CTX_NO_SQL` |
| `query_writer_agent` | `CTX_FULL` **+ `CTX_UI_SCOPE`** |
| `answer_agent` | `CTX_NO_SQL` **+ `CTX_MODULE_VIZ` + `CTX_UI_SCOPE`** |
| `root_agent` (`MMM_agent`) | `CTX_NO_SQL` **+ `CTX_MODULE_VIZ` + `CTX_UI_SCOPE`** |

- **`CTX_SQL_RULES`** + la syntaxe SQL (`SELECT MAX`, `SAFE_DIVIDE`) ne vont **qu'au `query_writer`** → retire l'amorce qui faisait halluciner les agents à toolset limité (cf. §6).
- **`CTX_MODULE_VIZ`** : bloc **bilingue EN/FR** (libellés sourcés des fichiers i18n) décrivant les graphes du module + leurs contrôles → l'agent aide à la **lecture des graphiques**. Générique (aucune variable client). Injecté `answer` + `root`.
- **`CTX_UI_SCOPE`** (23/07) : scope écran **live** de l'utilisateur → contrat détaillé en §3bis. Injecté `query_writer` (pour le filtre SQL par défaut), `answer` et `root`.
- **Root routing durci** : `root_agent` **délègue au `data_pipeline` dès qu'un chiffre est nécessaire** (ne répond seul que pour les questions pures d'interface) → évite le « chart-only » sur une question data.
- **`run_discovery`** exclut la ligne agrégée `name = 'Media total'` (faux canal). Il reste **volontairement non scopé** par le scope écran : il dit ce qui *existe* (KPIs, dates, canaux), donc le scoper casserait les réponses à « qu'est-ce que je peux regarder d'autre ? ».

### Isolation inter-client (défense en profondeur)

La table BQ contient tous les clients. L'isolation ne repose **pas** sur des tables séparées mais sur l'injection du `client_id` côté serveur + plusieurs couches Python non contournables par le LLM :

| Couche | Mécanisme | Force |
| --- | --- | --- |
| Backend Mine (`agent.ts`) | Préfixe `[MINE_CLIENT_ID: <id>]` au message, **toujours en première ligne** | **Hard** (serveur) |
| `extract_and_set_client_context` | Regex Python extrait le `client_id` — **leftmost match** | **Hard** |
| `run_discovery` | SQL hardcodé `WHERE client_id = @client_id` | **Hard** |
| `validate_client_filter` | Vérif Python avant toute exécution BQ | **Hard** |
| `sql_guard` (SequentialAgent) | Garantit checker avant executor | **Hard** |
| `query_writer_agent` | Instruction d'inclure `client_id` dans le SQL | Soft |

Isolation validée en test (07/07, Dan) : depuis Opel DE, une requête Peugeot a été refusée proprement. Guardrail P0.

**Ce qui donne réellement la garantie (précisé le 23/07)** : c'est le **leftmost match**, pas l'ancrage de la regex. Le backend émettant toujours sa ligne en premier, un marqueur forgé plus loin dans le texte utilisateur ne peut pas détourner l'extraction.

⚠️ Piège identifié en revue : **ancrer la regex en début de ligne *sans repli* est une régression**, pas un durcissement. Le `raw_message` est fourni **par un LLM** (`context_setter_agent` reçoit « le texte du premier message ») ; si Flash ajoute un préambule, des guillemets ou une espace, une regex ancrée seule échoue et renvoie une erreur fatale → **toutes** les questions data cassent, pas seulement celles avec un scope. Implémentation retenue : **ancrée, puis repli sur le motif large**.

### Déploiement

| | |
| --- | --- |
| Reasoning Engine (live) | `…/reasoningEngines/6241382153116975104` — display name `MMM_Agent_v3_live_ui_scope` |
| Framework | `google-adk` **pinné `==1.26.0`** — entrypoint `agent_engine_app.adk_app` |
| Service Account | `mmm-agent-sa@med-dtam-prd-mg.iam.gserviceaccount.com` |
| Région | `europe-west1` (RGPD) |
| Déploiement | `PYTHONUTF8=1 adk deploy agent_engine MMM_Agent --validate-agent-import` |

**Historique engines** : ~~`6073…`~~ (avril) → ~~`7899…`~~ (scoping, 20/07) → `2659…` (viz + fixes, 21-22/07, **rollback actuel**) → **`6241…` (scope écran live, 23/07, prod actuelle)**. `agent.ts` pointe sur `6241…`. Table engine↔commit dans le README du repo agent.

**Ménage du 23/07** : `6073` et `7899` **supprimés**. Chaque engine déployé facture son runtime en continu (~$35/mois même idle), donc laisser traîner des versions mortes coûte réellement. Il n'en reste que **deux** : la prod et son rollback. Règle : ne supprimer l'ancien qu'une fois le nouveau prouvé en monitoring.

⚠️ **Piège `display_name`** : `--display_name` n'est appliqué **qu'au moment où il est passé**. Un `adk deploy --agent_engine_id …` sans le flag retombe sur le `display_name` de `MMM_Agent/.agent_engine_config.json` et **écrase silencieusement** le nom versionné. Constaté le 23/07 : après l'update en place, l'engine v3 était redevenu `MMM_Agent`, recréant l'ambiguïté que le nommage versionné avait justement supprimée le 22/07. Le nom versionné vit maintenant dans le fichier de config, **à bumper à chaque nouvelle version**.

**Quand créer un nouvel engine plutôt qu'un update en place (23/07)** : dès que le changement touche **tout le trafic** et pas seulement la feature ajoutée, et que la vérification ne peut se faire qu'**après** déploiement. C'est le cas du correctif de regex `client_id` : un update en place de la prod aurait exposé tous les clients avant toute vérification. Nouvel engine → test en local dessus → cutover par PR. À l'inverse, un changement inerte sans la feature (ex. la règle de transparence) peut être poussé en place tant que la prod ne pointe pas encore dessus.

**Pièges de déploiement (appris 20-22/07)** :
- `adk deploy` lit le `requirements.txt` du **dossier agent** (`MMM_Agent/requirements.txt`), pas la racine → le pin va là. Sans pin, `google-adk 2.x` s'installe et **crash au démarrage** (`google.cloud.dataplex_v1` manquant).
- `--validate-agent-import` valide l'import **avant** de créer l'engine (échec rapide, pas d'engine mort).
- `PYTHONUTF8=1` : sinon le `✅` de fin plante sur la console Windows (cp1252) → « Deploy failed » alors que l'engine est bien créé.
- **Update en place** (sans re-pointer le backend) : `adk deploy … --agent_engine_id <id> --project med-dtam-prd-mg` — le `--project` explicite est **obligatoire** (sinon 404 mauvais projet).
- L'**ADC expire** (relogin `gcloud auth application-default login` d'un jour à l'autre).

---

## 3. Architecture d'intégration ConnectedHub

La partie **non** documentée par le repo agent : comment l'agent est branché dans Mine.

### Backend — `server/src/features/marketing-mix-modeling/routes/agent.ts`

Route **SSE** `POST /marketing-mix-modeling/agent` (`{ message, session_id, ui_context? }`). Étapes :

1. **Feature flag** — lecture Firestore `customers/{id}/products/marketing-mix-modeling` → champ `MMM_AI_Assistant`. Si absent/false → `403` (le frontend affiche la preview). C'est la feature payante.
2. **Auth Vertex** — `GoogleAuth` (`MASTER_CREDENTIALS`, scope `cloud-platform`) → token pour appeler l'Agent Engine (cross-project vers `med-dtam-prd-mg`).
3. **Session** — `getOrCreateVertexSession` : `Map` **en mémoire** clé `userId:session_id` → `adkSessionId` (via `async_create_session`). ⚠️ En mémoire → **perdue au redémarrage du backend** (cf. Bugs).
4. **Isolation + scope écran** — `composeAgentMessage(customerId, message, ui_context)` (module pur `routes/ui-context.ts`) compose :
   ```
   [MINE_CLIENT_ID: opel_fr]
   [MINE_UI_CONTEXT: kpi=…; kpi_label=…; period=2025-06-02..2026-05-11; tab=results; lang=fr]   (optionnelle)
   <question>
   ```
   Le `client_id` reste dérivé serveur, jamais fourni par l'utilisateur. Voir §3bis pour le contrat du scope.
5. **Streaming** — appel `:streamQuery` (`async_stream_query`), lecture du stream chunk par chunk :
   - À chaque changement d'`author` → envoi d'un event SSE `status` (mapping `AUTHOR_STATUS` / `FUNCTION_STATUS` → clés i18n) pour l'UX de progression.
   - Détection de la réponse finale (`isTerminal` + texte + pas de function_call + author `answer_agent`/`MMM_agent`) → event `done`.
   - Timeout **270 s** (AbortController) ; gestion du `client_disconnect` (`req.on('close')`).
   - **Filet retry (22/07)** : la tentative de stream est enveloppée dans une **boucle (1 retry, `MAX_ATTEMPTS=2`)** qui **ne retente que le `no_answer` transitoire** (pas les erreurs HTTP/stream, timeout, disconnect) — même session. Champs **`attempt` / `retried`** ajoutés aux logs `mmm_agent_exchange`. Logique de décision pure dans `routes/retry.ts` (`shouldRetryExchange`) + tests unitaires (vitest).
6. **Diagnostics & persistance** — sur chaque échange :
   - **GCS** `mmm-agent-chat-logs` (projet `med-dtam-prd-mg`), un JSON par échange, chemin `{clientId}/{ts}_{sessionId8}.json`. Schéma `ExchangeLog` : `timestamp, clientId, userId, sessionId, adkSessionId, userMessage, answer, status, durationMs, requestId` **+ `uiScope`, `uiScopePresent`** (23/07). `status ∈ success | error | timeout | no_answer | client_disconnect`.
   - **Cloud Logging** (projet `pmed-portal-prd-mg`) via `logger`, champ `metric: mmm_agent_exchange` + diagnostics (`authors`, `chunkCount`, `streamErrors`, `lastChunk`) sur les échecs. Depuis le 23/07 : **`uiScopePresent`**, **`uiScope`** (sanitizé, sur les échecs) et **`uiScopeRejected`** (champs refusés par la whitelist).

### 3bis. Contrat du scope écran (`ui_context`) — 23/07

**Ce que le front envoie** (tous les champs optionnels, l'objet entier peut être absent) :

| Champ | Source (`currentScopeWithSettings`) | Exemple | Usage |
| --- | --- | --- | --- |
| `kpi` | `name` | `Sell-out` | filtre SQL (scope contraignant) |
| `kpi_label` | `KPI_name` (traduit) | `Ventes` | nommer le KPI à l'utilisateur |
| `date_min` / `date_max` | **min/max réels** des lignes du scope | `2025-06-02` | filtre SQL (scope contraignant) |
| `tab` | route courante | `results` | déictique + navigation |
| `lang` | `i18next.language` | `fr` | langue de réponse |
| `market` / `market_label` | **conditionnel** : seulement si `Object.keys(scopes).length > 1` | `FR` | nommage uniquement, **jamais en SQL** |

⚠️ **Piège de nommage** : `currentScope.date_min` / `date_max` sont des **clés de semaine** (`2025-W23`) malgré leur nom, pas des dates. Ne jamais les envoyer telles quelles. Les vraies bornes se calculent depuis les `date` des lignes de `treatedResults.data`, où les DATE BigQuery arrivent en `{ value: 'YYYY-MM-DD' }` (même forme que `training_date`).

**Pourquoi des dates et pas des semaines** : la table n'a **ni colonne `year` ni colonne `week`** — elles sont dérivées côté ConnectedHub dans `get-data.ts` (`EXTRACT`). Et les deux bornes utilisaient deux définitions de semaine différentes (ISO vraie vs `EXTRACT(WEEK)` de BigQuery, dimanche/0-53), ce qui produisait des chiffres faux **sans erreur visible**. Passer en dates réelles supprime la question du contrat. A nécessité de ne plus exclure `date` du SELECT de `get-data.ts`.

**Sécurité** — `ui_context` **vient du navigateur**, donc non fiable, et atterrit juste à côté du garde-fou d'isolation :

- Whitelist stricte, **rejet** et non nettoyage : `^[\p{L}\p{N} _\-./&'(),%:+|]{1,64}$` pour les libellés, `^\d{4}-\d{2}-\d{2}$` pour les dates, énumérations pour `tab` et `lang`.
- Exclure `[`, `]`, `;`, `=` et les retours à la ligne rend un faux marqueur **structurellement inexprimable**. **C'est toute la propriété de sécurité, ne jamais la relâcher.**
- Rejet **champ par champ** : un `tab` invalide est ignoré, il ne casse jamais la conversation. Les champs rejetés sont loggés (sinon un rejet systématique de libellé serait invisible).
- Testé contre homoglyphes Unicode de crochets, caractères de contrôle, RTL override, astuces `toString`/prototype.

**Comportement de l'agent** (bloc `CTX_UI_SCOPE`) : scope **par défaut**, pas contrainte — la formulation explicite de l'utilisateur gagne toujours, **dimension par dimension**. Seule la ligne du **message courant** compte (les tours précédents sont de l'historique, pas une source de défaut). L'agent **annonce le scope sur sa 1re réponse** d'une conversation, puis se contente d'une courte incise.

> **Dégradation** : absent ou invalide → comportement strictement identique à l'avant-feature. Ce n'est pas un cas limite : `currentScope` est `undefined` tant que l'effet d'init n'a pas tourné alors que le chatbot est déjà joignable, donc les requêtes sans scope sont du **trafic normal**.

### Frontend — `client/src/features/marketing-mix-modeling/` + `routes/app/marketing-mix-modeling/`

- **`api/use-agent-mutation.ts`** — hook TanStack Query. `POST` SSE (`Accept: text/event-stream`, `responseType: text`), parsing du stream via `onDownloadProgress` d'axios (accumulation du `responseText`, découpage des lignes `event:` / `data:`). Events : `status` → callback `onStatus`, `done` → réponse, `error` → throw. Prend un argument optionnel `ui_context` transmis dans le body (23/07).
- **`context/data-context.tsx`** — le provider qui porte **à la fois** l'état du module (`currentScope`, `treatedResults`, `scopes`) **et** l'état du chat (`sendMessage`, `chatMessages`, `chatSessionId`). C'est ce qui rend la feature B triviale à câbler : le chatbot, même flottant, lit le scope sans plomberie. `buildUiContext()` y assemble le scope envoyé à chaque question. ⚠️ Le bloc mutation du chat a dû être **déplacé sous `currentScopeWithSettings`** (il était défini ~500 lignes avant la valeur qu'il doit lire).
- **`components/chatbot-floating.tsx`** — le chatbot **flottant** (reste visible en changeant de tab du module).
- **`components/markdown-message.tsx`** + `copy-button.tsx` — rendu markdown des réponses + copie.
- **`components/thinking-animation.tsx`** / `text-shimmer.tsx` + `hooks/use-loading-timer.ts` — animation de progression alimentée par les events `status` (montre l'étape en cours).
- **`components/input-message.tsx`** — saisie utilisateur ; bouton nouvelle conversation / continuité.
- **`routes/app/marketing-mix-modeling/chatbot.tsx`** — la vue chatbot ; **`settings.tsx`** — l'Admin Panel (toggle du flag payant).
- Autres livrés : photo de profil de l'utilisateur dans le chat, mémoire de conversation (même chat).

### Stockage & données

| Store | Usage |
| --- | --- |
| BigQuery `med-dtam-prd-mg.mine.tb_model_contributions` | Données modèle interrogées par l'agent |
| Firestore (`customers/{id}/products/…`) | Flag `MMM_AI_Assistant` (feature payante) |
| GCS `mmm-agent-chat-logs` | Log d'échange (1 JSON / question) |
| Cloud Logging `pmed-portal-prd-mg` | Logs backend + métrique `mmm_agent_exchange` |

### Décisions techniques clés

| Décision | Raison |
| --- | --- |
| Confidentialité via `client_id` injecté côté backend, pas via tables BQ séparées | Pas de tables BQ par client prévues — solution plus simple et centralisée |
| Chatbot flottant (suggestion Eddie) | Permet d'avoir l'assistant + les courbes visibles simultanément |

---

## 4. Observabilité & Debugging

Débloqué le 08/07 par le grant IAM sur le SA (cf. [[Session 2026-07-08]]).

### Observer — Trace Explorer

`https://console.cloud.google.com/traces/explorer?project=med-dtam-prd-mg`

- Tracing **intégré ADK** (`GOOGLE_CLOUD_AGENT_ENGINE_ENABLE_TELEMETRY=1`) → **un span par sous-agent** : `invocation` (run complet) → `invoke_agent <sous-agent>` → `call_llm` / `generate_content gemini-*` / `execute_tool`. ~48 spans / question, ~47 s pour une réponse normale.
- Les spans partent dans le **nouveau store `telemetry.googleapis.com`** (pas le Cloud Trace classique v1). Se lisent via le Trace Explorer, pas en API v1.
- ⚠️ Span status **toujours `UNSET`** (normal en OTEL). ADK ne stampe pas forcément `ERROR` sur un échec → **ne pas diagnostiquer par le status** mais par durée / waterfall / events.
- Bruit : les spans `/` viennent de `cf-budget-allocator-prod` (Cloud Function DS), pas de l'agent → filtrer `service.name = 6241382153116975104` (l'engine live). ⚠️ À mettre à jour à chaque cutover d'engine.

### Sources de diagnostic

| Source | Contenu | Rétroactif |
| --- | --- | --- |
| **Logs backend** (`pmed-portal-prd-mg`) | `authors` (où le pipeline meurt), `lastChunk`, `streamErrors`, `adkSessionId`, `status` | oui |
| **GCS** `mmm-agent-chat-logs` | Résumé de l'échange + `adkSessionId` (clé de jointure) | oui (index) |
| **Sessions API** (Agent Engine, clé `adkSessionId`) | Events internes : author par sous-agent, tool calls, SQL, réponses | **oui**, et **gratuit** |
| **Trace Explorer** | Timing / perf / structure des spans | non (à partir du grant) |

Requête Logs de triage :
```
jsonPayload.metric="mmm_agent_exchange" AND jsonPayload.status="no_answer"
```

Lecture des events d'une session (rétroactif) :
```
GET https://europe-west1-aiplatform.googleapis.com/v1beta1/projects/med-dtam-prd-mg/locations/europe-west1/reasoningEngines/6241382153116975104/sessions/<adkSessionId>/events
```

### Recette de diagnostic d'un `no_answer`

1. **Repérer** : requête Logs ci-dessus (ou l'alerte), récupérer `authors` + `adkSessionId`.
2. **Où ça meurt** : le champ `authors` (ex. `MMM_agent>context_setter_agent>schema_agent`) donne déjà le dernier sous-agent atteint.
3. **Confirmer / creuser** : Sessions API sur l'`adkSessionId` → events internes (rétroactif, même si la trace n'existe pas). Trace Explorer pour la perf (durée courte ~10 s vs ~47 s, waterfall tronqué).
4. Le `no_answer` transitoire (raté amont avalé) peut laisser le span `UNSET` : se fier à `lastChunk` / `streamErrors`.

### Alerte

- Alerte **log-based** `alertPolicies/527257283050504576` (*MMM agent — no_answer (log-based)*) → canal email `notificationChannels/5134431245708235828` (adajouin@publicisgroupe.net). Se déclenche sur chaque **ligne de log** `no_answer` (filtre `jsonPayload.metric="mmm_agent_exchange" AND jsonPayload.status="no_answer"`) et **extrait `authors` / `adkSessionId` / `customerId` dans la notification** → triage direct depuis le mail. Rate-limit 5 min (2 `no_answer` dans la même fenêtre → 1 notif).
- La log-based metric `mmm_agent_exchange` (DELTA/INT64, labels `status` + `customerId`) reste dispo pour dashboards/analytics.
- *Incidents Cloud Monitoring : notif à l'**ouverture** puis à la **résolution** (mail « resolved » quand la condition retombe — un `no_answer` étant ponctuel, chaque occurrence produit un couple open+resolved). Pour un strict 1-notif-par-événement (zéro merge en rafale) : sink Logging → Pub/Sub → Cloud Function (non fait, réservé si le volume grimpe).*

### Coût observabilité

Trace facturée au span : ~2,5 M spans/mois gratuits, puis ~$0,20/M. À ~48 spans/question → gratuit jusqu'à **~52 000 q/mois**. Usage « très intensif » (6 000 q/mois) reste 9× sous le seuil → **~€0**. Sampling non pertinent aux volumes actuels.

---

## 5. Coût technique (prod)

Projet GCP `med-dtam-prd-mg` **mutualisé** (pôle DTAM) — le Notebook (~$155/90j) et d'autres charges ne sont **pas** l'agent.

**Structure de coût** (~75% fixe) :

| Poste | Facturé quand | Montant |
| --- | --- | --- |
| **Agent Engine (runtime)** — Vertex AI, `MMM_Agent` europe-west1 | **fixe**, tant que l'agent est déployé (24/7, même idle) | ~$35/mois |
| **Tokens Gemini** | **variable**, par question | ~$0,05/question |
| Artifact Registry (images, partagé) + GCS + Scheduler + Secret Manager | continu | ~$6/mois |

**Coût par question ≈ $0,05** (jusqu'à ~$0,10 avec retries) : l'agent est un **pipeline multi-agents ADK** (7 sous-agents LLM principaux — 9 `LlmAgent` au total avec les gardes, cf. §2 —, **9 à 17 appels LLM/question**, dont **2 en `gemini-2.5-pro`** : `query_writer` + `answer`). Le contexte métier (blocs `CTX_*`, **scopé par sous-agent** — cf. §2) et les résultats BQ (jusqu'à 200 lignes) sont trimballés jusqu'aux appels finaux → ce sont les 2 appels Pro et le contexte accumulé qui dominent le coût.

**Coût par client, en prod** (1 client, base fixe portée entièrement par lui) :

| Usage | q/mois | Coût/mois |
| --- | --- | --- |
| Actif (~45/j) | 1 000 | ~$85 |
| Intensif (~135/j) | 3 000 | ~$185 |
| Très intensif (~270/j) | 6 000 | ~$335+ |

**Leviers d'optimisation** (baissent le $/question) : passer `answer_agent` de Pro à Flash ; réduire/agréger `max_query_result_rows=200` avant l'answer.

> **Écarté** : alléger le `BUSINESS_CONTEXT` pour économiser. Décision inverse — on l'**enrichit** (définitions variables/données) pour la qualité de réponse, cf. [[Synthèse & Roadmap Testing]]. La qualité prime sur ce micro-gain de coût.

*Note : dé-déployer l'Agent Engine ramène le fixe à ~$0 (redéploiement en quelques minutes).*

> Coût de l'**observabilité** (ingestion de traces) : voir §4 ci-dessus (~€0 aux volumes actuels).

---

## 6. Bugs & limites connus

| Sujet                           | État             | Détail                                                                                                                                                                                   |
| ------------------------------- | ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Sessions backend **en mémoire** | 🟠 limite connue | `Map` in-memory dans `agent.ts` → sessions perdues au redémarrage backend. La mémoire de conversation persistante par utilisateur est un chantier ouvert (cf. roadmap [[MMM AI Agent]]). |
| `context_setter_agent` dit « **premier** message » | 🟡 fragilité de wording | Son instruction demande « le texte du premier message utilisateur », alors que la règle RECENCY de `CTX_UI_SCOPE` exige **le message courant**. En pratique la récence marche (les agents lisent la ligne dans le transcript, et `ui_scope` en session state n'est lu par personne), **vérifié le 23/07**. Mais si un jour un tour répond sur un KPI périmé, c'est là qu'il faut regarder. Correction = une ligne. |
| `ui_scope` en session state **jamais lu** | 🟡 code mort utile | `extract_and_set_client_context` écrit `tool_context.state["ui_scope"]`, mais aucun agent ne lit cette clé (pas d'`output_key` ni de placeholder ADK). Le scope circule en réalité par le **transcript**. Inoffensif, mais ne pas croire que modifier cette clé change le comportement. |
| `currentScope` non typé (`any`) | 🟢 dette assumée | Assemblé dynamiquement depuis le payload BigQuery et lu dans ~20 endroits de `data-context.tsx`. Derrière un `eslint-disable` documenté. Le typer proprement est un chantier à part. |
| `__pycache__` **suivi par git** | 🟢 hygiène | `MMM_Agent/agent.cpython-313.pyc` est tracké dans le repo agent : sort dirty à chaque run et peut dériver dans un commit. À mettre en `.gitignore`. |

