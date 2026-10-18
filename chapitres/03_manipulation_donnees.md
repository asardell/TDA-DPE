# Chapitre 3 – Manipulation de dataframes

## Objectifs

Voici les objectifs de ce chapitre :
- [ ] sélectionner des lignes et des colonnes avec les crochets `[ , ]` de R de base
- [ ] filtrer, trier et transformer un dataframe avec `subset()` et `order()`
- [ ] transformer une variable qualitative en facteur
- [ ] enchaîner des opérations avec dplyr : `select()`, `filter()`, `arrange()`, `mutate()`, `slice()`
- [ ] calculer des agrégations par groupe avec `group_by()` et `summarise()`
- [ ] réorganiser et compléter des tableaux avec tidyr, et joindre deux tables avec `left_join()`

- [Chapitre 3 – Manipulation de dataframes](#chapitre-3--manipulation-de-dataframes)
  - [Objectifs](#objectifs)
  - [Introduction : R de base ou dplyr ?](#introduction--r-de-base-ou-dplyr-)
  - [Préparation](#préparation)
  - [Exercice 1 – Sélectionner avec `[ , ]`](#exercice-1--sélectionner-avec--)
    - [Mémo](#mémo)
  - [Exercice 2 – Filtrer et trier en R de base](#exercice-2--filtrer-et-trier-en-r-de-base)
    - [Mémo](#mémo-1)
  - [Exercice 3 – Facteurs](#exercice-3--facteurs)
    - [Mémo](#mémo-2)
  - [Exercice 4 – Enchaîner avec dplyr](#exercice-4--enchaîner-avec-dplyr)
    - [Mémo](#mémo-3)
  - [Exercice 5 – Créer des variables avec `mutate()`](#exercice-5--créer-des-variables-avec-mutate)
  - [Exercice 6 – Agréger par groupe](#exercice-6--agréger-par-groupe)
    - [Mémo](#mémo-4)
  - [Exercice 7 – Réorganiser et compléter avec tidyr](#exercice-7--réorganiser-et-compléter-avec-tidyr)
    - [Mémo](#mémo-5)
  - [Exercice 8 – Joindre deux tables](#exercice-8--joindre-deux-tables)
    - [Mémo](#mémo-6)
  - [Liens utiles](#liens-utiles)

## Introduction : R de base ou dplyr ?

R propose deux façons de manipuler un dataframe :
- les **fonctions de base** (`[ , ]`, `subset()`, `order()`), disponibles partout sans package ;
- le **tidyverse** (`dplyr`, `tidyr`), qui utilise des verbes lisibles et s'enchaîne avec le pipe `|>`.

Les deux approches donnent le même résultat. Le tidyverse est plus lisible dès que les opérations s'enchaînent. Les équivalents en Python sont :

| Opération | R de base | dplyr | pandas |
|-----------|-----------|-------|--------|
| Choisir des colonnes | `df[, c("a", "b")]` | `select(df, a, b)` | `df[["a", "b"]]` |
| Filtrer des lignes | `df[df$a > 5, ]` | `filter(df, a > 5)` | `df[df["a"] > 5]` |
| Trier | `df[order(df$a), ]` | `arrange(df, a)` | `df.sort_values("a")` |
| Créer une colonne | `df$c <- df$a * 2` | `mutate(df, c = a * 2)` | `df["c"] = df["a"] * 2` |
| Agréger par groupe | `aggregate()` | `group_by()` + `summarise()` | `groupby().agg()` |

<p align="center">
  <img src="https://education.rstudio.com/blog/2020/07/teaching-the-tidyverse-in-2020-part-1-getting-started/img/tidyverse-packages.png" alt="Source de l'image" width="600"/>
</p>

## Préparation

On charge les packages et on relit le fichier. Le résultat est un **tibble**, sur lequel dplyr fonctionne directement.

```r
install.packages(c("readr","dplyr","tidyr"))
library(readr)
library(dplyr)
library(tidyr)

dpe <- read_csv("data/dpe_villeurbanne_10000.csv")
```

Les exercices suivants travaillent sur `dpe`. Les colonnes utilisées sont : `numero_dpe`, `etiquette_dpe`, `type_batiment`, `surface_habitable_logement`, `type_energie_n1` et `conso_5_usages_ep`.

## Exercice 1 – Sélectionner avec `[ , ]`

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `[ , ]` | Sélectionne des lignes et/ou des colonnes. | `row_index` : lignes, `col_index` : colonnes | `dpe[1:5, c("numero_dpe", "etiquette_dpe")]` |
| `[ - , ]` | Exclut des lignes ou des colonnes. | Index négatif | `dpe[, -1]` |
| `$` | Extrait une colonne sous forme de vecteur. | | `dpe$etiquette_dpe` |

1. Afficher les 5 premières lignes des colonnes `numero_dpe` et `etiquette_dpe`.
<details>
<summary>Correction</summary>

```r
dpe[1:5, c("numero_dpe", "etiquette_dpe")]
```
</details>

2. Afficher toutes les colonnes sauf la première (`numero_dpe`), pour les 3 premières lignes.
<details>
<summary>Correction</summary>

```r
dpe[1:3, -1]
```
</details>

3. Afficher les lignes dont l'étiquette est `G`, en ne gardant que `numero_dpe` et `surface_habitable_logement`.
<details>
<summary>Correction</summary>

```r
dpe[dpe$etiquette_dpe == "G", c("numero_dpe", "surface_habitable_logement")]
```
</details>

## Exercice 2 – Filtrer et trier en R de base

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `subset()` | Retourne les lignes qui vérifient une condition et, facultativement, les colonnes choisies. | `x` : le dataframe, `subset` : la condition, `select` : les colonnes | `subset(dpe, surface_habitable_logement > 100)` |
| `order()` | Retourne les rangs de tri. | `x`, `decreasing` (FALSE par défaut) | `order(dpe$surface_habitable_logement, decreasing = TRUE)` |
| `is.na()` | Indique les valeurs manquantes. | `x` | `is.na(dpe$surface_habitable_logement)` |
| `!is.na()` | Indique les valeurs non manquantes. | `x` | `dpe[!is.na(dpe$surface_habitable_logement), ]` |
| `unique()` | Retourne les valeurs uniques. | `x` | `unique(dpe$type_batiment)` |

1. Afficher les logements de plus de 100 m² avec `subset()`, en ne gardant que `etiquette_dpe` et `surface_habitable_logement`. Combien y en a-t-il ?
<details>
<summary>Correction</summary>

```r
grands <- subset(
  dpe,
  surface_habitable_logement > 100,
  select = c(etiquette_dpe, surface_habitable_logement)
)
nrow(grands)
```
</details>

2. Trier le dataframe par surface décroissante avec `order()`, et afficher les 5 plus grandes surfaces. Que remarque-t-on sur la plus grande ?
<details>
<summary>Correction</summary>

```r
tri <- dpe[order(dpe$surface_habitable_logement, decreasing = TRUE), ]
head(tri[, c("numero_dpe", "surface_habitable_logement", "type_batiment")], n = 5)
# La valeur la plus grande est très éloignée des autres : c'est une valeur extrême,
# qu'on traitera au chapitre 5 (qualité des données).
```
</details>

3. Afficher les types de bâtiments distincts avec `unique()`, puis compter les valeurs manquantes de `surface_habitable_logement`.
<details>
<summary>Correction</summary>

```r
unique(dpe$type_batiment)
sum(is.na(dpe$surface_habitable_logement))
```
</details>

4. Garder uniquement les lignes où la surface est renseignée, dans `dpe_surface`, et vérifier qu'il ne reste aucune valeur manquante.
<details>
<summary>Correction</summary>

```r
dpe_surface <- dpe[!is.na(dpe$surface_habitable_logement), ]
sum(is.na(dpe_surface$surface_habitable_logement))
# [1] 0
```
</details>

## Exercice 3 – Facteurs

Une variable qualitative ordonnée, comme une étiquette DPE, gagne à être stockée en **facteur** : R connaît alors les modalités et leur ordre.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `as.factor()` | Convertit un vecteur en facteur, avec les modalités triées. | `x` | `as.factor(dpe$etiquette_dpe)` |
| `factor()` | Crée un facteur en imposant l'ordre des modalités. | `x`, `levels` : ordre des modalités | `factor(x, levels = c("A", "B"))` |
| `levels()` | Affiche les modalités d'un facteur. | `x` | `levels(f)` |

1. Afficher la classe de `dpe$etiquette_dpe`, puis la convertir en facteur avec `as.factor()`. Afficher ses modalités avec `levels()`.
<details>
<summary>Correction</summary>

```r
class(dpe$etiquette_dpe)
etiquette_f <- as.factor(dpe$etiquette_dpe)
levels(etiquette_f)
# [1] "A" "B" "C" "D" "E" "F" "G"
```
</details>

2. Créer `etiquette_f` avec `factor()` en imposant l'ordre `A` à `G`, puis compter les effectifs avec `table()`.
<details>
<summary>Correction</summary>

```r
etiquette_f <- factor(
  dpe$etiquette_dpe,
  levels = c("A", "B", "C", "D", "E", "F", "G")
)
table(etiquette_f)
```
</details>

3. Ajouter `etiquette_f` comme nouvelle colonne de `dpe` avec `$<-`, puis vérifier avec `class()`.
<details>
<summary>Correction</summary>

```r
dpe$etiquette_f <- etiquette_f
class(dpe$etiquette_f)  # "factor"
```
</details>

## Exercice 4 – Enchaîner avec dplyr

dplyr propose un verbe par action. Le pipe `|>` transmet le résultat d'une étape à la suivante.

> **Remarque** : on rencontre aussi le pipe `%>%`, venu du package magrittr et réexporté par dplyr. Il donne le même résultat que `|>` : `dpe %>% filter(...)` et `dpe |> filter(...)` sont équivalents. `|>` est le pipe natif de R (version 4.1 ou plus récente) et ne demande aucun package

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `select()` | Choisit des colonnes. | `...` : noms des colonnes, `-col` pour exclure | `select(dpe, numero_dpe, etiquette_dpe)` |
| `filter()` | Garde les lignes qui vérifient une condition. | `...` : conditions | `filter(dpe, etiquette_dpe == "G")` |
| `arrange()` | Trie les lignes. | `...` : colonnes ; `desc()` pour décroissant | `arrange(dpe, desc(surface_habitable_logement))` |
| `slice()` | Choisit des lignes par leur position. | `...` : positions | `slice(dpe, 1:5)` |
| `|>` | Pipe natif : transmet le résultat à l'étape suivante. | | `dpe |> filter(...) |> select(...)` |

1. Avec `select()`, garder `numero_dpe`, `etiquette_dpe`, `type_batiment` et `surface_habitable_logement`, puis `filter()` sur les maisons (`type_batiment == "maison"`).
<details>
<summary>Correction</summary>

```r
dpe |>
  select(numero_dpe, etiquette_dpe, type_batiment, surface_habitable_logement) |>
  filter(type_batiment == "maison")
```
</details>

2. Afficher les logements dont la surface est comprise entre 40 et 80 m², triés par surface décroissante, et ne garder que les 10 premiers avec `slice()`.
<details>
<summary>Correction</summary>

```r
dpe |>
  filter(surface_habitable_logement >= 40, surface_habitable_logement <= 80) |>
  arrange(desc(surface_habitable_logement)) |>
  slice(1:10)
```
</details>

3. Afficher les logements d'étiquette `F` ou `G` dont la surface est renseignée. Combien y en a-t-il ?
<details>
<summary>Correction</summary>

```r
dpe |>
  filter(etiquette_dpe %in% c("F", "G"), !is.na(surface_habitable_logement)) |>
  nrow()
```
</details>

## Exercice 5 – Créer des variables avec `mutate()`

`mutate()` ajoute une colonne calculée à partir des autres. La consommation par m² se calcule en divisant la consommation par la surface.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `mutate()` | Crée ou modifie des colonnes. | `...` : `nom = expression` | `mutate(df, double = a * 2)` |
| `ifelse()` | Renvoie une valeur ou une autre selon une condition. | `test`, `yes`, `no` | `ifelse(x > 0, "positif", "négatif")` |

1. Créer la colonne `conso_m2` égale à `conso_5_usages_ep / surface_habitable_logement`, puis afficher les 5 premières valeurs avec `numero_dpe` et `etiquette_dpe`.
<details>
<summary>Correction</summary>

```r
dpe <- dpe |>
  mutate(conso_m2 = conso_5_usages_ep / surface_habitable_logement)

dpe |>
  select(numero_dpe, etiquette_dpe, conso_m2) |>
  slice(1:5)
```
</details>

2. Créer la colonne `passoire` valant `"oui"` si l'étiquette est `E`, `F` ou `G`, et `"non"` sinon. Compter les effectifs.
<details>
<summary>Correction</summary>

```r
dpe <- dpe |>
  mutate(passoire = ifelse(etiquette_dpe %in% c("E", "F", "G"), "oui", "non"))

table(dpe$passoire)
```
</details>

## Exercice 6 – Agréger par groupe

`group_by()` définit des groupes, et `summarise()` calcule une valeur par groupe. Le résultat compte une ligne par groupe.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `group_by()` | Définit les groupes de calcul. | `...` : colonnes de groupe | `group_by(dpe, etiquette_dpe)` |
| `summarise()` | Calcule une valeur par groupe. | `...` : `nom = fonction(colonne)` | `summarise(df, moyenne = mean(a))` |
| `n()` | Compte les lignes du groupe (dans `summarise()`). | | `summarise(df, effectif = n())` |
| `count()` | Raccourci pour compter les lignes par groupe. | `...` : colonnes | `count(dpe, etiquette_dpe)` |

1. Compter le nombre de DPE par étiquette avec `count()`, puis avec `group_by()` et `summarise()`.
<details>
<summary>Correction</summary>

```r
dpe |> count(etiquette_dpe)

dpe |>
  group_by(etiquette_dpe) |>
  summarise(effectif = n())
```
</details>

2. Calculer, pour chaque type d'énergie `type_energie_n1`, le nombre de logements et la surface moyenne (en ignorant les valeurs manquantes), trié par effectif décroissant.
<details>
<summary>Correction</summary>

```r
dpe |>
  group_by(type_energie_n1) |>
  summarise(
    effectif = n(),
    surface_moyenne = mean(surface_habitable_logement, na.rm = TRUE)
  ) |>
  arrange(desc(effectif))
```
</details>

3. Calculer la part de passoires (`passoire == "oui"`) par type de bâtiment, à l'aide de `mean()` sur une condition.
<details>
<summary>Correction</summary>

```r
dpe |>
  group_by(type_batiment) |>
  summarise(part_passoires = mean(passoire == "oui"))
# mean() sur une condition donne la proportion de TRUE.
```
</details>

## Exercice 7 – Réorganiser et compléter avec tidyr

tidyr sert à mettre les tableaux dans la forme voulue : `pivot_wider()` transforme des lignes en colonnes, `drop_na()` supprime les lignes incomplètes.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `pivot_wider()` | Transforme les valeurs d'une colonne en plusieurs colonnes. | `names_from` : colonne qui donne les noms, `values_from` : colonne des valeurs, `values_fill` : valeur pour les cases vides | `pivot_wider(df, names_from = etiquette, values_from = n, values_fill = 0)` |
| `drop_na()` | Supprime les lignes qui contiennent des valeurs manquantes. | `...` : colonnes à examiner (toutes par défaut) | `drop_na(dpe, surface_habitable_logement)` |
| `replace_na()` | Remplace les valeurs manquantes. | `list` : colonne et valeur | `replace_na(df, list(a = 0))` |

1. Construire un tableau à deux entrées : pour chaque type d'énergie, le nombre de logements par étiquette. Les cases vides valent `0`.
<details>
<summary>Correction</summary>

```r
tableau_croise <- dpe |>
  count(type_energie_n1, etiquette_dpe) |>
  pivot_wider(
    names_from = etiquette_dpe,
    values_from = n,
    values_fill = 0
  )

tableau_croise
```
</details>

2. Supprimer les lignes où `surface_habitable_logement` est manquante avec `drop_na()`, et comparer le nombre de lignes avant et après.
<details>
<summary>Correction</summary>

```r
nrow(dpe)
dpe_complet <- dpe |> drop_na(surface_habitable_logement)
nrow(dpe_complet)
```
</details>

## Exercice 8 – Joindre deux tables

Une jointure rapproche deux tables à partir d'une colonne commune. Ici, on ajoute à chaque DPE une catégorie lisible, à partir d'une petite table de correspondance.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `left_join()` | Garde toutes les lignes de la table de gauche et ajoute les correspondances de droite. | `x`, `y`, `by` : colonne commune (`c("nom_gauche" = "nom_droite")` si les noms diffèrent) | `left_join(dpe, libelles, by = "etiquette_dpe")` |
| `distinct()` | Supprime les doublons d'une colonne ; `.keep_all = TRUE` garde toute la ligne. | `...` : colonnes, `.keep_all` | `distinct(adresses, id, .keep_all = TRUE)` |
| `read_delim()` | Lit un fichier dont le séparateur n'est pas la virgule, ici le point-virgule. | `file`, `delim`, `col_select` : colonnes à lire | `read_delim("adresses.csv", delim = ";", col_select = c(id, lon, lat))` |
| `tibble()` | Crée un tableau tidyverse. | `...` : colonnes | `tibble(a = 1:3, b = c("x", "y", "z"))` |

1. Créer la table `libelles` avec les colonnes `etiquette_dpe` (A à G) et `categorie`, pour les valeurs suivantes : `très performant`, `performant`, `assez performant`, `moyen`, `peu performant`, `très peu performant`, `très énergivore`.
<details>
<summary>Correction</summary>

```r
libelles <- tibble(
  etiquette_dpe = c("A", "B", "C", "D", "E", "F", "G"),
  categorie = c(
    "très performant", "performant", "assez performant", "moyen",
    "peu performant", "très peu performant", "très énergivore"
  )
)
```
</details>

2. Joindre `libelles` à `dpe` avec `left_join()` dans `dpe_libelle`, puis compter les logements par `categorie`, du plus au moins fréquent.
<details>
<summary>Correction</summary>

```r
dpe_libelle <- dpe |>
  left_join(libelles, by = "etiquette_dpe")

dpe_libelle |> count(categorie, sort = TRUE)
```
</details>

**Avant l'exercice 3 :** télécharger la Base Adresse Nationale du département 69 sur [adresse.data.gouv.fr](https://adresse.data.gouv.fr/data/ban/adresses/latest/csv-with-ids), décompresser l'archive et placer le CSV dans le dossier `data/` sous le nom `adresses-with-ids-69.csv`. Le fichier fait environ 100 Mo une fois décompressé : il n'est pas dans le dépôt.

3. Lire ce fichier (séparateur `;`) en ne gardant que les colonnes `id`, `lon` et `lat`, dans `adresses`. Compter les identifiants qui apparaissent plus d'une fois, dans `doublons`, puis afficher quelques lignes en doublon.
<details>
<summary>Correction</summary>

```r
adresses <- read_delim(
  "data/adresses-with-ids-69.csv",
  delim = ";",
  col_select = c(id, lon, lat)
)

doublons <- adresses |>
  count(id) |>
  filter(n > 1)

nrow(doublons)

adresses |>
  filter(id %in% doublons$id) |>
  arrange(id) |>
  head(4)
# Un même identifiant apparaît deux fois, avec des coordonnées qui diffèrent de quelques mètres.
```
</details>

4. Dédoublonner `adresses` avec `distinct()` dans `adresses_uniques`. Vérifier qu'il ne reste aucun identifiant en double, et combien de lignes ont été supprimées.
<details>
<summary>Correction</summary>

```r
adresses_uniques <- adresses |>
  distinct(id, .keep_all = TRUE)

nrow(adresses) - nrow(adresses_uniques)   # nombre de lignes supprimées

adresses_uniques |>
  count(id) |>
  filter(n > 1) |>
  nrow()                                  # [1] 0
# distinct() garde la première ligne rencontrée pour chaque identifiant.
# Ici les coordonnées ne diffèrent que de quelques mètres : garder la première ligne suffit.
```
</details>

5. Joindre `adresses` (sans dédoublonnage) directement à `dpe` dans `dpe_brut`, avec `identifiant_ban` côté DPE et `id` côté BAN. Comparer le nombre de lignes avant et après la jointure. Le nombre a-t-il changé ? Pourquoi ?
<details>
<summary>Correction</summary>

```r
dpe_brut <- dpe |>
  left_join(adresses, by = c("identifiant_ban" = "id"))

nrow(dpe)
nrow(dpe_brut)
# Pour les 10 000 DPE de Villeurbanne, le nombre de lignes ne change pas : aucun identifiant de DPE
# n'est en double dans la BAN. Le problème apparaît dès qu'un DPE pointe vers un identifiant en double :
# la jointure crée alors une ligne en plus pour chaque doublon.
```
</details>

6. Joindre `adresses_uniques` à `dpe` dans `dpe_gps`, puis vérifier que le nombre de lignes n'a pas changé et compter les DPE sans coordonnées.
<details>
<summary>Correction</summary>

```r
dpe_gps <- dpe |>
  left_join(adresses_uniques, by = c("identifiant_ban" = "id"))

nrow(dpe_gps) == nrow(dpe)   # TRUE : aucune ligne dupliquée
sum(is.na(dpe_gps$lon))      # DPE sans coordonnées (identifiant vide ou absent de la BAN)
```
</details>

## Liens utiles

- [Documentation dplyr](https://dplyr.tidyverse.org/)
- [Documentation tidyr](https://tidyr.tidyverse.org/)
- [Cheatsheet dplyr](https://posit.co/resources/cheatsheets/)
- [Données DPE de l'ADEME](https://data.ademe.fr/datasets/dpe03existant)
