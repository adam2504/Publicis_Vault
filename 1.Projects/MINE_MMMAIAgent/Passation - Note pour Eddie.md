---
type: note
projet: MMM AI Agent
---

> Note de travail : contenu destiné à **Eddie**, à lui envoyer tel quel. Copier à partir du titre ci-dessous, sans cette frontmatter ni ce bloc.

# Agent MMM — passation de la partie ConnectedHub

Salut Eddie,

Je pars le 21/08. Dan reprend l'agent lui même, c'est à dire le pipeline ADK sur Vertex dans `med-dtam-prd-mg`. Toi tu récupères tout ce qui est dans le repo ConnectedHub. Ce document couvre uniquement cette moitié là.

Je vais directement au fond, tu connais le repo mieux que moi.

---

## 1. La frontière avec Dan

Elle est nette et tient en trois lignes, celles que `composeAgentMessage` produit :

```
[MINE_CLIENT_ID: opel_fr]
[MINE_UI_CONTEXT: kpi=…; kpi_label=…; period=2025-06-02..2026-05-11; tab=results; lang=fr]
<question utilisateur>
```

Ce qui produit ces lignes est à toi. Ce qui les consomme est à Dan. Le format lui même est le contrat : si l'un des deux le change seul, l'autre casse **sans erreur visible**, juste avec des réponses fausses. C'est le seul endroit où il faut absolument se parler.

---

## 2. La surface de code

### Backend, `server/src/features/marketing-mix-modeling/routes/`

| Fichier | Rôle |
| --- | --- |
| `agent.ts` (609 l.) | La route SSE `POST /marketing-mix-modeling/agent`. Tout le flux y est |
| `ui-context.ts` (111 l.) | Module pur : validation du scope écran et composition du message. Aucun I/O |
| `ui-context.test.ts` | Tests de la whitelist, homoglyphes Unicode et caractères de contrôle compris |
| `retry.ts` (22 l.) | Décision de retry, pure. `shouldRetryExchange` et `MAX_ATTEMPTS` |
| `retry.test.ts` | Tests de cette décision |

La logique décidable est volontairement sortie de `agent.ts` dans deux modules purs testables. Ce qui reste dans `agent.ts` est du flux et de l'I/O, donc non couvert unitairement, c'est assumé.

### Frontend, `client/src/features/marketing-mix-modeling/`

| Fichier | Rôle |
| --- | --- |
| `api/use-agent-mutation.ts` (53 l.) | Hook TanStack. POST SSE, parsing du stream via `onDownloadProgress` d'axios |
| `context/data-context.tsx` (678 l.) | Porte **à la fois** l'état du module et l'état du chat. `buildUiContext()` y est, vers la ligne 575 |
| `components/chatbot-floating.tsx` (351 l.) | Le chatbot flottant, visible sur tous les onglets du module |
| `components/markdown-message.tsx`, `copy-button.tsx` | Rendu markdown et copie des réponses |
| `components/thinking-animation.tsx`, `text-shimmer.tsx`, `hooks/use-loading-timer.ts` | Progression alimentée par les events `status` |
| `components/input-message.tsx` | Saisie, bouton nouvelle conversation |
| `routes/app/marketing-mix-modeling/chatbot.tsx`, `settings.tsx` | La vue chatbot, et le toggle du flag payant dans l'Admin Panel |

---

## 3. Le point le plus important : l'ID de l'engine est en dur

```ts
const AGENT_RESOURCE =
  'projects/868239246901/locations/europe-west1/reasoningEngines/6241382153116975104';
```

`agent.ts`, ligne 19. **C'est le point de bascule de tout le système.**

Concrètement : quand Dan déploie une nouvelle version de l'agent, il crée un nouvel engine et le teste, mais c'est **toi** qui fais passer la prod dessus en changeant cette constante et en la déroulant par PR. Tant qu'elle n'a pas bougé, rien ne change pour les utilisateurs, quoi que Dan ait déployé.

Corollaire utile : **le rollback est une PR d'une ligne.** L'engine précédent est encore vivant et fonctionnel.

```
reasoningEngines/2659050124520456192   ← rollback, version du 22/07
reasoningEngines/6241382153116975104   ← prod actuelle, version du 23/07
```

Si l'agent part en vrille un jour, tu n'as besoin de personne côté Vertex : tu repointes sur `2659…` et tu déploies. Pas de redéploiement d'agent, pas d'attente.

Dan a pour consigne de garder exactement deux engines vivants, la prod et son rollback, parce que chaque engine facture environ $35 par mois même inutilisé.

---

## 4. Le flux, en une passe

1. **Feature flag.** Lecture Firestore `customers/{id}/products/marketing-mix-modeling`, champ `MMM_AI_Assistant`. Absent ou faux, c'est un `403` et le front affiche la preview. C'est la feature payante.
2. **Auth Vertex.** `GoogleAuth` avec `MASTER_CREDENTIALS`, scope `cloud-platform`. Appel cross-projet vers `med-dtam-prd-mg`.
3. **Session.** `getOrCreateVertexSession` : `Map` en mémoire, clé `userId:session_id` vers `adkSessionId`.
4. **Isolation et scope.** `composeAgentMessage(customerId, message, ui_context)` compose les trois lignes. Le `client_id` reste dérivé serveur, jamais fourni par le client.
5. **Streaming.** Appel `:streamQuery`, lecture chunk par chunk. À chaque changement d'`author`, un event SSE `status` part vers le front (mapping `AUTHOR_STATUS` et `FUNCTION_STATUS` vers des clés i18n) pour l'animation de progression. La détection de la réponse finale est `isTerminal` plus du texte, pas de `function_call`, et un author `answer_agent` ou `MMM_agent`. Timeout 270 s via `AbortController`, et `req.on('close')` gère le disconnect.
6. **Retry.** Toute la tentative est enveloppée dans une boucle à un seul retry (`MAX_ATTEMPTS = 2`), **sur la même session**. Elle ne retente **que** le `no_answer` transitoire : ni les erreurs HTTP, ni les erreurs de stream, ni le timeout, ni le disconnect. La raison est commentée dans `retry.ts` : un sous-agent hallucine parfois l'outil `execute_sql`, l'ADK renvoie un 498 et le stream se termine vide. C'est non déterministe, la tentative fautive meurt en quelques secondes, et un seul re-run passe presque toujours.

---

## 5. Sécurité : la propriété à ne jamais relâcher

`ui_context` **vient du navigateur**, donc n'est pas fiable, et il atterrit juste à côté du garde-fou d'isolation inter-client. C'est la seule partie du code que je te demanderais de relire avant d'y toucher.

La validation est dans `ui-context.ts`, en **whitelist stricte avec rejet**, pas en nettoyage :

- Libellés : `^[\p{L}\p{N} _\-./&'(),%:+|]{1,64}$`. Dates : `^\d{4}-\d{2}-\d{2}$`. Énumérations fermées pour `tab` et `lang`.
- **Exclure `[`, `]`, `;`, `=` et les retours à la ligne rend un faux marqueur `[MINE_CLIENT_ID: …]` structurellement inexprimable.** C'est toute la propriété de sécurité. Élargir la whitelist pour faire passer un libellé client exotique, c'est la casser.
- Le rejet est **champ par champ** : un `tab` invalide est ignoré et ne casse jamais la conversation. Les champs rejetés partent dans les logs sous `uiScopeRejected`, sinon un rejet systématique de libellé serait invisible.

Testé contre homoglyphes Unicode de crochets, caractères de contrôle, RTL override et astuces `toString` et prototype. Tout est dans `ui-context.test.ts`.

Côté agent, la garantie repose sur le **leftmost match** : le backend émet toujours sa ligne `[MINE_CLIENT_ID: …]` en premier, donc la première occurrence trouvée dans le message est toujours la valeur serveur. Tant que tu gardes cette ligne en tête de message, l'isolation tient.

---

## 6. Le piège des dates

`currentScope.date_min` et `date_max` sont des **clés de semaine** (`2025-W23`) malgré leur nom, pas des dates. Ne jamais les envoyer telles quelles dans le scope.

Les vraies bornes se calculent depuis les `date` des lignes de `treatedResults.data`, ce que fait `buildUiContext()`. Deux subtilités y sont commentées : les lignes ne sont pas garanties triées, donc min et max sont calculés explicitement, et les DATE BigQuery arrivent soit en string nue soit en `{ value: 'YYYY-MM-DD' }`, donc les deux formes sont gérées et tout le reste est jeté, période entière omise plutôt que date malformée envoyée.

Pourquoi des dates réelles et pas des semaines : la table n'a **ni colonne `year` ni colonne `week`**, elles sont dérivées côté ConnectedHub dans `get-data.ts` par `EXTRACT`. Et les deux bornes utilisaient deux définitions de semaine différentes, ISO vraie d'un côté et `EXTRACT(WEEK)` de BigQuery de l'autre (dimanche, 0-53). Ça produisait des chiffres faux **sans lever la moindre erreur**. Le passage en dates réelles supprime la question. Ça a nécessité de ne plus exclure `date` du SELECT de `get-data.ts`.

---

## 7. Observabilité

Deux destinations, sur chaque échange :

- **GCS** `mmm-agent-chat-logs` (projet `med-dtam-prd-mg`), un JSON par échange, chemin `{clientId}/{ts}_{sessionId8}.json`. Schéma : `timestamp, clientId, userId, sessionId, adkSessionId, userMessage, answer, status, durationMs, requestId, uiScope, uiScopePresent`. Avec `status ∈ success | error | timeout | no_answer | client_disconnect`.
- **Cloud Logging** (projet `pmed-portal-prd-mg`), champ `metric: mmm_agent_exchange`, plus sur les échecs `authors`, `chunkCount`, `streamErrors`, `lastChunk`, `uiScope` sanitizé, `uiScopeRejected`, et `attempt` et `retried` depuis le filet de retry.

Le champ qui sert vraiment est **`authors`** : il donne la chaîne des sous-agents atteints, donc l'endroit exact où le pipeline est mort. C'est la première chose à regarder sur un `no_answer`, et c'est aussi ce que tu transmets à Dan quand le problème est de son côté.

Une alerte log-based existe sur `status="no_answer"` et notifie Dan depuis le 19/08. Elle extrait `authors`, `adkSessionId` et `customerId` directement dans le mail.

---

## 8. Dette connue, de ton côté

- **Sessions en mémoire.** `const vertexSessions = new Map<string, string>()` dans `agent.ts`. Perdues au redémarrage du backend, donc l'utilisateur perd le fil de sa conversation en cours sans comprendre pourquoi. C'est aussi ce qui bloque la mémoire persistante par utilisateur.
- **`currentScope` non typé (`any`).** Assemblé dynamiquement depuis le payload BigQuery et lu dans une vingtaine d'endroits de `data-context.tsx`, derrière un `eslint-disable` documenté. Le typer proprement est un chantier à part que je n'ai pas ouvert.
- **`data-context.tsx` porte deux responsabilités**, l'état du module et l'état du chat. C'est ce qui a rendu le scope écran trivial à câbler, le chatbot flottant lisant le scope sans plomberie, mais le fichier fait 678 lignes. À noter : le bloc mutation du chat a dû être **déplacé sous `currentScopeWithSettings`**, il était défini environ 500 lignes avant la valeur qu'il doit lire.
- **`mapOptimizeFormData` renvoie un `target_name` que le backend ne lit pas.** `optimize.ts` ne déstructure que `temporal, kpi, budget_total, inflations, bounds, market, what`. La clé est envoyée puis silencieusement ignorée, et elle n'existe pas non plus sur `BudgetAllocationOptimizeParams`. Sans impact, probablement une des erreurs TS préexistantes du client, mais autant que tu le saches.

---

## 9. Ce qui reste ouvert de ton côté

- **Mémoire persistante par utilisateur**, pour reprendre une ancienne conversation. Suppose de sortir les sessions de la `Map`.
- **Chip de contexte dans le chatbot**, du type `Contexte : ROAS · juin 2025 – mai 2026`. Rend le scope visible en permanence, ne coûte aucun token, et signale un décalage avant même que l'utilisateur lise la réponse. C'est le complément visuel de l'annonce que l'agent fait déjà en texte sur sa première réponse. Petit chantier, bon rapport valeur sur effort.

---

## 10. Ce qui n'est pas à toi

Pour situer : le pipeline ADK, les prompts, le contexte métier, le déploiement des engines et le diagnostic des `no_answer` côté agent sont à Dan (`danphan2@publicisgroupe.net`). L'arbitrage produit sur ce que l'agent a le droit de recommander est chez Baptiste, et il n'est toujours pas tranché.

Une doc technique complète de la partie agent est sur Confluence, page **5. MMM AI Agent**, à côté de ta page *3. Integration in Mine*. Elle décrit le pipeline, l'isolation et les runbooks de déploiement et de diagnostic, si tu as besoin de comprendre ce qui se passe de l'autre côté du contrat.

---

Merci pour tout ce que tu m'as appris sur ce repo, j'y suis arrivé sans savoir écrire une ligne de React.

Adam
