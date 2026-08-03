---
type: resource
---

# Vault — Architecture & Conventions

Référence à consulter quand tu crées un projet, une note, ou que tu hésites où mettre quelque chose.

---

## Structure PARA

```
1.Projects/     → projets actifs avec un livrable et une fin définie
2.Areas/        → responsabilités récurrentes sans date de fin
3.Resources/    → connaissances de référence réutilisables, par domaine
4.Archive/      → tout ce qui est terminé ou en pause longue durée
_Templates/     → templates Templater — ne pas déplacer ni renommer
_Attachments/   → images, vidéos et fichiers embarqués dans les notes
```

> ⚙️ Configurer dans Obsidian : Settings → Files & Links → Default location for new attachments → `_Attachments`

**Règle** : un projet se termine, une Area dure. Si quelque chose n'a pas de livrable clair → Area. Si ça ne sera plus jamais utile → Archive.

Areas actives :

| Note                 | Rôle                                                                                                   |
| -------------------- | ------------------------------------------------------------------------------------------------------ |
| [[Accomplissements]] | Collecte en continu de preuves positives : messages, chiffres, réalisations remarquées                 |
| [[Bilan]]            | Synthèse périodique (mensuelle ou avant éval scolaire) — s'appuie sur Accomplissements et les sessions |
| [[TODO]]             | Tâches en cours, regroupées par projet                                                                 |
| `Carrière/`          | Notes de positionnement + CR des points RH/orga/évolution (adamscience, points Baptiste/Jules…)        |
| `Networking/`        | CR d'échanges réseau / benchmarks (autres alternants, autres agences du groupe…)                       |
| `Idées de Projets/`  | Idées de modules ou projets en attente de validation / ressources                                      |

> Les notes-index durables (Accomplissements, Bilan, TODO) restent à la racine de `2.Areas/`. Les CR de réunion carrière/positionnement vont dans `Carrière/`, les échanges réseau dans `Networking/`. Les points opérationnels récurrents (ex. Worklist Monday) restent à la racine tant qu'ils ne justifient pas leur propre sous-dossier.

---

## Nommage des dossiers projet

Format : `CLIENT_NomCourt` — client en majuscules, nom en PascalCase, pas d'espaces ni de tirets.

```
MINE_AMCAnalytics
MINE_FeedGenCategorisation
LOREAL_BrandStore
LOREAL_ViralBeauty
VERISURE_CreativeInsights
```

Clients connus : `LOREAL`, `MINE`, `VERISURE`, `PUBLICIS`, `STELLANTIS`, `LEAPMOTOR`

---

## Structure interne d'un projet

```
CLIENT_NomProjet/
  Nom du Projet.md           ← note de référence principale, nommée d'après le projet
  Sessions/                  ← une note par session de travail
    Session YYYY-MM-DD.md
  Réunions/                  ← comptes-rendus
    YYYY-MM-DD Sujet.md
  [notes thématiques libres] ← ex: "Sous Module - Full Funnel.md", "Décisions API.md"
```

La note principale porte le **nom du projet**, pas un nom générique. Cela la rend identifiable dans le graphe et évite les ambiguïtés de lien quand plusieurs projets coexistent.

```
MINE_AMCAnalytics/AMC Analytics.md
MINE_FeedGenCategorisation/FeedGen Catégorisation.md
LOREAL_BrandStore/Brand Store v2.md
```

Les sous-dossiers `Sessions/` et `Réunions/` ne sont créés que si le projet a du volume. Un petit projet peut n'avoir qu'une note principale.

### `## Dernières sessions` — résumé rolling dans la note principale

Chaque note de projet qui a des sessions doit inclure une section `## Dernières sessions` **à la fin de la note principale** (après Réunions / Ressources). Format : une ligne par session, du plus récent au plus ancien, limitée aux 4-6 dernières.

```markdown
## Dernières sessions

- **YYYY-MM-DD** — [résumé en 1-2 phrases : ce qui a été fait, décisions clés, état laissé]
- **YYYY-MM-DD** — ...
```

**But** : permettre de lire l'état récent d'un projet sans ouvrir les notes Sessions/ individuelles. Les sessions restent la trace détaillée ; cette section est le résumé navigable.

**Convention de mise à jour** : à compléter à la fin de chaque session de travail, en ajoutant la nouvelle entrée en tête de liste.

### Frontmatter des notes de projet

```yaml
---
type: project
statut: en cours # en cours | archivé
discipline: dev # dev | data-analyst | data-science | [dev, data-science]
implication: lead # lead | contributeur | participant | onboardé
client: LOREAL # LOREAL | MINE | VERISURE | PUBLICIS
---
```

`discipline` permet de filtrer les projets par type de travail dans la vue Propriétés d'Obsidian. Valeurs possibles : `dev`, `data-analyst`, `data-science` (liste YAML si hybride).

### `type` — toute note en a une

La vue Propriétés d'Obsidian ne sert à rien si des notes n'ont pas de `type`. **Aucune note ne doit être créée sans frontmatter**, même une note thématique de deux lignes.

| `type` | Pour quoi | Champs en plus |
| ------ | --------- | -------------- |
| `project` | Note principale d'un projet | `statut`, `discipline`, `implication`, `client` |
| `meeting` | Réunion **et** session de travail | `date`, `projet` |
| `note` | Note thématique dans un dossier projet | `projet` |
| `area` | Note-index durable de `2.Areas/` | — |
| `resource` | Référence réutilisable de `3.Resources/` | — |
| `client` | Note-hub d'un client dans `3.Resources/Clients/` | — |
| `idea` | Piste dans `2.Areas/Idées de Projets/` | `statut`, `client` |
| `context` | `CONTEXT.md` — synthèse régénérée | `Dernière mise à jour` |

### `implication` — mon niveau d'implication réel

Champ **obligatoire** sur toute note de projet. Il dit ce que *moi* (Adam) je fais sur le projet — indépendamment du volume ou du détail de la note.

| Valeur | Signification |
| ------ | ------------- |
| `lead` | Je porte le projet, je produis le livrable principal. C'est mon travail. |
| `contributeur` | J'apporte une contribution concrète (code, analyse) sans porter le projet. |
| `participant` | Points réguliers + knowledge projet, mais **pas** de contribution au livrable de ma part. |
| `onboardé` | Je connais le projet mais je n'ai **encore rien produit**. La note documente le travail de quelqu'un d'autre (voir « Personnes clés »). |

**Règle de lecture (pour un LLM ou pour moi) :**

- Ne jamais déduire mon implication du volume de la note ni de la présence de `Sessions/`. Une note `onboardé` peut être très détaillée (ex. [[EVBB Scoring]] documente le travail de Dan) ; une note `participant` peut rester minimale (ex. [[Brand Store v2]]).
- **Si `implication: onboardé` → ne jamais affirmer que j'ai travaillé sur ce projet.** C'est de la doc de référence, pas mon livrable. Attribuer le travail à la personne listée dans « Personnes clés ».
- En cas de doute sur ce que j'ai fait, se fier à ce champ, pas à la structure des dossiers.

**Placement selon `implication` :**

- `lead` / `contributeur` / `participant` → le projet vit dans `1.Projects/` (c'est un projet actif que je porte ou auquel je participe).
- `onboardé` → le projet vit dans `3.Resources/Onboardings/`, **pas** dans `1.Projects/`. Tant que je ne produis rien, c'est de la référence, pas un de mes projets actifs.

**Cycle de vie d'un `onboardé` :**

- Je commence à produire dessus → passer `onboardé` à `participant`, `contributeur` ou `lead` selon le cas **et** déplacer le dossier vers `1.Projects/`. Les liens `[[Nom]]` par nom de fichier restent valides après déplacement.
- Il ne se concrétise jamais et devient sans objet → déplacer vers `4.Archive/` (ou le supprimer si purement informatif).
- Règle de sortie : un `onboardé` qui n'a bougé ni vers Projects ni vers Archive **depuis ~3 mois** est à revoir — soit je m'y suis mis (→ Projects), soit c'est de la veille morte (→ Archive).

---

## Pièces jointes (images & vidéos)

Obsidian embarque images et vidéos directement dans les notes.

**Intégration** : glisser-déposer ou coller (`Ctrl+V`) → le fichier est copié dans `_Attachments/` et le lien inséré automatiquement.

**Syntaxe** :

```
![[screenshot.png]]          → image pleine largeur
![[screenshot.png|400]]      → image redimensionnée (largeur en px)
![[demo.mp4]]                → vidéo lue inline
```

**Formats supportés** : PNG, JPG, GIF, SVG, WebP — MP4, WebM, MOV pour les vidéos.

**Conseils** :

- Pour les messages reçus (Teams, mail) → screenshot PNG dans [[Accomplissements]]
- Pour les UIs ou démos → court clip MP4 ou GIF (éviter les longues vidéos, trop lourdes)
- Nommer les fichiers de façon descriptive : `2026-06-19-antoine-barbet-incident.png`

---

## Nommage des notes

| Type               | Format               | Exemple                                      |
| ------------------ | -------------------- | -------------------------------------------- |
| Note principale    | Nom du projet        | `AMC Analytics`, `Brand Store v2`            |
| Session de travail | `Session YYYY-MM-DD` | `Session 2026-06-19`                         |
| Réunion / point    | `YYYY-MM-DD Sujet`   | `2026-06-10 Cadrage hub`                     |
| Note thématique    | Nom court            | `Sous Module - Full Funnel`, `Accès & Rôles` |

Le préfixe `Session` (et le rangement dans un dossier `Sessions/`) distingue nettement les sessions des réunions (qui commencent par la date). C'est ce qui permet de les **exclure du graphe** sans toucher aux liens : le graphe Obsidian est filtré par `-path:Sessions/` (voir `.obsidian/graph.json`). Les sessions restent accessibles via leur lien d'en-tête et le panneau backlinks.

---

## Templates

Plugin : **Templater** (community plugin, activé).
Usage : ouvrir une note → `Ctrl+P` → _Templater: Open Insert Template Modal_.

> ⚠️ Il faut qu'une note soit ouverte avant d'insérer un template.

| Template  | Usage                        | Variables insérées             |
| --------- | ---------------------------- | ------------------------------ |
| `Projet`  | Nouvelle note de projet      | Date du jour, titre de la note |
| `Session` | Log d'une session de travail | Date longue + courte           |
| `Réunion` | Compte-rendu de réunion      | Date du jour, titre de la note |

---

## Liens entre notes

### Règles générales

- **Les notes de `3.Resources/Clients/`** et les notes de type MOC (hubs : `Mine Platform`) utilisent des chemins complets : `[[1.Projects/MINE_AMCAnalytics/AMC Analytics]]`
- **Partout ailleurs**, utiliser le nom de fichier seul (Obsidian résout automatiquement) : `[[AMC Analytics]]`
- Toujours lier une session vers son projet en en-tête : `[[AMC Analytics]]`
- **Ne pas lister les sessions en bloc depuis la note principale du projet** — trop répétitif, ça bruite le graphe. Elles restent accessibles via leur lien d'en-tête et le panneau backlinks. Les **réunions** (jalons, peu nombreuses) peuvent, elles, être listées.
	- **Exception : la citation inline est autorisée.** Un `cf. [[Session 2026-07-23]]` accroché à une affirmation précise (une décision, un chiffre, un correctif) est de la traçabilité, pas du listing. Ce qui est proscrit, c'est la *section* qui énumère toutes les sessions — le rolling `## Dernières sessions` la remplace.
- Les personnes sont mentionnées en **texte brut** (prénom suffit) — pas de lien vers Équipe & Contacts

### Liens avec alias (texte affiché différent du nom de fichier)

```
[[NomDuFichier|Texte affiché]]
```

Exemple : `[[Mine Platform|ConnectedHub]]` → affiche "ConnectedHub", pointe vers Mine Platform

### Liens dans les tableaux

Le pipe `|` est réservé par Markdown pour les colonnes de tableaux. Pour mettre un lien avec alias **dans une cellule**, échapper le pipe avec `\|` :

```
[[Mine Platform\|ConnectedHub]]
```

---

## Philosophie du graphe

Trois types de nœuds dans le graphe :

| Rôle           | Note                  | Comportement                                                                                          |
| -------------- | --------------------- | ----------------------------------------------------------------------------------------------------- |
| Hub plateforme | Mine Platform         | Reçoit des liens de tous les projets Mine ; ne lie vers rien d'autre que ses modules et ses incidents |
| Répertoire     | Équipe & Contacts     | Peut recevoir des liens explicites depuis d'autres notes ; n'a aucun lien sortant                     |
| Note de projet | `AMC Analytics`, etc. | Lie vers Mine Platform en en-tête ; personnes citées en texte brut dans le corps                      |

**Règles qui en découlent :**

- Toute note d'un projet Mine commence par `Plateforme : [[Mine Platform]]`
- `[[Équipe & Contacts]]` n'a aucun lien sortant — c'est un répertoire, pas un hub de navigation
- Les sessions lient leur projet en en-tête (`[[AMC Analytics]]`), pas Équipe & Contacts
- Les notes thématiques et les réunions lient leur projet principal et les personnes concernées, rien d'autre

---

## Resources — organisation par domaine

```
3.Resources/
  Vault - Architecture & Conventions.md   ← ce fichier
  Équipe & Contacts.md                    ← annuaire de l'équipe (aucun lien sortant)
  Asana - Sync & Conventions.md           ← mapping vault → Asana (cf. /asana-sync) : [[Asana - Sync & Conventions]]
  MMM — Notions.md                        ← notions métier MMM
  Mine Platform/    ← hub ConnectedHub : modules actifs & archivés, stack, incidents
  AMC/              ← doc Amazon Marketing Cloud : notions, contrat d'ingestion BQ
  Clients/          ← une note par client : LOREAL, STELLANTIS, VERISURE
  Onboardings/      ← projets où je suis onboardé sans encore produire (implication: onboardé)
```

**Chaque note client est un hub** : elle liste les projets actifs et archivés du compte (chemins complets) + les contacts. Tout projet doit être atteignable depuis sa note client — un projet qu'aucun hub ne référence est une note orpheline.

Dossiers à créer selon les besoins :

```
  Looker/   → dashboards, embedding iframes
  Métier/   → glossaire, définitions KPIs, conventions media
```

---

## Archive

Quand archiver un projet :

- Livrable livré et validé
- Projet annulé ou indéfiniment suspendu
- Plus aucune action prévue dans les 3 prochains mois

**Procédure d'archivage** — les 5 étapes, dans l'ordre. Archiver n'est pas qu'un déplacement de dossier : c'est ce qui décroche le projet des hubs et du TODO. En sauter une laisse des liens qui mentent.

1. Déplacer le dossier entier dans `4.Archive/` (`git mv` pour garder l'historique).
2. Passer `statut: archivé` dans la frontmatter de la note principale.
3. Ajouter en tête de la note principale une section **`## Statut — archivé le YYYY-MM-DD`** : pourquoi c'est clos, ce qui reste réutilisable, et à quelle condition ça pourrait repartir. Le reste du contenu n'est pas modifié.
4. **Mettre à jour les hubs** : déplacer la ligne de « Projets actifs » vers « Projets archivés » dans la note client, et dans `Mine Platform` si c'est un module ConnectedHub.
5. **Retirer les tâches du [[TODO]]** — sinon le projet reste vivant dans Asana et dans les weeklies.
