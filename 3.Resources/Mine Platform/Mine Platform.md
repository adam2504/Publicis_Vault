---
aliases:
  - Mine
  - ConnectedHub
  - Connected Hub
  - Mine Platform
type: resource
---

# Mine Platform (ConnectedHub)

Plateforme interne Publicis — anciennement appelée **Mine**, renommée **ConnectedHub**. Monorepo React + Node.js hébergé sur GCP, permettant aux équipes data, trading et conseil de Publicis d'accéder à des outils métier centralisés.

---

## Modules actifs

| Module        | Projet                                         | Statut   |
| ------------- | ---------------------------------------------- | -------- |
| MMM AI Agent  | [[1.Projects/MINE_MMMAIAgent/MMM AI Agent]]    | 🟢 Actif |
| AMC Analytics | [[1.Projects/MINE_AMCAnalytics/AMC Analytics]] | 🟢 Actif |
| Creative Insights | [[1.Projects/MINE_CreativeInsights/Creative Insights]] | 🟡 Cadrage (10/08/2026) |

## Modules archivés

| Module                 | Projet                                                           | Archivé le |
| ---------------------- | ---------------------------------------------------------------- | ---------- |
| FeedGen Catégorisation | [[4.Archive/MINE_FeedGenCategorisation/FeedGen Catégorisation]] | 2026-08-03 |

---

## Stack technique

- **Client** : React 19 + Vite + TypeScript + Tailwind CSS 4 + React Router 7
- **Server** : Node.js 24 + Express + Firebase Admin + Google Cloud SDKs
- **Shared** : types et schémas Zod partagés via `@shared/*`
- **Infra** : GCP — Cloud Build, Firestore, BigQuery, Cloud Functions
- **Branches** : `develop` → staging auto-deploy / `main` → production

## Pistes de modules

*(aucune piste ouverte : la piste Creative Insights Leapmotor est devenue le module Creative Insights le 10/08/2026)*

## Incidents & notes techniques

- [[Workflow Git & Merge]]
- [[Incident 2026-06-19 Node.js 24.17]]
