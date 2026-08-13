---
type: area
---

# TODO

## MINE - Module MMM

- [ ] Ajouter dans Simulation -> Optimisation, que l'objectif est journalier, comme pour ventiler un budget

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

- [ ] Suivre le ticket IT : rôles sur `zen-creativeinsights-dev-mg` pour `interface@pmed-portal-prd-mg.iam.gserviceaccount.com`
- [ ] Relancer Hajar sur la granularité par projet (`project_id` dans les CF et la table d'assets) — **débloque tout le reste**
- [ ] Demander une colonne `asset_source` écrite à la collecte (le scrapper connaît la branche, ne l'enregistre pas)
- [ ] **TikTok et Snapchat ne remontent pas `ad_id`** : aucune jointure performance possible. Prérequis DS avant tout dev multi-plateformes
- [ ] Faire évoluer les Cloud Run : filtre de période, multi-campagnes, retour HTTP honnête (200 en cas d'erreur)
- [ ] Trancher la règle d'attribution performance `ad_id` vers `asset_id` (un ad porte plusieurs assets, risque de double-compte façon AMC)
- [ ] Proposer le renversement du modèle : `analysis_json` canonique, colonnes par client en vues dérivées
- [ ] Trancher où le module s'arrête : export CSV avec le dashboard de Thomas, ou restitution native
- [ ] Récupérer la liste de features client de Léonie, jamais formalisée
- [ ] Étape 4 : localiser les prompts Gemini et l'exécution de l'extraction (absents du projet GCP exploré)
- [ ] Signaler à Eddie deux défauts du repo : `Button` variant `secondary` typé mais sans style, `onRowClick.logic` mal typé

## Réseau/Rencontres

- [ ] Parler avec Anael Cabrol
- [ ] Parler avec Yann Legrand
