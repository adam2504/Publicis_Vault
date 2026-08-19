---
type: note
projet: MMM AI Agent
---
# 5. MMM AI Agent

## Introduction

The MMM AI Agent is a conversational assistant embedded in the Marketing Mix Modeling module of ConnectedHub. It answers questions about a client's MMM results in natural language, such as ROAS per media, contributions, period comparisons or saturation, without requiring the user to read a chart or ask a data strategist.

The agent is a multi-agent pipeline built with the Google Agent Development Kit (ADK) and deployed on Vertex AI Agent Engine. It queries the same BigQuery data that feeds the Results page described in *3. Integration in Mine*.

> ConnectedHub is the current name of the platform previously called Mine. Older pages in this space, and several technical identifiers such as the `MINE_CLIENT_ID` prefix and the `mine` BigQuery dataset, still carry the former name. They refer to the same platform.

It is a paid add-on: access is controlled by a feature flag per client. Clients without the flag see a preview instead of the assistant.

*Screenshot of an example of the AI Assistant tab:*

## Scope of this page

The agent has two halves, documented separately:

* **This page** covers the agent itself: the ADK pipeline, its prompts, its security model, and how to deploy, monitor and troubleshoot it. It lives in the `med-dtam-prd-mg` project.
* ***3. Integration in Mine*** covers the ConnectedHub interface. The backend route, the React frontend, the feature flag and the session handling belong there.

The boundary between the two halves is the format of the message the agent receives:

```
[MINE_CLIENT_ID: opel_fr]
[MINE_UI_CONTEXT: kpi=…; kpi_label=…; period=2025-06-02..2026-05-11; tab=results; lang=fr]
Which media has the best ROAS?
```

The first line is mandatory and carries the client identifier, derived server side from the authenticated session and never supplied by the user. The second line is optional and describes what the user had on screen when they sent the message. Everything below is the question.

Changing this format requires a coordinated change on both sides. A one sided change breaks the other half silently.

## Environment

| | |
| --- | --- |
| GCP project | `med-dtam-prd-mg` |
| Region | `europe-west1` (GDPR constraint) |
| Framework | `google-adk`, pinned to `1.26.0` |
| Runtime | Vertex AI Agent Engine (Reasoning Engine) |
| Service account | `mmm-agent-sa@med-dtam-prd-mg.iam.gserviceaccount.com` |
| Source table | `med-dtam-prd-mg.mine.tb_model_contributions` |
| Repository | `github.com/Publicis-Media-France-FR5140/MMM_AI_Agent` |
| Engine in production | `6241382153116975104`, display name `MMM_Agent_v3_live_ui_scope` |
| Rollback engine | `2659…`, available without redeployment |

## Architecture

The agent is not a single model with a large prompt. It is a chain of nine `LlmAgent` instances, each with a narrow role and a restricted set of tools.

```
MMM_agent (Flash) — gatekeeper and router
│   Decides whether the question is precise enough. If not, it asks for
│   clarification. If it is, it transfers to data_pipeline.
│
└── data_pipeline (SequentialAgent)
      ├── context_setter_agent (Flash) → extract_and_set_client_context
      │     Python regex, writes client_id to session state
      ├── schema_agent (Flash) → get_table_info
      │     Live schema of the BigQuery table
      ├── discovery_agent (Flash) → run_discovery
      │     Hardcoded SQL. Ground truth: available KPIs, training_date per
      │     KPI, channels per KPI
      ├── refinement_loop (LoopAgent, max 3 iterations)
      │     ├── query_writer_agent (Pro) — writes SQL, has no tools
      │     ├── sql_guard (SequentialAgent)
      │     │     ├── filter_checker_agent (Flash) → validate_client_filter
      │     │     └── executor_agent (Flash) → execute_sql (WriteMode.BLOCKED)
      │     └── validator_agent (Flash) → exit_loop
      └── answer_agent (Pro) — natural language synthesis
```

A single question triggers 9 to 17 LLM calls, two of which run on `gemini-2.5-pro` (`query_writer_agent` and `answer_agent`). A normal answer takes about 47 seconds.

Three design choices are structural and should be understood before modifying anything:

1. **`context_setter_agent` is the first step of a `SequentialAgent`**, not a tool the root agent has to remember to call. Ordering is therefore enforced by the framework rather than by an instruction a model may ignore. This is what guarantees the client identifier is in session state before any query runs.
2. **Writing SQL is separated from executing it.** `query_writer_agent` has no tools and produces text. This is what allows `filter_checker_agent` to validate the query before it reaches BigQuery.
3. **`sql_guard` is a `SequentialAgent`**, which guarantees the checker runs before the executor. Structural again, not instructional.

### Business context scoping

The business knowledge given to the agent is not one monolithic block. It is split into composable blocks (`CTX_INTRO`, `CTX_DISPLAY`, `CTX_DISCOVERY_NOTE`, `CTX_SQL_RULES`, `CTX_FIELDS`, `CTX_METRICS`, `CTX_KPI_SELECTION`), assembled into three profiles and injected selectively.

| Agent | Context injected |
| --- | --- |
| `schema_agent` | none (`CTX_NONE`), plus "exactly one tool: `get_table_info`" |
| `discovery_agent` | `CTX_NO_SQL`, plus "exactly one tool: `run_discovery`" |
| `query_writer_agent` | `CTX_FULL` and `CTX_UI_SCOPE` |
| `validator_agent` | `CTX_NO_SQL` |
| `answer_agent` | `CTX_NO_SQL`, `CTX_MODULE_VIZ` and `CTX_UI_SCOPE` |
| `root_agent` | `CTX_NO_SQL`, `CTX_MODULE_VIZ` and `CTX_UI_SCOPE` |

The rule to preserve: `CTX_SQL_RULES` and SQL syntax go to `query_writer_agent` only. Giving SQL rules to an agent that has no execution tool gives it the material to hallucinate both a query and its results. This split is the fix for an actual hallucination problem observed in July 2026, not a stylistic preference.

Two blocks deserve a note:

* **`CTX_MODULE_VIZ`** describes the module's charts and controls, tab by tab, so the agent can help users read them. It is bilingual, with labels sourced from the application i18n files, and generic, with no client specific variables.
* **`CTX_UI_SCOPE`** describes how to use the on screen scope. Two rules matter most: the scope is a **default and not a constraint**, so anything the user states explicitly always wins, dimension by dimension; and only the line on the **current message** counts, earlier turns being history rather than a source of defaults.

`run_discovery` is deliberately left outside the on screen scope. It reports what **exists** for the client, so scoping it would break answers to questions such as "what else can I look at?".

## Client isolation

The BigQuery table holds every client. Isolation does not rely on separate tables. It relies on server side injection of the client identifier plus several Python layers the model cannot bypass.

| Layer | Mechanism | Strength |
| --- | --- | --- |
| ConnectedHub backend | Prefixes `[MINE_CLIENT_ID: <id>]`, always as the first line | Hard, server side |
| `extract_and_set_client_context` | Python regex, leftmost match | Hard |
| `run_discovery` | Hardcoded `WHERE client_id = @client_id` | Hard |
| `validate_client_filter` | Python check before any BigQuery call | Hard |
| `sql_guard` | `SequentialAgent` enforcing checker before executor | Hard |
| `query_writer_agent` | Instructed to include `client_id` in the SQL | Soft |

Tested on 7 July 2026: from an Opel DE session, a query targeting Peugeot was rejected cleanly.

### What actually provides the guarantee

The guarantee comes from the **leftmost match**, not from the anchoring of the regex. The backend always emits its line first, so the first match found in the message is always the server derived value. A forged marker placed further down in user text cannot hijack the extraction.

The code holds two patterns:

```python
CLIENT_ID_RE = re.compile(r'^\[MINE_CLIENT_ID:\s*([^\]\n]+)\]\s*$', re.MULTILINE)
CLIENT_ID_RE_FALLBACK = re.compile(r'\[MINE_CLIENT_ID:\s*([^\]]+)\]')
```

**Removing the fallback is a regression, not a hardening.** The raw message is not read off the wire directly. It is passed along by `context_setter_agent`, which is an LLM asked to forward the text of the user message verbatim. A model may add a preamble, wrap the line in quotes, or leave leading whitespace, none of which the anchored pattern tolerates. Without the fallback, extraction returns a fatal error and **every** data question fails, not only those carrying a scope. The anchor is defence in depth on top of the leftmost match. It is not the source of the guarantee. This behaviour is locked by a test in `tests/test_ui_scope.py`.

### The on screen scope is untrusted input

`ui_context` originates in the browser and lands next to the isolation guard, so it is validated in the ConnectedHub backend with a strict whitelist that **rejects** rather than sanitises:

* Labels must match `^[\p{L}\p{N} _\-./&'(),%:+|]{1,64}$`, dates must match `^\d{4}-\d{2}-\d{2}$`, and `tab` and `lang` are closed enumerations.
* Excluding `[`, `]`, `;`, `=` and newlines makes a forged marker structurally inexpressible. This is the entire security property and must not be relaxed.
* Rejection is field by field, so an invalid `tab` is ignored and never breaks the conversation. Rejected fields are logged, otherwise a systematic rejection would be invisible.

## Repository and tests

```
MMM_Agent/
  agent.py                     the whole pipeline, the CTX_* blocks and the tools
  agent_tester.py              local query script
  requirements.txt             this is the file adk deploy reads
  .env                         project, region and telemetry flags, no secrets
  .agent_engine_config.json    display name, description, service account
tests/                         35 tests, pytest
```

```bash
python -m pytest tests -q
```

The tests are not decorative. They lock properties that a prompt change would break without raising an error. `test_context_scoping.py` checks that each interpreter receives its intended context profile, `test_ui_scope.py` locks the leftmost match and the nine scope rules, and `test_module_viz.py` checks that the root agent still delegates data questions despite carrying the visualization block.

The docstrings of `extract_and_set_client_context`, `run_discovery` and `validate_client_filter` are read by the ADK and sent to the model as the tool descriptions. Editing them changes the agent's behaviour. They are not ordinary comments.

## Deployment

Refresh application default credentials first. They expire from one day to the next, and this is the first thing to check when a command fails for no apparent reason.

```bash
gcloud auth application-default login
```

Create a new engine:

```bash
PYTHONUTF8=1 adk deploy agent_engine MMM_Agent --validate-agent-import
```

Update an existing engine in place:

```bash
adk deploy … --agent_engine_id <id> --project med-dtam-prd-mg
```

### Pitfalls

All of the following were observed in production between 20 and 23 July 2026.

| Pitfall | What happens if missed |
| --- | --- |
| `google-adk` must stay pinned to `1.26.0` in `MMM_Agent/requirements.txt`, not the root one | `adk deploy` reads the agent folder's requirements. Without the pin, version 2.x is installed and the engine crashes on startup with a missing `google.cloud.dataplex_v1` |
| `--validate-agent-import` | Validates the import before creating the engine. Without it, a dead engine is created and has to be found and deleted later |
| `PYTHONUTF8=1` | On a Windows console in cp1252, the final character breaks the output and the command reports "Deploy failed" although the engine was created. Leads to pointless redeployments and orphan engines |
| Explicit `--project` on an in place update | Without it, the call 404s against the wrong project |
| `--display_name` only applies when passed | An update without the flag falls back to the display name in `.agent_engine_config.json` and silently overwrites the versioned name. The versioned name therefore lives in that file and must be bumped for each new version |

### New engine or in place update

Create a **new engine** whenever the change affects all traffic rather than only the feature being added, and whenever verification is only possible after deployment.

The reference case is the `client_id` regex fix: updating production in place would have exposed every client before any verification was possible. The correct sequence there is a new engine, local testing pointed at it, then a cutover through a pull request on the ConnectedHub side.

Conversely, a change that is inert until production points at it can safely be pushed in place.

### Engine lifecycle

Keep exactly two live engines, production and its rollback. Delete the old one only once the new one has proven itself in monitoring.

This is not cosmetic. Every deployed engine bills its runtime continuously, around $35 per month even when idle. Engines `6073…` and `7899…` were deleted on 23 July 2026 for this reason.

## Monitoring and troubleshooting

### Sources

| Source | Content | Retroactive |
| --- | --- | --- |
| Backend logs (`pmed-portal-prd-mg`) | `authors`, `lastChunk`, `streamErrors`, `adkSessionId`, `status` | yes |
| GCS `mmm-agent-chat-logs` (`med-dtam-prd-mg`) | One JSON per exchange, with `adkSessionId` as the join key | yes |
| Agent Engine Sessions API | Internal events: author per sub-agent, tool calls, SQL, answers | yes, and free |
| Trace Explorer | Timing, waterfall, structure. About 48 spans per question | no |

Backend logs live in a **different GCP project** from the agent, which is why troubleshooting requires access to both projects.

### Diagnosing a `no_answer`

1. Locate the exchange, from the email alert or from this logging query, then collect `authors` and `adkSessionId`:
   ```
   jsonPayload.metric="mmm_agent_exchange" AND jsonPayload.status="no_answer"
   ```
2. Read `authors`. A value such as `MMM_agent>context_setter_agent>schema_agent` names the last sub-agent reached, which is where the pipeline died.
3. Query the Sessions API on the `adkSessionId` for the internal events. This works even when no trace exists.
   ```
   GET https://europe-west1-aiplatform.googleapis.com/v1beta1/projects/med-dtam-prd-mg/locations/europe-west1/reasoningEngines/6241382153116975104/sessions/<adkSessionId>/events
   ```
4. Use Trace Explorer for performance only. A short duration, around 10 seconds instead of 47, with a truncated waterfall, indicates an upstream failure.

Two warnings. Span status is always `UNSET`, which is normal OpenTelemetry behaviour since the ADK does not stamp `ERROR` on failure, so never diagnose from the status. And the Trace Explorer filter is `service.name = 6241382153116975104`, which must be updated at every engine cutover, otherwise the view shows noise from the `cf-budget-allocator` cloud function instead.

### Alerting

A log based alert policy named *MMM agent — no_answer (log-based)* fires on every `no_answer` log line and extracts `authors`, `adkSessionId` and `customerId` into the notification, so triage can start from the email itself. Rate limited to one notification per five minutes.

The log based metric `mmm_agent_exchange` (DELTA/INT64, labelled by `status` and `customerId`) remains available for dashboards.

## Cost

The cost structure is roughly 75% fixed.

| Item | Nature | Amount |
| --- | --- | --- |
| Agent Engine runtime | Fixed, billed continuously even when idle | about $35 per month |
| Gemini tokens | Variable, per question | about $0.05, up to $0.10 with retries |
| Artifact Registry, GCS, Scheduler, Secret Manager | Continuous | about $6 per month |

Per client, with a single client carrying the entire fixed base: about $85 per month at 1 000 questions, $185 at 3 000, and $335 or more at 6 000.

Levers if cost becomes a concern: move `answer_agent` from Pro to Flash, or reduce `max_query_result_rows=200` before the synthesis call.

Trimming the business context to save tokens has been explicitly ruled out. The decision goes the other way: it is being enriched to improve answer quality.

Tracing is free at current volumes. About 2.5 million spans per month are included, which covers roughly 52 000 questions.

## Known issues and limitations

| Item | Severity | Detail |
| --- | --- | --- |
| Backend sessions held in memory | Known limitation | The ConnectedHub backend keeps sessions in an in memory map, so they are lost when the backend restarts. Persistent per user memory is an open item. This sits on the ConnectedHub side, not the agent side |
| `context_setter_agent` says "first message" | Wording fragility | Its instruction asks for the text of the **first** user message, while the recency rule of `CTX_UI_SCOPE` requires the **current** one. It works in practice, verified on 23 July 2026, because agents read the line from the transcript. If an answer ever uses a stale KPI, this is where to look. The fix is one line |
| `ui_scope` in session state is never read | Dead code | `extract_and_set_client_context` writes `tool_context.state["ui_scope"]`, but no agent reads that key. The scope actually travels through the transcript. Harmless, but do not expect changing this key to change behaviour |
| Market column | Documented inaccuracy | `level` is the market key across the system: `cf-budget-allocator` reads `level = request_json["market"]`, and the frontend groups its scopes by `level`. The description in `CTX_FIELDS`, "granularity of the analysis", is incomplete at best. Leaving `market` out of the agent's SQL rules remains correct, since its table is `tb_model_contributions` and no multi market client exists today, but this must be revisited when one does |

## Roadmap

* **Enrich the business context** with variable and data definitions from the data science team. Testing showed that users need to know what a variable *means* and why a performance moved, more than they need another number. Highest value item in the backlog.
* **Add `cf-budget-allocator-prod` as an agent tool** (the deployed service behind the "Optimize allocation" button described in *3. Integration in Mine*), covering saturation and budget allocation questions. Nothing implemented yet. Three traps produce wrong numbers without raising an error: `bounds`, `inflations` and the response are **positional** arrays with no media names, `inflations` is expressed in **percent** rather than as a ratio, and `t` is a **start index** rather than a duration. The media order is derived from `tb_model_configuration` and must never be composed by the model. Requires `roles/run.invoker` on that single service.
* **Per client knowledge**: sector, competitors and client specific framing. This converges with the media expertise and the educational layer needed before opening the module to clients, and should be treated as one project rather than three.
* **Access decision, pending**: how far the agent is allowed to recommend rather than state facts. This is structural and gates the budget allocation tool, since launching an optimization is a recommendation rather than a fact.

## Note for the DTAM team

`cf-budget-allocator` interpolates the client identifier directly into its SQL, without parameters, and runs with the `internal@med-dtam-prd-mg` service account.

```python
WHERE client_id = '{client_id}' AND level = '{level}' AND kpi = '{kpi}'
```

This is safe today because the value always comes from the ConnectedHub backend. It would become an exploitable SQL injection the day a value originating from a model reached that payload, which is why injecting the client identifier from session state is a requirement rather than a good practice.

## Contacts

Agent POC: Dan Phan (danphan2@publicisgroupe.net)

ConnectedHub POC: Eddie Ratignier (eddratig@publicisgroupe.net)

Original author: Adam Jouini, September 2024 to September 2026. Built the agent, wrote this page, then left. Gone, unlike the fallback regex. Please keep it that way. ([linkedin.com/in/adam-jouini](https://www.linkedin.com/in/adam-jouini))
