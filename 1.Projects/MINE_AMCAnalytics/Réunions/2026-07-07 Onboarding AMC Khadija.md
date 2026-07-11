---
type: meeting
date: 2026-07-07
projet: "[[AMC Analytics]]"
participants: Khadija
---
# 2026-07-07 Onboarding AMC Khadija

Projet : [[AMC Analytics]]
Plateforme : [[Mine Platform]]

## Contexte

Onboarding sur les sujets AMC avec Khadija (Data Scientist consultante — interlocutrice principale côté data pour AMC Analytics : audiences AMC, extracts AMC, dashboards Looker Studio, pipeline GCP → BQ).

- Point onboarding AMC commerce initial organisé le **30/06** — cette note démarre le suivi consolidé de l'onboarding.
- Point précédent : [[2026-06-25 Point AMC Khadija]] (répartition dashboards L'Oréal / commerce Publicis).
- Récurrence : point hebdo AMC commerce avec Khadija, tous les mardis.

## Participants

- Khadija
- Adam

## Ordre du jour / points à couvrir

### Pipeline & données AMC
- [ ] Comprendre le pipeline GCP → BigQuery : sources, fréquence, tables clés
- [ ] Process d'extraction des extracts AMC (manuel aujourd'hui vs. futur data querying direct)
- [ ] Structure des exports CSV AMC exploités par le module (Full Funnel, etc.)

### Dashboards Looker → React
- [ ] État des dashboards Looker Studio existants (L'Oréal + commerce Publicis)
- [ ] Calendrier de migration Looker → React (question ouverte du projet)
- [ ] Priorisation : audiences-insights / mm-p2c (L'Oréal), Etude Efficacité Vidéo Amazon (commerce)

### Audiences AMC
- [ ] AMC Modeled Audiences — périmètre, lien avec Hajar
- [ ] Cas d'usage audiences pour L'Oréal

### Organisation
- [ ] Répartition des rôles data (Khadija) / dev (Adam, Eddie) sur le module
- [ ] Canaux et rythme de collaboration

## Points abordés

### 1. Fonctionnalités Amazon Marketing Cloud (tour d'horizon)

- **Tables de données** : plusieurs tables disponibles dans AMC couvrant média, conversion et audience → analyse globale possible par pays / marché.
- **Segmentation des audiences** : segmentation par genre, démographie et comportement d'achat. ⚠️ Données issues d'**algorithmes Amazon** (probabilistes, non catégoriques) → fiabilité limitée, à garder en tête pour l'interprétation.
- **Limites de l'interface AMC** : peu flexible pour le regroupement et le détail des données ; certaines analyses nécessitent des requêtes spécifiques ou des exports manuels.
- **API AMC** : documentée, permet d'automatiser la création d'audiences, l'exécution de requêtes et l'export de données ; possibilité de **workflows récurrents** pour des audiences spécifiques.

### Comptes & instances AMC (Khadija)

Détail consolidé dans la note de référence → [[AMC — Notions]].

En bref : 2 comptes (L'Oréal — dont l'instance ACR hors périmètre — et Publicis, 1 instance par client type Bouygues/Coty). **Une requête AMC = une seule instance** → Khadija fait 1 requête/instance puis regroupe dans Looker. Renforce l'intérêt de centraliser les requêtes par marque/instance.

### 2. Extraction & intégration des données

- **Process actuel (manuel)** : téléchargement manuel des fichiers CSV depuis AMC → dépôt dans des buckets → traitement.
- **Automatisation via API** : l'API AMC permettrait d'extraire et d'intégrer les données sans étape manuelle → accélération du process.
- **Limites techniques à tester** : requêtes API potentiellement lentes ; limites possibles de caractères / performance à valider, surtout pour les extractions volumineuses.
- **Sécurité / confidentialité** : recommandation de **séparer les requêtes par type d'étude** (médiatisation, commercial…) pour limiter les croisements de données sensibles.

### 3. Centralisation vers les dashboards (Looker / Data Studio)

- Connexion des vues de données AMC à Looker Studio → dashboards dynamiques sans modifier directement les tables sources.
- **Séparation dev / prod** : vues distinctes pour sécuriser les modifications et garantir la stabilité des dashboards.
- Cible : connecter directement les tables extraites à Looker Studio, en automatisant le flux AMC → dashboards, **sans CSV intermédiaire**.

### 4. Comparaison plateformes concurrentes

- Avantage AMC vs. concurrents (ex. plateforme Leclerc) : **API bien documentée** permettant la création d'audiences et une meilleure intégration des données.

## Décisions / orientations

- Commencer par des **solutions simples et évolutives**, avant d'automatiser l'ensemble via l'API AMC.
- **Centraliser les requêtes** utilisées pour générer les CSV, par marque, pour les transformer en tables connectées directement aux dashboards.
- **Séparer les requêtes par type d'étude** (sécurité / confidentialité).
- **Cadrer le projet avec les référents** et valider les choix techniques avant tout déploiement à grande échelle.

## Actions

| Action                                                                                                                                                                                                 | Responsable    | Deadline  |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------- | --------- |
| Connexion données AMC → dashboards : extraire toutes les données des CSV par marque (ex. Azzaro, Mugler), les mettre en table, préparer les requêtes pour connecter ces tables aux dashboards d'études | Khadija        | À définir |
| Automatisation extraction : tester le transfert automatique des fichiers AMC vers un bucket sans étape manuelle                                                                                        | Khadija        | À définir |
| Vérifier avec Hajar quel projet GCP utiliser pour grow l'automatisation/centralisation des sujets AMC                                                                                                  | Adam & Khadija | À définir |
| Demande d'accès : faire une demande (ticket) pour accéder au projet AMC et à l'API (certains accès gérés en Inde)                                                                                      | Adam           | À définir |
| Connecter la table BQ au module AMC (sous module Full Funnel)                                                                                                                                          | Adam           |           |

## Ressources

- Notions outil AMC : [[AMC — Notions]]
- Doc API AMC (lien dans le projet) : [[AMC Analytics]]
