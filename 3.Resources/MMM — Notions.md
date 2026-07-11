# MMM — Notions

Notions de référence sur le Marketing Mix Modeling, réutilisables sur tous les sujets MMM (modèles clients, [[MMM AI Agent]], propales).

---

## Types de modèles : additif vs multiplicatif

Un MMM relie une variable à expliquer (ventes, trafic, CA on/offline) à des drivers : investissements média, baseline, facteurs endogènes (promos, events) et exogènes (saisonnalité, vacances). La **forme** du modèle change la façon dont les effets se combinent et s'interprètent.

### Modèle additif

Les ventes sont la **somme** des contributions :

```
Ventes = baseline + contrib_média_1 + contrib_média_2 + … + autres facteurs
```

- Chaque levier apporte un **incrément indépendant et fixe** ; les effets ne s'influencent pas entre eux.
- Coefficients = **unités absolues** (ex : X ventes générées par GRP, par € investi).
- **Décomposition directe** : chaque terme du modèle *est* la contribution → contributions et ROI/ROAS très lisibles.
- Estimé par régression linéaire sur les niveaux.
- **Limites** : suppose l'indépendance des leviers (pas de synergies sauf à les ajouter explicitement), peut prédire des valeurs négatives, ne capture pas qu'un facteur en amplifie un autre (ex : média plus efficace en haute saison).

### Modèle multiplicatif

Les ventes sont un **produit** de facteurs :

```
Ventes = baseline × f(média_1) × f(média_2) × …
```

- Souvent estimé en **log** (log-log) : `log(ventes) = log(baseline) + Σ β·log(média)` → les coefficients deviennent des **élasticités** (% de variation des ventes pour 1 % de variation du driver).
- Les effets **interagissent** : l'impact d'un levier dépend du niveau des autres → capture naturellement les **synergies** et les **rendements décroissants** (saturation).
- Empêche les prédictions négatives ; effets proportionnels au niveau d'activité.
- **Limites** : décomposition des contributions moins directe (calcul par décomposition), interprétation en élasticités moins intuitive pour un non-initié, exige des valeurs positives.

### En pratique

| | Additif | Multiplicatif |
| --- | --- | --- |
| Combinaison des effets | Somme | Produit |
| Coefficients | Incréments absolus | Élasticités (%) |
| Contributions | Lecture directe | Par décomposition |
| Synergies / interactions | Non (sauf ajout explicite) | Oui, nativement |
| Rendements décroissants / saturation | Mal captés | Mieux captés |
| Risque de prédiction négative | Oui | Non |

Beaucoup de modèles réels sont **hybrides** (semi-log) ou combinent les deux logiques selon les leviers.

### Lien avec l'agent MMM

Le type de modèle conditionne **comment interpréter** ce que l'agent renvoie :
- en additif, les contributions sont directement lisibles ; en multiplicatif, l'agent manipule des élasticités et des décompositions.
- la question des **synergies** (remontée comme limite sur Stellantis lors du testing du 30/06) dépend de la forme : un modèle additif sans terme d'interaction ne *contient* pas de synergies. Voir [[MMM AI Agent]].
