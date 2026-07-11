# Incident Plateforme — 19 juin 2026

## Rupture BigQuery sur /develop · Node.js 24.17.0

**Statut** : Résolu  
**Environnement touché** : staging (/develop) uniquement — production (/main) non affectée  
**Durée** : ~24h (18 juin milieu de journée → 19 juin après-midi)  
**Modules impactés** : tous les outils connectés à BigQuery

---

## Ce qui s'est passé (version courte)

Une version Node.js défaillante (24.17.0) a été publiée le 18 juin. Le Dockerfile de ConnectedHub n'épinglait pas de version patch (`FROM node:24`), donc le build automatique déclenché par un push Yield Studio ce jour-là a récupéré cette version cassée. Résultat : toutes les requêtes BigQuery en staging échouaient systématiquement.

**Eddie a publié un post-mortem officiel** confirmant que l'origine est strictement externe et n'est pas liée au code poussé par qui que ce soit sur la branche.

---

## Ce que j'ai vécu côté contacts

### Antoine Barbet — Head of Technology Operations for Media

Antoine me contacte à 15h11 pour me demander si j'ai fait des commits hier après-midi. Je confirme que oui (sur ma branche, avec un PR mergé ce matin). Il répond qu'ils vont renforcer la gouvernance sur les merges _"parce qu'on a des choses qui ont cassé"_. Il me dirige vers Cyrille Masson (Yield Studio).

Cyrille observe dans mon merge un diff massif : 455 fichiers modifiés, 338 ajoutés, 33 supprimés, 13 remplacés — dont des fichiers que je n'ai pas touchés fonctionnellement (ex: `budgetCalculator`). Antoine conclut à 15h29 : _"ça serait toi"_.

À 16h04, après avoir appelé Cyrille et investigué, je réponds à Antoine que la source potentielle est un bug Node.js récupéré par le build auto lors d'un push Yield. Je lui indique qu'Eddie est en train de repush develop sur la version antérieure.

À 16h37, je le tiens informé : _"la version develop devrait fonctionner mtn, eddie à rollback sur la version ultérieure de node"_.

À 17h07, Antoine demande _"on avait MAJ node ?"_. Je lui explique à 17h22 : _"la version de node dans le dockerfile était pas fixée, ça prenait automatiquement la dernière version 'stable' pour build"_.

À 17h31, Antoine clôt l'incident : _"okay — bon tout ça est OK désormais — merci encore"_.

### Jean-Marc Orhan — Directeur Général Publicis Performics

Jean-Marc m'a également contacté dans la journée au sujet de l'incident, signe que l'impact remontait jusqu'aux clients Performics utilisant les outils data de la plateforme.

---

## Les deux problèmes distincts qui ont été confondus

### 1. Les ~800 fichiers "fantômes" dans mon merge

**Ce que Cyrille a vu** : un diff massif de fichiers pas liés à mes développements AMC.

**Cause réelle** : VS Code avait `"editor.formatOnSave": true` activé globalement. À chaque sauvegarde automatique, Prettier reformatait silencieusement les fichiers ouverts, même ceux juste consultés. Ces reformatages s'accumulaient comme de vraies modifications dans le diff de ma branche.

**Ce que ce n'était pas** : aucune modification logique ou fonctionnelle — uniquement de la mise en forme (indentation, guillemets, virgules trailing, sauts de ligne). Aucun code cassé, aucun comportement changé.

**Fix** : `formatOnSave` désactivé dans les settings VS Code.

### 2. Les modules BigQuery cassés

**Cause réelle** : régression Node.js 24.17.0 sur la réutilisation des sockets HTTP keep-alive (`ERR_STREAM_PREMATURE_CLOSE`). Les libs Google Cloud (`gaxios`, `@google-cloud/bigquery`) utilisent exactement ce mécanisme.

Ref : [nodejs/node#63989](https://github.com/nodejs/node/issues/63989)  
Dernière version stable : **24.16.0**

**Déclencheur** : push Yield Studio sur /develop le 18 juin → build auto → `docker pull node:24` → résout sur 24.17.0.

**Aucun lien avec mon code.**

**Fix** : épinglage `FROM node:24.16.0-slim` dans le Dockerfile, déployé par Eddie.

---

## Corrections appliquées

1. Rollback de staging vers une version Node.js stable antérieure (immédiat)
2. Épinglage strict `node:24.16.0-slim` dans le Dockerfile sur toutes les branches
3. Désactivation de `formatOnSave` dans mon VS Code (ne se reproduira plus)

