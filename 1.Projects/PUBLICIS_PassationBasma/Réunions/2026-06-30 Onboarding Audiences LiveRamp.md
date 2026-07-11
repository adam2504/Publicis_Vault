---
type: meeting
date: 2026-06-30
---
# Onboarding — Création d'audiences via LiveRamp

Projet : [[Passation Basma]]

**Participants** : Adam, Ilyass, Basma (onboarding)

## Contexte

Onboarding mené par Basma sur la création d'audiences via LiveRamp, dans le cadre de la passation L'Oréal. Adam et Ilyass intégrés au groupe de passation.

## Points clés

- L'équipe Data est la **seule chez Publicis** à créer des audiences sur LiveRamp. Raison : L'Oréal paye 5 FTEs, et historiquement c'est nous qui travaillions le plus sur la division **CPD** → on gère leurs audiences sur cette division, sur lesquelles on fait ensuite des analyses / études.
- Sur **AMC**, on ne lance que les audiences **1P (first party)**.
- On utilise la **licence de Manu** pour LiveRamp.
- Déclenchement : on reçoit des mails qui demandent la création d'audiences → on les crée sur LiveRamp → puis on les pousse sur différents canaux (**Meta, Snapchat, Amazon**).

## Accès LiveRamp

- **Surface** : LiveRamp **Safe Haven** → https://safehaven.liveramp.fr/ (le produit clean room de LiveRamp, pas Connect classique)
- **Compte Basma** :
  - Login : `basma.kadri@publicismedia.com`
  - Mdp : `DataAnalyst`
- En prod aujourd'hui, l'équipe utilise la **licence de Manu**.
- **Plusieurs comptes / workspaces** accessibles dans Safe Haven :
  - `CRF L'Oreal` → compte **Carrefour × L'Oréal** (lié au brief Carrefour + cadre contractuel L'Oréal × Carrefour)
  - `L'Oreal FR` (compte principal observé)
  - `L'Oreal France - LorealParis`, `- NYX`, `- M6` → comptes par marque/division
  - → l'**option API peut être souscrite par compte**, pas globalement : la question devient « **sur quels comptes** a-t-on l'API ? ». Scope multi-comptes à cadrer si l'automatisation doit couvrir plusieurs marques.

### Contacts LiveRamp (via Jules)

| Contact | Email | Rôle |
| --- | --- | --- |
| Thomas Guerre | thomas.guerre@liveramp.com | **Product Operations @ LiveRamp** — connaît les capacités/options du produit (distribution d'audiences, tables de données, data science) → interlocuteur clé pour la faisabilité API / automatisation |
| Younes Ahagoul | younes.ahagoul@liveramp.com | **CSM / commercial** — contrat, droits, dérogations (provisioning API) |
| Ali Lamamra | ali.lamamra@liveramp.com | Alternant LiveRamp, rôle inconnu (secondaire) |

## LiveRamp — modèle mental

LiveRamp = le fournisseur / plateforme d'identité. Plusieurs produits : **Connect** (onboarding + identité RampID), **Data Marketplace**, **ATS**, et **Safe Haven** (= le produit **clean room**). L'Oréal utilise **Safe Haven**.

« Clean room » et « Safe Haven » désignent **la même chose** ici : un environnement neutre où on croise des données sans exposer les données brutes. Safe Haven est le nom du produit.

Les « rôles / interfaces » ne sont **pas** des produits distincts → ce sont des **niveaux d'accès aux modules d'une seule plateforme (Safe Haven)** :

| Module Safe Haven | Rôle d'accès | Usage |
| --- | --- | --- |
| Audiences (Audience Builder + Distribution) | créateur d'audiences | créer + pousser des audiences (Meta, Snap, Amazon) |
| Dashboards (Tableau) | data analyst | visualisation / reporting |
| Analytics Environment (Python/PySpark/BigQuery) | data science | grosses requêtes — **accès via VM** |

### Les 3 sujets LiveRamp chez Publicis

| Sujet | Qui | Module |
| --- | --- | --- |
| **Création d'audiences** ← sujet de l'automatisation visée | Basma | Audiences |
| Extract de data | Data scientists (Lydia / Hajar) | Analytics Environment (VM) |
| Études / reporting | Basma | Dashboards (Tableau) |

> ⚠️ **Cadrage du besoin** : l'automatisation/API visée concerne la **création d'audiences** (sujet Basma), **PAS** le module data science. La VM et la confidentialité concernent l'Analytics Environment (extract DS) — un autre module. Ne pas confondre les deux quand on pose les questions à LiveRamp.

## Les 3 types de briefs

| Type de brief | Source |
| --- | --- |
| Brief AMC | via Renaud |
| Brief 1P | via le Conseil |
| Brief Carrefour | via le client |

## Piste automatisation / API — module ConnectedHub

Croisement workflow réel × API LiveRamp :

`Mail de brief` → `création audience LiveRamp` → `push Meta / Snapchat / Amazon`

**Scope** : automatiser la **création d'audiences** de Basma (objectif n°1), + leur **distribution** si possible ensuite. Pas le data science.

**Donnée déjà dans Safe Haven** : pas de problème d'import. Les dossiers existent déjà dans l'UI Audiences (`My Data`, `Permissioned Taxonomies`, `Analytics Environment Data`, `Saved Audiences`, `Lookalike Group`). Le besoin = **construire des audiences à partir de ce qui est déjà là**, pas importer des fichiers.

| Maillon | Automatisable ? | Piste |
| --- | --- | --- |
| Réception du brief (mail) | Oui (côté ConnectedHub) | — |
| Création de l'audience | Selon process exact + option API | API audiences (si souscrite) |
| Push multi-canal (Meta/Snap/Amazon) | Oui | Distribution : **Always-On** / **Scheduled** dispo nativement dans l'UI, même sans API |

**Briefs majoritairement 1P** (AMC = 1P only, brief 1P via Conseil) → first-party, le chemin le plus simple. Risque résiduel sur la *logique de segmentation* si elle est faite à la main au cas par cas.

### Findings API (lecture doc détaillée — 2026-06-30)

Lecture du portfolio LiveRamp + références Activation API + page API Safe Haven :

- **Activation API** : orientée monde **Connect**. Un segment s'y crée en uploadant un CSV sur un **SFTP** (col. A = Client Customer ID, col. O = nom du segment) ; l'API est **read-only sur les segments** (statut/liste/détail) et pilote la **distribution** (Distribution Managers, Integration Connections OAuth, Deliveries). **« Safe Haven » n'apparaît nulle part** dans ses références → ce n'est pas le pipeline Safe Haven, et l'import SFTP est de toute façon hors sujet (donnée déjà dans SH).
- **Job Management API** : automatise des jobs Python/PySpark/BigQuery dans l'**Analytics Environment** = module **data science**, pas la création d'audiences.
- **Clean Room API** : « set up and manage clean rooms, **questions, and flows** programmatically » → vocabulaire du produit **ex-Habu**. Seule piste qui *pourrait* exprimer « construire une audience à partir de données clean room », **mais** non confirmé que (a) ça couvre la création d'audience, ni (b) que notre instance Safe Haven l'expose.
- Provisioning Activation API = **Service Account (clé JSON via 1Password) délivré par un représentant LiveRamp** → pas self-serve, confirme la dimension contrat/option.

**Conclusion** : d'après la doc publique, **la création d'audience dans Safe Haven n'a pas d'API publique documentée — c'est l'UI (Audience Builder / Advanced Audiences)**. La seule porte programmatique plausible = Clean Room API (questions/flows), non confirmée. Le besoin n°1 (créer par code) **n'est pas validé par la doc** → à trancher avec Thomas (capacité contractée / non publique).

Safe Haven ≠ LiveRamp Clean Room (ex-Habu) ≠ Connect : 3 produits distincts. Bien préciser « Safe Haven » à chaque question.

**Le vrai verrou est CONTRACTUEL (recadrage Jules)** : l'API LiveRamp est une **option du contrat**. Verdict de Jules : *« si c'est en option et qu'on a pas l'option, c'est kaput »* — *« ou ça se discute »*.

- C'est **binaire** : l'option API est souscrite dans le contrat → on peut, sinon → non.
- Si non souscrite, ce n'est **pas mort pour autant** : ça se **négocie / s'ajoute** au contrat (question commerciale, pas juridique/confidentialité).
- La VM des DS (Lydia/Hajar) concerne le **module data science** (Analytics Environment), pas le module Audiences → ne préjuge pas de l'accès API côté création d'audiences. Confidentialité = sujet DS, distinct du nôtre.

**Contrat L'Oréal × Carrefour** : entre en jeu sur le **périmètre de l'option / qui paie quoi**, plutôt que sur de la confidentialité de données.

→ Le **go/no-go appartient au commercial** : **Younes Ahagoul (CSM)** = « a-t-on l'option API ? sinon, coût / négociable ? ». **Thomas Guerre (technique)** n'intervient qu'une fois l'option confirmée.

**Si l'option API n'est pas disponible / pas négociable** : repli sans API, sur le module Audiences existant —
- exploiter au max **Always-On / Scheduled** dans l'UI pour la distribution multi-canal ;
- **standardiser** la création via templates de segments / checklists réutilisables (gain process, pas de code) ;
- ConnectedHub limité à l'amont (réception/parsing des briefs mail), sans toucher à LiveRamp.

## Suites

> Toutes les demandes API doivent être scopées **« API de création / distribution d'audiences »** (sujet Basma), pas « une API » en général — sinon les interlocuteurs répondront sur le module data science.

**En cours :**

- [x] **Pierre** : a validé que Thomas & Younes sont les bons contacts. N'avait pas l'info « API au contrat » lui-même → bascule sur Younes.
- [x] **Mail envoyé à Thomas + Younes** (01/07) → en attente de retour.

Questions posées dans le mail (pour mémoire) :
- **Thomas (Product Ops)** — (1) API pour **créer une audience Safe Haven** à partir des données déjà présentes (Clean Room API / autre) ou UI only ? (2) **distribution par API** des audiences Safe Haven vers Meta/Snap/Amazon (Activation API les voit-elle) ?
- **Younes (CSM)** — option API **incluse au contrat, et sur quels comptes** (L'Oreal FR, CRF L'Oreal, marques) ? sinon coût / périmètre / négociable ?
