---
type: meeting
date: 2026-06-24
---
# Onboarding EVBB Scoring

Projet : [[EVBB Scoring]]

**Participants** : Adam, Dan

## Contexte

- **EVBB** = Enhanced VBB — projet Stellantis de custom bidding
- Emilie (trader/business) pilote le sujet ; Pierre est le référent Data Strat côté data
- [Confluence — Stellantis Enhanced VBB](https://confluence.publicismedia.com/spaces/PMTFR/pages/892863970/Stellantis+Enhanced+VBB#StellantisEnhancedVBB-Context)

## Technique

- Pas de repo local — tout sur GCP : `gbl-psa-analytics-publicis`
- Stack full SQL sur BigQuery (views, tables, modèle)
- Structure 95% identique entre pays — seuls les détails changent (pays, marque dans queries)

## Périmètre

- Déjà en production : France (Peugeot), Espagne (Citroën)
- Déploiement prévu début juillet : Opel DE, Fiat IT, Opel GB

## Objectif

Custom bidding **entre pages** (quelle page mettre en avant pour maximiser les conversions ?), pas versus concurrents.

Pipeline :
1. Tracking des events sur les sites des marques
2. Agrégation aux données de conversions / orders
3. Flagging des events significatifs
4. Entraînement d'un modèle de régression

## Suites

- [ ] Accès au projet GCP `gbl-psa-analytics-publicis`
