# Tool `cf-budget-allocator` — cadrage intégration

Projet : [[MMM AI Agent]]

Note de préparation (23/07) pour l'item de backlog **« GCF as tools »** : brancher la Cloud Function d'optimisation budget comme outil de l'agent. Rien n'est encore implémenté — ce document rassemble le contrat, les contraintes IAM et les questions à trancher avant de coder.

> Contrat **lu dans la source de la CF** (`gs://run-sources-med-dtam-prd-mg-europe-west1/services/cf-budget-allocator-prod/…zip` → `main.py`), pas déduit de l'appelant. C'est la référence.

## La ressource

| | |
| --- | --- |
| Service | `cf-budget-allocator-prod` — **service Cloud Run**, pas une Cloud Function enregistrée (`gcloud functions describe` renvoie 404). Déployée depuis source, base `python312`, function-target `main` |
| URL prod | `https://cf-budget-allocator-prod-868239246901.europe-west1.run.app` |
| URL dev | `https://cf-budget-allocator-dev-868239246901.europe-west1.run.app` |
| Région | `europe-west1` |
| SA d'exécution | `internal@med-dtam-prd-mg` (c'est **elle** qui lit BigQuery, pas l'appelant) |
| Auth appelant | **Token OIDC** (`getIdTokenClient`), pas un access token |
| Ressources | 1 vCPU / 1 Gi, `maxScale: 5` |
| Déployée par | brilhost@publicisgroupe.net, janvier 2026 |

> Une variante **dev** existe : idéale pour développer et tester le tool sans toucher la prod.

> `maxScale: 5` + un solveur JAX : ce n'est pas un endpoint à marteler. Si l'agent peut le déclencher, prévoir que ça reste un appel rare.

## Contrat réel (relevé dans le code, pas supposé)

Appelé aujourd'hui par `server/src/features/marketing-mix-modeling/routes/optimize.ts`, lui-même appelé depuis l'onglet Simulation (`components/response-per-media-section/optimize-modal.tsx`).

**Payload envoyé à la CF :**

```json
{
  "t": <temporal>,
  "kpi": "<kpi>",
  "client_id": "<client>",
  "market": "<market>",
  "inflations": [0.02, 0, 0.05, ...],
  "budget": <budget_total>,          // mode "budget"
  "bounds": [[mins...], [maxs...]]
}
```

En mode **`goal`** (« Atteindre un objectif »), deux différences :
- `target_goal` remplace `budget`
- les bornes hautes sont **écrasées à `1e9`** : `[[...mins], [...maxs.map(() => 1000000000)]]`, donc de fait non bornées vers le haut.

**Réponse** : `number[]` — un budget par média, positionnel.

### Ce que la source apprend (et que l'appelant ne disait pas)

- **`market` est en fait le `level` du modèle.** La CF fait `level = request_json["market"]` puis interroge `tb_model_configuration WHERE client_id AND level AND kpi`. **Ça referme la question laissée ouverte le 23/07** sur le nom de la colonne marché : il n'y a pas de colonne « market », c'est `level`.
- **`inflations` est en POURCENTS**, pas en ratio : la CF calcule `1.0 + inflations / 100`. Envoyer `5` pour 5 %, surtout pas `0.05`. Erreur silencieuse garantie sinon.
- **`t` est un `t_start`**, un indice de départ dans la timeline du modèle, pas une durée. Le front envoie `historicDates.length - noHistoricDates.length + 1`.
- **Exactement un** de `budget` / `target_goal`. Les deux ensemble → `ValueError: Allocation to reach a target with a budget is not yet supported`. Aucun des deux → `ValueError` aussi.
- **La CF va chercher le `model_configuration` elle-même** dans BigQuery (dernier `training_date`). L'agent n'a donc **pas** à fournir la config du modèle, seulement le triplet `client_id` / `market` / `kpi`.
- Champs requis : `client_id`, `market`, `kpi`, `t`, `bounds`, `inflations`. Il n'y a **aucune valeur par défaut côté CF** : les défauts (`min 0`, `max = budget`, `inflation 0`) sont une convention du **front**, à réimplémenter côté agent si on veut le même comportement.
- Le solveur vient de `dtam==0.1.12`, un package privé d'Artifact Registry → **impossible de répliquer l'optimisation en local**, il faut appeler la CF.

**Origine de chaque paramètre côté UI :**

| Param | Origine | Défaut si l'utilisateur ne remplit pas |
| --- | --- | --- |
| `temporal` | `historicDates.length - noHistoricDates.length + 1` (hook `useDates`) | — calculé, jamais saisi |
| `kpi` / `market` | `currentScope.name` / `currentScope.market` | — |
| `bounds[0]` (mins) | contrainte min par média | `0` |
| `bounds[1]` (maxs) | contrainte max par média | **le budget total** |
| `inflations` | inflation par média | `0` |
| `budget_total` | saisie utilisateur | `1` |
| `what` | toggle des deux modes | `budget` |

## Les points durs

### 1. Tout est **positionnel** — c'est le vrai risque

`bounds`, `inflations` et la **réponse** sont des tableaux positionnels. Rien dans le payload ne nomme les médias. Ordre différent = le budget de Search atterrit sur la TV : **des chiffres faux, sans erreur**. Même classe de bug que la définition de semaine trouvée le 23/07 (cf. [[Session 2026-07-23]]).

**Bonne nouvelle : l'ordre est dérivable, pas à deviner.** La CF lit `model_configuration` depuis `tb_model_configuration` (dernier `training_date` pour le triplet `client_id`/`level`/`kpi`). L'agent peut interroger **exactement la même ligne** pour reconstruire l'ordre des médias en Python. Donc : ne jamais laisser le LLM composer ces tableaux, les assembler depuis la même source que la CF, et vérifier la longueur avant l'appel.

### 2. `t` (le `t_start`) n'existe pas côté agent

Il vient de la logique de dates du frontend (`useDates`, `historicDates` / `noHistoricDates`). L'agent n'a pas cet objet. Options : recalculer côté Python depuis la config du modèle, ou le faire passer par le `ui_context`.

> La piste `ui_context` est tentante puisque le tuyau existe depuis la feature B, mais ce serait la **première** valeur du scope écran qui sert à *calculer* et non à *filtrer*. À peser, et ça rendrait le tool dépendant du front.

### 3. Isolation `client_id` — et une injection SQL en embuscade

La CF prend un `client_id` en payload. Même règle que partout : **injecté en Python depuis le session state**, jamais fourni par le LLM. Patron de `run_discovery`.

⚠️ Raison supplémentaire, trouvée dans la source : la CF interpole le `client_id` **directement dans le SQL**, sans paramètre :

```python
WHERE client_id = '{client_id}' AND level = '{level}' AND kpi = '{kpi}'
```

Aujourd'hui c'est sans risque, la valeur vient du backend Mine. Mais si un jour un `client_id` (ou un `kpi`, ou un `market`) issu du LLM atteignait ce payload, ce serait une **injection SQL exploitable**, exécutée avec le SA `internal@med-dtam-prd-mg`. Ça élève le « injecter depuis le session state » du niveau bonne pratique au niveau **obligation**.

### 4. IAM à demander — bloquant, à lancer en premier

Vérifié côté GCP :

- La policy IAM **du service est vide**, et **personne n'a `roles/run.invoker`** au niveau projet.
- `mmm-agent-sa@med-dtam-prd-mg` a 7 rôles (`aiplatform.user`, `bigquery.dataViewer`, `bigquery.jobUser`, `cloudtrace.agent`, `logging.logWriter`, `monitoring.metricWriter`, `telemetry.tracesWriter`) — **aucun ne permet d'invoquer la CF**.
- Si le backend Mine y arrive aujourd'hui, c'est via `interface@pmed-portal-prd-mg` qui porte **`roles/cloudfunctions.developer`** sur le projet.

**Ne pas demander le même rôle pour l'agent** : `cloudfunctions.developer` permettrait aussi de déployer et modifier des functions. La demande juste est **`roles/run.invoker` sur le seul service `cf-budget-allocator-prod`**. Même type de démarche que le grant de tracing obtenu le 08/07.

## Questions produit à trancher

- **Est-ce que l'agent doit pouvoir *lancer* une optimisation, ou seulement *lire* et expliquer une allocation existante ?** Lancer une optimisation, c'est produire une recommandation budgétaire, pas répondre à un fait. Ça touche directement l'item de roadmap ouvert « jusqu'où l'agent recommande vs se limite aux faits », à trancher avec Baptiste. **À clarifier avant de coder**, sinon on livre une capacité que le cadrage produit refusera peut-être.
- Si oui : les **deux modes** (ventiler un budget / atteindre un objectif) ou seulement le premier ?
- Que fait l'agent des **contraintes** ? Les demander à l'utilisateur en langage naturel est une conversation à plusieurs tours, ce que le pipeline actuel ne fait pas. Version simple : n'accepter que les défauts (min 0, max = budget, inflation 0) et renvoyer vers l'onglet Simulation dès qu'il faut des contraintes fines.
- Le coût : un appel CF + un tour LLM de plus. Marginal face aux ~$0,05/question, mais à vérifier si l'optimisation devient fréquente.

## Détail relevé au passage

`mapOptimizeFormData` renvoie un `target_name` (`Pick<..., 'target_name' | ...>`) que **le backend ne lit pas** : `optimize.ts` ne déstructure que `temporal, kpi, budget_total, inflations, bounds, market, what`. La clé est envoyée puis silencieusement ignorée, et elle n'existe pas non plus sur `BudgetAllocationOptimizeParams` — probablement une des erreurs TS préexistantes du client. Sans impact, mais à ne pas reproduire dans le tool.

## Prochaine étape

Brainstorm de cadrage : périmètre (lire vs lancer), puis spec. Ne pas commencer par le code — le point bloquant est le grant IAM et la question produit, pas la plomberie.
