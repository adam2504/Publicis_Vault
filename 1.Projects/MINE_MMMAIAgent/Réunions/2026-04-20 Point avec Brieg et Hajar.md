---
type: meeting
date: 2026-04-20
---
Projet : [[MMM AI Agent]]

# 20 Avril — Point avec Brieg + Hajar

## Infos

- Testing en cours par Hajar + Brieg → feedbacks à venir
- 1 GCF d'optimisation : `cf-budget-allocator-prod`
- Courbes de saturation et autres fonctions directement dans le code de Mine
- Feature décidée : **bonus payant** (tab cachée si non souscrite)

## Next Steps

1. ~~Corriger erreur UI (réponse OK dans Vertex mais erreur côté interface)~~
2. Feature bonus :
   - ~~Ajouter dans Firebase~~
   - ~~Créer preview si non payée~~
   - ~~Toggle dans Admin Panel~~
   - ~~Vérifier sécurité du lock (écran chatbot apparaissait 1sec même sans accès)~~
3. ~~Ajouter photo de profil utilisateur dans le chat~~
4. ~~Ajouter contexte de discussion (mémoire dans le même chat)~~
5. Guardrails pour éviter double injection `client_id`
6. Ajouter GCF as tools (cf-budget-allocator-prod + fonctions built-in Mine)

## Idées en suspens

- Stockage messages dans bucket GCS (avec bucket par client ?)
- Mémoire constante par utilisateur
- Idée Eddie : chatbot flottant dans le coin du module (reste visible en changeant de tab)
