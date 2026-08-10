---
type: note
projet: Creative Insights
---

# Patterns ConnectedHub réutilisables

Projet : [[Creative Insights]]

Exploration du repo `C:\dev\Mine - ConnectedHub\mine - repo` le 10/08/2026, après le briefing de Hajar et Abhishek. Objectif : voir ce que la chaîne cible du module peut reprendre au lieu de le réinventer.

**Conclusion principale** : les sept étapes cadrées en réunion existent déjà, éclatées entre deux modules. Presque rien n'est à concevoir, tout est à assembler.

---

## SIMBA = le module `custom-bidding`

Le nom commercial est dans `shared/src/utils/tools.ts` : la clé `custom-bidding` porte `name: 'SIMBA™'`, catégorie `analyze-measure`, `is_private: false`, avec trois sous-produits (`sea`, `youtube`, `display`).

### Ce que SIMBA fait, architecturalement

| Aspect | Implémentation |
| --- | --- |
| État du projet | **BigQuery, pas Firestore.** Table `zen-pm-custom-bidding-prd-amg.monitoring.tb_youtube_briefs`, suffixe `_dev` en staging. Créer un projet = un `INSERT INTO`. |
| Gating | Colonne `custom_bidding_status` : `SUBMITTED`, `PRE-APPROVED`, `FAILED` |
| Journal | Colonne `activity_logs ARRAY<STRUCT<log_time TIMESTAMP, log_text STRING, user_id STRING>>`, embarquée dans la ligne du projet |
| Validation amont | `services/get-eligibility.ts` lance des checks avant insertion. Un check `critical` bloque en `FAILED`, sinon les échecs partent en commentaires |
| Traitement lourd | **Jamais dans le serveur ConnectedHub.** Appel de Cloud Run avec ID token (`GOOGLE_AUTH_INSTANCE.getIdTokenClient`) : `cf-agent-caller` pour le résumé IA, `cf-monitoring-dashboard-update` pour le dashboard |
| Upload de config | Stream vers GCS, bucket `custom_bidding_simba_training`, préfixe `training_configurations/` |
| Notifications | SendGrid, templates dynamiques, envoi au créateur puis aux admins via `getUsersAdmin()` |
| Alerting planifié | Cloud function dédiée `cloud-functions/custom-bidding-alert-end-brief` qui query BQ sur les campagnes du jour et envoie les mails |
| Validation payload | Zod (`OneSuiteYoutubeProjectSchema`) |

Fichiers de référence : `server/src/features/custom-bidding/routes/create-onesuite-youtube-project.ts`, `cloud-functions/custom-bidding-alert-end-brief/src/index.ts`.

---

## Mais le vrai modèle est `feed-manager` (FeedGen)

Le CR dit « comme dans SIMBA », or c'est **feed-manager** qui contient les primitives dont le module a besoin. SIMBA donne la forme (projet en BQ, statut, appels Cloud Run), feed-manager donne la mécanique.

| Besoin Creative Insights | Existe déjà dans feed-manager |
| --- | --- |
| Étapes verrouillées séquentiellement | `shared/src/features/feed-manager/utils/global-status.ts` : `stepsStatus: { import, build, exports }` en `SUCCESS` / `PENDING` / `FAIL`, avec court-circuit dès qu'une étape échoue ou est en attente |
| Bloquer les actions pendant un traitement | Middleware `check-feed-not-pending.ts` |
| **Prompt par défaut + prompt custom** | `routes/create-prompt.ts` : champ `where` à deux valeurs. `publicis` écrit le prompt global dans `products/feed-manager.prompts.{id}` (admin seulement), `advertiser` écrit le prompt client dans `customers/{id}/products/feed-manager.prompts.{id}` |
| Job asynchrone paramétrable | `routes/create-feedgen-job.ts` : doc Firestore sous `customers/{id}/products/feed-manager/feeds/{feedId}/feedgen/{jobId}`, avec `prompt`, `temperature`, `limit`, `scoring`, `language`, `version` |
| Étape de validation humaine | `routes/update-human-feedback.ts` |
| Upload de fichiers | `routes/get-url-local-upload.ts`, `create-additional-source.ts` |
| Traçabilité | `utils/log-action.ts` et `routes/get-logs.ts` |
| Chaîne de traitement déportée | 7 cloud functions dédiées : `feed-manager-dispatch`, `feed-manager-create-workspace`, `feed-manager-categorization-batch-builder`, `feed-manager-enhancement-categorization`, `cf-feed-manager-feedgen-job`, `cf-feed-manager-load-file-to-bigquery`, `cf-feed-manager-create-exports` |

Le module a aussi un `README.md` documentant ses endpoints, seul module du repo à en avoir un.

---

## Mapping de la chaîne cible sur l'existant

| # | Étape cadrée en réunion | Pattern à reprendre |
| --- | --- | --- |
| 1 | Création de projet (advertiser IDs, plateformes, période) | `create-onesuite-youtube-project.ts` : Zod, INSERT BQ, statut initial, activity log |
| 2 | Déclenchement des cloud functions d'extraction | Appel Cloud Run authentifié par ID token, comme `update-dashboard.ts` |
| 3 | Validation humaine des assets | `update-human-feedback.ts` plus `check-feed-not-pending.ts` |
| 4 | Prompt par défaut plus prompt custom | `create-prompt.ts` avec son champ `where`, quasiment tel quel |
| 5 | Scheduled queries de mapping et notification | Cloud function planifiée sur le modèle de `custom-bidding-alert-end-brief` |
| 6 | Notebooks d'analyse | Hors ConnectedHub, invoqués via Cloud Run authentifié |
| 7 | Restitution dashboard | `update-dashboard.ts` pour le rafraîchissement, ou restitution native selon l'arbitrage |

---

## La décision technique que ça fait remonter

**SIMBA stocke l'état du projet en BigQuery, feed-manager le stocke en Firestore.** Les deux modules coexistent avec des choix opposés, donc il faut trancher explicitement plutôt que de copier au hasard.

- **BigQuery** si les DS doivent lire et écrire l'état du projet depuis leurs propres pipelines, hors de l'app. C'est précisément pourquoi SIMBA le fait : les briefs sont consommés par des traitements externes.
- **Firestore** pour de l'état purement applicatif, avec le scoping natif par `customers/{id}/products/{code-name}` et la config par client.

Pour Creative Insights, les DS lancent leurs traitements en dehors de ConnectedHub et doivent savoir quels projets traiter. Le pattern SIMBA (état en BQ) semble donc le bon pour la table de projets, éventuellement en hybride avec Firestore pour la config et les prompts, comme le fait feed-manager. **À valider avec Eddie**, c'est une décision d'architecture, pas un détail d'implémentation.

---

## Ce que ça change pour le projet

L'effort de dev réel n'est pas dans la mécanique, il est dans les points déjà identifiés comme non tranchés : la clé de jointure créa vers performance, le contrat d'ingestion, et l'arbitrage sur la restitution. Le squelette du module est du montage.

Corollaire pour la répartition avec Abhishek : ces patterns sont du code ConnectedHub, dans un monorepo avec ses conventions (features par code-name, Zod partagé, middlewares d'auth, traductions en et fr obligatoires). C'est un argument concret pour que la couche plateforme soit portée côté ConnectedHub plutôt que reconstruite à côté.
