---
type: resource
---

# Asana — Sync & Conventions

Référence pour la synchro **vault → Asana**. But : donner de la visibilité aux managers (Manu, Jules) sur les projets ouverts et leurs grandes étapes, **sans sur-organiser**.

La synchro se lance depuis Claude Code avec la commande `/asana-sync` (déclenchement manuel, ~1×/jour). Elle lit `[[TODO]]` + cette note, propose un récap, et n'écrit dans Asana qu'après validation.

---

## Règles de sync

- **Grandes étapes uniquement** → une grande étape = une tâche Asana. Les mini-steps restent dans le vault.
- **Repérage sémantique** : l'orga puces/sous-puces du TODO est mouvante, on ne s'y fie pas rigidement.
- **Récap puis validation** avant toute écriture.
- **Préférer l'existant** : ne pas créer de projet/section Asana sans validation explicite.
- **Non synchronisé** : sections perso/meta du TODO (`Réseau/Rencontres`, `Outillage`).

---

## Identifiants

| Entité | gid |
| --- | --- |
| Workspace `publicismedia.com` | `161888073952935` |
| Moi (Adam Jouini) | `1212826067798031` |

---

## Champs des tâches (conventions)

À renseigner à la création : **Assignee** = moi, **Estimated time** (durée estimée, champ **en minutes** → estimer en heures × 60), **Status** (défaut **TODO**), **échéance** (`due_on`) seulement si le vault donne une date. Détail des règles dans la commande `/asana-sync`.

### Custom fields — projets TECH 2026 & DATA 2026

*(mêmes gids sur les deux projets ; vérifier via l'API pour tout autre projet)*

| Champ | gid |
| --- | --- |
| Estimated time (**en minutes** — heures × 60) | `1203941520545343` |
| Status (enum) | `1212840479994727` |
| ✨ Tps passé (j) — *ne pas utiliser pour l'estimé* | `1209898296746645` |
| Effort (enum) | `1212840479994717` |
| Priority (enum) | `1212840479994722` |

**Options du champ Status :**

| Option | gid |
| --- | --- |
| BACKLOG | `1212841158029562` |
| OK | `1212841158029563` |
| KO | `1212841158029564` |
| TODO | `1212841158029565` |
| RUN | `1212841158029566` |
| CADRAGE | `1212841158029567` |
| FIL ROUGE | `1212841158029568` |

---

## Projets & sections Asana

### p/DATA ANALYST/ALL PROJETS TECH 2026 — `1212840479994695`
*Organisé par chantier tech.*

| Section | gid |
| --- | --- |
| ALL | `1213662099502635` |
| SHOPPING PRO MAX (SPM) | `1212840479994750` |
| MARKETING MIX MODELING (MMM) | `1212898628689485` |
| ETUDES MATHILDE | `1212913303238781` |
| CREATION D'AUDIENCES LIVERAMP | `1216264462374940` |
| AMC ANALYTICS | `1216264462374946` |
| SIMBA | `1213338868312610` |
| STLA | `1213662099502638` |

### p/DATA ANALYST/ALL PROJETS DATA 2026 — `1212841158029655`
*Organisé par client.*

| Section | gid |
| --- | --- |
| STL - STELLANTIS | `1212841158029660` |
| OAF - L'ORÉAL | `1212841158029663` |
| MANUTAN | `1212841158029666` |
| ITM - INTERMARCHÉ | `1212841158029669` |
| AIR FRANCE | `1212841158029674` |
| MMA | `1212841158029677` |
| MCDO | `1212841158029680` |
| CIO | `1212841158029683` |
| BONDUELLE | `1212841158029686` |
| SRP - SHOWROOMPRIVÉ | `1212841158029689` |

### p/DATA ANALYST/ALL CLIENTS 2026 — `1212836379735592`

| Section | gid |
| --- | --- |
| CLIENTS | `1212836379735596` |

### Projets clients dédiés

| Projet | gid |
| --- | --- |
| p/L'ORÉAL/LR+AMZN (ALL) | `1213135387234460` |
| p/AMC/MODULE ANALYSE | `1213662099502473` |

---

## Correspondances chantier vault → Asana

> Statut : **confirmé** = validé lors d'un sync, à réutiliser tel quel. **à confirmer** = proposition à trancher au prochain sync.

| Chantier vault (TODO) | Projet / section Asana | Statut |
| --- | --- | --- |
| MINE - MMM AI Agent | TECH 2026 / MARKETING MIX MODELING (MMM) | **confirmé** (2026-07-09) |
| MINE - FeedGen Catégorisation | *(pas de section évidente — à trancher)* | à confirmer |
| MINE × L'OREAL - AMC Analytics | TECH 2026 / AMC ANALYTICS | **confirmé** (2026-07-08) |
| LOREAL - Passation Basma & Audiences | TECH 2026 / CREATION D'AUDIENCES LIVERAMP | à confirmer |
| Réseau/Rencontres | *(non synchronisé — perso)* | — |
| Outillage - Asana MCP | *(non synchronisé — meta)* | — |

*Cette table est enrichie au fil des syncs : chaque correspondance validée passe en « confirmé ».*
