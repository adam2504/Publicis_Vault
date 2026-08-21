---
type: meeting
date: 2026-08-21
projet: Creative Insights
---

Projet : [[Creative Insights]]
Plateforme : [[Mine Platform]]

# 2026-08-21 Point Hajar et Abhishek, état du module

**Note de préparation.** Le compte rendu sera ajouté en bas après la réunion.

## Ce que je veux en sortir

1. Des **retours sur l'interface** et sur le parcours utilisateur, avant de figer l'architecture des pages
2. Leur **accord tacite** sur ce que j'ai touché dans GCP, et la transparence sur la copie de leurs Cloud Functions
3. Une **réponse claire sur la performance** : de quoi a-t-on besoin, et quand

---

## 1. Démo

**En local**, la production n'ayant pas encore tous les droits. Deux manques, détaillés plus bas : Vertex AI pour l'extraction, et la lecture des buckets TikTok et Snapchat.

Parcours à montrer, dans cet ordre :

| Écran | Ce qu'il illustre |
| --- | --- |
| Accueil du module | liste des projets, avec l'état d'avancement dérivé |
| Création de projet | le **contrôle de périmètre** avant de lancer quoi que ce soit |
| Page projet | collecte, journal d'activité, carte d'extraction |
| Bibliothèque de créas | filtres, sélection, vignettes, et le **panneau de features** sous une créa agrandie |
| Bibliothèque de prompts | le prompt Verisure avec ses 54 features |

**Trois moments qui valent la démonstration** : le contrôle de périmètre qui répond en moins d'une seconde et nomme une campagne invalide, le panneau de features ouvert sur une vidéo, qui montre concrètement ce que le modèle a lu, et une **bibliothèque TikTok**, qui prouve que le module ne se limite plus à Meta.

**À dire d'emblée**, pour éviter qu'on le découvre en cours de route : le prompt de la bibliothèque est **calibré Verisure** alors que le compte de test est immobilier. Les features sectorielles sortent donc vides, et c'est le comportement correct. La mécanique est prouvée, la pertinence ne l'est pas encore.

---

## 2. Les trois plateformes : ce qui marche, ce qui bloque

**Les trois plateformes collectent.** Meta, TikTok et Snapchat sont câblées et remplissent la bibliothèque.

Ce qui les sépare n'est pas la collecte, c'est le **rattachement à la performance**. Les trois scrappers n'interrogent pas la même classe d'API :

| Plateforme | Ce que l'API renvoie |
| --- | --- |
| **Meta** | la **structure publicitaire** : campagnes, puis annonces, puis créas. Donc `ad_id` |
| **TikTok** | `/v1.3/file/video/ad/search/`, soit la **bibliothèque vidéo** de l'annonceur |
| **Snapchat** | `/v1/adaccounts/<id>/media`, soit **tous les médias** du compte |

TikTok et Snapchat répondent « voici tout ce que l'annonceur possède », pas « voici ce qui a été diffusé ». Sans notion d'annonce, **aucune jointure vers la performance n'est possible**, et ajouter une colonne n'y changerait rien.

**Ce n'est pas une limite des plateformes non plus.** TikTok expose `/v1.3/ad/get/` et Snapchat ses endpoints d'annonces. La liaison créa vers annonce est atteignable, mais elle demande **un appel supplémentaire à développer**, pas un champ de plus.

C'est le vrai chiffrage à leur donner : ce n'est pas un correctif, c'est une évolution du scrapper.

### Deux conséquences concrètes

- Le **contrôle de périmètre** avant collecte n'existe que pour Meta, seul à pouvoir compter sans collecter. Chez les deux autres, la seule façon de savoir combien il y a, c'est de tout ramener. L'interface ne propose donc plus le bouton là où il ne veut rien dire.
- **Snapchat ne renvoie rien** sur le compte de test dont je dispose. Question directe pour Abhishek : compte réellement vide, ou périmètre d'API à demander ?

---

## 3. Retours à demander sur l'interface

Questions ouvertes, pas des validations :

- Le découpage **projet → bibliothèque de créas → extraction** correspond-il à leur façon de travailler ?
- La **sélection de créas** avant analyse a-t-elle du sens pour eux, ou écartent-ils plutôt après coup ?
- Les prompts par **industrie** sont-ils la bonne maille ? Ou faut-il du par client, voire du par étude ?
- Le **dictionnaire de features en JSON** est-il éditable pour eux, ou faut-il un formulaire ?
- Que manque-t-il à l'écran pour qu'ils puissent juger la qualité d'une extraction ?

> ⚠️ Je compte retravailler l'accueil du module et l'architecture des pages imbriquées, mais **après** l'étape 5, qui va probablement changer la forme d'un projet. Autant leur dire, pour que leurs retours portent sur le fond plutôt que sur la mise en page.

---

## 4. Ce que j'ai touché dans GCP

À annoncer sans détour, c'est le point délicat de la réunion.

**Créé dans `zen-creativeinsights-dev-mg` :**

- Dataset `connectedhub` et ses tables : projets, exclusions de sélection, features extraites, et **nos propres tables de créas** pour les trois plateformes
- **Trois services Cloud Run** : `cf-gather-meta-assets-ci`, `cf-gather-tiktok-assets-ci`, `cf-gather-snapchat-assets-ci`
- **Un bucket** `creative_assets_posters`, qui stocke les vignettes de vidéos

**Modifié chez eux :** une colonne `project_id` ajoutée à `meta.tb_api_asset_urls`. Elle est restée **à NULL sur les 1481 lignes**, puisqu'on a finalement basculé sur nos propres tables. Proposer de la retirer.

### La copie de leurs Cloud Functions

Le point à expliquer clairement. Les trois services `-ci` sont **leur code**, déployé sous un autre nom. Leurs services n'ont pas été touchés, leur dernière révision date toujours du 1er juin.

Deux changements sur les trois plateformes :

| Changement | Pourquoi |
| --- | --- |
| Écrit un `project_id` | sinon les créas d'un projet sont toutes celles de son compte annonceur |
| `DELETE` filtré sur compte et projet | **bug actif chez eux** : une créa utilisée sur deux comptes voit sa ligne supprimée par une collecte sur l'autre |

Deux changements propres à Meta :

| Changement | Pourquoi |
| --- | --- |
| Garde sur la pagination | **bug actif chez eux** : `_get_next_pages` lit `data` sans vérifier, donc une erreur Meta en cours de pagination fait tomber toute la fonction |
| Multi-campagnes | le formulaire collectait une liste dont seule la première entrée était envoyée |

Sur ces quatre changements, **deux sont des bugs de leurs services actuels** : le `DELETE` non filtré, qui touche les trois plateformes, et la garde sur la pagination, propre à Meta. Les deux autres sont des ajouts pour nous. C'est l'angle à privilégier : on leur remonte des correctifs, on ne fait pas bande à part.

**Tables séparées**, parce que leur `DELETE` non filtré tourne toujours : une table partagée laisserait leurs collectes emporter nos lignes.

**Ce qu'on n'a pas pu corriger** : la temporisation sur le quota et le filtre de période vivent dans les paquets Artifact Registry, pas dans `main.py`. Ça reste de leur ressort.

### Les vignettes, et pourquoi un bucket de plus

Une créa vidéo n'avait rien à afficher dans la bibliothèque, le seul visuel disponible étant à l'intérieur d'un fichier trop lourd pour une vignette : un master TikTok pèse 35 Mo en moyenne, et jusqu'à 150.

Le serveur en extrait donc une image, une fois, et la conserve. Mesuré sur un master de 30,6 Mo : **981 ms et 20 ko**, contre 10,4 s pour seulement télécharger le fichier. Rien n'est jamais rapatrié en entier, ffmpeg lisant l'objet par requêtes de plage.

C'est un détail d'implémentation, mais il explique le bucket, et il vaut d'être mentionné si la question du coût de stockage vient.

---

## 5. La performance, et la vraie question à poser

Hajar a annoncé une Cloud Function pour la jointure avec la Data Platform, mais pas tout de suite, faute de disponibilité. **C'est ce qui bloque l'étape 5.**

**La question à poser : a-t-on besoin d'une Cloud Function, ou d'un accès en lecture ?**

Ce que j'ai constaté :

- `verisure.mvw_dataplatform_meta_data` existe déjà dans le projet Creative Insights, agrégée par `day, campaign_id, adset_id, ad_id`
- Je **ne peux pas la lire** : elle pointe sur `zen-dataplatform-pm-prd-amg` et je n'ai aucun droit sur la table source
- Mais `verisure.tb_consolidated_data` est **lisible**, et contient déjà la jointure faite à la main : 73 lignes, 17 colonnes, avec `ad_id`, `buying_type`, `total_rdv_bruts`, `total_rdv_nets`, `total_ventes`, `total_cost`

Autrement dit, **la jointure est une jointure BigQuery, et il en existe une version aboutie**. Si c'est bien le cas, un droit de lecture ou une vue suffirait à prototyper, sans attendre le développement d'une fonction.

Ce qui resterait vraiment de leur ressort : la vue actuelle porte un `WHERE account_id = ...` **codé en dur**. La généraliser à n'importe quel annonceur est leur travail, mais c'est un chantier bien plus court que celui qu'ils repoussent.

### La décision qu'ils seuls peuvent prendre

Indépendamment de l'accès, **la règle d'attribution n'est pas tranchée** : la performance est à la maille annonce par jour, la créa à la maille asset, et une annonce porte plusieurs assets. Comment répartit-on la performance d'une annonce entre ses créas ?

C'est une décision méthodologique, pas technique. Sans elle, l'étape 5 ne peut pas être écrite, même avec tous les accès du monde.

---

## 6. Les droits qui manquent en production

Trois demandes, à porter au même ticket. Les deux premières valent d'être expliquées de vive voix, parce que les tentatives précédentes se sont trompées de bénéficiaire.

Toutes concernent le **compte de service** `interface@pmed-portal-prd-mg.iam.gserviceaccount.com`, et **pas mon compte utilisateur**.

| Demande | Sans quoi |
| --- | --- |
| `roles/aiplatform.user` sur `zen-creativeinsights-dev-mg` | l'extraction échoue en 403 dès qu'elle n'est plus lancée en local |
| `Storage Object Viewer` sur `creative_assets_tiktok` | aucune créa TikTok ne s'affiche en production, ni fichier ni vignette |
| `Storage Object Viewer` sur `creative_assets_snapchat` | idem pour Snapchat |

Le troisième point est **une découverte d'aujourd'hui**, et elle mérite d'être racontée : le compte de service ne dispose de `Storage Object Viewer` que sur `creative_assets_meta`, accordé par le ticket précédent. Les deux autres buckets ne lui accordent rien.

Ça ne se voyait pas, parce qu'en local tout retombe sur mon compte, qui est éditeur du projet et lit donc les trois. **Le test local ne pouvait pas révéler le problème.** C'est le genre d'angle mort à signaler, il les concernera dès qu'ils déploieront quoi que ce soit avec ce compte.

---

## Points à ne pas oublier

- Leur signaler le **job nocturne** qui retire des utilisateurs tout outil absent du registre de production. Ça les concernera dès qu'ils testeront un module en cours de développement.
- Récupérer la **liste de features demandées par le client**, que Léonie détenait sur le POC Verisure et qui n'a jamais été formalisée. C'est la seule expression de besoin client connue.
- Mentionner que `Plan modifications DS.md` est à réécrire : il présente encore comme des demandes des choses qu'on a livrées entre-temps.

---

## Compte rendu

Démo complète déroulée comme prévu : accueil, création de projet avec contrôle de périmètre, collecte, bibliothèque de créas, bibliothèque de prompts, puis l'interface vue par un non-admin. Aucune objection sur l'architecture des écrans ni sur le modèle de rôles.

L'annonce sur GCP est passée sans friction. La copie des Cloud Functions plutôt que leur modification a été reçue comme la bonne décision, pour la traçabilité et parce qu'elle laisse leurs pipelines intacts. Ils ont bien noté que les créas elles-mêmes restent dans leurs buckets existants, et que le seul bucket créé sert aux vignettes.

### Le vrai sujet de la réunion, qui n'était pas celui que j'avais préparé

J'arrivais avec la question « Cloud Function ou accès en lecture ». Hajar a déplacé le débat ailleurs : **la sélection manuelle des créas ne passe pas à l'échelle**. Cocher des assets un par un sur un projet qui en compte plusieurs centaines n'est pas un usage tenable, et elle veut que la sélection soit **pilotée par la performance** plutôt que faite à la main.

Abhishek a répondu par une proposition qui débloque la question d'origine : que **l'utilisateur désigne lui-même une table** contenant les données d'engagement, qu'elle vienne de la Data Platform ou d'une table créée via Adverity.

C'est structurant, et ça valide l'intuition de la préparation. On n'attend plus une Cloud Function pour prototyper : **le projet gagne un identifiant de table**, et l'intégration devient manuelle ou automatique selon ce que l'utilisateur fournit. La dépendance à leur disponibilité tombe.

### Ce que cette proposition ne résout pas

Deux choses à garder en tête, parce qu'elles n'ont pas été soulevées en séance :

- **La règle d'attribution reste à trancher.** La performance est à la maille annonce par jour, la créa à la maille asset, et une annonce porte plusieurs assets. Désigner une table dit d'où viennent les chiffres, pas comment les répartir. La question reste entière.
- **La clé de jointure manque toujours sur TikTok et Snapchat.** Leurs scrappers lisent une médiathèque et ne connaissent aucune annonce, donc aucune table d'engagement ne pourra s'y rattacher. La proposition résout la source, pas la clé.

### Ce qui n'a pas été abordé

À reprendre, aucun de ces points n'ayant été traité faute de temps :

- Les **droits manquants en production**, les trois du tableau plus haut. Rien n'a été demandé en séance.
- **Snapchat qui ne renvoie rien** sur le compte de test.
- La **maille des prompts**, par industrie ou par client.
- Le **job nocturne** et la liste de features de Léonie.

### Demandes retenues pour moi

- **Filtrer la bibliothèque par valeur de feature.** Besoin exprimé par Hajar : un data strategist doit pouvoir retrouver rapidement les créas illustrant une caractéristique précise pour les mettre dans une présentation. C'est l'usage concret du module côté métier, et ça n'existe pas aujourd'hui.
- **Déposer un fichier de prompt** en plus de l'éditeur JSON. Proposé par moi, approuvé par les deux. Les deux entrées cohabitent.

### De leur côté

- Hajar et Abhishek affinent l'approche de la sélection automatique et du rattachement à la performance **pendant mes congés**.
- Ils décident s'ils intègrent mes correctifs dans leurs Cloud Functions officielles ou s'ils maintiennent les copies séparément. C'est la suite logique de ce que je leur ai remonté.
