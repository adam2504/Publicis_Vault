---
type: meeting
date: 2026-08-10
---
Projet : [[Creative Insights]]
Plateforme : [[Mine Platform]]

# 10 Août — Cadrage module Creative Insights

Participants : Hajar (DS, porte le sujet), Abhishek (DS PGD, cloud functions et notebooks), Adam.

**Objet réel du point** : Hajar et Abhishek m'introduisent aux besoins et aux idées de dev d'un module Creative Insights dans ConnectedHub. Ce n'était pas une réunion d'arbitrage, c'était mon briefing de dev.

> Le résumé IA du facilitateur ne m'attribue aucune action et ne mentionne pas ConnectedHub. C'est un artefact du résumé, pas de la réunion.

---

## Chaîne actuelle, telle qu'elle tourne aujourd'hui

| # | Étape | Qui | Outil |
| --- | --- | --- | --- |
| 1 | Extraction des assets créa | DS | Cloud Functions (Meta, TikTok, Snapchat) |
| 2 | Regroupement assets + data client | DS | Data Platform |
| 3 | Extraction des features | DS | Prompt + Gemini |
| 4 | Modélisation (corrélation ou autre) | DS | Notebooks |
| 5 | Export des données | DS | CSV |
| 6 | Analyse | DA (Thomas) | À partir du CSV |

Tout est manuel et porté par les DS. Le module vise à rendre cette chaîne self-service.

## Chaîne cible cadrée en séance

1. Création de projet : advertiser IDs, plateformes, période
2. Déclenchement automatique des cloud functions pour télécharger les assets
3. Étape de validation : sélection des vidéos et images pertinentes
4. Extraction de features par prompt : prompt par défaut pour les variables de base, upload d'un prompt custom pour les variables sectorielles (texte ou JSON)
5. Scheduled queries de mapping (vidéos, engagement, features) avec notification quand le volume est suffisant
6. Notebooks prédéfinis pour l'analyse et la modélisation, sur le modèle de SIMBA
7. Restitution sur dashboard, avec accès stakeholders

Principes actés : étapes verrouillées séquentiellement, page de suivi d'avancement dédiée, sorties standardisées pour que les stakeholders sachent à quoi s'attendre.

---

## Existant réutilisable (point important du briefing)

Les cloud functions Meta, TikTok et Snapchat **existent déjà et ont déjà servi**. Des cas passés ont laissé des assets dans des buckets GCS, notamment l'analyse Leapmotor menée par Abhishek. Le module ne part donc pas de zéro sur l'amont : il orchestre de l'existant.

Sont également cités comme antérieurs et potentiellement réutilisables :

- le dashboard interface de Thomas, alimenté par CSV, à intégrer pour minimiser l'effort de dev
- des travaux de Brieg (parti le 12/06/2026, sans repreneur identifié)
- les notebooks d'analyse du pattern SIMBA

Hajar doit me communiquer les pointeurs GCP précis (projets, cloud functions, buckets) pour que j'explore la partie technique existante.

---

## Décisions

- Le sujet devient un **module ConnectedHub** de production, pas un POC client. Périmètre plateforme.
- **Priorité de dev** : la page de création de projet et la récupération des assets d'abord. Le reste suit.
- Réutiliser l'existant plutôt que réécrire : cloud functions, notebooks, dashboard de Thomas.
- Hajar prévient Jules du lancement du projet.

## Points non tranchés

1. **Répartition du dev.** Abhishek annonce démarrer le développement initial, alors que le point servait à me briefer sur le module. Qui construit la couche ConnectedHub (page projet, orchestration, gating, restitution) et qui garde le backend DS ? À clarifier avec Hajar et Jules avant que deux chantiers partent en parallèle.
2. **Où le module s'arrête.** Export CSV avec le dashboard de Thomas branché dessus, ou restitution native dans ConnectedHub. Voir l'analyse dans [[Creative Insights]].
3. **Premier client de démonstration.** Aucun n'a été nommé. Verisure et Leapmotor sont des antécédents, pas des engagements.
4. **Sponsor, priorité, calendrier.** Jules est informé, pas sollicité comme arbitre. Je suis déjà lead sur [[1.Projects/MINE_AMCAnalytics/AMC Analytics]] et [[1.Projects/MINE_MMMAIAgent/MMM AI Agent]], la question de la capacité n'a pas été posée.
5. **Clé de jointure créa vers performance**, contrat d'ingestion, coût de l'extraction Gemini, exécution des notebooks en prod, seuil de volume suffisant. Détaillés dans [[Creative Insights]].

## Actions

| Action | Qui | Pour quand |
| --- | --- | --- |
| Prévenir Jules du lancement du projet et du démarrage d'Abhishek | Hajar | Rapide |
| Transmettre les pointeurs GCP (cloud functions, buckets GCS des cas passés) | Hajar | À demander |
| Explorer l'existant technique dans GCP une fois les pointeurs reçus | Adam | À planifier |
| Clarifier la répartition du dev avec Hajar et Jules | Adam | Avant tout code |
| Démarrer le développement initial | Abhishek | Annoncé en séance |
