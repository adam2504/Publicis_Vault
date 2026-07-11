Projet : [[MMM AI Agent]]

# Testing Conseil (proxy client)

> **Phase 2 du testing.** Fait suite à [[Testings Internes]] (clôturée le 06/07/2026). Action proposée par Lara le 06/07 : faire tester l'agent par des profils **non-connaisseurs du MMM** — équipes conseil, voire leurs alternants.

L'angle change par rapport à la phase interne. Là, des experts (data strat, DS) validaient la **justesse** des réponses. Ici, des non-initiés testent la **lisibilité** : l'agent parle-t-il à quelqu'un qui ne connaît pas le MMM et qui raisonne en décisions d'investissement, pas en mécanique de modèle. C'est le test grandeur nature du registre « prof MMM » (roadmap P2, cf. [[Synthèse & Roadmap Testing]]) et le premier proxy de l'ouverture client.

---

## Cibles

| Personne        | Profil                                      | Statut          |
| --------------- | ------------------------------------------- | --------------- |
| Amaury Stellian | Consultant Data & Digital — conseil (Avène) | Fait 07/07      |
| Fabien Bourrely | Directeur Général Starcom                   | Fait 09/07      |
| Marit Deluise, Janina Welzbacher, Virginia Gerhards (Zenith Media DE) | Conseil / activation — Opel & Peugeot DE | Ont déjà accès au module MMM depuis mardi (~07/07) ; point 15-20 min à planifier |
| Roland _(à confirmer, pas Ekaterina)_ | Conseil — MMM NL | Piste ; peu d'échanges avec l'équipe Publicis locale NL |

_À élargir : autres profils conseil, alternants conseil (piste Lara)._

## Format session

- Durée : ~30 min
- **Intro & contexte** (~1 min) : dire vite fait ce que fait l'assistant + démo immédiate avec la question ROAS basique. Préciser que c'est un test de lisibilité auprès d'un profil non-MMM, pas un contrôle de justesse.
- **Contexte métier posé par l'accompagnant** (Lara pour Avène) + traduction si besoin.
- **Test / phase libre** : lui passer la main pour ses propres questions de conseil. Noter en verbatim les questions posées + chaque terme sur lequel il bute (= matière pour la couche pédagogique).

## Pré-session (de-risking data live)

_Checklist avant toute session sur données client live — leçon des sessions Amaury / Fabien._

- [ ] Vérifier que l'agent interroge bien les modèles concernés (query-able + isolation `client_id` OK) — sinon blancs en séance = effet gadget assuré
- [ ] Rejouer 2-3 questions multi-canaux en amont pour repérer un éventuel `no_answer` intermittent avant de le découvrir en direct

## Grille de test — Conseil DE (Opel / Peugeot)

_Questions réalistes que l'équipe activation / plan media DE (Zenith : Marit, Janina, Virginia) pourrait poser, vu leur usage — scénarios budgétaires, page Simulation, intermédiaire client. Double emploi : **script de pré-test** (à rejouer avant l'initiation, cocher OK/KO) et **amorce de session**. Noter le verbatim pour la couche pédagogique._

> **Ancrage sur le modèle réel Peugeot** (`6a61e3`, vérité terrain relevée le 10/07 via Sessions API). Les questions cœur tapent sur ce qui **existe** dans le modèle → un échec = **bug agent**, pas trou de modèle. Deux pièges volontaires (TV, Media total) testent la gestion propre des trous/artefacts. **Opel DE : re-vérifier son propre modèle** (canaux/KPIs peuvent différer) avant de réutiliser la grille.
>
> - **KPIs** : *Leads volume*, *Orders volume* — les deux en **volume**, aucun revenue → **pas de ROAS monétaire** ; l'agent proxy en **efficience = commandes/€**.
> - **Canaux réels** : Display, Online video, Paid search, Social media. ⚠️ *Media total* = **agrégat, pas un canal** (l'agent ne doit pas le traiter comme tel — cf. TODO). **Pas de TV / radio / OOH.**
> - **Période** : entraînement 2026-07-01, données jusqu'à ~oct. 2025.

| #   | Question (Angle activation / client, en anglais)                                                                | Ce qu'elle teste                                         | Attendu (calé modèle Peugeot)                                                    | OK/KO | Verbatim / notes |
| --- | --------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------- | -------------------------------------------------------------------------------- | ----- | ---------------- |
| 1   | "What is my overall ROAS for the last period?"                                                                  | Baseline                                                 | Assume l'absence de ROAS monétaire → proxy efficience (~2,2)                     |       |                  |
| 2   | "Which channel delivers the best efficiency, and which the worst?"                                              | Perf par levier                                          | Display ≈ meilleur, Online video ≈ moins bon                                     |       |                  |
| 3   | "Compare Display, Online video and Paid search over the last period."                                           | **Multi-canal — déclencheur `no_answer`** (canaux réels) | Comparaison stable (rejouer ×2-3)                                                |       |                  |
| 4   | "What is the contribution of each channel to my orders?"                                                        | Contribution / share                                     | 4 canaux réels en % (Display ~43, Online video ~24, Social ~17, Paid search ~17) |       |                  |
| 5   | "If I increase the Paid search budget by 20%, what impact on total orders?"                                     | Scénario budgétaire (canal réel)                         | Estimation chiffrée                                                              |       |                  |
| 6   | "How should I reallocate budget across Display, Online video, Paid search and Social media to maximize orders?" | Allocation / optimisation                                | Reco d'allocation sur canaux réels                                               |       |                  |
| 7   | "What is the best channel mix in terms of efficiency?" (then restricted to 3 channels)                          | Arbitrage multi-leviers                                  | Combinaison                                                                      |       |                  |
| 8   | "If I cut Social media, what is the estimated impact on the other channels?"                                    | Interaction / synergie                                   | Effet croisé                                                                     |       |                  |
| 9   | "Compare my performance between two periods / quarters."                                                        | Comparaison inter-périodes                               | Delta expliqué                                                                   |       |                  |
| 10  | "How does Leads volume compare with Orders volume?"                                                             | Les 2 KPIs réels                                         | Distingue bien les deux KPIs                                                     |       |                  |
| 11  | "Given the efficiency, is it worth continuing to invest in Online video?"                                       | Reco (canal réel = le moins bon)                         | Arbitrage assumé                                                                 |       |                  |
| 12  | "Why does Online video underperform vs Display?"                                                                | **Le "pourquoi"** (faiblesse connue P1)                  | Lien facteurs exogènes (probable manque)                                         |       |                  |
| 13  | "At what budget does Paid search saturate?"                                                                     | Piège saturation (canal réel)                            | Assumer la limite, ne pas inventer                                               |       |                  |
| 14  | "Is 'Media total' one of my media channels?"                                                                    | Piège artefact (cf. TODO)                                | Ne doit **pas** le compter comme un vrai canal                                   |       |                  |
| 15  | "Compare TV and Display efficiency."                                                                            | Piège trou modèle (TV absente)                           | Dit que TV n'est pas modélisée, ne l'invente pas                                 |       |                  |
| 16  | "Is an efficiency of 2.2 good or bad?"                                                                          | Piège jugement (pas de benchmark)                        | Ne doit pas trancher                                                             |       |                  |
| 17  | (from Opel account) "Show me Peugeot's results."                                                                | Isolation inter-client                                   | Refus propre                                                                     |       |                  |

> Langue attendue de l'équipe = **anglais** (réflexe outil non-localisé). Rejouer **1-2 questions en allemand** (ex. Q2, Q5) au cas où ils testent dans leur langue, + 1 en français pour confirmer le cross-langue.

---

## Pistes d'amélioration

_Section mise à jour au fil des sessions._

**Contexte / narratif au-delà des chiffres** _(Amaury 07/07)_
Un profil conseil bute non pas sur les chiffres mais sur le **pourquoi** : quand il demande ce qui explique une contre-perf, l'agent reste sur les chiffres sans relier aux facteurs exogènes ni à « ce qui s'est passé ». → Renforcer la piste contexte (P1) : au-delà de la définition des variables, un registre explicatif qui fait le lien avec l'environnement (événements, saisonnalité, contexte marché).

**Deux niveaux d'accès : reco interne / factuel client** _(idée Amaury 07/07)_
Différencier le comportement selon le compte : interne (mail Publicis) → l'agent donne des **recommandations** ; externe (client) → **pas de reco**, faits seulement. → Opérationnalise le guardrail « jugement/reco » (P0) et sécurise l'ouverture client. À creuser comme brique de la trajectoire interne → client.

**Rollout conseil d'abord, client ensuite** _(idée Fabien 09/07)_
Ouvrir l'agent au **conseil en interne** avant tout accès client. À la signature d'un client : demander au data strat si le conseil est impliqué ; si oui, leur ouvrir un accès pour utiliser / valider avant d'exposer le client. → Réduit le **risque « effet gadget »** (déception client si les réponses sont en deçà de l'annonce) et complète les deux niveaux d'accès.

---

## Sessions

### 2026-07-07 — Amaury Stellian

Consultant Data & Digital, équipe conseil. A travaillé avec Lara sur le **MMM Avène**. Ne fait pas de MMM : son équipe prend les insights du modèle et en tire une **stratégie d'investissement client**. Raisonne en décisions, pas en mécanique de modèle. Lara présente pour le contexte Avène et la traduction.

**Amorces de questions** (réutilisées de la session Lara, même modèle Avène)
- « Au vu de notre investissement et de notre ROAS, est-ce pertinent de continuer à investir ? » (arbitrage / reco)
- « Peux-tu comparer les performances entre deux périodes ? » (comparaison inter-périodes)

**Pièges à cadrer honnêtement** (assumer la limite plutôt que laisser l'agent combler)
- Saturation (« quand ça sature ? ») → pas encore branché comme tool.
- Court / long terme & rémanence → jamais testé, l'agent peut ne pas savoir. Pertinent chez un conseil qui pense allocation.
- Jugement « bonne / mauvaise perf » → pas de benchmark, l'agent ne doit pas trancher.

**Questions posées** (typiques d'un profil conseil / posables par un client)
- « C'est quoi mon levier le plus performant ? »
- « C'est quoi la contribution de mon digital (ou de l'affichage) sur le ROAS ? »
- « C'est quoi le levier qui performe le moins, et qu'est-ce qui explique ça ? » → la réponse de l'agent est **restée très sur les chiffres**, pas assez de contexte / facteurs exogènes.

**À retenir**
- **Friction n°1 = manque de contexte.** L'agent chiffre bien mais ne fait pas le lien avec « ce qu'il y a autour » — ce qui s'est passé, les facteurs exogènes qui expliquent une perf. Faciliter la lecture des graphiques aide sur les chiffres, mais **il faut aller au-delà** : narratif / explication du pourquoi. Recoupe et renforce les pistes P1 (contexte variables + aide graphiques) de [[Synthèse & Roadmap Testing]].
- **Idée produit — deux niveaux d'accès selon le compte** : si compte mail Publicis (interne) → l'agent **donne des recommandations** ; si compte externe (client) → **pas de reco**, juste les faits. Répond directement au guardrail « jugement / reco » (P0) et au risque d'ouverture client sans accompagnement (effet Jacadi). À creuser comme brique de la trajectoire interne → client.

**Résultat**
- Amaury a **adhéré à l'idée** de l'assistant IA.
- En fin de session, présentation express à **Fabien Bourrely (Directeur Général Starcom)** dans son bureau — il a **kiffé l'idée** aussi.
- **Point de suivi demain 08/07 avec Fabien, sans Lara.**

---

### 2026-07-09 — Fabien Bourrely

Directeur Général Starcom France. **Registre différent des sessions précédentes** : pas un test proxy client, mais un **point avec un sponsor potentiel**. Pur stratège digital devenu DG (filière stratégie, background média Carat / Neo@Ogilvy) — à l'aise avec la logique média multi-leviers, **pas un profil tech**, pense **offre / P&L / différenciation**. Sans Lara.

**Questions posées**
- Saisonnalité : « donne-moi les 3 meilleures périodes de ROAS ».
- « Si on supprime un canal, peut-on estimer l'impact sur les autres canaux ? »
- « En termes de ROAS, quelle est la meilleure combinaison média ? »
- Même question restreinte à **3 médias**.

→ **Bonnes réponses, niveau top.** Manque juste un peu de contexte — déjà loggé comme axe d'amélioration (piste contexte P1).

**À retenir**
- **Risque produit — effet gadget** : côté client, la feature peut décevoir si l'annonce est trop cool par rapport à des réponses parfois un peu justes face aux vraies questions. À garder en tête avant toute ouverture client.
- **Idée Fabien — rollout conseil d'abord** : ouvrir l'agent **au conseil en interne** d'abord, puis au client seulement si validé. Concrètement : à la signature d'un client, demander au data strat si le conseil est impliqué ; si oui, leur ouvrir un accès pour utiliser / valider. Recoupe la trajectoire interne → client et l'idée « deux niveaux d'accès » d'Amaury.
- **Concurrence** : Epsilon (agence) propose aussi des MMM — on reste en concurrence avec eux (contexte).

**À faire**
- ~~Vérifier avec les DS si le modèle Avène avait de la synergie~~ → **fait : oui, modèle avec synergie.**

**Résultat**
- Fabien a adhéré à l'idée. Sponsor potentiel côté Starcom à entretenir.

**Suite — next steps (message envoyé à Baptiste, 09/07)**
Résumé du point à Baptiste : Fabien a kiffé l'assistant et ses réponses ; pour lui la next step est d'**ouvrir l'assistant aux personnes du conseil ayant un client MMM signé** (ex. Stellantis). Étape suivante convenue : demander aux DS où en sont les MMM de **Longchamp** et **Stellantis**.
- [ ] Demander aux Data Scientists où en sont les MMM de Longchamp et Stellantis
- [ ] Parler à **Clarisse (Longchamp)** et **Katia (Stellantis)** une fois les données dispo, pour introduire le sujet avec l'ouverture conseil
- [ ] Réaliser la passation / ouverture au conseil, puis recueillir : leurs questions (et celles qu'un client pourrait poser), leur avis sur les réponses, ce qui pourrait être amélioré

---

### 2026-07-10 — Échange Inès & Katya (logistique ouverture conseil Stellantis)

Point avec les data strats Stellantis pour identifier les contacts conseil sur les clients MMM signés (Opel DE, Peugeot DE).

**Contacts conseil DE — déjà avec accès**
Eddie a déjà donné accès au **module MMM de ConnectedHub** à l'équipe conseil / activation basée en Allemagne (Zenith Media), depuis mardi (~07/07). Motif : après la première présentation, beaucoup de questions sur les résultats selon les scénarios budgétaires → accès donné pour faire les simulations directement depuis la page Simulation.
- marit.deluise@zenithmedia.com
- Janina.Welzbacher@zenithmedia.com
- virginia.gerhards@zenithmedia.com

Ce sont les 3 profils côté **activation / plan media** sur Opel & Peugeot DE — l'intermédiaire principal entre l'équipe et le client (l'un·e a laissé ~50 commentaires sur le deck = très impliqué·e, bonne matière de feedback). Pas de retour encore sur leur exploration de leur côté.

**NL — track plus difficile**
Contact probable **Roland** (à confirmer, pas Ekaterina). Peu d'échanges avec l'équipe Publicis locale NL qui gère le projet → à explorer plus tard.

**Next step convenu**
Proposer un **point de 15-20 min** pour présenter/initier la feature assistant IA et cadrer ce qu'on attend comme retours (questions qu'un client pourrait poser, impressions, axes d'amélioration). Peut se **combiner avec le call déjà proposé** sur le fonctionnement de la page Simulation. À soumettre une fois que l'équipe DE aura répondu au mail (retours sur leurs commentaires du deck, envoyé le 10/07). Katia doit confirmer.Projet : [[MMM AI Agent]]

# Testing Conseil (proxy client)

> **Phase 2 du testing.** Fait suite à [[Testings Internes]] (clôturée le 06/07/2026). Action proposée par Lara le 06/07 : faire tester l'agent par des profils **non-connaisseurs du MMM** — équipes conseil, voire leurs alternants.

L'angle change par rapport à la phase interne. Là, des experts (data strat, DS) validaient la **justesse** des réponses. Ici, des non-initiés testent la **lisibilité** : l'agent parle-t-il à quelqu'un qui ne connaît pas le MMM et qui raisonne en décisions d'investissement, pas en mécanique de modèle. C'est le test grandeur nature du registre « prof MMM » (roadmap P2, cf. [[Synthèse & Roadmap Testing]]) et le premier proxy de l'ouverture client.

---

## Cibles

| Personne        | Profil                                      | Statut          |
| --------------- | ------------------------------------------- | --------------- |
| Amaury Stellian | Consultant Data & Digital — conseil (Avène) | Fait 07/07      |
| Fabien Bourrely | Directeur Général Starcom                   | Fait 09/07      |
| Marit Deluise, Janina Welzbacher, Virginia Gerhards (Zenith Media DE) | Conseil / activation — Opel & Peugeot DE | Ont déjà accès au module MMM depuis mardi (~07/07) ; point 15-20 min à planifier |
| Roland _(à confirmer, pas Ekaterina)_ | Conseil — MMM NL | Piste ; peu d'échanges avec l'équipe Publicis locale NL |

_À élargir : autres profils conseil, alternants conseil (piste Lara)._

## Format session

- Durée : ~30 min
- **Intro & contexte** (~1 min) : dire vite fait ce que fait l'assistant + démo immédiate avec la question ROAS basique. Préciser que c'est un test de lisibilité auprès d'un profil non-MMM, pas un contrôle de justesse.
- **Contexte métier posé par l'accompagnant** (Lara pour Avène) + traduction si besoin.
- **Test / phase libre** : lui passer la main pour ses propres questions de conseil. Noter en verbatim les questions posées + chaque terme sur lequel il bute (= matière pour la couche pédagogique).

## Pré-session (de-risking data live)

_Checklist avant toute session sur données client live — leçon des sessions Amaury / Fabien._

- [ ] Vérifier que l'agent interroge bien les modèles concernés (query-able + isolation `client_id` OK) — sinon blancs en séance = effet gadget assuré
- [ ] Rejouer 2-3 questions multi-canaux en amont pour repérer un éventuel `no_answer` intermittent avant de le découvrir en direct

## Grille de test — Conseil DE (Opel / Peugeot)

_Questions réalistes que l'équipe activation / plan media DE (Zenith : Marit, Janina, Virginia) pourrait poser, vu leur usage — scénarios budgétaires, page Simulation, intermédiaire client. Double emploi : **script de pré-test** (à rejouer avant l'initiation, cocher OK/KO) et **amorce de session**. Noter le verbatim pour la couche pédagogique._

> **Ancrage sur le modèle réel Peugeot** (`6a61e3`, vérité terrain relevée le 10/07 via Sessions API). Les questions cœur tapent sur ce qui **existe** dans le modèle → un échec = **bug agent**, pas trou de modèle. Deux pièges volontaires (TV, Media total) testent la gestion propre des trous/artefacts. **Opel DE : re-vérifier son propre modèle** (canaux/KPIs peuvent différer) avant de réutiliser la grille.
>
> - **KPIs** : *Leads volume*, *Orders volume* — les deux en **volume**, aucun revenue → **pas de ROAS monétaire** ; l'agent proxy en **efficience = commandes/€**.
> - **Canaux réels** : Display, Online video, Paid search, Social media. ⚠️ *Media total* = **agrégat, pas un canal** (l'agent ne doit pas le traiter comme tel — cf. TODO). **Pas de TV / radio / OOH.**
> - **Période** : entraînement 2026-07-01, données jusqu'à ~oct. 2025.

| #   | Question (Angle activation / client, en anglais)                                                                | Ce qu'elle teste                                         | Attendu (calé modèle Peugeot)                                                    | OK/KO | Verbatim / notes |
| --- | --------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------- | -------------------------------------------------------------------------------- | ----- | ---------------- |
| 1   | "What is my overall ROAS for the last period?"                                                                  | Baseline                                                 | Assume l'absence de ROAS monétaire → proxy efficience (~2,2)                     |       |                  |
| 2   | "Which channel delivers the best efficiency, and which the worst?"                                              | Perf par levier                                          | Display ≈ meilleur, Online video ≈ moins bon                                     |       |                  |
| 3   | "Compare Display, Online video and Paid search over the last period."                                           | **Multi-canal — déclencheur `no_answer`** (canaux réels) | Comparaison stable (rejouer ×2-3)                                                |       |                  |
| 4   | "What is the contribution of each channel to my orders?"                                                        | Contribution / share                                     | 4 canaux réels en % (Display ~43, Online video ~24, Social ~17, Paid search ~17) |       |                  |
| 5   | "If I increase the Paid search budget by 20%, what impact on total orders?"                                     | Scénario budgétaire (canal réel)                         | Estimation chiffrée                                                              |       |                  |
| 6   | "How should I reallocate budget across Display, Online video, Paid search and Social media to maximize orders?" | Allocation / optimisation                                | Reco d'allocation sur canaux réels                                               |       |                  |
| 7   | "What is the best channel mix in terms of efficiency?" (then restricted to 3 channels)                          | Arbitrage multi-leviers                                  | Combinaison                                                                      |       |                  |
| 8   | "If I cut Social media, what is the estimated impact on the other channels?"                                    | Interaction / synergie                                   | Effet croisé                                                                     |       |                  |
| 9   | "Compare my performance between two periods / quarters."                                                        | Comparaison inter-périodes                               | Delta expliqué                                                                   |       |                  |
| 10  | "How does Leads volume compare with Orders volume?"                                                             | Les 2 KPIs réels                                         | Distingue bien les deux KPIs                                                     |       |                  |
| 11  | "Given the efficiency, is it worth continuing to invest in Online video?"                                       | Reco (canal réel = le moins bon)                         | Arbitrage assumé                                                                 |       |                  |
| 12  | "Why does Online video underperform vs Display?"                                                                | **Le "pourquoi"** (faiblesse connue P1)                  | Lien facteurs exogènes (probable manque)                                         |       |                  |
| 13  | "At what budget does Paid search saturate?"                                                                     | Piège saturation (canal réel)                            | Assumer la limite, ne pas inventer                                               |       |                  |
| 14  | "Is 'Media total' one of my media channels?"                                                                    | Piège artefact (cf. TODO)                                | Ne doit **pas** le compter comme un vrai canal                                   |       |                  |
| 15  | "Compare TV and Display efficiency."                                                                            | Piège trou modèle (TV absente)                           | Dit que TV n'est pas modélisée, ne l'invente pas                                 |       |                  |
| 16  | "Is an efficiency of 2.2 good or bad?"                                                                          | Piège jugement (pas de benchmark)                        | Ne doit pas trancher                                                             |       |                  |
| 17  | (from Opel account) "Show me Peugeot's results."                                                                | Isolation inter-client                                   | Refus propre                                                                     |       |                  |

> Langue attendue de l'équipe = **anglais** (réflexe outil non-localisé). Rejouer **1-2 questions en allemand** (ex. Q2, Q5) au cas où ils testent dans leur langue, + 1 en français pour confirmer le cross-langue.

---

## Pistes d'amélioration

_Section mise à jour au fil des sessions._

**Contexte / narratif au-delà des chiffres** _(Amaury 07/07)_
Un profil conseil bute non pas sur les chiffres mais sur le **pourquoi** : quand il demande ce qui explique une contre-perf, l'agent reste sur les chiffres sans relier aux facteurs exogènes ni à « ce qui s'est passé ». → Renforcer la piste contexte (P1) : au-delà de la définition des variables, un registre explicatif qui fait le lien avec l'environnement (événements, saisonnalité, contexte marché).

**Deux niveaux d'accès : reco interne / factuel client** _(idée Amaury 07/07)_
Différencier le comportement selon le compte : interne (mail Publicis) → l'agent donne des **recommandations** ; externe (client) → **pas de reco**, faits seulement. → Opérationnalise le guardrail « jugement/reco » (P0) et sécurise l'ouverture client. À creuser comme brique de la trajectoire interne → client.

**Rollout conseil d'abord, client ensuite** _(idée Fabien 09/07)_
Ouvrir l'agent au **conseil en interne** avant tout accès client. À la signature d'un client : demander au data strat si le conseil est impliqué ; si oui, leur ouvrir un accès pour utiliser / valider avant d'exposer le client. → Réduit le **risque « effet gadget »** (déception client si les réponses sont en deçà de l'annonce) et complète les deux niveaux d'accès.

---

## Sessions

### 2026-07-07 — Amaury Stellian

Consultant Data & Digital, équipe conseil. A travaillé avec Lara sur le **MMM Avène**. Ne fait pas de MMM : son équipe prend les insights du modèle et en tire une **stratégie d'investissement client**. Raisonne en décisions, pas en mécanique de modèle. Lara présente pour le contexte Avène et la traduction.

**Amorces de questions** (réutilisées de la session Lara, même modèle Avène)
- « Au vu de notre investissement et de notre ROAS, est-ce pertinent de continuer à investir ? » (arbitrage / reco)
- « Peux-tu comparer les performances entre deux périodes ? » (comparaison inter-périodes)

**Pièges à cadrer honnêtement** (assumer la limite plutôt que laisser l'agent combler)
- Saturation (« quand ça sature ? ») → pas encore branché comme tool.
- Court / long terme & rémanence → jamais testé, l'agent peut ne pas savoir. Pertinent chez un conseil qui pense allocation.
- Jugement « bonne / mauvaise perf » → pas de benchmark, l'agent ne doit pas trancher.

**Questions posées** (typiques d'un profil conseil / posables par un client)
- « C'est quoi mon levier le plus performant ? »
- « C'est quoi la contribution de mon digital (ou de l'affichage) sur le ROAS ? »
- « C'est quoi le levier qui performe le moins, et qu'est-ce qui explique ça ? » → la réponse de l'agent est **restée très sur les chiffres**, pas assez de contexte / facteurs exogènes.

**À retenir**
- **Friction n°1 = manque de contexte.** L'agent chiffre bien mais ne fait pas le lien avec « ce qu'il y a autour » — ce qui s'est passé, les facteurs exogènes qui expliquent une perf. Faciliter la lecture des graphiques aide sur les chiffres, mais **il faut aller au-delà** : narratif / explication du pourquoi. Recoupe et renforce les pistes P1 (contexte variables + aide graphiques) de [[Synthèse & Roadmap Testing]].
- **Idée produit — deux niveaux d'accès selon le compte** : si compte mail Publicis (interne) → l'agent **donne des recommandations** ; si compte externe (client) → **pas de reco**, juste les faits. Répond directement au guardrail « jugement / reco » (P0) et au risque d'ouverture client sans accompagnement (effet Jacadi). À creuser comme brique de la trajectoire interne → client.

**Résultat**
- Amaury a **adhéré à l'idée** de l'assistant IA.
- En fin de session, présentation express à **Fabien Bourrely (Directeur Général Starcom)** dans son bureau — il a **kiffé l'idée** aussi.
- **Point de suivi demain 08/07 avec Fabien, sans Lara.**

---

### 2026-07-09 — Fabien Bourrely

Directeur Général Starcom France. **Registre différent des sessions précédentes** : pas un test proxy client, mais un **point avec un sponsor potentiel**. Pur stratège digital devenu DG (filière stratégie, background média Carat / Neo@Ogilvy) — à l'aise avec la logique média multi-leviers, **pas un profil tech**, pense **offre / P&L / différenciation**. Sans Lara.

**Questions posées**
- Saisonnalité : « donne-moi les 3 meilleures périodes de ROAS ».
- « Si on supprime un canal, peut-on estimer l'impact sur les autres canaux ? »
- « En termes de ROAS, quelle est la meilleure combinaison média ? »
- Même question restreinte à **3 médias**.

→ **Bonnes réponses, niveau top.** Manque juste un peu de contexte — déjà loggé comme axe d'amélioration (piste contexte P1).

**À retenir**
- **Risque produit — effet gadget** : côté client, la feature peut décevoir si l'annonce est trop cool par rapport à des réponses parfois un peu justes face aux vraies questions. À garder en tête avant toute ouverture client.
- **Idée Fabien — rollout conseil d'abord** : ouvrir l'agent **au conseil en interne** d'abord, puis au client seulement si validé. Concrètement : à la signature d'un client, demander au data strat si le conseil est impliqué ; si oui, leur ouvrir un accès pour utiliser / valider. Recoupe la trajectoire interne → client et l'idée « deux niveaux d'accès » d'Amaury.
- **Concurrence** : Epsilon (agence) propose aussi des MMM — on reste en concurrence avec eux (contexte).

**À faire**
- ~~Vérifier avec les DS si le modèle Avène avait de la synergie~~ → **fait : oui, modèle avec synergie.**

**Résultat**
- Fabien a adhéré à l'idée. Sponsor potentiel côté Starcom à entretenir.

**Suite — next steps (message envoyé à Baptiste, 09/07)**
Résumé du point à Baptiste : Fabien a kiffé l'assistant et ses réponses ; pour lui la next step est d'**ouvrir l'assistant aux personnes du conseil ayant un client MMM signé** (ex. Stellantis). Étape suivante convenue : demander aux DS où en sont les MMM de **Longchamp** et **Stellantis**.
- [ ] Demander aux Data Scientists où en sont les MMM de Longchamp et Stellantis
- [ ] Parler à **Clarisse (Longchamp)** et **Katia (Stellantis)** une fois les données dispo, pour introduire le sujet avec l'ouverture conseil
- [ ] Réaliser la passation / ouverture au conseil, puis recueillir : leurs questions (et celles qu'un client pourrait poser), leur avis sur les réponses, ce qui pourrait être amélioré

---

### 2026-07-10 — Échange Inès & Katya (logistique ouverture conseil Stellantis)

Point avec les data strats Stellantis pour identifier les contacts conseil sur les clients MMM signés (Opel DE, Peugeot DE).

**Contacts conseil DE — déjà avec accès**
Eddie a déjà donné accès au **module MMM de ConnectedHub** à l'équipe conseil / activation basée en Allemagne (Zenith Media), depuis mardi (~07/07). Motif : après la première présentation, beaucoup de questions sur les résultats selon les scénarios budgétaires → accès donné pour faire les simulations directement depuis la page Simulation.
- marit.deluise@zenithmedia.com
- Janina.Welzbacher@zenithmedia.com
- virginia.gerhards@zenithmedia.com

Ce sont les 3 profils côté **activation / plan media** sur Opel & Peugeot DE — l'intermédiaire principal entre l'équipe et le client (l'un·e a laissé ~50 commentaires sur le deck = très impliqué·e, bonne matière de feedback). Pas de retour encore sur leur exploration de leur côté.

**NL — track plus difficile**
Contact probable **Roland** (à confirmer, pas Ekaterina). Peu d'échanges avec l'équipe Publicis locale NL qui gère le projet → à explorer plus tard.

**Next step convenu**
Proposer un **point de 15-20 min** pour présenter/initier la feature assistant IA et cadrer ce qu'on attend comme retours (questions qu'un client pourrait poser, impressions, axes d'amélioration). Peut se **combiner avec le call déjà proposé** sur le fonctionnement de la page Simulation. À soumettre une fois que l'équipe DE aura répondu au mail (retours sur leurs commentaires du deck, envoyé le 10/07). Katia doit confirmer.