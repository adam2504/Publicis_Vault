---
type: resource
---

Plateforme : [[Mine Platform]]

# Workflow Git & Merge

Convention d'équipe sur ConnectedHub (transmise par Eddie, 30/06/2026).

## Invariant

**`main` ne contient jamais plus de choses que `develop`.** Chacun merge sa branche dans `develop` _avant_ de la merger dans `main`. Si tout le monde respecte ça, ton merge vers `main` reste trivial et tu peux merger en toute tranquillité.

## Étapes pour une feature ou un fix

1. **Brancher depuis `main`** (pas depuis `develop`). Ça garde le diff de ta branche limité à ton seul travail, car `develop` peut contenir des commits non release d'autres équipes.
2. **Ne garder que tes propres commits.** Si tu as une branche tirée de `develop` (donc qui embarque le travail des autres), créer une **branche de transition basée sur `main`** et **cherry-picker uniquement tes commits** dessus. Ne jamais merger une branche basée sur `develop` directement dans `main`.
3. **Vérifier que le diff ne touche que tes fichiers** avant de merger (`git diff --stat origin/main <branche>`). Un merge qui modifie des fichiers hors de ton périmètre = il a embarqué du travail tiers → corriger avant (sinon risque de conflits/régressions pour les autres ; selon Eddie, « Antoine va grogner s'il y a une couille »).
4. **Merger dans `develop` d'abord**, vérifier que rien ne casse sur staging.
5. **Merger ensuite la même branche dans `main`.**
6. Une fois les deux merges faits, **supprimer les branches** de travail / transition (local + remote).

## Note back-port

Un fix qui arrive sur `main` en premier doit aussi être reporté sur `develop` (ex : cherry-pick), sinon `develop` serait en retard — ce qui casse l'invariant.

## Exemple vécu — AMC Analytics (30/06/2026)

- Branche `feature/AMC-Analytics` tirée de `develop` → diff vers `main` de 250 fichiers (176 commits tiers). Refait via branche de transition `feature/AMC-Analytics-to-main` basée sur `main` + cherry-pick des seuls commits AMC → diff propre de 32 fichiers.
- Fixes suivants (accès dashboards par client, rôle global) : branchés sur `main`, mergés sur `develop` puis `main`, puis branches supprimées.
