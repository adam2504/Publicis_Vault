---
type: resource
---

# Incident Plateforme — 11 août 2026

## Compte GitHub compromis · payload obfusqué force-pushé sur toutes les branches

**Statut** : Poste vérifié sain, repo local purgé — remote en attente de validation
**Environnement touché** : branches de travail du repo `Publicis-Media-France-FR5140/mine` — `main` et `develop` (donc staging et production) non affectés
**Vecteur** : compte GitHub d'Adrien Neto-Ferreira (Yield Studio), `adrnetof_publicis`
**Modules impactés** : aucun en exécution — l'attaque vise les postes de développement, pas les environnements déployés

---

## Ce qui s'est passé (version courte)

Le 11 août au matin, le compte GitHub d'Adrien a poussé un payload obfusqué sur quasiment toutes les branches du repo, toutes à la même heure, suivant le même format : reprise du message de commit précédent, ajout du payload, force push. Eddie a détecté et alerté en fin d'après-midi, retiré les accès au compte, puis retiré les accès de la team Yield globale faute d'avoir la main sur la gestion des teams GitHub.

L'attaque cible les **points d'entrée exécutables** du repo. Elle ne fait rien tant qu'un développeur ne matérialise pas les fichiers infectés sur son disque et ne lance pas un build ou un serveur de dev.

---

## Le payload

Injecté dans ~25 fichiers par branche, tous exécutables :

```ts
import { createRequire } from 'module';
const require = createRequire(import.meta.url);
// ... code légitime inchangé ...
program.parse();
eval("global.o='5-745-du';"+atob('<7 345 octets de base64>'))
```

`createRequire` sert à retrouver `require()` en contexte ESM, donc à charger des modules Node depuis un fichier module. La couche décodée fait 5 507 octets de JavaScript obfusqué par table de permutation (`var _$_b9af=(function(p,j){…}`).

Cibles et conditions de déclenchement :

| Fichier                              | Ce qui l'exécute                          |
| ------------------------------------ | ----------------------------------------- |
| `server/src/index.ts`, `config/index.ts` | `npm run dev` du backend              |
| `client/postcss.config.mjs`          | tout `npm run dev` / build du client       |
| 18 × `esbuild.mjs` (cloud functions) | tout build ou déploiement GCP              |
| `cli/src/index.ts` + scripts serveur | appel du CLI via tsx                       |

**Le point le plus vicieux** : l'attaquant a réécrit des commits existants en conservant auteur, date, message et contenu réel. Mes propres commits sont revenus signés de mon nom, avec mes messages, plus le payload. Un contrôle sur l'auteur ne détecte rien — il faut scanner les diffs ou le contenu des refs.

---

## Ce que j'ai vécu côté contacts

**Eddie Ratignier**, 16h48, alerte l'équipe : le compte d'Adrien semble compromis, il pousse sur toutes les branches avec des payloads obfusqués. À 17h21 : *"Tout le monde aucun merge jusqu'à nouvel ordre."* À 17h54 il me contacte directement pour me demander si je suis sur ma branche locale `chore/amc-jules-feedback-main`, me dit de ne faire ni pull ni fetch, et de chercher des références à `eval(` n'importe où.

**Antoine Barbet** réagit à 16h58 — *"c'est pas bon du tout ça"*, *"quelles incidences ?"* — et ouvre à 17h35 un ticket GSO (Global Security Office), REQ6524231, livraison estimée au 12 août. Eddie répond à 17h05 que main et prod ne sont pas impactées, mais qu'il ne faut ni utiliser les autres branches, ni les run en local, ni tenter de merge.

**James Hemery**, 18h47 : *"Ok j'ai fini de restore correctement le git"*, suivi de *"Je pense que ton ordi et celui de Cyrille ont des chances d'être infecté"* et *"Probablement à reset + rotation des keys + mot de passe par sécurité"*. Eddie ajoute à 18h50 : *"adam aussi potentiellement du coup j'imagine"*.

Le lendemain 12 août, **Cyrille Masson** demande à 9h12 s'il existe une procédure pour détecter si on est infecté ou pour nettoyer le poste. Antoine à 9h59 : *"il faudra qu'on sache pour le call de demain en tout cas"*.

---

## Ce que j'ai vérifié sur mon poste

L'hypothèse d'Eddie et James — mon poste potentiellement infecté — s'est révélée fausse, et c'est vérifiable point par point.

Les objets malveillants **sont** bien descendus chez moi : l'auto-fetch de VS Code les a récupérés le **11/08 à 12:31:38** (`fetch origin --quiet: forced-update`), isolés dans un pack de 228 Ko. C'est plus de 4 heures avant la détection de 16h48 — ce qui donne une borne haute sur le début de l'attaque, plus précise que l'heure d'alerte.

Mais un fetch écrit dans `.git/objects`, **pas dans le working tree**. Ce sont deux zones distinctes du disque, et seuls checkout, merge, pull, rebase ou cherry-pick matérialisent un blob en vrai fichier source. Le payload n'a donc jamais existé sous forme de fichier exécutable chez moi.

| Contrôle                                              | Résultat                                     |
| ----------------------------------------------------- | -------------------------------------------- |
| Index git des fichiers cibles                         | Hash d'origine — jamais passés par l'index    |
| Reflog HEAD complet du 11/08                          | Aucun checkout d'un ref infecté               |
| Working tree, toutes extensions                       | Marqueur absent partout                       |
| Clés Run HKCU/HKLM, dossier Démarrage                 | Rien d'anormal (corporate uniquement)         |
| Tâches planifiées depuis le 11/08                     | Aucune                                        |
| Scripts déposés dans APPDATA / Temp / profil          | Aucun d'inexpliqué                            |
| `.gitconfig`, `.npmrc`, `.git-credentials`, clés SSH  | Inchangé depuis mars / inexistants            |
| ADC gcloud                                            | Modifié à 11h08, soit **avant** l'arrivée du payload |

Étendue dans ma copie : **148 refs `origin/*` infectés**, 6 sains (`main`, `develop`, `HEAD`, `Feature_FusionVideoAdmin`, `feat/amc-saved-analyses`, `feat/amc-saved-analyses-main`). Mes **14 branches locales : toutes saines**, scannées sur l'intégralité de leur arbre.

---

## Purge du repo local — 12 août

Plutôt qu'un reclone (1,9 Go de `node_modules` à réinstaller, plus les `.env`, `CLAUDE.md` et docs gitignorés à reporter à la main), purge chirurgicale du repo existant :

1. Suppression des 148 refs `refs/remotes/origin/*` infectés, conservation des 6 sains
2. `git gc --prune=now` — `.git` passe de 99 à 68 Mo
3. Vérification sur **toute la base d'objets**, pas seulement les refs : **0 occurrence** du marqueur sur les 68 158 objets (blobs, commits, trees)
4. `git fsck` propre, 14 branches locales aux mêmes tips, working tree identique

Puis mise en sécurité en attendant la validation du remote :

- `main` local avancé sur l'ancien `origin/main` sain (`c0385d487`) — devient la base de branchement
- `git remote remove origin` — plus aucun remote ni ref distant, `fetch` inopérant et `pull` en échec
- `git.autofetch` déjà à `false` dans VS Code

Sauvegarde du travail non commité au moment de l'incident : `C:\dev\backup-connectedhub-2026-08-12\`.

---

## Ce qui reste en attente

Message envoyé à James pour savoir si le remote est déclaré sain, et surtout **si `main` a été réécrit** pendant sa restauration. Si `c0385d487` n'est plus un ancêtre du nouveau `main`, un rebase sera nécessaire avant tout push.

Séquence de reconnexion prévue :

```bash
git remote add origin https://github.com/Publicis-Media-France-FR5140/mine.git
git fetch origin main
grep -r "createRequire(import.meta.url)" .          # doit être vide
git merge-base --is-ancestor c0385d487 origin/main  # sinon rebase nécessaire
```

Mes branches AMC n'ont pas besoin d'être restaurées côté remote : tout est déjà dans `main`, et la convention veut qu'elles soient supprimées après merge. Un seul commit reste hors `main` (`76dbb7212`), sauvegardé en patch.

---

## Ce que j'en retiens

**Les métadonnées git ne sont pas une preuve.** Mon premier verdict sur ces branches était « auteurs légitimes, rien à signaler » — parce que j'avais lu les auteurs et les messages sans ouvrir les diffs. C'est exactement ce sur quoi l'attaque compte.

**Distinguer le repo contaminé du poste infecté.** Avoir des objets malveillants dans `.git` n'est pas être infecté. Cette distinction change la réponse : pas de reformatage de poste, pas de panique, une purge ciblée et une vérification.

**Une purge se prouve, elle ne se promet pas.** Scanner l'intégralité de la base d'objets après coup transforme « je pense que c'est propre » en constat vérifiable, et rend le reclone inutile.

Voir aussi [[Workflow Git & Merge]] et [[Incident 2026-06-19 Node.js 24.17]].
