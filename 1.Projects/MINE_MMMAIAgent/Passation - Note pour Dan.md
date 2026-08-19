---
type: note
projet: MMM AI Agent
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
2. **Lire `MMM_Agent/agent.py` en entier.** Tout l'agent tient dans ce fichier, prompts compris. 

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

- **Mon compte disparaît le 04/09.** L'alerte `no_answer` a déjà été rebasculée sur toi (tu es destinataire depuis le 19/08), mais mon adresse est encore rattachée en parallèle. 
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

## 8. Le contexte produit, pour que tu saches où va l'agent

Tu ne portes pas l'offre, mais tu as besoin de ce contexte pour arbitrer ce que tu développes.

**Ce qu'on vend aujourd'hui**, c'est le modèle MMM et son analyse, restituée au client par les data strats en slides. Le module ConnectedHub n'est pas encore un produit client. L'agent sert donc pour l'instant **en interne**, aux data strats, en sanity-check sur leur propre analyse : valider une intuition, pas découvrir.

**La cible**, c'est d'ouvrir le module aux clients et de leur vendre l'assistant en **add-on payant**. Le flag payant est déjà en place dans le code, avec une preview si le client n'y a pas droit. Ce n'est pas une hypothèse lointaine, c'est ce pour quoi la feature a été construite.

**Le pricing envisagé** : abonnement fixe mensuel autour de 400 à 600 € par client, avec un fair-use vers 3 000 questions par mois. Le fixe colle à la structure de coût, qui est fixe à 75%, et le fair-use protège du power user. Le coût technique n'est qu'un **plancher de marge**, on vend à la valeur : aujourd'hui le client dépend d'une restitution manuelle du data strat, donc si l'agent remplace 2 à 4 heures de data strat par mois, ça vaut déjà 200 à 600 € mensuels. Trois inconnues restent à lever avant de figer un prix : le prix du module MMM de base, le fait de savoir si l'agent est un add-on ou un produit d'appel, et le nombre de clients cibles.

### Les prochaines étapes produit

1. **Trancher jusqu'où l'agent recommande** (Baptiste). Deux options : recommandation pleine en interne mais faits seuls côté client, ou assistant pur sans recommandation nulle part. C'est le blocage principal, il traîne depuis juillet et il conditionne aussi ton backlog technique.
2. **Ouvrir l'agent aux équipes conseil sur un MMM client signé.** C'est le vrai prochain jalon, et il n'est pas technique. La trajectoire retenue est « conseil d'abord, client ensuite », validée par Fabien Bourrely et Baptiste. La cible est **Stellantis** (Opel DE et Peugeot DE) : le modèle est disponible, rien ne bloque côté data, mais le conseil DE ne répond plus depuis l'envoi du deck. Sur Longchamp, le MMM v2 vient d'être signé mais il n'y a pas encore de modèle, donc rien à montrer avant.
3. **Puis seulement l'ouverture client**, qui suppose une couche pédagogique solide. Le précédent Jacadi, un client à qui on a donné un accès direct à l'interface sans accompagnement, a produit un flot de questions. Ouvrir sans vulgariser reproduirait ça à l'échelle.

La disponibilité annoncée en interne était **fin d'été**, sous réserve du go de validation. Cette date est à réactualiser avec Baptiste, elle date d'avant mon départ.

Ce qui reste hors de ton périmètre malgré tout : le pricing, l'ouverture commerciale, et côté ConnectedHub la mémoire persistante des conversations et le chip de contexte du chatbot.

