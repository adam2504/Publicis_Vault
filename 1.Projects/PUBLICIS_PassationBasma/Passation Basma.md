---
type: project
statut: en cours
discipline:
  - data-analyst
  - data-science
implication: lead
client: PUBLICIS
aliases:
  - Assistant Audiences
---
# Passation Basma

Départ de Basma le **23/07** (référente analyse L'Oréal, dont [[Viral Beauty]]). Double objectif, désormais un seul fil :

1. **Capturer son knowledge** (audiences LiveRamp : nomenclatures, fishing rules, segments, briefs historiques) avant qu'il parte avec elle.
2. **Brancher un agent Copilot** sur ce knowledge pour **assister la création d'audiences** — d'abord pour la remplaçante, potentiellement comme module ConnectedHub plus tard.

> ⚖️ **Scope resserré (arbitrage Jules, 07/07).** La vision initiale de Manu (module IA à KB dynamique, MVP 15j) était **overkill**. On part simple : remplir le knowledge via la passation, brancher le Copilot existant, industrialiser plus tard si ça tient. *(Fusion de l'ex-note « Assistant Audiences ».)*

---

## Plan (validé avec Jules, 07/07)

1. **Attendre le retour de Basma** → elle remplit son knowledge.
2. **Continuer les points de passation**, qui alimentent en continu le knowledge (à **nettoyer / segmenter** ensuite).
3. **Brancher le Copilot** (créé par Manu) sur ce knowledge → il assiste la création d'audiences.
4. *(Optionnel)* ajouter du **knowledge connexe** (historique des audiences, documentation LiveRamp…) pour améliorer l'aide / l'UX.
5. **Plus tard, éventuellement** : un module dans ConnectedHub — soit via **API LiveRamp directe**, soit en **connectant un agent** à ce knowledge.

---

## État réel du knowledge (dossier `BASMA K/knowledge`)

Dossier OneDrive/SharePoint synchronisé localement (`…/CLIENTS/LOREAL/BASMA K/knowledge`), 5 fichiers `.md` :
`01-nomenclature-audiences`, `02-fishing-rules-par-marque`, `03-segments-liveramp`, `04-exemples-brief-to-audience`, `05-glossaire-publicis` (+ un fichier inventaire/export).

**Constat (lecture du 07/07)** : le **schéma est excellent mais scaffoldé (IA)** — squelette là (nomenclature `LOREAL_{MARQUE}_{CIBLE}_{TYPE}_{GEO}_{DATE}`, fishing rules par marque, mapping segments, 15 exemples brief→audience), **mais les vraies données de Basma manquent** (`[À COMPLÉTER]` partout, arbre LiveRamp réel absent, catalogue synthétique).

→ **Un agent branché maintenant serait confiant et faux.** L'agent ne vaut que ce que la KB a de réel (même discipline que le MMM : chiffres vérifiés). La valeur est à ~90 % dans la **capture**, pas dans l'agent. C'est le travail irremplaçable à faire avant le 23/07.

---

## Piste outil — Docmost (knowledge dynamique + accessible IA)

[Docmost](https://docmost.com/) : wiki self-hosted, **collaboratif temps réel** (esprit Google Docs de Manu), import **Markdown** natif (nos `.md` s'y chargent directement), **serveur MCP** + recherche sémantique → un agent (Copilot de Manu, Claude, Cursor) interroge le knowledge via un standard. **Self-hosted = données L'Oréal jamais exposées** (argument fort vs Notion cloud).

**Modèle éco (à connaître) : pas de cloud managé — on héberge dans tous les cas.** Les tiers débloquent des features via licence, sur notre infra :

| Tier | Prix | Ce que ça débloque |
| --- | --- | --- |
| Community | Gratuit (AGPL) | Wiki + collaboratif. **Pas l'IA/MCP** |
| Business | **3,50 $/siège/mois**, annuel, **min 10 sièges** | SSO + **AI & MCP** + permissions fines + import Confluence |
| Enterprise | Sur devis | + SCIM, audit logs, workflow validation, support |

> ⚠️ **La feature « AI & MCP » (le pont agent↔knowledge) est en tier Business, pas gratuit.** Le scénario « SharePoint dynamique + IA » = Business obligatoire. Ordres de grandeur : 20 pers ≈ **840 $/an** de licence + **~30-60 €/mois** de VM GCP → ~1,5 k€/an tout compris.

**Où ça se place** : **PAS** dans la fenêtre Basma (avant 23/07). C'est de l'**industrialisation** (étape 4-5 du plan). La capture reste sur SharePoint, sans friction pour Basma.

> 📣 **Proposé à Jules & Manu (Teams, 08/07)** : pitch « SharePoint knowledge vivant + interrogeable par IA, self-hosted » + angle différenciateur vs Copilot/SharePoint (boucle read/write gouvernée sur MCP vs lecture RAG figée).
>
> ❌ **Abandonné (09/07).** On ne part pas sur Docmost. → retour au plan resserré : capture Basma dans SharePoint + Copilot de Manu branché dessus.

**Séquence dérisquée :**
1. Court terme : capture knowledge dans SharePoint (rien ne change).
2. **Démo pilote = Docmost en Docker local (`docker compose up`), 0 € / ~1h**, knowledge de Basma chargé, présenté en partage d'écran au weekly. Pas de VM, jetable.
3. **Si Manu/Jules kiffent** → hosting réel = décision d'**équipe** : projet GCP Publicis (budget équipe), **Eddie owner infra**, licence Business validée par Jules/Manu.

> 🛡️ **Garde-fous perso** : ne **pas auto-financer** (ni CB ni VM perso — la « Community gratuite » gratuite en licence mais la VM se paie) ; ne **pas s'enraciner dans l'exploitation d'infra** (domaine Eddie, pas le mien — cf. ressenti 07/07). Mon apport = l'idée + la démo qui convainc, pas le déploiement pérenne.

---

## Les agents Copilot

- **Deux agents distincts** (pas un seul) : celui de l'**AI Office** (Lou) et celui de **Manu**.
- L'agent de **Manu est plus avancé que le mien** → autant **converger sur celui de Manu** plutôt que dupliquer, et le brancher sur le knowledge de Basma une fois rempli.
- Objectif commun : agent alimenté par la doc/connaissances de Basma, que la remplaçante interroge directement.
- **Blocage connu** : autorisation d'ajouter des fichiers au Copilot en attente **avec Bradley**.

---

## Contexte

- Basma est référente data analyst sur les sujets L'Oréal (dont [[Viral Beauty]]).
- Réorg L'Oréal en cours autour de son départ (voir [[2026-07-07 Worklist Adam — Premier weekly Jules & Manu|weekly du 07/07]]).

---

## Personnes clés

| Rôle | Personne |
| --- | --- |
| Besoin métier / a créé le Copilot | Manu |
| Arbitrage scope | Jules |
| Source du knowledge (départ 23/07) | Basma |
| AI Office (construction agent) | Lou |
| Autorisation ajout fichiers Copilot | Bradley |
| Export / accès base LiveRamp | Lydia |
| Référent technique ConnectedHub (si module plus tard) | Eddie |

---

## Réunions

- [[2026-06-23 Point AI Office - Agent Copilot]]
- [[2026-06-30 Onboarding Audiences LiveRamp]]

---

## Prochaines étapes

- [ ] Continuer les points de passation Basma → remplir le knowledge avec le réel (nomenclatures, fishing rules, arbre segments, briefs historiques).
- [ ] Nettoyer / segmenter le knowledge une fois rempli.
- [ ] Brancher le Copilot sur le knowledge + tester l'aide à la création — *débloquer l'autorisation d'ajout de fichier avec Bradley*.
- [ ] *(Optionnel)* ajouter du knowledge connexe (historique audiences, doc LiveRamp).
- [ ] *(Plus tard)* évaluer un module ConnectedHub — API LiveRamp directe ou agent connecté au knowledge.
