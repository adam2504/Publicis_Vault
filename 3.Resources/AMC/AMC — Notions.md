---
type: resource
---

# AMC — Notions

Notions de référence sur **Amazon Marketing Cloud** (l'outil d'Amazon Ads), réutilisables sur tous les sujets AMC (module [[AMC Analytics]], dashboards, audiences, data querying).

AMC est la *clean room* d'Amazon Ads : un environnement où l'on requête des données média / conversion / audience pseudonymisées, sans accès aux données individuelles brutes.

---

## Comptes & instances

L'organisation se fait à deux niveaux : **compte** (par entité) puis **instance** (par périmètre/client) à l'intérieur d'un compte.

Chez nous, Khadija gère **2 comptes AMC** :

- **Compte L'Oréal** — 2 instances :
  - Instance **ACR / audiences** (modernisation d'audience) → hors périmètre de notre module
  - Instance **« normale »** L'Oréal
- **Compte Publicis** — usage Publicis interne / commerce (alimente les dashboards commerce déjà présents dans ConnectedHub, côté client Publicis). Contient **une instance par client** (ex. Bouygues, Coty…).

## Limite de requêtage : 1 requête = 1 instance

Une requête AMC ne peut cibler **qu'une seule instance** à la fois. Pour consolider plusieurs clients/périmètres, il faut donc **une requête par instance** puis regrouper les résultats en aval.

Aujourd'hui Khadija procède ainsi : 1 requête par instance → regroupement → connexion à Looker Studio pour consolider dans le dashboard. C'est ce qui motive la **centralisation des requêtes par marque/instance** côté [[AMC Analytics]].

## Tables de données

Plusieurs tables couvrent **média**, **conversion** et **audience**, permettant une analyse croisée par pays / marché.

## Segmentation des audiences

Segmentation par genre, démographie et comportement d'achat. ⚠️ Ces attributs sont issus d'**algorithmes Amazon** (probabilistes, non catégoriques) → fiabilité limitée, à garder en tête lors de l'interprétation.

## Limites de l'interface

L'interface AMC est peu flexible pour le regroupement et le détail des données ; beaucoup d'analyses passent par des **requêtes SQL spécifiques** ou des **exports manuels** (CSV).

## API

AMC expose une **API documentée** permettant d'automatiser la création d'audiences, l'exécution de requêtes et l'export de données, avec possibilité de **workflows récurrents**. Point de comparaison : API mieux documentée que certaines plateformes retail concurrentes (ex. Leclerc), notamment pour la création d'audiences.

Documentation : voir le lien dans [[AMC Analytics]].
