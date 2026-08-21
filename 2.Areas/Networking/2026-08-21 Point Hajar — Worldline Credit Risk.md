---
type: meeting
date: 2026-08-21
---
# Point informel Hajar, Data Scientist Credit Risk / KYC chez Worldline

Échange informel initié suite à la découverte de son parcours chez Worldline (data scientist credit risk). Pertinent car proche du rôle visé chez Amex (Credit Risk Data Analyst, sept 2026).

## Contexte

- Hajar a travaillé comme Data Scientist chez Worldline sur des sujets credit risk
- Compétences LinkedIn : KYC, détection d'anomalies, transactions financières, ML, gestion des risques, analyse prédictive, probabilités, NLP, validation de modèles, SQL, R, anti-blanchiment (AML), analyse statistique, MongoDB, analyse des risques, modélisation des données, Power BI, Python, risque de crédit, services de paiement, évaluation des risques, deep learning
- Lors d'un échange précédent, en évoquant Amex (credit risk sur les particuliers, pas les entreprises), elle a identifié ça comme du KYC

## Questions préparées

- Quotidien concret en credit risk/KYC chez Worldline : quelles données (transactionnelles, applicatives), quels modèles (scoring, détection d'anomalies/fraude)
- KYC en pratique : pipeline vérification d'identité, scoring de risque, monitoring continu
- Validation de modèles et gouvernance : contraintes réglementaires spécifiques à la finance/paiement (explicabilité, biais, audit), différent de l'ad-tech
- AML : recoupement avec la détection d'anomalies/fraude côté credit risk
- Pourquoi elle a quitté Worldline, ce qu'elle recommande d'apprendre d'ici septembre 2026
- Toolkit ML classique/stats minimal indispensable pour le credit risk (vs. IA générative/agents, plus fort chez moi)

## Notes de l'échange

**Périmètre chez Worldline :** deux scopes, Credit Risk et KYC.

**Credit Risk (côté entreprises, B2B) :**
- Si un client entreprise fait faillite, les paiements dus restent à la charge de Worldline (responsable légalement du paiement). Worldline calcule donc ce risque tous les mois.
- Le calcul prend en compte plusieurs facteurs, avec un reporting mensuel pour suivre l'évolution.
- Différents types d'exposure au risque : nette et brute.
- Équipe transaction monitoring dédiée à la vérification de l'historique des transactions clients. Elle avait accès à de la data transactionnelle peu granulaire.
- Modèles utilisés : segmentation des clients (par typologie, par industrie) pour détecter des anomalies, avec DBSCAN et Local Outlier Factor.
- MCC Code (code catégorie marchand) mentionné comme feature clé.
- Beaucoup d'acronymes propres au secteur, à apprivoiser.

**AML/KYC :**
- Objectif : vérifier qu'il n'y a pas de fraudeurs parmi les clients à qui Worldline fournit ses services.
- Utilisation d'algos de NLP dans ce cadre.
- Régulation : avant plus floue, aujourd'hui beaucoup plus stricte. Plusieurs sous-filiales de Worldline ont dû fermer à cause de ce durcissement réglementaire.

**Sur Amex :**
- Amex est la plus grosse boîte du secteur payment/credit risk. Bon point pour le CV.
- Elle pense que leur data est propre ("data clean").

## Ce que j'ai appris / points intéressants

- Le credit risk chez Worldline est côté B2B (risque de défaut des entreprises clientes), alors que le rôle visé chez Amex est côté particuliers (émission de carte) : logique KYC/scoring différente à creuser.
- KYC et AML sont très liés à la détection d'anomalies/fraude, pas juste de la vérification d'identité statique.
- Segmentation + détection d'anomalies (DBSCAN, LOF) reviennent comme outils clés côté modélisation, pas seulement du scoring supervisé classique.
- Le secteur paiement/risk a un jargon dense (MCC Code, exposure nette/brute, etc.) à maîtriser avant d'arriver chez Amex.
- Contexte réglementaire mouvant : la régulation AML/KYC s'est durcie récemment (a fait fermer des filiales Worldline), donc probablement un sujet chaud aussi chez Amex.

## À voir / rechercher

- Tous les KPIs credit risk au niveau particuliers (vs. entreprises)
- AML/KYC : définition précise, implications concrètes, comment ça s'applique aux particuliers
- Vérifier s'il y a un critère d'image/standing sur qui peut obtenir une carte Amex
- Partenariats d'Amex avec des entreprises sur les solutions utilisées : ce qui est fait en interne, sous-traité, ou en partenariat
- Explorer les équipes proches chez Amex (credit analyst, hybrid risk, etc.) pour avoir le point de vue des experts métier

## Suite éventuelle

-