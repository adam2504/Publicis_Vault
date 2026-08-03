---
type: resource
---

# Contrat d'ingestion AMC → BigQuery

Projet : [[AMC Analytics]] · Notions outil : [[AMC — Notions]]

*Comment ajouter un nouvel extract AMC pour qu'il apparaisse dans le module AMC Analytics (Full Funnel).*

Le module lit ses données **directement dans BigQuery**. Pour qu'un extract soit visible et exploitable côté produit, il faut respecter les 3 étapes ci-dessous **à chaque import** (dans la Cloud Function qui charge le CSV du bucket vers BQ).

---

## Où charger

- **Projet** : `amira-test`
- **Région** : `EU` (multi-région)
- **Dataset** : un par client → `AMC_ConnectedHub_<customerId>`
  - L'Oréal = **`AMC_ConnectedHub_7cR1jE`**
  - (Publicis aura son propre dataset le moment venu)

Deux objets par dataset : les **tables d'analyse** (une par étude × marque × période) + une table **`registry`** (le catalogue lu par le module).

---

## Étape 1 — Nommer la table

Format strict :

```
<étude>__<marque>__<période>
```

- **minuscules ASCII** (accents translittérés : `é`→`e`), pas d'espaces
- séparateur **`__`** (double underscore) **entre** les dimensions ; underscore **simple** autorisé *à l'intérieur* d'une dimension
- **période** = slug court : `2025_q4`, `2026_h1`, `2026_nov_dec`…

Exemples :

```
full_funnel__mugler__2025_q4
full_funnel__azzaro__2025_q4
full_funnel__yves_saint_laurent__2026_q1
```

> Le nom technique n'est jamais montré à l'utilisateur : le module affiche les libellés du `registry` (voir étape 3).

**Schéma** : identique à `tb_media_mix_path_to_conversion` (le tien). Charger la table **non partitionnée**.

---

## Étape 2 — Transformer le CSV avant le `load` ⚠️

Les exports AMC ne se chargent **pas** tels quels dans BQ. Il faut convertir, **colonne par colonne** :

| Problème dans le CSV | Correction |
| --- | --- |
| Séparateur `;` | `--field_delimiter=';'` au load |
| Décimales à la **virgule** (`0,4948`) | virgule → point, **uniquement sur les colonnes numériques** |
| Dates en **`JJ/MM/AAAA`** (`23/04/2026`) | → ISO `AAAA-MM-JJ` sur les colonnes DATE |
| Colonne `path` (et `description`) contient des **virgules** | **ne pas y toucher** — ce sont des STRING |

> 🔑 Piège principal : ne **jamais** faire un « remplacer toutes les virgules par des points » global → ça casserait `path` (ex. `[1/PRIME, 2/OFF-SITE]`). La conversion doit cibler **seulement** les colonnes FLOAT/DATE d'après le schéma.

Exemple de transfo (Python, à adapter dans la CF) — pilotée par le schéma cible :

```python
import csv, datetime

# types issus du schéma tb_media_mix_path_to_conversion : {nom_colonne: "DATE"|"FLOAT"|"INTEGER"|"STRING"}
def clean_value(name, value, types):
    v = (value or "").strip()
    if v == "":
        return ""
    t = types[name]
    if t == "DATE":
        return datetime.datetime.strptime(v, "%d/%m/%Y").strftime("%Y-%m-%d")
    if t == "FLOAT":
        return v.replace(" ", "").replace(",", ".")
    return v  # INTEGER / STRING : inchangé (path préservé)

def transform(src_path, out_path, types):
    with open(src_path, encoding="utf-8-sig", newline="") as f_in, \
         open(out_path, "w", encoding="utf-8", newline="") as f_out:
        r = csv.reader(f_in, delimiter=";")
        w = csv.writer(f_out, delimiter=";")
        header = next(r)
        w.writerow(header)
        for row in r:
            w.writerow([clean_value(h, val, types) for h, val in zip(header, row)])
```

Puis `bq load --source_format=CSV --field_delimiter=';' --skip_leading_rows=1 --replace <dataset>.<table> <csv_propre> <schema.json>`.

---

## Étape 3 — Mettre à jour le `registry`

Après le load, la CF exécute ce `MERGE` (idempotent : ré-import = mise à jour de la ligne). À remplir avec les métadonnées de l'extract :

```sql
MERGE `amira-test.AMC_ConnectedHub_7cR1jE.registry` T
USING (
  SELECT
    'full_funnel__mugler__2025_q4'        AS table_id,
    'full_funnel'  AS study,       'Full Funnel' AS study_label,
    'mugler'       AS brand,       'Mugler'      AS brand_label,
    '2025_q4'      AS period_slug, 'Q4 2025'     AS period_label,
    DATE '2025-11-10' AS period_start,   -- début réel de l'étude
    DATE '2025-12-31' AS period_end,     -- fin réelle
    DATE '2026-04-23' AS extract_date,   -- date de l'export AMC
    (SELECT COUNT(*) FROM `amira-test.AMC_ConnectedHub_7cR1jE.full_funnel__mugler__2025_q4`) AS row_count
) S
ON T.table_id = S.table_id
WHEN MATCHED THEN UPDATE SET
  study=S.study, study_label=S.study_label, brand=S.brand, brand_label=S.brand_label,
  period_slug=S.period_slug, period_label=S.period_label,
  period_start=S.period_start, period_end=S.period_end,
  extract_date=S.extract_date, row_count=S.row_count,
  created_at=CURRENT_TIMESTAMP(), is_active=TRUE
WHEN NOT MATCHED THEN INSERT
  (table_id, study, study_label, brand, brand_label, period_slug, period_label,
   period_start, period_end, extract_date, row_count, created_at, is_active)
VALUES
  (S.table_id, S.study, S.study_label, S.brand, S.brand_label, S.period_slug, S.period_label,
   S.period_start, S.period_end, S.extract_date, S.row_count, CURRENT_TIMESTAMP(), TRUE);
```

Colonnes du `registry` (rappel) :

| colonne | ex. | rôle |
| --- | --- | --- |
| `table_id` | `full_funnel__mugler__2025_q4` | nom BQ exact de la table (clé) |
| `study` / `study_label` | `full_funnel` / « Full Funnel » | slug + libellé affiché |
| `brand` / `brand_label` | `mugler` / « Mugler » | slug + libellé affiché |
| `period_slug` / `period_label` | `2025_q4` / « Q4 2025 » | slug + libellé affiché |
| `period_start` / `period_end` | dates | période réelle de l'étude |
| `extract_date` | date | date de l'export AMC |
| `row_count` | nombre | contrôle d'intégrité |
| `is_active` | `TRUE` | mettre `FALSE` pour masquer sans supprimer |

---

## Checklist par extract

1. [ ] CSV transformé (décimales, dates ; `path` intact)
2. [ ] Table chargée en `<étude>__<marque>__<période>`, non partitionnée, dans `AMC_ConnectedHub_<customerId>`
3. [ ] `MERGE` du `registry` exécuté avec les bonnes métadonnées

→ L'étude apparaît automatiquement dans le module (picker « Data Available »).
