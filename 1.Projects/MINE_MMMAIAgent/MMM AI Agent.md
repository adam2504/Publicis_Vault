---
type: project
statut: en cours
discipline:
  - dev
  - data-science
implication: lead
client: LOREAL
---
# MMM AI Agent

Plateforme : [[Mine Platform]]

Ajout d'un assistant IA conversationnel dans le module MMM de ConnectedHub. L'agent répond à des questions sur les données MMM du client (ROI par média, comparaisons périodiques, etc.) en accédant aux données via Vertex AI Agent Engine.

> 🧭 **Project Brief** — contexte produit & business (ce fichier). Le technique (pipeline, intégration ConnectedHub, observabilité, coût, bugs) est dans [[Architecture technique]].

---

## Ce qui a été livré

- Agent déployé sur **Vertex AI Agent Engine**
- Intégré dans le module MMM de Mine (tab chatbot)
- Confidentialité inter-clients gérée via injection du `client_id` depuis le backend Mine dans le `BUSINESS_CONTEXT` + `query_agent`
- Feature verrouillée derrière un flag payant : preview si non activé, toggle dans l'Admin Panel du module
- Photo de profil de l'utilisateur dans le chat
- Contexte de discussion (mémoire dans le même chat)
- Bouton nouvelle conversation + continuité de conversation
- Chatbot flottant dans le coin du module (reste visible si changement de tab)
- Enregistrement des messages dans un bucket GCS
- Ajout dans Firebase
- Diagnostics d'observabilité sur le stream de l'agent (`no_answer` / `stream_error` : `authors`, `chunkCount`, `reachedAnswerAgent`, `streamErrors`, `lastChunk`) — livrés en prod (07/07)
- Message d'erreur du chatbot pointant vers l'alias support `help.connectedhub@publicismedia.com` (tranché par Eddie, client-safe)

## Ce qui reste ouvert

- [ ] Guardrails pour éviter double injection de `client_id` — *isolation inter-client validée en test par Dan le 07/07 (depuis Opel DE, requête Peugeot refusée proprement) ; reste à formaliser le guardrail*
- [ ] Ajouter `cf-budget-allocator-prod` (GCF d'optimisation budget) comme tool de l'agent
- [ ] Explorer les fonctions built-in de Mine (courbes de saturation) comme tools
- [ ] Mémoire persistante par utilisateur (reprendre une ancienne conversation)
- [ ] Phase de testing élargie par les Data Scientists pour valider la sécurité sur un grand nombre de questions
- [x] **Observabilité** : grant accordé au SA `mmm-agent-sa` (4 rôles) le 08/07 → 403 `telemetry.traces.write` résolu, tracing ADK **par sous-agent opérationnel** dans Cloud Trace (le grant était le fix entier, aucun code). Alerte email `no_answer` créée sur `pmed-portal-prd-mg`. Cf. [[Session 2026-07-08]].
- [x] **`no_answer` — hallucination de tool corrigée (20/07)** : root cause confirmée contre le code — `schema_agent` (get_table_info only) amorcé par le `BUSINESS_CONTEXT` complet hallucinait `execute_sql` → ADK 498 → `no_answer`. Fix livré = décomposition du `BUSINESS_CONTEXT` en blocs `CTX_*` scopés par sous-agent (`schema_agent` → aucun contexte + instruction durcie ; `CTX_SQL_RULES` → `query_writer` seul). Déployé sur un **nouvel engine `reasoningEngines/7899832715896160256`** (blue/green) et **basculé en prod** (`AGENT_RESOURCE` mergé sur `main`). Validé : les canaris définitionnels ne produisent plus de `no_answer` (vs l'ancien engine). Cf. [[Session 2026-07-20]].
- [x] **Dette backend** : fix `reachedAnswerAgent` corrigé et mergé (PR #1647/#1648). ID resource périmé du README corrigé (`4105…` → live `6073…`) + table de mapping engine↔commit ajoutée (20/07).
- [x] **Livraison 22/07 — module viz + fiabilité (prod sur engine `2659`)** : bloc **`CTX_MODULE_VIZ`** (aide à la lecture des graphiques, bilingue EN/FR, générique, injecté `answer`+`root`) ; **filet retry backend** sur `no_answer` transitoire (l'hallucination `execute_sql` s'avère **non-déterministe** et migre entre sous-agents — schema→discovery→context_setter — donc le retry est le fix uniforme, en plus du retrait de priming résiduel + durcissement `discovery`) ; **root routing durci** (délègue dès qu'un chiffre est nécessaire → plus de « chart-only ») ; **« Media total »** exclu comme faux canal. Cutover ConnectedHub mergé (`develop`→`main`). **Prod = `reasoningEngines/2659050124520456192`** (supersede `7899…`). Cf. [[Session 2026-07-22]].
- [ ] **Deploy hygiene (20/07)** : les déps non pinnées faisaient installer `google-adk 2.5.0` dans le container → crash au démarrage (`dataplex_v1`). Pin `google-adk==1.26.0` posé dans `MMM_Agent/requirements.txt` (le fichier qu'`adk deploy` lit). **Reste** : propager ce pin sur la branche principale du repo `Agent MMM`, et supprimer l'ancien engine `6073…` une fois la nouvelle version prouvée en prod. Cf. [[Session 2026-07-20]].
- [ ] **Cadrage produit — recommandation** : jusqu'où l'agent recommande vs se limite aux faits (surtout côté client) — à trancher avec Baptiste + data strats

> Testing interne clôturé (06/07). Base de connaissances + roadmap priorisée dans [[Synthèse & Roadmap Testing]]. Phase 2 (profils conseil / proxy client) en cours dans [[Testing Conseil (proxy client)]] — Amaury Stellian (conseil, Avène) testé le 07/07 (adhésion), puis point sponsor avec Fabien Bourrely (DG Starcom) le 08/07.

## Roadmap & mise en prod

- **v1 — prête, en validation.** Répond sur les données du modèle (ROI par média, comparaisons de périodes, allocation budget). Code et déploiement en place ; reste la validation par les Data Scientists + Baptiste sur un grand volume de questions (sécurité, fiabilité, isolation inter-clients).
	- *Avancement validation (07/07, session Dan) : isolation inter-client OK ; bug `no_answer` intermittent sur questions multi-canaux diagnostiqué (raté transitoire amont, non déterministe) → correctif d'observabilité livré en prod.*
	- *Avancement observabilité (08/07) : grant IAM accordé → tracing détaillé par sous-agent opérationnel (Cloud Trace) + alerte email `no_answer`. Diagnostic désormais outillé (logs `authors` + Sessions API rétroactive). Reste à rejouer les questions multi-canaux de Dan pour capturer un raté transitoire en live. Cf. [[Session 2026-07-08]].*
- **v2 — enrichissement.** *Aide à la lecture des graphiques **livrée** (bloc `CTX_MODULE_VIZ`, engine 2659 en prod).* Reste : tools saturation/allocation (`cf-budget-allocator-prod`), enrichissement `BUSINESS_CONTEXT` par les DS, contexte/knowledge par client, live-suivi UI (feature B). Backlog priorisé dans [[Session 2026-07-22]].
- **Cible de dispo** : fin d'été (annoncé à Clarisse le 29/06), sous réserve du go de validation. Si prêt avant, on avance la date et on met à jour Clarisse.

## Coût & tarification

**Coût technique (synthèse)** — structure ~75% fixe : Agent Engine runtime ~$35/mois + tokens Gemini ~$0,05/question (~$0,10 avec retries) + ~$6/mois divers. Par client (1 client porte toute la base fixe) : ~$85/mois à 1 000 q, ~$185 à 3 000 q, ~$335+ à 6 000 q. **Détail complet (structure, coût par question, leviers, arbitrage `BUSINESS_CONTEXT`) : [[Architecture technique]] → §5 Coût technique.**

⚠️ **Prix ≠ coût.** Le coût technique n'est que le **plancher de marge** — on vend à la **valeur**, pas au coût de revient.

**Ancre de valeur** : aujourd'hui le client n'a pas la plateforme, il dépend d'une **présentation manuelle du data strat**. L'agent = accès self-service aux réponses ROI/média. S'il remplace ne serait-ce que 2–4h de data strat/mois → **€200–600/mois de valeur** (coût chargé évité).

**Modèle retenu (1ère idée) : abonnement fixe mensuel par client + fair-use.**
Le forfait fixe colle à la structure de coût (majoritairement fixe) ; le fair-use protège du power-user (tokens variables).

| Paramètre | Proposition |
| --- | --- |
| Prix cible | **~€400–600/mois/client** |
| Questions incluses (fair-use) | ~3 000/mois → coût plafonné ~$185 → **marge ≥ 65%** |
| Marge à usage normal (≤1 000 q) | ~85–90% |
| Seuil de rentabilité (à €500) | ~10 000 q/mois (au-delà → sous l'eau) |

**Inconnues à lever avant de figer un prix :**
- [ ] **Prix du module MMM de base** (ancre interne) — un add-on IA se positionne typiquement à **20–40% du prix du module** qu'il enrichit
- [ ] **Add-on vs produit d'appel** : l'agent enrichit un module déjà payé, ou c'est le point d'entrée du client sur ConnectedHub ? (change le prix plus que n'importe quel calcul de coût)
- [ ] Nombre de clients cibles (mutualise le fixe → baisse le coût/client)

## Décisions produit

| Décision | Raison |
| --- | --- |
| Feature "bonus" payante avec preview | Monétisation, pas inclus par défaut |

> Décisions **techniques** (injection `client_id` côté backend, chatbot flottant) : voir [[Architecture technique]] → §3.

## Personnes clés

| Rôle                                      | Personne                      |
| ----------------------------------------- | ----------------------------- |
| Dev — intégration Mine                    | Adam                          |
| Lead Data Scientist _(parti 12/06/2026)_  | Brieg                         |
| Data Scientist — testing                  | Hajar                         |
| Head of Data (futur pôle études & mesure) | Baptiste                      |
| Data Strat — Stellantis MMM               | Inès                          |
| Data Strat — Stellantis MMM               | Katya                         |
| Web Analyst — ex-Analyste Consultante MMM | Elsa                          |

## Contexte de reprise

Projet désarchivé le 23/06/2026. Baptiste (futur head of études et mesures) a suggéré lors d'un 1-to-1 d'organiser des **sessions de testing internes** ([[Testings Internes]]) avec des profils variés (data strats, DS) avant toute mise en avant client. Inès et Katya identifiées via Hajar comme data strats actives sur le MMM Stellantis. Elsa identifiée par Adam — Web Analyst avec un background Analyste Consultante MMM. Premières cibles pour les sessions de testing.

## Réunions

- [[2026-04-13 Point avec Brieg et Hajar]]
- [[2026-04-15 Point avec Brieg]]
- [[2026-04-20 Point avec Brieg et Hajar]]
- [[2026-04-27 Point avec Brieg]]
