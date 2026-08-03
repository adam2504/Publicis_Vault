---
type: project
statut: en cours
discipline:
  - dev
  - data-science
implication: lead
client: MINE
---
# MMM AI Agent

Plateforme : [[Mine Platform]] · Déploiement visé : [[STELLANTIS]] (Opel DE / Peugeot DE), puis Longchamp

> Produit **plateforme**, pas projet client : l'agent est une feature du module MMM de ConnectedHub, vendue en add-on à n'importe quel client ayant un MMM signé. D'où `client: MINE`. **Ce n'est pas un projet L'Oréal.**

Ajout d'un assistant IA conversationnel dans le module MMM de ConnectedHub. L'agent répond à des questions sur les données MMM du client (ROI par média, comparaisons périodiques, etc.) en accédant aux données via Vertex AI Agent Engine.

> 🧭 **Project Brief** — contexte produit & business (ce fichier). Le technique (pipeline, intégration ConnectedHub, observabilité, coût, bugs) est dans [[Architecture technique]]. Les notions métier MMM (variables, décomposition, saturation) sont dans [[MMM — Notions]].

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
- Aide à la lecture des graphiques (bloc `CTX_MODULE_VIZ`, bilingue EN|FR) — en prod le 22/07
- **Scope écran live (feature B)** : l'agent connaît le KPI, la période, l'onglet et la langue affichés et répond dans ce scope par défaut, surchargeable dimension par dimension. Annonce le scope utilisé sur sa 1re réponse. En prod le 23/07 (engine `6241`) — cf. [[Session 2026-07-23]]

## Ce qui reste ouvert

- [ ] Ajouter `cf-budget-allocator-prod` (GCF d'optimisation budget) comme tool de l'agent — contrat relevé et points durs dans [[Tool - cf-budget-allocator]]. **Deux préalables non techniques** : le grant IAM `run.invoker` sur la CF, et trancher si l'agent a le droit de *lancer* une optimisation (recommandation) ou seulement de lire
- [ ] Mémoire persistante par utilisateur (reprendre une ancienne conversation)
- [ ] **Cadrage produit, recommandation** : jusqu'où l'agent recommande vs se limite aux faits. Deux modèles sur la table, à trancher avec Baptiste :
	- **A. Deux niveaux selon le compte** (déjà proposé, vu avec Fabien et Amaury) : reco pleine en interne (conseil inclus) ; côté client, faits seulement, aucune reco.
	- **B. Assistant pur, aucune reco** (option plus simple, 24/07) : l'agent ne recommande jamais, ni interne ni client. L'expertise et la reco restent côté humain. Évite d'avoir à gérer deux comportements selon le compte.
	- Décision structurante : elle conditionne aussi le tool d'allocation budget (lancer une optimisation = recommander). En option B, ce tool ne pourrait que lire/expliquer une allocation, jamais en proposer une. Cf. [[Tool - cf-budget-allocator]].
- [ ] **Chip de contexte** dans le chatbot (`Contexte : ROAS · juin 2025 – mai 2026`) : rend le scope visible en permanence, sans coût en tokens, et signale un décalage avant même la lecture de la réponse. Complément visuel de l'annonce faite par l'agent (23/07)
- [x] ~~Guardrails pour éviter double injection de `client_id`~~ — *formalisé le 23/07 avec la feature B : extraction Python par regex ancrée + repli, le premier marqueur gagne, donc un marqueur forgé dans le texte utilisateur ne peut pas la détourner. Whitelist stricte sur tout ce qui vient du navigateur (exclut `[`, `]`, `;`, `=`, retours à la ligne). Isolation déjà validée en test par Dan le 07/07*

> Testing interne clôturé (06/07). Base de connaissances + roadmap priorisée dans [[Synthèse & Roadmap Testing]]. Phase 2 (profils conseil / proxy client) en cours dans [[Testing Conseil (proxy client)]] — Amaury Stellian (conseil, Avène) testé le 07/07 (adhésion), puis point sponsor avec Fabien Bourrely (DG Starcom) le 08/07.

## Roadmap & mise en prod

- **v1 — prête, en validation.** Répond sur les données du modèle (ROI par média, comparaisons de périodes, allocation budget). Code et déploiement en place ; reste la validation par les Data Scientists + Baptiste sur un grand volume de questions (sécurité, fiabilité, isolation inter-clients).
	- *Avancement validation (07/07, session Dan) : isolation inter-client OK ; bug `no_answer` intermittent sur questions multi-canaux diagnostiqué (raté transitoire amont, non déterministe) → correctif d'observabilité livré en prod.*
	- *Avancement observabilité (08/07) : grant IAM accordé → tracing détaillé par sous-agent opérationnel (Cloud Trace) + alerte email `no_answer`. Diagnostic désormais outillé (logs `authors` + Sessions API rétroactive). Reste à rejouer les questions multi-canaux de Dan pour capturer un raté transitoire en live. Cf. [[Session 2026-07-08]].*
- **v2 — enrichissement.** *Aide à la lecture des graphiques **livrée** (bloc `CTX_MODULE_VIZ`, 22/07). **Scope écran live (feature B) livrée** le 23/07 — engine `6241` en prod, cf. [[Session 2026-07-23]].* Reste : tools saturation/allocation (`cf-budget-allocator-prod`, cadrage dans [[Tool - cf-budget-allocator]]), enrichissement `BUSINESS_CONTEXT` par les DS, contexte/knowledge par client. Backlog priorisé dans [[Session 2026-07-22]].
- **Prochaine étape produit / déploiement — ouverture au conseil sur un MMM live.** Le prochain vrai jalon terrain n'est pas technique : **donner accès à l'agent aux équipes conseil qui travaillent sur un MMM client signé**, en commençant par **Stellantis** (Opel DE / Peugeot DE). C'est la trajectoire **« conseil d'abord, client ensuite »** de Fabien Bourrely, **validée par Fabien (DG Starcom) et Baptiste**. Contacts et logistique amorcés avec **Inès et Katia** (data strats Stellantis) ; l'équipe conseil / activation DE (Zenith Media) a déjà accès au module MMM de ConnectedHub. **État au 24/07** : sur Stellantis le modèle est dispo (rien ne bloque côté data), mais **le conseil DE ne répond plus** depuis le mail deck, à relancer ; sur **Longchamp** le MMM v2 vient d'être signé, **pas encore de modèle**, donc rien à montrer avant. Détail, contacts et to-dos dans [[Testing Conseil (proxy client)]].
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
| Data Strat — Stellantis MMM               | Katia                         |
| Web Analyst — ex-Analyste Consultante MMM | Elsa                          |

## Contexte de reprise

Projet désarchivé le 23/06/2026. Baptiste (futur head of études et mesures) a suggéré lors d'un 1-to-1 d'organiser des **sessions de testing internes** ([[Testings Internes]]) avec des profils variés (data strats, DS) avant toute mise en avant client. Inès et Katia identifiées via Hajar comme data strats actives sur le MMM Stellantis. Elsa identifiée par Adam — Web Analyst avec un background Analyste Consultante MMM. Premières cibles pour les sessions de testing.

## Réunions

- [[2026-04-13 Point avec Brieg et Hajar]]
- [[2026-04-15 Point avec Brieg]]
- [[2026-04-20 Point avec Brieg et Hajar]]
- [[2026-04-27 Point avec Brieg]]

## Dernières sessions

- **2026-07-23** : Feature B (scope écran live) cadrée, implémentée et livrée en prod (engine `6241`). Défaut critique corrigé : bornes de période passent en dates réelles (colonnes `year`/`week` inexistantes dans BQ). Guardrails whitelist durcis, security review validée. Monitoring à surveiller (`no_answer`, `uiScopeRejected`).
- **2026-07-22** : Bloc `CTX_MODULE_VIZ` (aide à la lecture des graphiques, bilingue EN/FR) livré. Retry backend sur `no_answer` transitoire implémenté. Engine `2659` en prod. UI polish : suggestions refaites, chatbot flottant repositionné, bug z-index portail corrigé.
- **2026-07-20** : Durcissement anti-hallucination. `BUSINESS_CONTEXT` décomposé en blocs `CTX_*` scopés par sous-agent. Engine v2 `7899` déployé. Deux bugs d'infra corrigés : deps pinnées (`google-adk==1.26.0`) et faux négatif Windows console.
- **2026-07-08** : Grant IAM accordé sur `mmm-agent-sa`, tracing par sous-agent opérationnel (Cloud Trace, ~48 spans/question). Zéro redéploiement nécessaire. Sessions API et GCS confirmés complémentaires pour le diagnostic `no_answer`.
- **2026-07-07** : Validation DS par Dan. Isolation inter-client OK (Opel DE ne peut pas requêter Peugeot). Diagnostics stream (`authors`, `chunkCount`, `streamErrors`, `lastChunk`) livrés en prod. Bug `no_answer` multi-canaux confirmé intermittent et transitoire.
