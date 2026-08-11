---
type: area
---
# TODO

MINE - MMM Module 
Ajouter dans Simulation -> Optimisation, que l'objectif est journalier, comme pour ventiler un budget

## MINE - MMM AI Agent

- [ ] Réaliser passation/ouverture au conseil, puis demander questions qu'ils ont/client pourrait avoir, ce qu'ils pensent des réponses, ce qui pourrait être améliorer etc
- [ ] Améliorer l'agent avec feedbacks
	- [ ] Enrichir le `BUSINESS_CONTEXT` avec les DS (définitions variables/données)
	- [ ] Tools saturation + allocation (`cf-budget-allocator-prod`) + registre narratif
	- [ ] Explorer possibilité de knowledge par client
## MINE × L'OREAL - AMC Analytics

- [ ] Avancer module avec feedbacks de Jules
- [ ] Traduire dashboards Looker en native React

## MINE - Creative Insights

- [ ] **Remonter à Hajar et Abhishek : TikTok et Snapchat ne remontent pas `ad_id`**, donc pas de jointure performance possible sur ces deux plateformes. Prérequis DS avant tout dev multi-plateformes
- [ ] Faire évoluer les Cloud Run : paramètre de période, notion de projet/client (destinations codées en dur), retour HTTP honnête (elles renvoient 200 en cas d'erreur)
- [ ] Trancher la règle d'attribution performance `ad_id` vers `asset_id` (un ad porte plusieurs assets, risque de double-compte façon AMC)
- [ ] Proposer le renversement du modèle : `analysis_json` canonique, colonnes par client en vues dérivées
- [ ] Trancher où le module s'arrête : export CSV avec le dashboard de Thomas, ou restitution native
- [ ] Récupérer la liste de features client de Léonie, jamais formalisée
- [ ] Trancher avec Eddie où vit l'état du projet : BigQuery comme SIMBA, ou Firestore comme feed-manager

## Réseau/Rencontres

- [ ] Parler avec Anael Cabrol
- [ ] Parler avec Yann Legrand