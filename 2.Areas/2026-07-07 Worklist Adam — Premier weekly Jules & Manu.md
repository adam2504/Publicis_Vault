---
type: meeting
date: 2026-07-07
---
# Worklist Adam — Premier weekly Jules & Manu (2026-07-07)

Première mise en pratique du **weekly Jules/Manu** acté au [[2026-07-03 Point positionnement & coordination Adam|point positionnement du 03/07]]. **Manu découvre le setup** (il n'était pas au point d'orga) → je resitue vite, même si Jules l'a sûrement déjà briefé.

> 🎯 **C'est du reporting, pas du tasking.** Je mène : j'annonce où j'en suis, ce qui avance, ce que je débloque. Deux blocs : (1) l'orga, (2) mes sujets.

Voir aussi : [[2026-06-22 Worklist Monday]] · [[MMM AI Agent]] · [[AMC Analytics]] · [[TODO]]

---

## 1. Orga (pour Manu surtout)

**Rappel de ce qui a été acté le 03/07** (Manu n'était pas là) :
- Cadence : **weekly Jules/Manu** + point **bi-mensuel avec Hajar** en parallèle. Règle : annuler si pas de sujet.
- Je reporte à Manu/Jules pour l'orga ; Hajar référente/coordinatrice côté DS ; Baptiste sponsor pour septembre.

**Asana — ce que j'ai déjà mis en place (montre que j'agis) :**
- J'ai parlé avec **Eddie** de sa manière d'utiliser Asana et je me suis **branché sur son fonctionnement**.
- Résultat concret : **vous pouvez suivre mon travail directement sur Asana**, et vous recevrez les **notifs dans le canal Teams**. → vous suivez l'avancement sans me relancer.

> Une phrase, pas un atelier outil. Le message = « c'est en place, vous n'avez rien à faire ».

**Où intégrer Eddie ? (point ouvert à discuter)**
- Baptiste a remonté vendredi qu'**Eddie** (référent technique ConnectedHub) devrait être **intégré quelque part dans l'orga**.
- Je ne sais pas encore **où ni comment** le placer → à cadrer avec vous.

---

## 2. Mes sujets

### 1) Assistant IA MMM
- **Testing interne (data strat) terminé.** L'assistant **fonctionne, il plaît, ses résultats sont safe** (chiffres vérifiés, sécurité inter-client OK).
- **Ça remonte côté conseil** : point aujourd'hui avec **Lara et Amaury** (suite à une proposition de Lara) → remonté jusqu'au **DG Starcom France, Fabien**, avec qui j'ai un **point demain**.
- **Prochaine phase = le faire évoluer.** Feedbacks : il manque du **contexte** —
  - contexte **métier / de cadrage**,
  - **contexte spatial du module** (connaissance des graphiques affichés),
  - + **ajouter des outils**.
- **Position produit (reporting, pour info)** : il **pourrait être vendu aux clients**, **mais** il n'est pas capable de faire des **recommandations** → je le vois plutôt **réservé aux utilisateurs internes** (où il fait déjà gagner un temps fou). *La décision se tranche avec les stakeholders (Baptiste, conseil) — ici je vous informe seulement.*

### 2) AMC Analytics
- **Dashboards de Khadija intégrés au Hub, accès granted.**
- **Onboarding avec Khadija à 15h aujourd'hui.**
- Beaucoup de **possibilités d'intégration** — notamment la **création d'audiences AMC via API directe depuis ConnectedHub** → fait le pont avec le sujet 3.

### 3) Création d'audiences LiveRamp
- **En attente** : pas de réponse au **mail LiveRamp**. → point de blocage à signaler, rien à faire de mon côté pour l'instant.

---

## Questions / points à trancher

- **Jour/heure** du weekly + articulation avec la [[2026-06-22 Worklist Monday|Worklist Monday]] 

---

## Vigilance

- **Manu = première fois** → resituer sans dérouler tout le 03/07 ; l'essentiel c'est le fonctionnement.
- **Asana** : une phrase (« c'est branché, vous suivez »), pas un atelier.
- **MMM** : la remontée jusqu'au DG est un signal fort — le dire posément, factuel, sans surjouer.
- **Concision** : 2-3 min par sujet.

---

## Compte-rendu (post-point)

> Synthèse manuelle + notes facilitator IA (à vérifier).

### Ce qui a été acté

> ⚠️ **Révisé le 07/07 (après point avec Jules)** : la vision de Manu ci-dessous était **overkill**. Scope resserré retenu → voir [[Passation Basma|Assistant Audiences]]. On attend que Basma remplisse le knowledge via la passation, puis on branche le Copilot de Manu ; module ConnectedHub plus tard.

**Nouveau chantier prioritaire — Module IA « gestion des audiences » (MVP sous 15 jours).**
Manu a posé le besoin : un **« cerveau centralisé »** des connaissances de Basma (audiences LiveRamp : nomenclatures, volumes, segments, catalogue), **non statique** — vivant, accessible à plusieurs, modifiable (esprit Google Docs). Concrètement :
- **Base = dossier Knowledge sur SharePoint** que Manu a déjà structuré (fichiers `.md` nomenclaturés). **Basma doit le compléter avant son départ.**
- **Module conversationnel MVP** dans le bac à sable d'Eddie : interroger le Knowledge, l'enrichir de nouveaux fichiers, **logguer les conversations/erreurs** pour améliorer les recos.
- **Sortie attendue** : des **recommandations d'audience** (segmentations, nomenclatures) à partir des briefs, en s'appuyant sur l'historique des audiences créées + le catalogue → un **Excel dense** (nomenclature, description, segments, fishing rules). Objectif : réduire drastiquement le temps de création manuelle d'audiences, **pallier le départ de Basma**.

**Priorités clarifiées** : les deux sujets prioritaires = **Module IA audiences + AMC**. La collab avec les data scientists sur des sujets annexes reste possible **en binôme**, mais **après** que le MVP soit fonctionnel. Suivi dans **Asana** (structurer la liste des modules, sans sur-organiser). Vigilance **dépenses GCP**.

**Eddie** : question tranchée — **on ne l'intègre pas** formellement dans l'orga. On sait où il est et comment le contacter si besoin. *(réponse de Jules)*

**Gouvernance / doc** : Manu insiste — les IA ont besoin d'une base bien documentée (exemple de son app iOS/Android où la doc dépasse le code). La gouvernance passe par **l'alimentation continue du prompt + du Knowledge**, et un **workflow GitHub** (chacun documente ses changements, prompts partagés entre les outils IA de l'équipe — Claude, Cursor…).

### Tâches de suivi (voir [[TODO]])
- [ ] **Module MVP agent audiences** : interroger/structurer le Knowledge SharePoint, logguer l'historique, intégrer les nouveaux fichiers, générer l'Excel dense (nomenclature, description, segments, fishing rules). — *Adam*
- [ ] **Alimenter le Knowledge** : demander à Basma de remplir les 5 fichiers avant son départ (structure respectée, assez détaillée). — *Manu*

### Ressenti (perso)
> « Je reste pieds et poings liés par des racines qui tirent un peu et empêchent d'avancer. »

Le legacy opérationnel L'Oréal (passation Basma, audiences) me retient alors que je vise le DS/IA. Nuance : ce module **transforme justement ce legacy en sujet IA** — potentiellement le pont vers mon positionnement (3e brique : automatiser/intégrer dans Mine), mais ça reste du périmètre Basma. À surveiller pour ne pas m'y enraciner.