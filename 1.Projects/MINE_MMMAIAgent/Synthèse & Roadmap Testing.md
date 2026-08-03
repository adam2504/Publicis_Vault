---
type: note
projet: MMM AI Agent
---

Projet : [[MMM AI Agent]]

# Synthèse & Roadmap — Testing MMM Agent

Consolidation de la phase de testing interne (clôturée le 06/07/2026 — sessions brutes dans [[Testings Internes]]). Deux temps : d'abord **ce que le testing nous a appris** (base de connaissances durable), puis **la roadmap d'actions** pour enrichir l'agent. Support du point à venir avec Baptiste.

> Depuis, une **phase 2 « proxy client »** (profils conseil non-initiés) a démarré — cf. [[Testing Conseil (proxy client)]] ; ses premiers acquis (Amaury, 07/07) sont intégrés ci-dessous. La **validation DS** a aussi avancé (session Dan, 07/07 — voir §2 Gate de validation).

---

# 1. Base de connaissances

> Ce que le testing nous a appris sur le MMM, les clients et l'usage de l'agent. Connaissance transversale, à garder même une fois les pistes livrées — indépendante des actions ci-dessous.

## Ce qui est vendu, et à qui sert l'agent

- **Aujourd'hui** : on vend au client **le modèle MMM et son analyse** (restituée par les data strat en slides) ; le module ConnectedHub n'est pas encore un produit client. L'agent sert donc pour l'instant en **interne**, aux data strat — mode d'usage type *(Lara)* : **sanity-check / second regard** sur leur propre analyse (valider une intuition, pas découvrir).
- **Cible produit** : ouvrir le **module aux clients** et leur vendre **l'assistant IA en add-on payant** (flag payant déjà en place ; pistes de prix dans [[MMM AI Agent]]). Le testing interne prépare cette ouverture — ce n'est pas la finalité.
- **Cas Jacadi** (client ayant eu un accès direct à l'interface → beaucoup de questions) : montre que l'accès client **sans accompagnement** pose problème → argument fort pour la couche pédagogique le jour de l'ouverture (cf. « prof MMM », roadmap P2).

## Valeur & positionnement de l'agent

- **Point d'entrée unique / couche de centralisation** des données MMM. Argument renforcé par Longchamp v2 : montée en granularité (divisions produit) → interface moins lisible → l'agent centralise et rend les données interrogeables. *(Clarisse)*
- **Valeur même sans réponse parfaite** : expliquer un concept fait déjà gagner du temps, même sans le chiffre exact. *(Inès)*
- **Feature monétisée + argument d'offre** : l'assistant est destiné à être vendu en **add-on payant** ; sur une propale MMM (Feu Vert) il différencie aussi l'offre face à la concurrence. *(30/06)*
- **Le chiffre ne suffit pas — il faut le narratif.** Un profil conseil (non-MMM) bute sur le *pourquoi* d'une perf, pas sur le chiffre : l'agent doit relier aux facteurs exogènes / « ce qui s'est passé », pas seulement restituer des nombres. Friction n°1 confirmée hors du cercle expert. *(Amaury, phase 2)*

## Trajectoire : interne aujourd'hui → client demain

Usage actuel interne (data strat), mais la **cible est l'accès client + assistant IA payant**. Deux implications pour la roadmap :
- L'ouverture client **suppose un mode pédagogique fort** (vulgariser le MMM aux non-initiés) — sinon effet Jacadi. C'est la piste « prof MMM » (P2), alignée sur la cible produit, pas un « au cas où ».
- Fiabilité et isolation deviennent **critiques** dès qu'un client paie et interroge lui-même → renforce la priorité des guardrails P0.

## Ce qu'un client attend vraiment du MMM — cadrage de référence (Feu Vert)

Le meilleur jeu de questions client qu'on ait. Sert de **banc de test** réutilisable (validation + non-régression). Le besoin Feu Vert tourne autour d'une décision de coupe des investissements branding.

**Premier rendu :**
1. **Contribution / ROAS** — incrément business du media sur trafic & CA on/offline (2 ans) ; media vs facteurs endogènes (promos, events) / exogènes (saisonnalité) ; branding vs reste ; par levier (les plus roistes).
2. **Impact court / long terme** de chaque levier ; rémanence du branding ; à partir de quand après la coupe les invests 2026 ne génèrent plus rien.
3. **Saturation** — seuil de saturation par levier / semaine ; niveau optimal ; palier déjà atteint ?
4. **Simulation** — meilleur scénario d'allocation pour les PM futurs ; volume trafic/CA avec vs sans branding + rentabilité.

**Récurrences semestrielles :** évolution trafic/CA après coupure ; perte cumulée 6 / 12 mois ; jusqu'à quand le gain court terme de la coupe compense la perte long terme ; en cas de réinvest, incrément attendu / levier à prioriser / nouveau mix optimal.

## Connaissance métier des médias (« ADN des leviers »)

- Chaque levier sert des objectifs différents — ex : l'**OOH** sert aussi des objectifs de **branding**, pas seulement la perf. L'agent devrait porter cette lecture de conseil, pas seulement les chiffres.
- **Non universel** : cette expertise varie selon **secteur / pays / client**. Ne pas la figer → elle appelle une couche contextuelle (rejoint la personnalisation par client).

## Deux types de limites à distinguer

Ce que le testing a clarifié — et qui fonde le guardrail P0 :
- *Limite données / modèle* — l'info n'existe pas dans le modèle (ex : GRP non intégrés, module synergies non fait sur Stellantis) → l'agent ne peut pas inventer.
- *Limite d'interprétation* — juger une perf « bonne / mauvaise » sans benchmark → l'agent ne devrait pas trancher, faute de référentiel.

---

# 2. Roadmap

> Actions pour enrichir l'agent. Chaque piste renvoie à l'acquis de KB (§1) qu'elle sert.

## Gate de validation (Baptiste + DS)

La v1 est fonctionnelle et déployée ; il reste **le go de validation (Baptiste + DS)** avant l'ouverture — d'abord aux data strat en interne, puis aux clients (add-on payant) :
- **Sécurité / isolation inter-clients** — **validée en test par Dan (07/07)** : depuis Opel DE, une requête Peugeot a été refusée proprement. Reste à formaliser le guardrail double injection `client_id` (cf. P0).
- **Fiabilité sur volume** — testing élargi par les DS (Hajar) sur un grand nombre de questions, en s'appuyant sur le banc Feu Vert (§1). Un bug `no_answer` intermittent (questions multi-canaux) a été diagnostiqué le 07/07 : raté transitoire amont, non déterministe. Correctif d'observabilité livré en prod ; tracing détaillé par sous-agent en attente d'un grant IAM (ticket support).

**Jalon** : présenter cette synthèse à Baptiste → obtenir le go. Cible dispo annoncée à Clarisse : **fin d'été**, sous réserve.

## Pistes priorisées

### P0 — Fiabilité & sécurité (bloquant pour le go)

- **Guardrails « je n'ai pas l'info » plutôt que combler.** Faire respecter la distinction *données/modèle* vs *interprétation* (→ KB « limites connues »). L'agent signale la limite au lieu d'inventer ou de juger sans référentiel.
- **Guardrail double injection `client_id`.** Verrouiller l'isolation inter-clients — prérequis du go Baptiste. *(Comportement d'isolation déjà validé en test par Dan le 07/07 ; reste la formalisation.)*
- **Deux niveaux d'accès : reco interne / factuel client.** Selon le compte : interne (mail Publicis) → l'agent recommande ; externe (client) → faits seulement, pas de reco. Opérationnalise le guardrail « jugement / reco » et sécurise l'ouverture client (anti-effet Jacadi). **Arbitrage produit à trancher avec Baptiste + data strats** (jusqu'où laisser recommander). *(Amaury, phase 2 ; incohérence de reco aussi remontée par Dan)*

### P1 — Enrichissement contexte (fort impact, récurrent)

- **Contexte sur les variables et les données du modèle.** Expliquer ce que représentent les variables et le « pourquoi » (baseline, contribution à X %). Enrichir le `BUSINESS_CONTEXT` avec les DS (Dan / Hajar). *(Inès, Clarisse)*
- **Aide à la lecture des graphiques.** Donner à l'agent une connaissance « spatiale » des graphes du module : ce qu'ils montrent, comment les lire, ce qu'on en tire (→ KB : l'interprétation compte plus que le chiffre brut). *(friction n°1 d'Inès)*
- **Registre explicatif / narratif (« le pourquoi »).** Au-delà de définir les variables, relier une perf à son contexte (événements, saisonnalité, marché) : l'agent chiffre bien mais n'explique pas le *pourquoi*. Confirmé côté conseil — même sans chiffre exact, expliquer fait gagner du temps. *(Inès, Amaury)*

### P1/P2 — Nouveaux tools

- **Saturation + optimisation / allocation.** Courbes de saturation + fonctions built-in de Mine + budget-allocator (`cf-budget-allocator-prod`) comme tools. Couvre « quand ça sature ? » et « meilleur scénario d'allocation ». *(Inès, Clarisse, cadrage Feu Vert)*
- **Analyse temporelle : court / long terme & rémanence.** Besoin client explicite (Feu Vert, §1) **jamais testé** contre l'agent → d'abord vérifier ce qu'il sait répondre, puis combler (données/modèle avec les DS, ou tool dédié).

### P2 — Couche contextuelle (v2, chantier unique)

Les trois pistes suivantes convergent vers **une même brique** : une couche de contexte modulable (secteur / pays / client / niveau d'expertise de l'audience). À traiter d'un bloc, pas en silos.

- **« ADN des leviers » / conseil média** — vraie expertise média (→ KB « ADN des leviers »).
- **Personnalisation par client** — contexte secteur, concurrents, cadrage spécifique par `client_id`. *(Clarisse)*
- **Couche pédagogique « prof MMM »** — vulgariser et guider les clients non-initiés ; prérequis de l'ouverture client (cf. Trajectoire §1), registre distinct du conseil média. *(Lara)*

## Backlog produit (hors testing)

- **Mémoire persistante par utilisateur** (reprendre une ancienne conversation) — open item existant, cf. [[MMM AI Agent]]. Pas issu du testing, gardé ici pour ne pas le perdre de vue.

## Prochaines actions

**Immédiat — débloquer le go :**
- [x] **Validation DS — isolation inter-clients** : OK (test Dan 07/07, Opel DE → Peugeot refusé).
- [x] **Débloquer l'observabilité** : obtenir le grant `telemetry.tracesWriter` au SA `mmm-agent-sa` (ticket support), puis faire rejouer à Dan ses questions multi-canaux en prod pour confirmer la cause du `no_answer`.
- [ ] **Validation DS — guardrail « je ne sais pas »** : faire valider le refus de combler (donnée hors modèle) et de juger sans benchmark.
- [ ] **Enrichir le `BUSINESS_CONTEXT`** avec les DS (Dan / Hajar) : définitions des variables et des données du modèle.
- [ ] **Trancher les deux niveaux d'accès** (reco interne / factuel client) avec Baptiste + data strats — point d'agenda du go.
- [ ] **Présenter à Baptiste** la synthèse → obtenir le go.

**Ensuite — enrichissement agent :**
- [ ] Tools **saturation + allocation** (`cf-budget-allocator-prod`) et **aide à la lecture des graphiques**.
- [ ] Vérifier puis combler l'**analyse temporelle** (court / long terme, rémanence).
- [ ] **Couche contextuelle** (ADN des leviers + perso par client) — chantier v2.
