---
type: note
projet: MMM AI Agent
---

# Accès & Rôles à ouvrir pour Dan

Projet : [[MMM AI Agent]]

Liste établie le 19/08/2026 pour la passation. Objectif : Dan doit pouvoir **déployer**, **diagnostiquer** et **faire évoluer** l'agent seul, sans dépendre d'un compte qui n'existera plus après le 04/09.

**Bénéficiaire** : Dan Phan, identité GCP `danphan2@publicisgroupe.net`

> Son autre adresse, `dan.phan@publicismedia.com`, n'est **pas** l'identité qui porte les accès GCP. Toute demande d'IAM se fait sur `danphan2@publicisgroupe.net` : une demande sur la mauvaise identité est acceptée sans rien ouvrir, et ça ne se voit qu'au premier déploiement raté.

> Le point non intuitif : ça touche **deux projets GCP**. L'agent vit dans `med-dtam-prd-mg`, mais les logs qui servent au diagnostic des `no_answer` sont écrits par le backend ConnectedHub dans `pmed-portal-prd-mg`. Un accès au seul projet DTAM permet de déployer, pas de diagnostiquer.

---

## 1. Projet `med-dtam-prd-mg` (l'agent)

| Rôle | Pourquoi | Sans lui |
| --- | --- | --- |
| `roles/aiplatform.user` | Créer, mettre à jour, supprimer les Reasoning Engines. Appeler `streamQuery`. Lire la Sessions API | Aucun déploiement, aucun test |
| `roles/iam.serviceAccountUser` **sur le seul SA `mmm-agent-sa@med-dtam-prd-mg.iam.gserviceaccount.com`** | Déployer un engine qui **tourne sous** ce SA | `adk deploy` échoue en `PERMISSION_DENIED` alors même que `aiplatform.user` est accordé. **C'est le piège le plus courant de cette liste** |
| `roles/storage.objectAdmin` sur le bucket de staging de `adk deploy` | La commande y dépose le package de l'agent avant création de l'engine | Déploiement bloqué à l'upload |
| `roles/storage.objectViewer` sur `mmm-agent-chat-logs` | Relire un échange complet (question, réponse, `uiScope`, `adkSessionId`) | Perte d'une des deux sources rétroactives de diagnostic |
| `roles/bigquery.dataViewer` + `roles/bigquery.jobUser` | Rejouer à la main les requêtes sur `mine.tb_model_contributions`, vérifier ce que l'agent a lu | Impossible de confirmer un chiffre remonté par l'agent |
| `roles/cloudtrace.user` | Lire le Trace Explorer (~48 spans par question) | Diagnostic de performance à l'aveugle. Les autres sources restent utilisables, donc ce rôle est le moins critique de la liste |

> À ne **pas** demander : le grant `roles/run.invoker` sur `cf-budget-allocator-prod`. Il n'est utile que le jour où le tool d'allocation budget est développé, et cette décision n'est pas prise (cf. [[Tool - cf-budget-allocator]]). Le demander maintenant, c'est ouvrir un droit sans usage.

## 2. Projet `pmed-portal-prd-mg` (diagnostic et alerte)

| Rôle | Pourquoi | Sans lui |
| --- | --- | --- |
| `roles/logging.viewer` | Lire les logs `metric="mmm_agent_exchange"` : `authors` (où le pipeline meurt), `lastChunk`, `streamErrors`, `adkSessionId`, `status` | **La recette de diagnostic d'un `no_answer` ne démarre pas.** C'est le point d'entrée de tout le triage |
| `roles/monitoring.viewer` | Voir la métrique log-based et les incidents ouverts | Pas de vue d'ensemble du volume d'erreurs |
| `roles/monitoring.editor` | Modifier la policy `alertPolicies/527257283050504576` et son notification channel | Dan ne peut pas reprendre l'alerte à son nom, ni l'ajuster |

> Ce projet est celui de ConnectedHub, pas celui du pôle DTAM. L'ouverture passe donc probablement par Eddie ou par l'IT, pas par le même circuit que les rôles du §1. **À lancer en premier, c'est le plus lent.**

## 3. Hors GCP

| Accès | Détail |
| --- | --- |
| **GitHub** | Droit d'écriture sur `Publicis-Media-France-FR5140/MMM_AI_Agent` (repo *internal*, poussé le 19/08). La lecture est acquise pour les membres de l'organisation, l'écriture doit être accordée explicitement |
| **Confluence** | Droit d'édition sur l'espace qui porte la doc de l'agent |
| **Alerte `no_answer`** | Le notification channel `5134431245708235828` pointe aujourd'hui sur `adajouin@publicisgroupe.net`. À rebasculer sur Dan, ou mieux sur une alias d'équipe qui survit au prochain départ |

### Procédure — rebasculer l'alerte

À faire **avec Dan**, pour qu'il voie où ça se règle et reçoive le premier mail de test.

1. Console → **Monitoring → Alerting**, projet `pmed-portal-prd-mg` → policy *MMM agent — no_answer (log-based)* (`alertPolicies/527257283050504576`).
2. **Créer un nouveau notification channel** de type Email plutôt que modifier l'existant. L'ancien (`5134431245708235828`) reste en place le temps de vérifier que le nouveau reçoit bien, on le retire ensuite.
3. Destinataire : **une alias d'équipe si elle existe**, sinon `dan.phan@publicismedia.com`. L'alias est préférable, sinon le prochain départ repose exactement le même problème.
4. Rattacher le nouveau channel à la policy, **sans retirer l'ancien tout de suite**.
5. Vérifier la réception : soit attendre un `no_answer` réel, soit poser une question multi-canaux dans le module pour en provoquer un (le bug est intermittent, donc pas garanti du premier coup).
6. Une fois la réception confirmée, **retirer l'ancien channel** de la policy.

> Ne pas se contenter de changer l'adresse du channel existant : si la modification échoue en silence, plus personne ne reçoit rien et ça ne se voit que le jour d'un incident.

---

## Le SA de l'agent, pour mémoire

`mmm-agent-sa@med-dtam-prd-mg.iam.gserviceaccount.com` porte déjà les 7 rôles dont l'agent a besoin **à l'exécution**. Ils sont indépendants de ceux de Dan et il n'y a rien à y changer :

`aiplatform.user`, `bigquery.dataViewer`, `bigquery.jobUser`, `cloudtrace.agent`, `logging.logWriter`, `monitoring.metricWriter`, `telemetry.tracesWriter`

Ne pas confondre les deux listes : Dan a besoin de droits pour **piloter** l'agent, le SA a besoin de droits pour **faire tourner** l'agent.

---

## À vérifier avant d'envoyer la demande

- [x] ~~L'adresse exacte de Dan~~ — `dan.phan@publicismedia.com`, reste à confirmer que c'est l'identité GCP
- [ ] Le nom du bucket de staging utilisé par `adk deploy` (relevé à la prochaine exécution, il apparaît dans la sortie de la commande)
- [ ] Que la policy `527257283050504576` est bien dans `pmed-portal-prd-mg` et non dans `med-dtam-prd-mg`
- [ ] Les rôles déjà détenus par Dan sur les deux projets, pour ne demander que le delta
