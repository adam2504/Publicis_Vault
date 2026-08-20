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

## Le projet en clair

**Comprendre ce qui, dans une créa publicitaire, fait qu'une campagne marche.**

Un annonceur diffuse des dizaines ou des centaines de visuels et de vidéos sur Meta, TikTok et Snapchat. Certains performent, d'autres non. Les outils habituels disent *quelle* créa a bien marché, mais jamais **pourquoi** : est-ce le prix affiché à l'écran, la présence d'un visage, la durée de la vidéo, le fait qu'on y montre le produit en main ?

Ce projet rend la créa elle-même mesurable. Il la transforme en un ensemble de variables décrites, puis confronte ces variables à la performance réellement obtenue. On passe ainsi de « cette vidéo a bien marché » à « les créas qui affichent un prix final convertissent mieux », ce qui est **reproductible** par le client sur ses prochaines campagnes.

### La chaîne

1. **Récupérer les créas** depuis les plateformes, via leurs APIs (Meta, TikTok, Snapchat)
2. **Les lire avec un LLM** pour en extraire des variables typées : présence humaine, palette dominante, type d'accroche commerciale, durée, mentions légales, et ainsi de suite
3. **Récupérer la performance** côté client, à la maille de l'annonce
4. **Croiser les deux** et comparer la performance par modalité de variable
5. **Livrer au Data Analyst**, qui interprète et construit l'analyse remise au client

### Comment la performance est mesurée

Pas en volume brut, ce qui ne dirait rien : une créa peu diffusée générerait mécaniquement peu de résultats. La métrique retenue est le **lift**, soit le rapport entre les conversions observées et celles qu'on aurait attendues au vu des impressions reçues.

Ça neutralise le poids média et isole ce qui revient à la créa elle-même. Les créas trop peu exposées sont écartées, faute de sens statistique.

### Ce que le client reçoit

Une liste de ce qui marche et de ce qui ne marche pas, **chiffrée et actionnable**. Extrait d'une analyse réelle, sur 73 créas Verisure :

> **Ce qui marche** : les vidéos de 24 à 29 secondes obtiennent un KPI de 0,042 contre 0,022 pour les autres (N=13 contre 34). Les créas comportant un témoignage client atteignent 0,035 contre 0,019 (N=20 contre 52).
>
> **Ce qui ne marche pas** : les vidéos de 4 à 20 secondes tombent à 0,013 contre 0,033 pour les autres.

Chaque constat porte son **effectif** et sépare explicitement ce qui relève de la créa de ce qui relève du média ou du type d'achat, deux causes qu'il serait facile de confondre.

Sur ce cas Verisure, la méthode est de la **comparaison de moyennes par modalité**, et non de la modélisation. Ça se défend sur 73 créas, où un modèle donnerait une fausse impression de rigueur : les livrables signalent d'ailleurs eux-mêmes les biais et les effectifs faibles.

> ⚠️ **Ce n'est pas la seule méthode pratiquée.** De la modélisation a bien été faite sur d'autres cas d'usage. Le choix dépend du volume disponible et de la question posée, et le module doit donc pouvoir porter les deux, pas seulement le descriptif.

### Pourquoi ce projet existe

**C'est une prestation, pas un outil interne.** L'analyse de créas fait partie de ce que Publicis Media propose à ses annonceurs, au titre de son métier d'agence média. Ce n'est donc pas un confort d'équipe : la qualité, la répétabilité et le délai de production ont une portée commerciale directe.

**Et aujourd'hui, chaque étude est artisanale.** Un Data Scientist monte le pipeline dans des notebooks pour un client donné, puis passe la main au Data Analyst par un export CSV. Deux équipes, deux outils, un transfert manuel, et rien de réutilisable d'une étude à l'autre.

L'industrialisation vise donc deux choses à la fois : **rendre le travail des DS répétable**, et **réunir dans un même endroit ce que se partagent aujourd'hui deux métiers**. Le module met la partie data science et la partie analyse dans [[Mine Platform|ConnectedHub]], là où les analystes travaillent déjà sur le reste de leurs campagnes. Un analyste crée un projet, choisit son périmètre et lance la chaîne sans écrire de code.

Le périmètre est transverse, pas mono-client.

### Ce qui rend le sujet difficile

**Les créas sont multimodales.** Analyser une vidéo demande un modèle capable de la regarder, pas seulement d'en lire les métadonnées.

**Les variables doivent être comparables entre créas**, sinon rien ne se corrèle. D'où un dictionnaire de features typées et un schéma de sortie contraint, plutôt qu'un texte libre.

**Un LLM n'est pas déterministe.** Le pipeline des DS interroge le modèle trois fois par créa et consolide par vote majoritaire, ou par médiane pour les valeurs numériques. Sans ça, deux lectures de la même vidéo pourraient donner deux réponses différentes, et la corrélation porterait sur du bruit.

**Le croisement avec la performance est le vrai point dur.** La performance se mesure à la maille de l'annonce et par jour, la créa à la maille de l'asset, et une annonce porte plusieurs assets. La règle d'attribution entre les deux n'est pas tranchée.

---

## État d'avancement au 20/08/2026

**Les étapes 1 à 4 du cadrage sont construites et fonctionnent.** Le module crée des projets, déclenche la collecte Meta, présente la bibliothèque de créas avec sélection, et extrait des features par prompt d'industrie. Tout est vérifié sur données réelles.

**Les étapes 5 à 7 n'existent pas** : mapping avec la performance, notebooks d'analyse, restitution.

| Étape | État |
| --- | --- |
| 1. Création de projet | livrée, avec contrôle de périmètre avant collecte |
| 2. Récupération des assets | livrée, Meta uniquement |
| 3. Validation / sélection | livrée |
| 4. Extraction de features | livrée |
| 5. Mapping avec la performance | **bloquée**, voir zones d'ombre |
| 6. Analyse (notebooks) | non commencée |
| 7. Restitution | non tranchée, voir le point de design ci-dessous |

**Seul Meta est câblé.** TikTok et Snapchat ont un formulaire mais aucune collecte, et surtout aucune jointure performance possible (voir zones d'ombre).

> ⚠️ **Le code n'est ni poussé ni mergé au 20/08.** Quatorze commits locaux sur `feat/creative-insights-scoping`. Ce qui est en production s'arrête aux rôles et à la bibliothèque de créas, livrés le 13/08.

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

1. ✅ **Création de projet** : advertiser IDs, plateformes, période
2. ✅ **Récupération des assets** : déclenchement automatique des Cloud Functions existantes
3. ✅ **Validation** : sélection humaine des vidéos et images à analyser
4. ✅ **Extraction de features** : prompt par défaut, plus prompts par industrie
5. ⛔ **Mapping** : scheduled queries reliant vidéos, données d'engagement et features, avec notification quand le volume est suffisant
6. ⬜ **Analyse** : notebooks prédéfinis, sur le modèle de SIMBA
7. ⬜ **Restitution** : dashboard, accès stakeholders

Étapes verrouillées séquentiellement : une étape ne s'ouvre que si la précédente est terminée. Page de suivi d'avancement dédiée. Sorties standardisées, pour que chacun sache ce qu'il obtient en fin d'analyse.

---

## Le point de design structurant : où le module s'arrête

Aujourd'hui la chaîne se termine par un CSV parce que c'est le passage de main du DS vers le DA. Ce CSV n'est pas un format, c'est une frontière d'équipe. Deux options :

| Option | Conséquence |
| --- | --- |
| Reproduire la frontière | Le module produit un export propre, on branche le dashboard CSV existant de Thomas dessus. Effort de dev minimal, la valeur ajoutée reste l'automatisation amont. |
| Supprimer la frontière | Restitution native dans ConnectedHub, le DA travaille dans l'app. Plus de dev, mais c'est ce qui fait du module un produit et pas un ordonnanceur. |

**Premier utilisateur réel du module : Thomas** (Data Analyst), pas Verisure ni Leapmotor. Il n'était pas dans la réunion de cadrage.

---

## Existant technique à explorer

Une partie de la chaîne tourne déjà et a servi sur des cas passés. À cartographier avant tout dev.

- [x] Cloud Functions Meta, TikTok, Snapchat : ce sont des Cloud Run gen2 dans `zen-creativeinsights-dev-mg`, contrat d'appel et pièges documentés dans [[Existant technique GCP]]
- [x] Buckets GCS : `creative_assets_<plateforme>/<ad_account_id>/<asset_id>.<format>`, volumétrie relevée (prototype arrêté depuis le 01/06)
- [x] Data Platform : c'est `zen-dataplatform-pm-prd-amg`, branchée par vue matérialisée avec un compte client codé en dur
- [x] **Prompts Gemini et notebooks : trouvés le 19/08**, dans [creative-insights-analysis](https://github.com/Publicis-Media-France-FR5140/creative-insights-analysis). Ce n'est pas un notebook mais un package Python structuré, avec un dictionnaire de 54 features typées et un pipeline Vertex AI. Le modèle est `gemini-2.5-flash`, le coût mesuré est de l'ordre de 8 900 tokens par image et 12 200 par vidéo. Détail dans [[Session 2026-08-20]]
- [ ] Dashboard CSV de Thomas : stack, alimentation, réutilisable ou non
- [ ] Travaux antérieurs de Brieg et Thomas référencés en réunion (Brieg est parti le 12/06/2026, sans repreneur identifié)
- [x] **Côté ConnectedHub : fait le 10/08.** SIMBA est le module `custom-bidding`, et les sept étapes de la chaîne cible existent déjà entre `custom-bidding` et `feed-manager`. Voir [[Patterns ConnectedHub réutilisables]]

---

## Zones d'ombre techniques

1. ~~**Clé de jointure créa vers performance.**~~ **Tranché par l'exploration du 10/08, et c'est pire que prévu.** La jointure passe par `ad_id`, colonne que **seul Meta possède**. TikTok et Snapchat ne remontent aucune hiérarchie de campagne, donc la jointure y est impossible aujourd'hui. Reste ouvert : la règle d'attribution, car la performance est à la maille `ad_id` par jour alors que la créa est à la maille `asset_id`, et un ad porte plusieurs assets. Voir [[Existant technique GCP]].
2. **Contrat d'ingestion.** Instruit le 10/08 : les features sont stockées en **colonnes physiques propres à chaque client** (environ 70 colonnes Verisure), Stellantis a une forme entièrement différente. `analysis_json` existe déjà et doit devenir le stockage canonique, sinon « sorties standardisées » est intenable.
3. ~~**Coût de l'extraction multimodale.**~~ **Chiffré le 19/08** : environ 8 900 tokens pour une image, 12 200 pour une vidéo, soit une dizaine d'euros pour une passe complète sur 1 456 créas. Ce n'est pas le coût qui contraint, c'est la durée : 6 h en séquentiel, ramenées à une vingtaine de minutes avec une concurrence de 16.
4. **Notebooks en backend de production.** Fragile par nature : versioning, exécution, maintenance quand ils cassent.
5. **Seuil « assez de données »** pour déclencher la notification : non défini, décision DS. C'est un prérequis de l'étape 5.
6. ~~**Où vit l'état du projet.**~~ **Tranché : BigQuery**, façon SIMBA, parce que les DS lisent les projets depuis leurs pipelines, hors de l'application. Seuls les prompts sont en Firestore, façon feed-manager, n'ayant aucun consommateur hors de l'app.

---

## Décisions structurantes

Les décisions durables, avec leur raison. Le comment est dans les sessions, le pourquoi est ici.

**Un projet est défini par son périmètre de collecte, pas par un compte annonceur.** C'était la contrainte la plus lourde du départ : la Cloud Function n'écrivait aucun identifiant de projet, donc les créas d'un projet étaient toutes celles de son compte. Sur le compte de test, un projet ayant collecté 35 créas en affichait 1 456. Levé le 20/08 en écrivant `project_id` nous-mêmes.

**Les features sont typées et le schéma de réponse en est dérivé.** C'est ce qui rend deux créas comparables. Un prompt en texte libre aurait produit des sorties inexploitables. Repris du pipeline des DS, qui a raison sur ce point.

**Les prompts vivent par industrie, pas par client ni par projet.** Un prompt par projet casserait la comparabilité, qui est la raison d'être du typage. La notion d'industrie ne peut pas venir du champ `business` des annonceurs : il n'est renseigné que sur 55 sur 300, en texte libre, avec des doublons et des coquilles. Elle est donc portée par le prompt.

**L'extraction est séparée de l'analyse de performance**, alors que le pipeline des DS les fait dans le même appel. Les features n'ont pas besoin des chiffres de performance, donc l'extraction tourne dès la collecte au lieu d'attendre une jointure qui reste bloquée.

**Ce qui se déduit ne se stocke pas.** L'avancement, le caractère bloqué d'un run, le rôle de l'appelant, l'état global du projet : tout est dérivé côté serveur. Un compteur stocké finit toujours par diverger de la réalité, et une donnée dupliquée oblige chaque étape à penser à la mettre à jour.

**Toute dépense passe par un accord.** Collecte et extraction coûtent, l'une du quota API, l'autre de l'argent. Un éditeur demande, un administrateur décide. La modification d'un projet, elle, ne coûte rien et n'est donc pas soumise à validation : la tracer dans le journal d'activité suffit.

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

## Reprise du projet

Ce qu'il faut savoir pour reprendre, au-delà du code.

**Le module utilise sa propre copie de la Cloud Function.** Depuis le 20/08, la collecte passe par `cf-gather-meta-assets-ci` et écrit dans `connectedhub.tb_meta_asset_urls`, pas dans le service ni la table des DS. C'est une **divergence de code assumée** : leur service reste intact et ils continuent de l'utiliser, mais les corrections qu'on y a apportées ne sont pas chez eux.

> ⚠️ **Hajar n'en a pas été informée au 20/08.** C'est la première chose à faire. Le [[Plan modifications DS]] décrit encore comme des demandes plusieurs corrections qu'on a nous-mêmes livrées, il est à réécrire avant tout envoi.

**Ce qu'on a corrigé de leur côté et qu'ils n'ont pas** : le `project_id` écrit en colonne, le `DELETE` d'idempotence qui ne filtrait que sur `asset_id` (bug actif chez eux, une créa partagée entre deux comptes voit sa ligne supprimée par une collecte sur l'autre), le `KeyError` de pagination qui fait tomber toute la fonction dès que Meta répond une erreur, et le multi-campagnes.

**Ce qu'on ne peut pas corriger** : `main.py` n'est qu'une coquille, le travail réel vient de deux paquets privés Artifact Registry (`simba` et `creative-insights`). Tout ce qui est dans les paquets, notamment la temporisation sur le quota et le filtre de période, reste du ressort des DS.

**Le prompt d'amorçage est calibré Verisure** et c'est le seul de la bibliothèque. Il fonctionne techniquement sur n'importe quelle créa, mais ses features sectorielles (détection de cambriolage, règles de prix) ne veulent rien dire hors du secteur. **La pertinence des extractions n'a jamais été validée sur un vrai jeu Verisure.**

**Le compte de test n'est pas Verisure.** C'est un compte immobilier, `1349117765200572`, dont les campagnes s'appellent `Site4`, `Hubert`, `Seloger`. Il contient 16 575 annonces, dont seules 1 431 ont jamais été collectées par le prototype.

**Un piège de la plateforme, sans rapport avec ce module** : un job nocturne retire des utilisateurs tout outil absent du registre déployé en production. Un module en cours de développement disparaît donc chaque nuit des comptes de ceux qui le testent. Voir [[Session 2026-08-20]].

## Notes techniques

- [[Plan modifications DS]] : les 5 évolutions attendues côté Cloud Functions, à destination de Hajar et Abhishek
- [[Existant technique GCP]] : inventaire de `zen-creativeinsights-dev-mg`, contrat des Cloud Run, modèle de données, volumétrie
- [[Patterns ConnectedHub réutilisables]] : ce que `custom-bidding` (SIMBA) et `feed-manager` fournissent déjà

## Réunions

- [[2026-08-10 Cadrage module Creative Insights]] : briefing dev, chaîne cible, répartition à confirmer

## Dernières sessions

- **2026-08-20** — Grosse session du 17 au 20, **14 commits non poussés**. **Étape 4 livrée** : bibliothèque de prompts par industrie, extraction Vertex AI en `gemini-2.5-flash` avec schéma typé dérivé du dictionnaire de features, worker à concurrence 16 et reprenable, affichage des résultats dans la bibliothèque de créas. Le tout derrière le même circuit de validation que la récupération, puisqu'une passe coûte de l'argent. **On a repris la main sur la Cloud Function** : service dupliqué en `cf-gather-meta-assets-ci`, avec `project_id` écrit, `DELETE` d'idempotence corrigé, `KeyError` de pagination réglé, multi-campagnes et mode comptage. Table séparée `connectedhub.tb_meta_asset_urls`. Statuts refondus, `gathering_status` plus un `stage` dérivé. Droits IT enfin en place après deux erreurs de bénéficiaire, et découverte d'un **job nocturne qui retirait le module** à tous les utilisateurs tant qu'il n'était pas en production. Détail dans [[Session 2026-08-20]].
- **2026-08-13** — Trois lots. Les deux premiers mergés et déployés : rôles éditeur/administrateur avec circuit de validation, mail aux admins et carte d'activité (#1738, #1739) ; sélecteur de rôle dans l'admin panel et icône du module (#1740, #1741). Le rôle est désormais renvoyé par le serveur, le lire depuis le `localStorage` du navigateur affichait des autorisations périmées. **Découverte structurante** : sans identifiant de campagne, le scrapper énumère les 1545 campagnes du compte et se fait couper par le quota Meta, donc un projet visant un compte entier n'est pas exécutable. Fragilité de l'observateur détaché constatée en vrai, voir [[Session 2026-08-13]]. **Lot 3 poussé sans PR** : colonne `project_id` créée par nos soins sur `meta.tb_api_asset_urls` et périmètre à repli côté module, de sorte que la livraison de Hajar suffira à activer la fonctionnalité sans redéploiement. [[Plan modifications DS]] rédigé, pas encore envoyé. Ticket IT toujours pas traité, staging non fonctionnel.
- **2026-08-12** — Étapes 1 et 2 livrées, étape 3 en cours. Projets persistés en BigQuery (dataset `connectedhub` créé), déclenchement Meta branché et **validé de bout en bout** (35 créas récupérées en 1 min 35, fichiers GCS horodatés du jour). Bibliothèque de créas avec filtres et vue agrandie, non commitée. Deux commits locaux : `575304be6` (ossature), `5062ebf88` (persistance et gathering). Ticket IT envoyé pour les droits du service account `interface@`.
