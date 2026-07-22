---
type: area
---
# TODO

## MINE - MMM AI Agent

- [ ] Réaliser passation/ouverture au conseil, puis demander questions qu'ils ont/client pourrait avoir, ce qu'ils pensent des réponses, ce qui pourrait être améliorer etc
- [ ] Améliorer l'agent avec feedbacks
	- [x] canal "Media total" n'est pas un vrai canal — exclu de la discovery + noté comme agrégat (22/07)
	- [x] Durcir contre l'hallucination de tool — scoping `BUSINESS_CONTEXT` par sous-agent + durcissement (schema, discovery) + **filet retry backend** (bug non-déterministe qui migre → retry = fix uniforme). En prod (22/07)
	- [x] Aide à la lecture des graphiques — bloc `CTX_MODULE_VIZ` (bilingue EN/FR) livré (22/07)
	- [ ] Enrichir le `BUSINESS_CONTEXT` avec les DS (définitions variables/données)
	- [ ] Tools saturation + allocation (`cf-budget-allocator-prod`) + registre narratif
	- [ ] *(Backlog)* live-suivi UI (feature B) ; knowledge par client
## MINE - FeedGen Catégorisation

- [ ] Vrai jeu d'éval avec Manu (GPC vérifié à la main + titre brut si possible)
- [ ] Explorer le cross-encoder en prod (serving batch + fine-tuning)
- [ ] Se rapprocher de Dan : brancher le RAG en amont de sa solution LLM
## MINE × L'OREAL - AMC Analytics

- [ ] Finaliser étude Full Funnel avec bons graphiques, une fois que prez finale validée
- [ ] Traduire dashboards Looker en native React
- [ ] Ajouter étude Brand Store

## L'OREAL - Passation Basma & Audiences

- [ ] Continuer les points de passation Basma → remplir le knowledge avec le réel (le schéma est fait, les vraies données manquent)
- [ ] Nettoyer / segmenter le knowledge une fois rempli
- [ ] Brancher le Copilot de Manu (le plus avancé) sur le knowledge + tester l'aide à la création — *débloquer l'autorisation d'ajout de fichier avec Bradley*
- [ ] *(Optionnel)* ajouter du knowledge connexe (historique audiences, doc LiveRamp)
- [ ] *(Veille)* offres **MCP + agents LiveRamp** à venir — Julien Guého revient vers moi
## Réseau/Rencontres

- [ ] Parler avec Sidy Diallo
- [ ] Parler avec Anael Cabrol
- [ ] Parler avec Yann Legrand