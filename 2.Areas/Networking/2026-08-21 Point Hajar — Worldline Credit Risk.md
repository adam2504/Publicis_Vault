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

**KPIs credit risk niveau particuliers :**
- Taux de délinquence (% du solde en retard vs. solde total actif), avec des buckets 30-59, 60-89, 90-119 jours de retard
- Taux de défaut ou charge-off (crédit passé en perte)
- Taux d'utilisation de la carte (somme des soldes / somme des plafonds accordés)
- Facteurs qui influencent ces KPIs : taux d'intérêt, chômage, niveau d'endettement, accès au crédit pour profils plus risqués

**AML/KYC pour particuliers :**
- KYC, vérifier et authentifier l'identité du client à l'ouverture de compte, et évaluer son niveau de risque individuel
- Concrètement pour une demande de carte : vérification identité (pièce d'identité), adresse, revenus, croisement avec bases de données (bureaux de crédit)
- Screening automatique contre listes de sanctions, PEP (personnes politiquement exposées), médias négatifs
- Pas un one-shot, monitoring continu des transactions après l'ouverture du compte
- KYC est un sous-composant de l'AML (dispositif anti-blanchiment plus large)

**Critère d'image/standing sur les cartes Amex :**
- Score FICO généralement 670+ pour les cartes d'entrée de gamme (Blue Cash Everyday), 690+ pour le milieu de gamme (Gold, Blue Cash Preferred), 720+ pour le haut de gamme (Platinum, Centurion/Black)
- Pas de seuil de revenu minimum officiellement communiqué, mais revenu pris en compte dans la décision (salaire, sécurité sociale, chômage, pension, revenus d'investissement)
- Condition de base : 18 ans minimum, bon crédit, revenu suffisant pour couvrir les paiements mensuels, SSN ou ITIN

**Partenariats Amex :**
- Deux modèles de cartes co-brand. D'un côté Amex est l'émetteur/souscripteur (ex. Delta, Hilton, Charles Schwab, Lowe's, via Centurion Bank) et porte le risque de crédit. De l'autre une banque partenaire est l'émetteur (ex. Bank of America, ex-MBNA), porte le risque et gère le compte sur ses propres systèmes en utilisant le réseau et la marque Amex
- Historiquement Amex avait environ 80 partenariats d'émission dans environ 90 pays

**Équipes proches côté risk chez Amex (pour le rôle Credit Risk Data Analyst) :**
- Credit Risk Analyst au sein du CRU (Credit Risk Unit). Note et souscrit les expositions de crédit par région, industrie et business line, calcule la probabilité de défaut (PD) et la perte en cas de défaut (LGD), rédige des mémos de risque, doit connaître Bâle et le cycle macroéconomique
- Ce rôle CRU semble plus orienté institutionnel/corporate (cartes entreprise, facilités de working capital) que particuliers, à vérifier si le rôle visé (particuliers) dépend d'une autre équipe (ex. Consumer Risk / US Consumer Services)

## Sources

- [Consumer credit risk](https://en.wikipedia.org/wiki/Consumer_credit_risk)
- [Credit Card Delinquency Rates and Charge-Offs](https://wallethub.com/edu/cc/credit-card-charge-off-delinquency-statistics/25536)
- [What is AML and KYC in Payments?](https://www.clearlypayments.com/blog/what-is-aml-and-kyc-in-payments/)
- [Credit Card Application KYC Requirements Explained](https://www.vouched.id/learn/blog/credit-card-application-kyc)
- [What is KYC in Banking? - Experian Insights](https://www.experian.com/blogs/insights/what-is-kyc-in-banking/)
- [American Express Credit Score Requirements by Card](https://wallethub.com/answers/cc/american-express-credit-score-requirements-1000338-2140726659/)
- [Requirements to Get a Credit Card - American Express](https://www.americanexpress.com/en-us/credit-cards/credit-intel/credit-card-requirements/)
- [Credit Risk Analyst @ American Express](https://jobs.anitab.org/companies/american-express/jobs/54699088-credit-risk-analyst)
- [Analyst-Risk Management - American Express Careers](https://careers.americanexpress.com/en/sites/CX_1/job/26011119/)
- [The Anatomy of a Co-Branded Credit Card](https://thefinancebuff.com/anatomy-co-branded-credit-card.html)

## Suite éventuelle

-