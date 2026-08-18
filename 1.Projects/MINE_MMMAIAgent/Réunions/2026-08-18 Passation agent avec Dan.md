---
type: meeting
date: 2026-08-18
projet: MMM AI Agent
---

# 2026-08-18 Passation agent avec Dan

Projet : [[MMM AI Agent]]

Point d'onboarding 15h avec Dan. **Périmètre : la moitié « agent » uniquement** (pipeline ADK sur Vertex AI Agent Engine, projet `med-dtam-prd-mg`). La moitié « intégration ConnectedHub » (backend `agent.ts`, frontend React, feature flag, sessions) fera l'objet d'un point séparé avec Eddie.

**Contexte** : dernier jour de travail le 21/08. Trois jours ouvrés après ce point, donc tout ce qui demande une action de ma part doit sortir d'ici avec une date.

---

## Frontière agent / intégration

À poser en ouverture, c'est ce qui rend le découpage lisible. Le seul contrat entre les deux moitiés :

```
[MINE_CLIENT_ID: opel_fr]
[MINE_UI_CONTEXT: kpi=…; period=YYYY-MM-DD..YYYY-MM-DD; tab=…; lang=fr]   (optionnelle)
<question utilisateur>
```

composé côté backend Mine, envoyé en `streamQuery`, la réponse revenant en chunks (un par étape de sous-agent). Tout ce qui est **au-dessus** de ces trois lignes est du ressort d'Eddie. Tout ce qui est **en dessous** passe à Dan.

Conséquence à énoncer : le `client_id` est dérivé serveur et n'est jamais fourni par l'utilisateur. Si un jour l'agent change de source de vérité sur ce point, c'est une décision conjointe agent + intégration, pas une décision agent.

---

## Agenda proposé (60 min)

| Temps | Bloc | Objectif |
| --- | --- | --- |
| 5 min | Périmètre et frontière | Ce qui passe à Dan, ce qui attend Eddie |
| 10 min | Passation matérielle (accès, repo, alerte) | Rien ne casse le 22/08 |
| 15 min | Architecture du pipeline | Dan sait lire le code et le modifier |
| 10 min | Runbook déploiement | Dan sait déployer sans se faire piéger |
| 10 min | Runbook diagnostic | Dan sait traiter un `no_answer` seul |
| 10 min | Backlog, décisions ouvertes, qui porte quoi | Rien ne tombe dans le vide |

---

## 1. Passation matérielle (le bloc à ne pas rater)

C'est ce qui casse concrètement après le 21/08 si on ne le fait pas maintenant.

- [ ] **Repo du code agent**. Aujourd'hui le repo `Agent MMM` est un scratch personnel, la doc de référence étant la note [[Architecture technique]] du vault. Trancher pendant le point : où le code atterrit (repo d'équipe DS, ou repo ConnectedHub), et qui le pousse. **Sans ça, il n'y a plus de source pour redéployer.**
- [ ] **Alerte `no_answer`**. `alertPolicies/527257283050504576` notifie `notificationChannels/5134431245708235828`, c'est à dire `adajouin@publicisgroupe.net`. À rebasculer sur Dan (ou une alias d'équipe) avant le 21/08, sinon l'alerte part dans le vide.
- [ ] **Accès GCP** : projet `med-dtam-prd-mg`, région `europe-west1`. Vérifier avec Dan qu'il a bien les droits sur Vertex AI (Agent Engine), Cloud Logging, Trace Explorer, le bucket GCS `mmm-agent-chat-logs`, et BigQuery sur `mine.tb_model_contributions`.
- [ ] **Le SA `mmm-agent-sa@med-dtam-prd-mg`** et ses 7 rôles actuels (`aiplatform.user`, `bigquery.dataViewer`, `bigquery.jobUser`, `cloudtrace.agent`, `logging.logWriter`, `monitoring.metricWriter`, `telemetry.tracesWriter`). Rien de plus, et notamment **pas** de droit d'invoquer la Cloud Function d'allocation (cf. §5).
- [ ] **Les deux engines vivants**, et la règle qui va avec.

| Engine | Rôle | Display name |
| --- | --- | --- |
| `6241382153116975104` | Prod actuelle, scope écran live | `MMM_Agent_v3_live_ui_scope` |
| `2659…` | Rollback disponible sans redéploiement | viz + fixes (21-22/07) |

> Règle transmise : chaque engine déployé facture son runtime en continu (~$35/mois même idle). On garde **exactement deux** engines, la prod et son rollback, et on ne supprime l'ancien qu'une fois le nouveau prouvé en monitoring. `6073…` et `7899…` ont déjà été supprimés le 23/07 pour cette raison.

---

## 2. Architecture du pipeline (ce qu'il faut vraiment comprendre)

Support : [[Architecture technique]] §2. Trois choses à faire passer, le reste se lit.

**a. Le pipeline est une chaîne de 9 `LlmAgent`, pas un agent unique.**

```
MMM_agent (Flash, gatekeeper)
└── data_pipeline (Sequential)
      ├── context_setter_agent  → extraction client_id (Python)
      ├── schema_agent          → get_table_info
      ├── discovery_agent       → run_discovery (SQL hardcodé)
      ├── refinement_loop (max 3)
      │     ├── query_writer_agent (Pro) → écrit le SQL
      │     ├── sql_guard : filter_checker (Python) puis executor (SELECT only)
      │     └── validator_agent → exit_loop
      └── answer_agent (Pro) → réponse NL
```

9 à 17 appels LLM par question, dont 2 en Gemini 2.5 Pro. Ce sont ces 2 appels Pro et le contexte accumulé qui font le coût (~$0,05/question).

**b. Le contexte métier est scopé par sous-agent, ce n'est pas un prompt monolithique.** Les blocs `CTX_*` sont injectés sélectivement. Le point non intuitif à souligner : `CTX_SQL_RULES` ne va **qu'au** `query_writer`, parce que donner la syntaxe SQL aux agents à toolset limité les faisait halluciner. Toute évolution du contexte doit respecter ce découpage.

**c. L'isolation inter-client est une défense en profondeur Python, pas une confiance au LLM.** Backend qui préfixe toujours en première ligne, extraction regex en **leftmost match**, `run_discovery` hardcodé `WHERE client_id = @client_id`, `validate_client_filter` avant toute exécution BQ, `sql_guard` qui garantit l'ordre checker puis executor. Validé en test par Dan lui-même le 07/07 (depuis Opel DE, une requête Peugeot refusée proprement).

> Piège à transmettre explicitement, parce qu'il ressemble à un durcissement alors que c'est une régression : **ancrer la regex `client_id` en début de ligne sans repli casse tout**. Le `raw_message` est fourni par un LLM (Flash peut ajouter un préambule ou des guillemets), et une regex ancrée seule échoue alors en erreur fatale sur **toutes** les questions data. L'implémentation retenue est : ancrée, **puis repli** sur le motif large. Ce qui donne la garantie de sécurité, c'est le leftmost match, pas l'ancrage.

---

## 3. Runbook déploiement

Commande de référence :

```
PYTHONUTF8=1 adk deploy agent_engine MMM_Agent --validate-agent-import
```

Update en place (sans re-pointer le backend) :

```
adk deploy … --agent_engine_id <id> --project med-dtam-prd-mg
```

Les pièges, tous appris à la dure entre le 20 et le 23/07 :

| Piège | Effet si oublié |
| --- | --- |
| `google-adk` pinné `==1.26.0` dans **`MMM_Agent/requirements.txt`** (pas à la racine) | `google-adk 2.x` s'installe et crash au démarrage (`google.cloud.dataplex_v1` manquant) |
| `--validate-agent-import` | Engine mort créé au lieu d'un échec rapide |
| `PYTHONUTF8=1` | Le `✅` final plante sur console Windows (cp1252), « Deploy failed » alors que l'engine est créé |
| `--project` explicite sur un update en place | 404, mauvais projet |
| `--display_name` n'est appliqué **que** quand il est passé | Un update sans le flag retombe sur `.agent_engine_config.json` et écrase silencieusement le nom versionné. Le nom versionné vit dans ce fichier, à bumper à chaque version |
| ADC expire d'un jour à l'autre | `gcloud auth application-default login` |

**Règle nouvel engine vs update en place** : créer un nouvel engine dès que le changement touche **tout le trafic** et pas seulement la feature ajoutée, et que la vérification ne peut se faire qu'après déploiement. Exemple type : le correctif de regex `client_id`, où un update en place aurait exposé tous les clients avant toute vérification. À l'inverse, un changement inerte tant que la prod ne pointe pas dessus peut être poussé en place.

---

## 4. Runbook diagnostic

Quatre sources, dont deux rétroactives et gratuites, ce qui est le point important.

| Source | Contenu | Rétroactif |
| --- | --- | --- |
| Logs backend (`pmed-portal-prd-mg`) | `authors` (où le pipeline meurt), `lastChunk`, `streamErrors`, `adkSessionId`, `status` | oui |
| GCS `mmm-agent-chat-logs` | Résumé d'échange, `adkSessionId` comme clé de jointure | oui |
| **Sessions API** (clé `adkSessionId`) | Events internes : author par sous-agent, tool calls, SQL | **oui, et gratuit** |
| Trace Explorer | Timing, waterfall, structure des spans (~48 spans, ~47 s/question) | non |

Triage :

```
jsonPayload.metric="mmm_agent_exchange" AND jsonPayload.status="no_answer"
```

Recette : repérer via les logs, lire `authors` pour savoir **où** ça meurt, confirmer via la Sessions API sur l'`adkSessionId`, et n'aller au Trace Explorer que pour la perf.

> Deux avertissements à donner. Le span status est **toujours `UNSET`** (normal en OTEL, ADK ne stampe pas `ERROR`), donc ne jamais diagnostiquer par le status. Et le filtre du Trace Explorer est `service.name = 6241382153116975104`, **à mettre à jour à chaque cutover d'engine**, sinon on lit le bruit de `cf-budget-allocator-prod`.

---

## 5. Backlog agent qui passe à Dan

Ce qui reste ouvert **côté agent** (le reste est côté Mine, donc Eddie).

- [ ] **Enrichir le `BUSINESS_CONTEXT`** (définitions des variables et des données du modèle). C'était déjà identifié comme un sujet Dan/Hajar dans la roadmap testing, donc c'est le point de reprise le plus naturel. Décision déjà prise et à ne pas rouvrir : on **enrichit** le contexte pour la qualité de réponse, on ne l'allège pas pour économiser des tokens.
- [ ] **`cf-budget-allocator-prod` comme tool.** Cadrage complet dans [[Tool - cf-budget-allocator]], rien n'est implémenté. À transmettre en priorité :
  - Décision du 24/07 : **prototyper A (l'agent invoque la CF) et B (logique embarquée)** puis trancher sur pièces. Mon analyse penche A (un seul solveur partagé avec l'onglet Simulation, donc chiffres non divergents par construction, et pas de JAX dans un déploiement déjà fragile). Séquencer A d'abord.
  - **Prérequis bloquant** : `roles/run.invoker` sur le seul service `cf-budget-allocator-prod`. Surtout **ne pas** demander `cloudfunctions.developer` (ce que porte le SA du backend Mine), qui permettrait aussi de déployer et modifier des functions.
  - Trois pièges qui produisent des chiffres faux **sans erreur** : tout est **positionnel** (`bounds`, `inflations`, la réponse), `inflations` est en **pourcents** et non en ratio, et `t` est un **indice de départ** et non une durée. L'ordre des médias se dérive de `tb_model_configuration` (dernier `training_date` pour le triplet `client_id`/`level`/`kpi`), donc il ne se devine pas et le LLM ne doit jamais composer ces tableaux.
  - À signaler aux DS quelle que soit l'option : la CF **interpole le `client_id` directement dans le SQL** sans paramètre, exécuté avec le SA `internal@`. Sans risque tant que la valeur vient du backend, mais ça rend « injecter depuis le session state » obligatoire et pas seulement recommandé.
- [ ] **Explorer le knowledge par client** (contexte secteur, concurrents, cadrage par `client_id`). Chantier v2, converge avec la couche pédagogique et l'ADN des leviers, à traiter d'un bloc et pas en silos.

**Dette et fragilités connues côté agent**, à mentionner sans s'y attarder :

- Le `context_setter_agent` dit « premier message » dans son instruction alors que la règle de récence exige le **message courant**. En pratique ça marche (vérifié le 23/07), mais si un tour répond sur un KPI périmé, c'est là qu'il faut regarder. Correction : une ligne.
- `ui_scope` écrit en session state n'est **lu par personne**, le scope circule par le transcript. Inoffensif, mais ne pas croire que modifier cette clé change le comportement.
- `__pycache__` tracké par git dans le repo agent, à mettre en `.gitignore` au moment du transfert du repo.

---

## 6. Ce qui ne passe PAS à Dan (à dire explicitement)

Pour qu'il ne se retrouve pas à porter des sujets qui ne sont pas les siens.

| Sujet | Qui | Pourquoi |
| --- | --- | --- |
| Backend `agent.ts`, SSE, feature flag Firestore, sessions in-memory | Eddie | Intégration ConnectedHub |
| Chip de contexte dans le chatbot, mémoire persistante par utilisateur | Eddie | Frontend et session côté Mine |
| **Cadrage produit « recommande vs faits »** (option A deux niveaux, option B assistant pur) | Baptiste | Décision structurante, elle conditionne le périmètre du tool d'allocation. Toujours non tranchée |
| Ouverture au conseil DE Stellantis (Zenith Media, silencieux depuis le mail deck) | Katia, Inès | Jalon terrain, pas technique |
| Pricing et positionnement de l'add-on | Baptiste, Jules | Hors agent |

---

## Questions ouvertes à poser à Dan pendant le point

1. Où atterrit le code de l'agent, et qui le pousse ? (bloquant, tout le reste en dépend)
2. Qui reçoit l'alerte `no_answer` à partir du 22/08 ?
3. Est ce qu'il reprend le sujet en propre, ou en binôme avec Hajar ? Ça change à qui je transmets quoi dans les trois jours qui restent.
4. Est ce qu'on cale un second point court avant le 21/08 pour un déploiement fait par lui, avec moi en observateur ? C'est le seul moyen de vérifier que le runbook passe.

---

## Compte-rendu

_À compléter après le point._
