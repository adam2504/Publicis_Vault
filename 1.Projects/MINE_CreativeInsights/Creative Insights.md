---
type: project
statut: en cours
discipline:
  - dev
  - data-science
implication: lead
client: MINE
---
# Creative Insights

Plateforme : [[Mine Platform]]

Module ConnectedHub d'automatisation des Creative Insights : industrialiser en self-service la chaîne aujourd'hui exécutée à la main par les Data Scientists, de la récupération des assets créa jusqu'à la restitution au Data Analyst.

Périmètre plateforme (transverse, pas mono-client). Plateformes média couvertes à l'entrée : Meta, TikTok, Snapchat.

> ⚠️ **Répartition du dev non tranchée.** Le CR du 10/08 indique qu'Abhishek démarre le développement initial, alors que le point servait à me briefer sur le module ConnectedHub. `implication: lead` reflète que je porte la couche plateforme, à confirmer avec Hajar et Jules.

---

## Chaîne actuelle (manuelle, portée par les DS)

| # | Étape | Qui | Outil |
| --- | --- | --- | --- |
| 1 | Extraction des assets créa | DS | Cloud Functions (Meta, TikTok, Snapchat) |
| 2 | Regroupement assets + data client | DS | Data Platform |
| 3 | Extraction des features | DS | Prompt + Gemini |
| 4 | Modélisation (corrélation ou autre) | DS | Notebooks |
| 5 | Export des données | DS | CSV |
| 6 | Analyse | DA (Thomas) | Hors chaîne, à partir du CSV |

## Chaîne cible (module)

1. **Création de projet** : advertiser IDs, plateformes, période
2. **Récupération des assets** : déclenchement automatique des Cloud Functions existantes
3. **Validation** : sélection humaine des vidéos et images à analyser
4. **Extraction de features** : prompt par défaut, plus upload d'un prompt custom pour les variables sectorielles (texte ou JSON)
5. **Mapping** : scheduled queries reliant vidéos, données d'engagement et features, avec notification quand le volume est suffisant
6. **Analyse** : notebooks prédéfinis, sur le modèle de SIMBA
7. **Restitution** : dashboard, accès stakeholders

Étapes verrouillées séquentiellement : une étape ne s'ouvre que si la précédente est terminée. Page de suivi d'avancement dédiée. Sorties standardisées, pour que chacun sache ce qu'il obtient en fin d'analyse.

---

## Le point de design structurant : où le module s'arrête

Aujourd'hui la chaîne se termine par un CSV parce que c'est le passage de main du DS vers le DA. Ce CSV n'est pas un format, c'est une frontière d'équipe. Deux options :

| Option | Conséquence |
| --- | --- |
| Reproduire la frontière | Le module produit un export propre, on branche le dashboard CSV existant de Thomas dessus. Effort de dev minimal, la valeur ajoutée reste l'automatisation amont. |
| Supprimer la frontière | Restitution native dans ConnectedHub, le DA travaille dans l'app. Plus de dev, mais c'est ce qui fait du module un produit et pas un ordonnanceur. |

**Précédent à ne pas répéter** : sur [[1.Projects/MINE_AMCAnalytics/AMC Analytics]], le deck de vues figées a dû être entièrement refait en workspace après le retour de Jules (« on ne peut pas faire ce qu'on veut »). Sorties standardisées ne doit pas dériver vers sorties figées.

**Premier utilisateur réel du module : Thomas** (Data Analyst), pas Verisure ni Leapmotor. Il n'était pas dans la réunion de cadrage.

---

## Existant technique à explorer

Une partie de la chaîne tourne déjà et a servi sur des cas passés. À cartographier avant tout dev.

- [x] Cloud Functions Meta, TikTok, Snapchat : ce sont des Cloud Run gen2 dans `zen-creativeinsights-dev-mg`, contrat d'appel et pièges documentés dans [[Existant technique GCP]]
- [x] Buckets GCS : `creative_assets_<plateforme>/<ad_account_id>/<asset_id>.<format>`, volumétrie relevée (prototype arrêté depuis le 01/06)
- [x] Data Platform : c'est `zen-dataplatform-pm-prd-amg`, branchée par vue matérialisée avec un compte client codé en dur
- [ ] Prompts Gemini existants : où ils sont versionnés, quel modèle, quel coût par asset (**non trouvé en CLI**, à demander à Hajar)
- [ ] Notebooks d'analyse façon SIMBA : où ils tournent, comment ils sont versionnés (**hors du projet GCP exploré**)
- [ ] Dashboard CSV de Thomas : stack, alimentation, réutilisable ou non
- [ ] Travaux antérieurs de Brieg et Thomas référencés en réunion (Brieg est parti le 12/06/2026, sans repreneur identifié)
- [x] **Côté ConnectedHub : fait le 10/08.** SIMBA est le module `custom-bidding`, et les sept étapes de la chaîne cible existent déjà entre `custom-bidding` et `feed-manager`. Voir [[Patterns ConnectedHub réutilisables]]

> Adam doit obtenir de Hajar les pointeurs GCP précis avant d'ouvrir quoi que ce soit. Le reste de la liste concerne l'infra DS, invisible depuis le repo.

---

## Zones d'ombre techniques

1. ~~**Clé de jointure créa vers performance.**~~ **Tranché par l'exploration du 10/08, et c'est pire que prévu.** La jointure passe par `ad_id`, colonne que **seul Meta possède**. TikTok et Snapchat ne remontent aucune hiérarchie de campagne, donc la jointure y est impossible aujourd'hui. Reste ouvert : la règle d'attribution, car la performance est à la maille `ad_id` par jour alors que la créa est à la maille `asset_id`, et un ad porte plusieurs assets. Voir [[Existant technique GCP]].
2. **Contrat d'ingestion.** Instruit le 10/08 : les features sont stockées en **colonnes physiques propres à chaque client** (environ 70 colonnes Verisure), Stellantis a une forme entièrement différente. `analysis_json` existe déjà et doit devenir le stockage canonique, sinon « sorties standardisées » est intenable.
3. **Coût de l'extraction multimodale.** Un prompt Gemini sur des centaines de vidéos n'est pas gratuit, aucun chiffrage fait.
4. **Notebooks en backend de production.** Fragile par nature : versioning, exécution, maintenance quand ils cassent.
5. **Seuil « assez de données »** pour déclencher la notification : non défini, décision DS.
6. **Où vit l'état du projet.** SIMBA le met en BigQuery, feed-manager en Firestore. Les deux modules du repo ont fait des choix opposés, il faut trancher avec Eddie. Détail dans [[Patterns ConnectedHub réutilisables]].

---

## Personnes clés

| Rôle | Personne |
| --- | --- |
| Data Scientist, porte le sujet Creative Insights | Hajar |
| Data Scientist PGD, cloud functions et notebooks, antécédent Leapmotor | Abhishek |
| Data Analyst, consommateur final et auteur du dashboard CSV | Thomas |
| Data Strategist, besoin client côté Verisure | Léonie |
| Manager, à informer | Jules |

---

## Antécédents

- **POC Creative Insights Verisure** : porté par Hajar et Léonie, jamais rien produit de mon côté. Note supprimée du vault le 10/08/2026, seuls les contacts subsistent dans [[VERISURE]]. Point à retenir : Léonie détenait une liste de features demandées par le client, jamais formalisée nulle part. C'est la seule expression de besoin client connue sur le sujet, et elle est à récupérer.
- [[Creative Insights Leapmotor]] : piste de module restée en attente, cas d'usage passé d'Abhishek. Ses cloud functions et ses assets GCS deviennent la brique amont du module. Reprise par ce projet.

## Notes techniques

- [[Existant technique GCP]] : inventaire de `zen-creativeinsights-dev-mg`, contrat des Cloud Run, modèle de données, volumétrie
- [[Patterns ConnectedHub réutilisables]] : ce que `custom-bidding` (SIMBA) et `feed-manager` fournissent déjà

## Réunions

- [[2026-08-10 Cadrage module Creative Insights]] : briefing dev, chaîne cible, répartition à confirmer

## Dernières sessions

- **2026-08-13** — Deux lots mergés : rôles éditeur/administrateur avec circuit de validation, mail aux admins et carte d'activité (#1738, #1739) ; sélecteur de rôle dans l'admin panel et icône du module (#1740, #1741). Le rôle est désormais renvoyé par le serveur, le lire depuis le `localStorage` du navigateur affichait des autorisations périmées. **Découverte structurante** : sans identifiant de campagne, le scrapper énumère les 1545 campagnes du compte et se fait couper par le quota Meta, donc un projet visant un compte entier n'est pas exécutable. Fragilité de l'observateur détaché constatée en vrai, voir [[Session 2026-08-13]]. Ticket IT toujours pas traité, staging non fonctionnel.
- **2026-08-12** — Étapes 1 et 2 livrées, étape 3 en cours. Projets persistés en BigQuery (dataset `connectedhub` créé), déclenchement Meta branché et **validé de bout en bout** (35 créas récupérées en 1 min 35, fichiers GCS horodatés du jour). Bibliothèque de créas avec filtres et vue agrandie, non commitée. Deux commits locaux : `575304be6` (ossature), `5062ebf88` (persistance et gathering). Ticket IT envoyé pour les droits du service account `interface@`.
