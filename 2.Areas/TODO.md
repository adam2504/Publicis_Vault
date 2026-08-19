---
type: area
---
# TODO

## MINE - MMM AI Agent

### Passation à Dan (deadline 21/08, dernier jour)

- [x] Pousser le code dans le repo GitHub d'équipe `Publicis-Media-France-FR5140/MMM_AI_Agent`
- [x] Enlever les commentaires du code
- [x] Faire la liste des rôles GCP nécessaires pour Dan
- [x] Rédiger la doc complète — **reste à la coller dans Confluence**
- [x] Modifier l'email d'alerte `no_answer` — channel de Dan rattaché en parallèle
- [ ] **Confirmer avec Dan qu'il reçoit l'alerte, puis retirer mon channel `5134431245708235828`** (avant le 04/09)
- [ ] **Envoyer la demande d'accès GCP pour Dan** (`danphan2@publicisgroupe.net`) — deux projets, `pmed-portal-prd-mg` est le circuit le plus lent
- [ ] Donner à Dan le droit d'écriture sur le repo GitHub

### Backlog produit (transmis)

- [ ] Réaliser passation/ouverture au conseil, puis demander questions qu'ils ont/client pourrait avoir, ce qu'ils pensent des réponses, ce qui pourrait être améliorer etc
- [ ] Améliorer l'agent avec feedbacks
  - [ ] Enrichir le `BUSINESS_CONTEXT` avec les DS (définitions variables/données)
  - [ ] Tools saturation + allocation (`cf-budget-allocator-prod`) + registre narratif
  - [ ] Explorer possibilité de knowledge par client

## MINE × L'OREAL - AMC Analytics

- [ ] Réaliser feedbacks PowerPoint de Jules
- [ ] Explorer feedbakcs mail de Jules
- [ ] Traduire dashboards Looker en native React

## MINE - Creative Insights

- [ ] Suivre le ticket IT : rôles sur `zen-creativeinsights-dev-mg` pour `interface@pmed-portal-prd-mg.iam.gserviceaccount.com`
- [ ] Relancer Hajar sur la granularité par projet (`project_id` dans les CF et la table d'assets) — **débloque tout le reste**
- [ ] **TikTok et Snapchat ne remontent pas `ad_id`** : aucune jointure performance possible. Prérequis DS avant tout dev multi-plateformes
- [ ] Faire évoluer les Cloud Run : filtre de période, multi-campagnes, retour HTTP honnête (200 en cas d'erreur)
- [ ] Trancher la règle d'attribution performance `ad_id` vers `asset_id` (un ad porte plusieurs assets, risque de double-compte façon AMC)
- [ ] Proposer le renversement du modèle : `analysis_json` canonique, colonnes par client en vues dérivées
- [ ] Trancher où le module s'arrête : export CSV avec le dashboard de Thomas, ou restitution native
- [ ] Récupérer la liste de features client de Léonie, jamais formalisée
- [ ] Étape 4 : localiser les prompts Gemini et l'exécution de l'extraction (absents du projet GCP exploré)
- [ ] Signaler à Eddie deux défauts du repo : `Button` variant `secondary` typé mais sans style, `onRowClick.logic` mal typé
