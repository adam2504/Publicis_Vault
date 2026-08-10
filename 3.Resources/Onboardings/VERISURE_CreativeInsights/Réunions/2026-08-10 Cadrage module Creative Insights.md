---
type: meeting
date: 2026-08-10
---

Projet : [[Creative Insights Verisure]]
Plateforme : [[Mine Platform]]

# 10 Août — Cadrage automatisation Creative Insights (module ConnectedHub)

> ⚪ Note de **préparation**. Le compte-rendu se remplit pendant/après la réunion (sections Décisions et Actions).

Participants : Hajar (DS, porte le POC), Abhishek (DS, PGD, piste Leapmotor), moi.
Absente à confirmer : Léonie (Data Strat, interlocutrice features côté Verisure).

---

## Objet

Transformer les Creative Insights, aujourd'hui un POC porté par les DS, en **module ConnectedHub**. Périmètre d'entrée retenu : **Verisure d'abord**, les autres comptes (Leapmotor) en cas d'usage suivant.

---

## Ce que le vault sait déjà

| Élément | État |
| --- | --- |
| POC Verisure | Porté par Hajar (DS) et Léonie (Data Strat). Une seule réunion tracée : [[2026-06-18 Point avec Léonie & Hajar]] |
| Liste de features | Léonie a une liste de features demandées par le client. Elle n'a **jamais été écrite dans le vault** |
| Piste Leapmotor | [[Creative Insights Leapmotor]], idée de module ConnectedHub, statut « en attente d'Abhishek », **aucune relance depuis la création** |
| Mon implication | `onboardé` sur Verisure. Aucune contribution au livrable à ce jour |

**Trou de contexte assumé** : je n'ai aucune trace de ce que produit concrètement le POC (quelles données en entrée, quel type d'insight en sortie, sous quelle forme il est livré au client aujourd'hui). C'est le premier bloc de questions.

---

## Questions à poser

### À Hajar (le POC existant)

- Qu'est-ce qui est produit aujourd'hui, concrètement ? Sortie = fichier, deck, notebook, dashboard ?
- Quelles données en entrée, et d'où : plateformes média, assets créa, taxonomie de nommage des créas ?
- Quelle est la partie **manuelle et répétitive** du process actuel ? C'est elle qui définit ce que « automatisation » veut dire ici.
- À quelle fréquence le livrable est-il refait (par campagne, mensuel, ad hoc) ? Le volume justifie-t-il un module ?
- Où tourne le code aujourd'hui, et est-il réutilisable en l'état ou à réécrire ?
- Où en est la liste de features de Léonie, et est-elle arbitrée ou encore une liste de souhaits ?

### À Abhishek (généralisation)

- Le besoin Leapmotor est-il le même que Verisure, ou juste le même mot ?
- Qu'est-ce qui bloquait depuis la création de la piste : priorité, données, ou absence de demandeur ?
- Sur SIMBA et FeedGen, existe-t-il déjà de la brique réutilisable côté ingestion ou catégorisation de créas ?

### Aux deux (cadrage module)

- Qui sont les utilisateurs cibles dans ConnectedHub : DS, conseil, ou client final ? Ça change tout le front.
- Un module transverse multi-clients, ou un module Verisure qu'on généralise ensuite ?

---

## Points à trancher

1. **Automatiser quoi.** Le pipeline de calcul (côté DS), la restitution (côté module), ou les deux. Tant que ce n'est pas tranché, « Creative Insight Automation » ne veut rien dire de précis.
2. **Mon rôle.** Je passe de `onboardé` à contributeur ou lead, ou je reste en support. Si je porte le module, la note Verisure sort de `3.Resources/Onboardings/` pour aller dans `1.Projects/` (dossier `MINE_CreativeInsights` ou `VERISURE_CreativeInsights` selon le point 3).
3. **Client de rattachement.** Module plateforme `MINE` vendable à plusieurs comptes, ou livrable `VERISURE` spécifique. Détermine le nommage du dossier et le rattachement au hub.
4. **Qui possède la logique métier.** Si la définition d'un insight reste chez les DS, le module n'est qu'une couche de restitution et il faut un contrat d'interface clair (table BQ ou API).
5. **Sponsor et priorité.** À valider avec Jules et Manu avant d'engager du temps de dev : je suis déjà lead sur AMC Analytics et MMM AI Agent.

---

## Ce que j'apporte (à dire si utile)

- Le pattern module ConnectedHub est déjà rodé deux fois : [[1.Projects/MINE_AMCAnalytics/AMC Analytics]] (ingestion BQ, workspace pivot, config Firestore par customer) et [[1.Projects/MINE_MMMAIAgent/MMM AI Agent]] (couche IA sur données modèle).
- Leçon AMC directement transposable : **ne pas livrer un deck de vues figées**. Le retour de Jules (« on ne peut pas faire ce qu'on veut ») a imposé une refonte complète en workspace. Si les Creative Insights partent en slides figées, on repayera la même refonte.
- Deuxième leçon AMC : les doubles comptes viennent de l'empilement de niveaux d'analyse dans une même table. À poser dès le contrat d'ingestion, pas après.

---

## Décisions

> À remplir pendant la réunion.

---

## Actions

> À remplir pendant la réunion.

| Action | Qui | Pour quand |
| --- | --- | --- |
| | | |
