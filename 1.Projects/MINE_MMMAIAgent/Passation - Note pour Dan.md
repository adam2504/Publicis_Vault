---
type: note
projet: MMM AI Agent
---

> Note de travail : contenu destiné à **Dan**, à lui envoyer tel quel (mail ou message). Copier à partir du titre ci-dessous, sans cette frontmatter ni ce bloc.

# Agent MMM — ce que tu reprends

Salut Dan,

Je pars le 21/08, donc voici tout ce qu'il te faut pour reprendre l'agent sans avoir à me relancer. C'est volontairement court : la doc technique complète est sur Confluence, ici je te donne l'état réel, l'ordre dans lequel attaquer, et les choses que tu ne devinerais pas.

---

## 1. Ce que tu reprends, et ce que tu ne reprends pas

L'agent a deux moitiés. **Tu reprends la première.**

**À toi** : le pipeline ADK déployé sur Vertex AI Agent Engine, dans le projet `med-dtam-prd-mg`. Le code, les prompts, le déploiement, le diagnostic, le contexte métier.

**À l'équipe ConnectedHub (Eddie)** : la route backend qui appelle l'agent, le frontend React, le feature flag payant, la gestion des sessions. Un point séparé est prévu avec lui.

La frontière entre les deux, c'est le format du message que tu reçois :

```
[MINE_CLIENT_ID: opel_fr]
[MINE_UI_CONTEXT: kpi=…; period=2025-06-02..2026-05-11; tab=results; lang=fr]
Quel est mon meilleur média en ROAS ?
```

Tout ce qui est en dessous de ces lignes est ton périmètre. Tout ce qui les produit est celui d'Eddie. Si un jour ce format doit changer, c'est une décision à prendre à deux, pas unilatéralement : un changement d'un côté casse l'autre en silence.

---

## 2. L'état réel au 19/08

L'agent **tourne en production** et est utilisé. Ce n'est pas un prototype.

| | |
| --- | --- |
| Engine en prod | `6241382153116975104`, display name `MMM_Agent_v3_live_ui_scope` |
| Engine de rollback | `2659…`, disponible sans redéploiement |
| Dernière mise en prod | 23/07/2026 |
| Repo | `github.com/Publicis-Media-France-FR5140/MMM_AI_Agent` |
| Tests | 35, tous verts |

Ce qui est livré et fonctionne : isolation inter-client (que tu as toi-même validée le 07/07), aide à la lecture des graphiques, scope écran live, observabilité complète avec alerte mail, retry automatique sur les `no_answer` transitoires.

Ce qui est ouvert, dans l'ordre où je m'y remettrais : enrichir le contexte métier avec les définitions de variables, brancher l'allocation budget comme outil, puis le knowledge par client.

---

## 3. Par quoi commencer, dans cet ordre

1. **Demander tes accès tout de suite.** C'est le plus long et ça bloque tout le reste. La liste précise est dans la page Confluence. Attention, il y a **deux projets GCP** : `med-dtam-prd-mg` pour l'agent, et `pmed-portal-prd-mg` pour les logs de diagnostic. Le second passe par l'IT ou par Eddie, c'est le circuit le plus lent, lance le en premier.
2. **Cloner le repo et lancer les tests** (`python -m pytest tests -q`). Si les 35 passent, ton environnement est bon.
3. **Lire `MMM_Agent/agent.py` en entier.** Tout l'agent tient dans ce fichier, prompts compris. Une heure de lecture te donne 90% du sujet.
4. **Faire un déploiement pendant que je suis encore là.** C'est le seul point de cette liste qui a une date : si tu bloques sur un piège de déploiement après le 21/08, tu perdras une journée à le retrouver seul. Si tu ne dois faire qu'une chose avec moi, fais celle là.

---

## 4. Les pièges qui coûtent une journée

Ceux là ne se devinent pas, ils m'ont tous coûté du temps.

- **`roles/iam.serviceAccountUser` sur `mmm-agent-sa`.** Sans ce rôle, `adk deploy` échoue en `PERMISSION_DENIED` alors que tu as `aiplatform.user` et que tout semble en ordre. Le message d'erreur ne pointe pas vers la bonne cause.
- **`google-adk` doit rester pinné `==1.26.0`, dans `MMM_Agent/requirements.txt`** et pas celui de la racine. `adk deploy` lit celui du dossier agent. Sans le pin, la 2.x s'installe et l'engine crash au démarrage sur un module manquant.
- **`PYTHONUTF8=1` devant `adk deploy`.** Sinon la commande affiche « Deploy failed » sur console Windows **alors que l'engine a bien été créé**. Tu redéploies pour rien et tu accumules des engines fantômes.
- **Chaque engine facture environ $35/mois même inutilisé.** Règle : exactement deux engines vivants, la prod et son rollback. On ne supprime l'ancien qu'une fois le nouveau prouvé.
- **Le span status est toujours `UNSET` dans le Trace Explorer**, y compris sur un échec. Ne diagnostique jamais par le status, regarde `authors`, `lastChunk` et `streamErrors` dans les logs.
- **Ne supprime pas le repli de la regex `client_id`** dans `agent.py`. Il ressemble à une redondance, il ne l'est pas : le message t'arrive via un LLM qui peut ajouter un préambule ou des guillemets, et sans le repli **toutes** les questions data cassent. La garantie de sécurité vient du leftmost match, pas de l'ancrage. C'est détaillé sur Confluence et verrouillé par un test.

---

## 5. Ce qui va casser tout seul si personne n'y touche

- **Mon compte disparaît le 04/09.** L'alerte `no_answer` a déjà été rebasculée sur toi (tu es destinataire depuis le 19/08), mais mon adresse est encore rattachée en parallèle. Elle doit être retirée avant cette date.
- **L'ADC expire d'un jour à l'autre.** Quand une commande gcloud échoue sans raison apparente, `gcloud auth application-default login` est le premier réflexe, avant de chercher plus loin.
- **Le filtre du Trace Explorer contient l'ID de l'engine** (`service.name = 6241382153116975104`). À mettre à jour à chaque bascule d'engine, sinon tu lis le bruit d'une Cloud Function qui n'a rien à voir.

---

## 6. Qui voir pour quoi

- **Eddie** : tout ce qui touche au backend, au frontend, au feature flag, aux sessions. Et l'accès au projet `pmed-portal-prd-mg`.
- **Baptiste** : la décision produit non tranchée, jusqu'où l'agent a le droit de recommander plutôt que de s'en tenir aux faits. C'est structurant, ça conditionne le périmètre de l'outil d'allocation budget. À relancer, ça traîne depuis juillet.
- **Hajar** : la connaissance métier MMM et l'enrichissement du contexte, c'est un sujet que vous portiez déjà à deux.
- **Inès et Katia** : l'ouverture aux équipes conseil sur le MMM Stellantis. Sujet terrain, pas technique. Le conseil DE ne répond plus depuis le mail deck, à relancer.

---

## 7. Mon conseil sur le fond

Deux choses, si tu ne devais en retenir que ça.

**Le prochain gain n'est pas technique, il est dans le contexte métier.** L'agent sait déjà chercher et calculer. Ce qui lui manque, c'est de savoir ce que les variables *veulent dire* et de relier une performance à son pourquoi. C'est remonté par tous les profils testés, côté data strat comme côté conseil. C'est exactement ton terrain, et c'est le meilleur rapport effort sur valeur du backlog.

**Ne code pas l'outil d'allocation budget avant que Baptiste ait tranché.** Si la décision tombe sur « assistant pur, aucune recommandation », cet outil ne pourra que lire une allocation existante, jamais en proposer une. Coder avant, c'est risquer de livrer une capacité que le cadrage produit refusera.

---

## 8. Ce qui n'est pas ton problème

Pour que tu ne repartes pas en pensant porter le produit entier : le pricing et le positionnement de l'add-on, l'ouverture commerciale aux clients, la mémoire persistante des conversations et le chip de contexte dans le chatbot (côté ConnectedHub), et l'arbitrage produit sur la recommandation. Tu portes l'agent, pas l'offre.

---

Bonne reprise, et n'hésite pas à me pinger sur LinkedIn si un truc reste obscur après mon départ.

Adam
