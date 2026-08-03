---
type: client
---

# Stellantis

Groupe automobile (Peugeot, Citroën, Opel, Fiat…). Publicis reprend une partie des marchés européens sur le custom bidding, avec sa propre méthodologie — Dragonfly (équipe interne UK) gérait auparavant les POC EVBB en France.

Stellantis est la **première cible de déploiement** de [[1.Projects/MINE_MMMAIAgent/MMM AI Agent]] : le modèle MMM est dispo (Opel DE / Peugeot DE), rien ne bloque côté data. Le conseil / activation DE (Zenith Media) a déjà accès au module MMM de ConnectedHub, mais ne répond plus depuis le mail deck — à relancer via Inès et Katia.

## Projets

| Projet       | Discipline   | Implication | Note                                                              |
| ------------ | ------------ | ----------- | ----------------------------------------------------------------- |
| EVBB Scoring | data-science | onboardé    | [[3.Resources/Onboardings/STELLANTIS_EVBBScoring/EVBB Scoring]]   |

> ⚠️ `implication: onboardé` — la note EVBB Scoring documente le travail de **Dan**, pas le mien.

## Contacts

| Personne            | Rôle                                             |
| ------------------- | ------------------------------------------------ |
| Emilie              | Trader / Business — pilote EVBB                  |
| Pierre              | Data Strategist                                  |
| Dan                 | Data Scientist — porte le pipeline EVBB          |
| Inès                | Data Strat — contact ouverture conseil MMM       |
| Katia               | Data Strat — contact ouverture conseil MMM       |
| Thibaut Vouloir     | Création de comptes Stellantis                   |
| Iker Zabala Ibanez  | Contact technique — accès BigQuery               |

## Infrastructure

- Projet de travail : `gbl-psa-analytics-publicis` (dataset `enhanced_vbb_pmf`)
- Données GA4 (read only) : `gbl-psa-analytics-cdf-prod`, dataset `analytics_ga4_eu`
- Comptes Stellantis distincts des comptes Publicis — navigateur séparé ou navigation privée
