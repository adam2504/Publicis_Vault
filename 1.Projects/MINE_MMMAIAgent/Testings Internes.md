---
type: note
projet: MMM AI Agent
---

Projet : [[MMM AI Agent]]

# Testing interne

> **Phase clôturée le 06/07/2026.** 8 profils testés (Inès, Katia, Clarisse, Lydia, Léonie, Stanislas, Lara + Hajar sur la V1). Synthèse (base de connaissances + roadmap) dans [[Synthèse & Roadmap Testing]]. Cette note reste l'archive de la phase 1.

Objectif : faire tester l'agent MMM par des profils internes non-techniques avant toute mise en avant client. Approche suggérée par Baptiste lors du point du 22/06.

---

## Cibles

| Personne  | Profil                                    | Statut          |
| --------- | ----------------------------------------- | --------------- |
| Inès      | Data Strat — MMM Stellantis (actif)       | Fait 23/06      |
| Katia     | Data Strat — MMM Stellantis (actif)       | Fait 30/06      |
| Elsa      | Web Analyst — ex-Analyste Consultante MMM | À contacter     |
| Hajar     | Data Scientist — testing V1               | Déjà au courant |
| Lydia     | Data Scientist                            | Fait 30/06      |
| Clarisse  | Data Strat — MMM Longchamp                | Fait 29/06      |
| Lara      | Data Strat — MMM Avène                    | Fait 06/07      |
| Léonie    | Data Strat — propal MMM Feu Vert          | Fait 30/06      |
| Stanislas | Data Strat — propal MMM Feu Vert          | Fait 30/06      |

## Format session

- Durée : ~30 min
- **Intro & contexte** (~1 min) : dire vite fait ce que fait l'assistant + démo immédiate avec la question ROAS basique ; rappeler que ça découle du point avec Baptiste (faire connaître et faire tester en interne) et le statut actuel (~75 %, en attente des retours data strat / clients).
- **Expérience MMM & clients** : leur demander comment ils travaillent avec le MMM et les clients — questions qui reviennent, ce qui serait utile, feedbacks s'ils ont déjà bossé avec le module.
- **Test / phase libre** : leur passer la main pour poser leurs propres questions à l'agent ; recueillir les feedbacks en live (l'agent répond-il bien ? qu'est-ce qui manque ?).

---

## Pistes d'amélioration

_Section mise à jour au fil des sessions. Pour le contexte complet → voir les sessions individuelles._

**Aide à la lecture des graphiques**
Inès a eu du mal à comprendre à quoi servent les graphiques concrètement, pas sur les chiffres.
→ Donner à l'agent une connaissance "spatiale" des graphiques disponibles dans le module MMM : ce qu'ils montrent, comment les lire, ce qu'on peut en tirer.

**Saturation + optimisation / allocation budgétaire comme tools** _(Inès, Clarisse, + cadrage Feu Vert 30/06)_
L'agent ne peut pas répondre directement aux questions de saturation ("quand arrive la saturation média ?") ni aux besoins d'**allocation** (« meilleur scénario d'allocation pour les PM futurs », module de simulation demandé par Feu Vert).
→ Ajouter les courbes de saturation, les fonctions built-in de Mine et le budget-allocator (`cf-budget-allocator-prod`) comme tools de l'agent.

**Analyse temporelle : impact court / long terme & rémanence** _(cadrage Feu Vert 30/06)_
Besoin client explicite jamais testé contre l'agent : impact court vs long terme de chaque levier, rémanence du branding, « à partir de quand après la coupe les invests ne génèrent plus rien ».
→ Vérifier ce que l'agent sait déjà répondre là-dessus, puis combler (données/modèle avec les DS, ou tool dédié).

**Contexte sur les variables et les données du modèle**
Les réponses manquent d'explication sur ce que représentent les variables, les données et leurs impacts (ex : pourquoi la baseline est ce qu'elle est, pourquoi la contribution média est de X %).
→ Enrichir le BUSINESS_CONTEXT avec des descriptions des variables et la définition des données — à faire avec les DS (Dan/Hajar).

**Personnalisation par client** _(Clarisse)_
Besoin d'adapter l'agent au client : contexte secteur, concurrents, cadrage spécifique.
→ Explorer une couche de contexte par `client_id` (au-delà de l'isolation des données déjà en place).

**Connaissance métier des leviers / couche conseil média** _(session 30/06)_
L'agent répond bien aux chiffres mais n'a pas l'« ADN des leviers » (OOH, TV…) : à quoi sert chaque média, quels objectifs il sert, limites d'interprétation.
→ Donner à l'agent une vraie expertise média (réponses de conseil) — ex : OOH sert aussi des objectifs branding. À moduler par secteur/pays/client (recoupe la personnalisation par client).

**Savoir dire « je n'ai pas l'info » plutôt que combler** _(session 30/06)_
Distinguer deux limites : données/modèle (GRP, module synergies non fait sur Stellantis) vs interprétation (juger une perf bonne/mauvaise sans benchmark).
→ Cadrer l'agent pour qu'il signale ces limites au lieu d'inventer ou de juger sans référentiel (guardrails « benchmark » et « hors modèle »).

**Couche pédagogique « prof MMM » pour clients non-initiés** _(session 06/07, Lara)_
Si l'agent est ouvert aux clients, la plupart ne connaissent pas le MMM : il doit vulgariser et guider pas à pas, pas seulement répondre aux chiffres.
→ Prévoir un registre pédagogique (explication des concepts MMM) distinct du registre conseil média. Recoupe la personnalisation par client, ici selon le niveau d'expertise de l'audience.

---

## Sessions

### 2026-06-23 — Inès

**Contexte appris**
- Pour Stellantis, le client n'a pas accès au module MMM — c'est l'équipe conseil/data strat qui l'utilise pour préparer leurs slides et présenter les résultats des modèles MMM. La cible utilisateur réelle = équipe interne, pas self-service client.

**Feedbacks sur les réponses**
- "Quand arrive la saturation média ?" → pas de réponse directe sur la saturation, mais bonne explication de ce que représente la courbe — lui aurait fait gagner du temps (à améliorer en ajoutant fonctions de saturation/optimisation)
- "Quelle variable a l'impact le plus négatif sur le volume d'orders ?" → bonne réponse ; le manque de contexte sur les variables vient des données/modèle, pas de l'agent (à améliorer côté infos Data Scientists)

**Point de friction principal**
- Galère surtout sur la compréhension des graphiques (à quoi ça sert concrètement), pas sur les chiffres
- Piste d'amélioration : ajouter une aide à la lecture des graphes — qu'est-ce qu'ils représentent, qu'est-ce qu'on peut en tirer (à améliorer contexte/connaissance de l'agent sur les graphiques dispos et ou dans le module, sorte d'intelligence "spatiale" de sa part)

**À retenir**
- L'agent a de la valeur même sans réponse parfaite (explication de concept = gain de temps réel)
- Axe prioritaire : aide à l'interprétation des visualisations MMM

---

### 2026-06-29 — Clarisse

Lead Data Strategist (scope L'Oréal), active sur le MMM Longchamp.

**Contexte appris**
- Comme pour Stellantis, le client n'interroge pas le module directement : Longchamp n'a pas eu accès, les résultats sont présentés en slides PowerPoint. Jacadi a eu accès à l'interface mais a généré beaucoup de questions → l'accès direct sans accompagnement soulève des interrogations.
- Longchamp v2 à venir avec plus de granularité (divisions produit : cuir, nylon, etc.). Risque que ce soit moins lisible dans l'interface → argument fort pour l'agent comme point d'entrée unique pour centraliser et interroger les données.

**Pistes d'amélioration**
- Enrichir le contexte avec la **définition des données**, pas seulement le modèle — permettre de comprendre le « pourquoi » (ex : pourquoi la baseline est ce qu'elle est).
- **Personnalisation de l'agent par client** : contexte secteur, concurrents, cadrage spécifique.
- Ajouter les éléments de **saturation et d'optimisation** (récurrent, déjà remonté par Inès).

**Questions posées en phase libre**
- « Pourquoi la contribution média est seulement de 17 % ? » → typique du besoin de contexte sur les données (cf. piste ci-dessus).

**À retenir**
- La cible reste l'équipe interne (slides client), mais la montée en granularité de Longchamp v2 renforce la valeur de l'agent comme couche de centralisation.
- Demande spécifique de profil lead : personnalisation par client + explication des données, au-delà des chiffres bruts.

---

### 2026-06-30 — Stanislas, Léonie, Katia, Lydia (session de groupe)

**Profils**
- Stanislas (Lead Data Strat) + Léonie (Data Strat) → sur une **propale MMM pour Feu Vert**, client qui hésite et regarde la concurrence. Angle commercial.
- Katia → active sur le MMM Stellantis.
- Lydia (Data Scientist) → angle fiabilité / confiance dans les réponses.

**Préparation / angle du point**
- Intro + démo courte (5-10 min), pas un show — qu'ils comprennent vite ce que c'est.
- **Test en live = le cœur** : leur passer la main pour qu'ils posent leurs propres questions. Noter verbatim les questions tentées + ce qui échoue (= jeu de test + roadmap).
- **Recueillir leur expérience MMM** : comment les clients consomment les livrables aujourd'hui, questions qui reviennent en boucle, points de friction / ce qui est pénible à traiter à la main.
- **Angle Feu Vert** : l'agent comme différenciateur dans la propale. « Qu'est-ce que Feu Vert devrait voir pour être convaincu ? » → transforme un prospect hésitant en scénario de validation concret.
- **Message timeline** (si « c'est pour quand ? ») : v1 fonctionnelle en phase de validation, cible **fin d'été** sous réserve du go Baptiste + DS ; v2 derrière (graphiques, contexte client). Cohérent avec ce qui a été dit à Clarisse.

**Contexte appris — cadrage de la propale Feu Vert**

Le besoin MMM de Feu Vert tourne autour d'une **décision de coupe des investissements branding** : mesurer l'impact de cette coupe et arbitrer. C'est le meilleur cadrage client qu'on ait pour benchmarker ce que l'agent doit savoir répondre.

Questions du **premier rendu** :
1. **Contribution / ROAS** — incrément business (contribution) des investissements media sur trafic et CA on/offline sur 2 ans, et ROAS. Décliné : media global vs facteurs endogènes (promos, events) et exogènes (saisonnalité, vacances) ; branding global vs reste ; par levier (les plus roistes). → chiffrer le CA qui n'aurait pas existé sans le branding.
2. **Impact court / long terme** de chaque levier sur trafic et CA on/offline. → mettre en évidence la rémanence du branding ; à partir de quand après la coupe les investissements 2026 ne génèrent plus rien.
3. **Saturation** — à partir de quel niveau d'investissement/semaine un levier sature ; niveau optimal par levier. → rationaliser le passé (palier de saturation atteint ?) et revoir l'allocation du mix.
4. **Module de simulation** — meilleur scénario d'allocation pour les PM futurs ; volume trafic/CA avec vs sans branding, et rentabilité (ROAS).

Questions des **récurrences semestrielles** : évolution trafic/CA après coupure ; perte cumulée à 6 / 12 mois ; jusqu'à quand le gain court terme de la coupe compense la perte long terme ; en cas de réinvestissement, incrément attendu / levier à prioriser / nouveau mix optimal.

**Feedbacks sur les réponses testées en live**
- « Les perfs sont-elles bien ou non ? » → **piège du benchmark** : l'agent ne devrait pas juger une perf bonne/mauvaise sans référentiel. À cadrer (éviter le jugement de valeur sans benchmark).
- « Y a-t-il des synergies ? » → **piège** : le module synergies n'a pas été fait sur Stellantis → limite données/modèle, pas un défaut de l'agent.
- Demandes impliquant d'**additionner des canaux** → bien répondu à tout.
- « Combien de GRP faut-il mettre en TV à la semaine ? » → **info absente du modèle** (côté Data Scientists), l'agent ne peut pas inventer.

**À retenir — axe majeur qui ressort : "ADN des leviers" / couche conseil média**
- L'agent devrait avoir une **vraie connaissance métier des médias** (OOH, TV, …) → axe = **réponses de conseil média**. Ex : l'OOH répond aussi à des objectifs branding ; investir xxx € sur un TF sur 3 jours peut être pertinent ; clarifier les limites d'interprétation.
- **MAIS** cette expertise n'est pas universelle : elle varie selon **secteur / pays / client** → ne pas figer, prévoir une couche contextuelle (rejoint la personnalisation par client déjà remontée par Clarisse).
- Deux types de limites à distinguer clairement pour l'agent : **limites de données/modèle** (GRP, synergies non modélisées) vs **limites d'interprétation** (jugement de perf sans benchmark) — l'agent doit savoir dire « je n'ai pas cette info » plutôt que combler.

---

### 2026-07-06 — Lara

Data Strat sur le MMM Avène (L'Oréal, dermocosmétique / circuit pharmacie). Dernière session de la série de testing interne.

**Usage observé**
- Surtout des questions d'analyse et de compréhension des chiffres : elle s'en sert pour **vérifier son analyse et sa logique** après avoir eu une première intuition. Mode d'usage = sanity-check de son raisonnement, pas découverte brute — l'agent a de la valeur comme second regard.

**Questions posées en phase libre**
- « Au vu de notre investissement et de notre ROAS, est-ce pertinent de continuer à investir ? » → question d'arbitrage / recommandation.
- « Est-ce capable de faire de l'analyse comparative entre périodes ? » → test de capacité (comparaison inter-périodes).

**À retenir**
- **Angle client = « prof MMM »** : si on ouvre l'agent aux clients, il doit vulgariser et les tenir par la main — la plupart ne connaissent pas le MMM. Besoin d'une couche pédagogique explicite (cf. Pistes d'amélioration).
- **Action proposée par Lara** : faire tester l'agent par des profils **non-connaisseurs du MMM** (équipes conseil, voire leurs alternants) — « quelles questions vous poseriez ? ». Élargit le testing au-delà des experts data strat / DS.