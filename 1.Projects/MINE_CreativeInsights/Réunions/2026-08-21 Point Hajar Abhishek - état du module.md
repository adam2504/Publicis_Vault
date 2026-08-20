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
2. Leur **accord tacite** sur ce que j'ai touché dans GCP, et la transparence sur la copie de leur Cloud Function
3. Une **réponse claire sur la performance** : de quoi a-t-on besoin, et quand

---

## 1. Démo

**En local**, la production n'ayant pas encore les droits Vertex AI. Le rôle `roles/aiplatform.user` sur `zen-creativeinsights-dev-mg` est demandé mais pas accordé, donc l'extraction échoue en déployé.

Parcours à montrer, dans cet ordre :

| Écran | Ce qu'il illustre |
| --- | --- |
| Accueil du module | liste des projets, avec l'état d'avancement dérivé |
| Création de projet | le **contrôle de périmètre** avant de lancer quoi que ce soit |
| Page projet | collecte, journal d'activité, carte d'extraction |
| Bibliothèque de créas | filtres, sélection, et le **panneau de features** sous une créa agrandie |
| Bibliothèque de prompts | le prompt Verisure avec ses 54 features |

**Deux moments qui valent la démonstration** : le contrôle de périmètre qui répond en moins d'une seconde et nomme une campagne invalide, et le panneau de features ouvert sur une vidéo, qui montre concrètement ce que le modèle a lu.

**À dire d'emblée**, pour éviter qu'on le découvre en cours de route : le prompt de la bibliothèque est **calibré Verisure** alors que le compte de test est immobilier. Les features sectorielles sortent donc vides, et c'est le comportement correct. La mécanique est prouvée, la pertinence ne l'est pas encore.

---

## 2. Pourquoi seul Meta fonctionne

C'est la question qui viendra, autant la traiter avant qu'elle soit posée.

**Ce n'est pas un manque de câblage de mon côté.** Les trois scrappers n'interrogent pas la même classe d'API :

| Plateforme | Ce que l'API renvoie |
| --- | --- |
| **Meta** | la **structure publicitaire** : campagnes, puis annonces, puis créas. Donc `ad_id` |
| **TikTok** | `/v1.3/file/video/ad/search/`, soit la **bibliothèque vidéo** de l'annonceur |
| **Snapchat** | `/v1/adaccounts/<id>/media`, soit **tous les médias** du compte |

TikTok et Snapchat répondent « voici tout ce que l'annonceur possède », pas « voici ce qui a été diffusé ». Sans notion d'annonce, **aucune jointure vers la performance n'est possible**, et ajouter une colonne n'y changerait rien.

**Ce n'est pas une limite des plateformes non plus.** TikTok expose `/v1.3/ad/get/` et Snapchat ses endpoints d'annonces. La liaison créa vers annonce est atteignable, mais elle demande **un appel supplémentaire à développer**, pas un champ de plus.

C'est le vrai chiffrage à leur donner : ce n'est pas un correctif, c'est une évolution du scrapper.

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

- Dataset `connectedhub` et ses **7 tables** : projets, exclusions de sélection, features extraites, et notre propre table de créas
- **Un service Cloud Run**, `cf-gather-meta-assets-ci`

**Modifié chez eux :** une colonne `project_id` ajoutée à `meta.tb_api_asset_urls`. Elle est restée **à NULL sur les 1481 lignes**, puisqu'on a finalement basculé sur notre propre table. Proposer de la retirer.

**Aucun bucket créé.** On réutilise `creative_assets_meta`, avec les mêmes chemins, donc rien n'est retéléchargé ni dupliqué.

### La copie de leur Cloud Function

Le point à expliquer clairement. `cf-gather-meta-assets-ci` est **leur code**, déployé sous un autre nom, avec quatre changements. Leur service n'a pas été touché, sa dernière révision date toujours du 1er juin.

| Changement | Pourquoi |
| --- | --- |
| Écrit un `project_id` | sinon les créas d'un projet sont toutes celles de son compte annonceur |
| `DELETE` filtré sur compte et projet | **bug actif chez eux** : une créa utilisée sur deux comptes voit sa ligne supprimée par une collecte sur l'autre |
| Garde sur la pagination | **bug actif chez eux** : `_get_next_pages` lit `data` sans vérifier, donc une erreur Meta en cours de pagination fait tomber toute la fonction |
| Multi-campagnes | le formulaire collectait une liste dont seule la première entrée était envoyée |

**Deux de ces quatre corrections sont des bugs de leur service actuel**, pas des ajouts pour nous. C'est l'angle à privilégier : on leur remonte des correctifs, on ne fait pas bande à part.

**Table séparée** `connectedhub.tb_meta_asset_urls`, parce que leur `DELETE` non filtré tourne toujours : une table partagée laisserait leurs collectes emporter nos lignes.

**Ce qu'on n'a pas pu corriger** : la temporisation sur le quota et le filtre de période vivent dans les paquets Artifact Registry, pas dans `main.py`. Ça reste de leur ressort.

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

## Points à ne pas oublier

- Demander le rôle **`roles/aiplatform.user`** pour `interface@pmed-portal-prd-mg.iam.gserviceaccount.com`, en précisant bien qu'il s'agit du **compte de service** et pas de mon compte utilisateur. Les deux demandes précédentes se sont trompées de bénéficiaire.
- Leur signaler le **job nocturne** qui retire des utilisateurs tout outil absent du registre de production. Ça les concernera dès qu'ils testeront un module en cours de développement.
- Récupérer la **liste de features demandées par le client**, que Léonie détenait sur le POC Verisure et qui n'a jamais été formalisée. C'est la seule expression de besoin client connue.

---

## Compte rendu

*À compléter après la réunion.*
