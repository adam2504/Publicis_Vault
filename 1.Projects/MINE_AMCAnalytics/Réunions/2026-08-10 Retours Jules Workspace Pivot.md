---
type: meeting
date: 2026-08-10
projet: "[[AMC Analytics]]"
participants: Jules
---

# 2026-08-10 Retours Jules Workspace Pivot

Projet : [[AMC Analytics]]
Plateforme : [[Mine Platform]]

## Participants

- Jules
- Adam

## Contexte

Premier passage en revue du workspace pivot par Jules après le merge des PR #1719 à #1723. Retours d'usage sur les vues existantes, les cas d'usage manquants et la navigation.

## Retours sur les vues

- **Mono vs multi** : une catégorie vide apparaît en plus, à comprendre et à supprimer.
- **Nombre de touches** : s'arrêter à 5 ou 6 touches (ou aligner le seuil sur ce que porte le parcours).
- **Titre de vue** : chaque vue doit afficher en haut la question à laquelle elle répond.
- **Place des leviers** : allonger le graphique, et ajouter une option d'affichage en absolu (plusieurs barres côte à côte par channel) en complément du relatif.
- **Score cards** : inverser la hiérarchie, le chiffre du filtre en grand, le total en petit.

## Cas d'usage à ajouter

- **Entrée et sortie simultanées** sur les leviers, pour voir si un levier est démarreur ou finisseur.
- **Analysis levels non exploités** : `global`, `time of`, `campaign performance`. À noter : `campaign performance` va changer dans les futurs templates pour devenir de la donnée découpée par campagne.
- **Question fréquente à couvrir** : quelles combinaisons de leviers génèrent le plus de reach et de conversions ?

## Navigation et structure

- **Slicer `analysis level` inutile** : un cas d'usage est d'office lié à un analysis level, donc pas besoin de l'exposer. À la place, créer un onglet d'exploration réellement libre, où tout est réglable.
- **Export du lien de vue sans intérêt en l'état** : en cliquant sur le lien, on retombe sur le choix du dataset, la vue est perdue.
- **S'inspirer de Meta Advanced Analytics** : enregistrer les analyses puis y accéder via un historique, pour ne plus repasser par le choix du dataset et le regroupement des leviers à chaque fois.

## Actions

| Action | Responsable | Deadline |
| ------ | ----------- | -------- |
| Corriger la catégorie vide sur mono vs multi | Adam | À définir |
| Plafonner le nombre de touches à 5 ou 6 | Adam | À définir |
| Ajouter la question traitée en tête de chaque vue | Adam | À définir |
| Place des leviers : graphique allongé + bascule relatif / absolu | Adam | À définir |
| Inverser la hiérarchie des score cards | Adam | À définir |
| Ajouter le cas d'usage entrée / sortie des leviers | Adam | À définir |
| Ajouter des cas d'usage sur les analysis levels non utilisés | Adam | À définir |
| Retirer le slicer analysis level + créer un onglet exploration libre | Adam | À définir |
| Corriger le lien de vue partagé (ne plus repasser par le choix du dataset) | Adam | À définir |
| Étudier l'historique d'analyses façon Meta Advanced Analytics | Adam | À définir |
