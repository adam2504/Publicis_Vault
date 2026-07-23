# Tool `cf-budget-allocator` — cadrage intégration

Projet : [[MMM AI Agent]]

Note de préparation (23/07) pour l'item de backlog **« GCF as tools »** : brancher la Cloud Function d'optimisation budget comme outil de l'agent. Rien n'est encore implémenté — ce document rassemble le contrat, les contraintes IAM et les questions à trancher avant de coder.

> Contrat **lu dans la source de la CF** (`gs://run-sources-med-dtam-prd-mg-europe-west1/services/cf-budget-allocator-prod/…zip` → `main.py`), pas déduit de l'appelant. C'est la référence.

## La ressource

| | |
| --- | --- |
| Service | `cf-budget-allocator-prod` (Cloud Run, gen2) |
| URL prod | `https://cf-budget-allocator-prod-868239246901.europe-west1.run.app` |
| URL dev | `https://cf-budget-allocator-dev-868239246901.europe-west1.run.app` |
| Région | `europe-west1` |
| Auth | **Token OIDC** (`getIdTokenClient`), pas un access token |

> Une variante **dev** existe : idéale pour développer et tester le tool sans toucher la prod.

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

**Réponse** : `number[]` — un budget par média.

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

`bounds`, `inflations` et la **réponse** sont des tableaux alignés sur l'ordre de `model[0].media`. Rien dans le payload ne nomme les médias. Si l'agent construit ces tableaux dans un ordre différent de celui qu'attend la CF, il attribue silencieusement le budget de Search à la TV : **des chiffres faux, sans erreur**.

C'est exactement la même classe de bug que la définition de semaine trouvée le 23/07 (cf. [[Session 2026-07-23]]). À traiter avec la même méthode : ne pas faire reconstruire l'ordre par le LLM, le dériver en Python depuis une source unique, et le vérifier.

### 2. `temporal` n'existe pas côté agent

Il vient de la logique de dates du frontend (`useDates`, `historicDates` / `noHistoricDates`). L'agent n'a pas cet objet. Trois options à évaluer : recalculer côté Python depuis les données du modèle, le faire passer par le `ui_context` (le front le connaît déjà), ou le dériver dans la CF elle-même.

> La piste `ui_context` est tentante puisque le tuyau existe depuis la feature B, mais attention : ce serait la **première** valeur du scope écran qui sert à *calculer* et non à *filtrer*. À peser.

### 3. Isolation `client_id`

La CF prend un `client_id` en payload. Même règle que partout ailleurs : **injecté en Python depuis le session state**, jamais fourni par le LLM. Reprendre le patron de `run_discovery`.

### 4. IAM à demander

L'agent tourne sous `mmm-agent-sa@med-dtam-prd-mg`. Pour invoquer la CF il lui faut **`roles/run.invoker`** sur le service Cloud Run, et savoir émettre un **token OIDC** (pas le token d'accès utilisé pour BigQuery). C'est un grant à demander, comme celui du tracing obtenu le 08/07. **À anticiper : c'est le seul point bloquant qui ne dépend pas de nous.**

## Questions produit à trancher

- **Est-ce que l'agent doit pouvoir *lancer* une optimisation, ou seulement *lire* et expliquer une allocation existante ?** Lancer une optimisation, c'est produire une recommandation budgétaire, pas répondre à un fait. Ça touche directement l'item de roadmap ouvert « jusqu'où l'agent recommande vs se limite aux faits », à trancher avec Baptiste. **À clarifier avant de coder**, sinon on livre une capacité que le cadrage produit refusera peut-être.
- Si oui : les **deux modes** (ventiler un budget / atteindre un objectif) ou seulement le premier ?
- Que fait l'agent des **contraintes** ? Les demander à l'utilisateur en langage naturel est une conversation à plusieurs tours, ce que le pipeline actuel ne fait pas. Version simple : n'accepter que les défauts (min 0, max = budget, inflation 0) et renvoyer vers l'onglet Simulation dès qu'il faut des contraintes fines.
- Le coût : un appel CF + un tour LLM de plus. Marginal face aux ~$0,05/question, mais à vérifier si l'optimisation devient fréquente.

## Détail relevé au passage

`mapOptimizeFormData` renvoie un `target_name` (`Pick<..., 'target_name' | ...>`) que **le backend ne lit pas** : `optimize.ts` ne déstructure que `temporal, kpi, budget_total, inflations, bounds, market, what`. La clé est envoyée puis silencieusement ignorée, et elle n'existe pas non plus sur `BudgetAllocationOptimizeParams` — probablement une des erreurs TS préexistantes du client. Sans impact, mais à ne pas reproduire dans le tool.

## Prochaine étape

Brainstorm de cadrage : périmètre (lire vs lancer), puis spec. Ne pas commencer par le code — le point bloquant est le grant IAM et la question produit, pas la plomberie.
