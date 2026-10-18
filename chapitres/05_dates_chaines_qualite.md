# Chapitre 5 – Dates, chaînes de caractères et qualité des données

## Objectifs

Voici les objectifs de ce chapitre :
- [ ] convertir une colonne de texte en date avec `as.Date()`, et en extraire l'année ou le mois avec `format()`
- [ ] contrôler le format des dates avant conversion, et convertir une colonne qui mélange deux formats
- [ ] ajouter un horodatage d'insertion avec `now()`
- [ ] calculer une durée entre deux dates
- [ ] manipuler du texte avec `nchar()`, `trimws()`, `grepl()`, `sub()` et `paste()`
- [ ] repérer les doublons avec `duplicated()` et les valeurs manquantes avec `is.na()`
- [ ] repérer les valeurs extrêmes avec la règle de l'écart interquartile (`IQR()`)
- [ ] vérifier la cohérence d'une base avec des contrôles simples

- [Chapitre 5 – Dates, chaînes de caractères et qualité des données](#chapitre-5--dates-chaînes-de-caractères-et-qualité-des-données)
  - [Objectifs](#objectifs)
  - [Préparation](#préparation)
  - [Exercice 1 – Manipuler les dates](#exercice-1--manipuler-les-dates)
    - [Mémo](#mémo)
  - [Exercice 2 – Manipuler les chaînes de caractères](#exercice-2--manipuler-les-chaînes-de-caractères)
    - [Mémo](#mémo-1)
  - [Exercice 3 – Repérer les doublons](#exercice-3--repérer-les-doublons)
    - [Mémo](#mémo-2)
  - [Exercice 4 – Repérer les valeurs manquantes](#exercice-4--repérer-les-valeurs-manquantes)
    - [Mémo](#mémo-3)
  - [Exercice 5 – Traiter les valeurs extrêmes](#exercice-5--traiter-les-valeurs-extrêmes)
    - [Mémo](#mémo-4)
  - [Exercice 6 – Contrôler la cohérence](#exercice-6--contrôler-la-cohérence)
    - [Mémo](#mémo-5)
  - [Liens utiles](#liens-utiles)

## Préparation

On charge le fichier en précisant que `date_reception_dpe` est lue comme du texte : on fera la conversion en date nous-mêmes, à l'exercice 1.

```r
library(readr)
library(dplyr)
library(lubridate)   # une seule fois : install.packages("lubridate")

dpe <- read_csv(
  "data/dpe_villeurbanne_10000.csv",
  col_types = cols(date_reception_dpe = col_character())
)
```

Les exercices utilisent les colonnes `date_reception_dpe`, `description_installation_chauffage_n1` (texte libre décrivant l'installation de chauffage), `surface_habitable_logement`, `etiquette_dpe` et `conso_5_usages_ep`.

## Exercice 1 – Manipuler les dates

R stocke une date sous forme de nombre de jours depuis le 1er janvier 1970. Une colonne de texte `"2023-03-29"` doit donc être convertie avec `as.Date()`, en indiquant le format du texte. Les dates peuvent ensuite être comparées, triées et soustraites.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `as.Date()` | Convertit un texte en date. | `x`, `format` : `"%Y-%m-%d"` pour `2023-03-29` | `as.Date("2023-03-29", format = "%Y-%m-%d")` |
| `format()` | Extrait une partie de la date ou de l'heure sous forme de texte. | `x`, `format` : `"%Y"` année, `"%m"` mois en chiffres, `"%B"` nom du mois, `"%d"` jour, `"%A"` nom du jour, `"%H:%M:%S"` heure, minute, seconde | `format(date, "%B %Y")` |
| `range()` | Retourne la plus petite et la plus grande valeur. | `x` | `range(dpe$date_reception)` |
| `Sys.Date()` | Retourne la date du jour. Le résultat change chaque jour : pour un exercice reproductible, on fixe une date. | | `Sys.Date()` |
| `coalesce()` | Prend, pour chaque ligne, la première valeur non `NA` parmi plusieurs vecteurs (dplyr). | `...` : vecteurs à essayer dans l'ordre | `coalesce(as.Date(x, "%Y-%m-%d"), as.Date(x, "%d/%m/%Y"))` |
| `now()` | Retourne la date et l'heure du moment, au format `POSIXct` (package lubridate). Chaque appel donne une nouvelle valeur. | `tzone` : fuseau horaire | `now(tzone = "Europe/Paris")` |
| `Sys.time()` | Équivalent de `now()` en R de base, sans lubridate. | | `Sys.time()` |
| `n_distinct()` | Compte les valeurs différentes d'une colonne (dplyr). | `x` | `n_distinct(dpe$insert_timestamp)` |


<p align="center">
  <img src="https://michaeltoth.me/images/common_r_date_formats.png" alt="Source de l'image" width="600"/>
</p>
 
1. Ajouter à `dpe` une colonne `date_reception` de classe `Date`, obtenue à partir de `date_reception_dpe`. Vérifier la classe, puis afficher la date la plus ancienne et la plus récente.
<details>
<summary>Correction</summary>

```r
dpe <- dpe |>
  mutate(date_reception = as.Date(date_reception_dpe, format = "%Y-%m-%d"))

class(dpe$date_reception)
range(dpe$date_reception)
```
</details>

2. Ajouter les colonnes `annee_reception` et `mois_reception` (texte, avec `format()`), puis compter les DPE par année. Quelle année est la plus représentée ?
<details>
<summary>Correction</summary>

```r
dpe <- dpe |>
  mutate(
    annee_reception = format(date_reception, "%Y"),
    mois_reception = format(date_reception, "%m")
  )

dpe |> count(annee_reception)
```
</details>

3. Fixer la date de référence `date_ref` au 17 octobre 2026, puis calculer l'ancienneté de chaque DPE en jours dans `anciennete_jours`. Afficher le résumé statistique de cette colonne.
<details>
<summary>Correction</summary>

```r
date_ref <- as.Date("2026-10-17")

dpe <- dpe |>
  mutate(anciennete_jours = as.numeric(date_ref - date_reception))

summary(dpe$anciennete_jours)
# as.numeric() transforme la durée (difftime) en nombre de jours.
```
</details>

4. Garder les DPE reçus en 2025 avec une comparaison de dates, puis les compter.
<details>
<summary>Correction</summary>

```r
dpe_2025 <- dpe |>
  filter(date_reception >= as.Date("2025-01-01"), date_reception <= as.Date("2025-12-31"))

nrow(dpe_2025)
```
</details>

5. Créer trois colonnes de dates à partir de `date_reception_dpe`, chacune avec un format différent : `date_ymd` (`%Y-%m-%d`, année-mois-jour), `date_dmy` (`%d/%m/%Y`, jour-mois-année) et `date_mdy` (`%m/%d/%Y`, mois-jour-année). Compter, pour chaque colonne, les dates converties (non `NA`). Quelle colonne convient à la base ?
<details>
<summary>Correction</summary>

```r
dpe <- dpe |>
  mutate(
    date_ymd = as.Date(date_reception_dpe, format = "%Y-%m-%d"),
    date_dmy = as.Date(date_reception_dpe, format = "%d/%m/%Y"),
    date_mdy = as.Date(date_reception_dpe, format = "%m/%d/%Y")
  )

colSums(!is.na(dpe[, c("date_ymd", "date_dmy", "date_mdy")]))
# Un format qui ne correspond pas donne NA sans erreur : c'est le nombre de conversions réussies qui compte.
# Attention : 01/02/2023 se lit à la fois en dmy (1er février) et en mdy (2 janvier).
# Pour trancher, regarder les dates qui ne sont convertibles que dans un seul des deux formats.
```
</details>

6. Faire l'inverse de la conversion : écrire `date_reception` sous forme de texte avec `format()`. Créer `date_texte` au format `JJ/MM/AAAA`, `mois_annee` avec le nom complet du mois (`%B`, puis l'année `%Y`) et `jour_semaine` avec le nom du jour (`%A`). Afficher les 3 premières lignes.
<details>
<summary>Correction</summary>

```r
dpe <- dpe |>
  mutate(
    date_texte = format(date_reception, "%d/%m/%Y"),
    mois_annee = format(date_reception, "%B %Y"),
    jour_semaine = format(date_reception, "%A")
  )

dpe |>
  select(date_reception, date_texte, mois_annee, jour_semaine) |>
  head(3)
# Les noms de mois et de jours suivent la langue de R. S'ils sortent en anglais :
# Sys.setlocale("LC_TIME", "French_France")
```
</details>

7. Compter les DPE par mois. Pour trier dans l'ordre chronologique, on crée une clé `mois_cle` au format `AAAA-MM`, et non `mois_annee`. Pourquoi ?
<details>
<summary>Correction</summary>

```r
dpe |>
  count(mois_cle = format(date_reception, "%Y-%m"))
# Trié sur le texte, "avril 2025" passerait avant "février 2025" : l'ordre alphabétique
# des noms de mois ne suit pas l'ordre du calendrier. "AAAA-MM" se trie correctement.
```
</details>

8. Ajouter la colonne `insert_timestamp`, qui contient l'heure d'insertion des données dans la session (fuseau de Paris). Vérifier sa classe, afficher la première valeur au format `AAAA-MM-JJ HH:MM:SS`, puis compter les valeurs différentes.
<details>
<summary>Correction</summary>

```r
dpe <- dpe |>
  mutate(insert_timestamp = lubridate::now(tzone = "Europe/Paris"))

class(dpe$insert_timestamp)                              # "POSIXct" "POSIXt"
format(dpe$insert_timestamp[1], "%Y-%m-%d %H:%M:%S")

n_distinct(dpe$insert_timestamp)                         # [1] 1
# lubridate::now() est appelé une seule fois par mutate() : toutes les lignes reçoivent la même heure.
# Contrairement à date_ref, la valeur change à chaque exécution du code.
# Pour garder l'heure d'un import précis, on l'enregistre une fois et on ne la recalcule pas.
```
</details>

> **Note : le `::` dans `lubridate::now()`**
>
> `lubridate::now()` appelle la fonction `now()` du package lubridate sans charger le package avec `library()`. Le préfixe indique donc d'où vient la fonction. C'est utile quand deux packages définissent une fonction du même nom : `filter()` existe dans dplyr et dans le package de base `stats` (pour les séries temporelles), et `lubridate::now()` lève toute ambiguïté. Le package doit tout de même être installé avec `install.packages("lubridate")`.

## Exercice 2 – Manipuler les chaînes de caractères

La colonne `description_installation_chauffage_n1` contient un texte libre, rédigé par les diagnostiqueurs. Elle n'est pas directement exploitable : on la nettoie, puis on en tire des variables utilisables.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `nchar()` | Compte les caractères d'un texte. Une valeur `NA` donne `NA`. | `x` | `nchar("gaz")` retourne `3` |
| `trimws()` | Supprime les espaces au début et à la fin. | `x` | `trimws("  gaz ")` retourne `"gaz"` |
| `gsub()` | Remplace toutes les occurrences d'un motif. | `pattern`, `replacement`, `x` | `gsub("\\s+", " ", x)` remplace les espaces multiples par un seul |
| `sub()` | Remplace la première occurrence d'un motif. | `pattern`, `replacement`, `x` | `sub("^Chaudière ", "", x)` |
| `grepl()` | Retourne `TRUE` si le motif est présent. | `pattern`, `x`, `ignore.case` | `grepl("gaz", x, ignore.case = TRUE)` |
| `paste()` | Assemble plusieurs textes. | `...`, `sep` | `paste(type_batiment, etiquette_dpe, sep = " - ")` |

1. Ajouter la colonne `longueur_descr` (nombre de caractères de la description). Combien de descriptions sont vides ? Afficher le résumé de la longueur.
<details>
<summary>Correction</summary>

```r
dpe <- dpe |>
  mutate(longueur_descr = nchar(description_installation_chauffage_n1))

sum(is.na(dpe$description_installation_chauffage_n1))
summary(dpe$longueur_descr)
# read_csv() lit les champs vides comme NA : nchar() renvoie alors NA, et non 0.
```
</details>

2. Compter les descriptions qui commencent ou finissent par un espace, ou qui contiennent deux espaces à la suite. Puis créer `description_propre`, sans espaces parasites, avec `trimws()` et `gsub()`.
<details>
<summary>Correction</summary>

```r
descr <- dpe$description_installation_chauffage_n1

sum(grepl("^\\s|\\s$|\\s\\s", descr), na.rm = TRUE)

dpe <- dpe |>
  mutate(
    description_propre = trimws(gsub("\\s+", " ", description_installation_chauffage_n1))
  )
```
</details>

3. Ajouter la colonne `chauffage_gaz`, valant `TRUE` si la description contient le mot « gaz », sans tenir compte de la casse. Compter les `TRUE` et les `FALSE`.
<details>
<summary>Correction</summary>

```r
dpe <- dpe |>
  mutate(chauffage_gaz = grepl("gaz", description_propre, ignore.case = TRUE))

table(dpe$chauffage_gaz, useNA = "ifany")
```
</details>

4. Certaines descriptions contiennent la partie « Emetteur(s): ... ». Extraire ce qui suit cette partie dans `emetteur`, et mettre `NA` quand elle est absente. Combien de descriptions ont un émetteur renseigné ?
<details>
<summary>Correction</summary>

```r
dpe <- dpe |>
  mutate(
    emetteur = if_else(
      grepl("Emetteur", description_propre),
      sub(".*Emetteur\\(s\\):\\s*", "", description_propre),
      NA_character_
    )
  )

sum(!is.na(dpe$emetteur))
# .* avale tout ce qui précède le motif ; le motif est donc supprimé avec le texte qui le précède.
```
</details>

5. Créer `energie_chauffage` à partir de mots-clés : `gaz`, `électrique`, `réseau de chaleur` (le mot « Réseau »), `bois`, sinon `autre`. Les descriptions vides doivent rester `NA`. Compter les effectifs.
<details>
<summary>Correction</summary>

```r
dpe <- dpe |>
  mutate(
    energie_chauffage = case_when(
      is.na(description_propre) ~ NA_character_,
      grepl("gaz", description_propre, ignore.case = TRUE) ~ "gaz",
      grepl("électrique", description_propre, ignore.case = TRUE) ~ "électrique",
      grepl("Réseau", description_propre) ~ "réseau de chaleur",
      grepl("bois", description_propre, ignore.case = TRUE) ~ "bois",
      TRUE ~ "autre"
    )
  )

table(dpe$energie_chauffage, useNA = "ifany")
# L'ordre des conditions compte : la première condition vraie est retenue.
```
</details>

## Exercice 3 – Repérer les doublons

Un doublon est une ligne répétée. Dans une base de DPE, chaque numéro doit être unique : c'est la première chose à vérifier.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `duplicated()` | Retourne `TRUE` pour chaque valeur déjà rencontrée plus haut dans le vecteur. | `x` | `duplicated(dpe$numero_dpe)` |
| `distinct()` | Supprime les lignes en double (voir chapitre 3). | `...`, `.keep_all` | `distinct(dpe, numero_dpe, .keep_all = TRUE)` |
| `bind_rows()` | Empile des tableaux. | `...` | `bind_rows(dpe, dpe[1:3, ])` |

1. Compter les numéros de DPE en double dans `dpe`.
<details>
<summary>Correction</summary>

```r
sum(duplicated(dpe$numero_dpe))
# [1] 0 : aucun numéro n'est répété dans le fichier.
```
</details>

2. Pour s'entraîner, créer `dpe_test` en ajoutant à `dpe` les trois premières lignes une seconde fois. Compter les doublons de `dpe_test`, puis le nombre de lignes après `distinct()`.
<details>
<summary>Correction</summary>

```r
dpe_test <- bind_rows(dpe, dpe[1:3, ])

sum(duplicated(dpe_test$numero_dpe))                       # [1] 3
nrow(distinct(dpe_test, numero_dpe, .keep_all = TRUE))     # [1] 10000
```
</details>

## Exercice 4 – Repérer les valeurs manquantes

Une valeur manquante est notée `NA`. Il faut savoir combien il y en a, et dans quelles colonnes, avant de calculer une moyenne ou de faire un graphique.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `is.na()` | Retourne `TRUE` pour chaque valeur manquante. | `x` | `is.na(dpe$surface_habitable_logement)` |
| `colSums()` | Calcule la somme de chaque colonne d'une matrice ou d'un tableau. Combiné à `is.na()`, il compte les `NA` par colonne. | `x` | `colSums(is.na(dpe))` |

1. Compter les valeurs manquantes pour `surface_habitable_logement`, `besoin_chauffage`, `conso_5_usages_ep` et `description_installation_chauffage_n1`.
<details>
<summary>Correction</summary>

```r
colSums(is.na(dpe[, c(
  "surface_habitable_logement",
  "besoin_chauffage",
  "conso_5_usages_ep",
  "description_installation_chauffage_n1"
)]))
```
</details>

2. Pour une colonne avec des `NA`, afficher le nombre de DPE qui restent une fois les `NA` retirés. Quelle colonne a le plus de manquants ?
<details>
<summary>Correction</summary>

```r
nrow(dpe)
nrow(dpe[!is.na(dpe$description_installation_chauffage_n1), ])
# La colonne description_installation_chauffage_n1 est la plus souvent vide.
```
</details>

## Exercice 5 – Traiter les valeurs extrêmes

Au chapitre 4, une surface d'environ 3300 m² tirait la moyenne vers le haut. Pour la repérer sans décision arbitraire, on utilise la règle de l'écart interquartile : une valeur est extrême si elle dépasse le troisième quartile de plus de 1,5 fois l'écart interquartile. C'est la règle qui fixe les moustaches de la boîte à moustaches.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `IQR()` | Écart interquartile : `quantile(0.75) - quantile(0.25)`. | `x`, `na.rm` | `IQR(surface, na.rm = TRUE)` |
| `if_else()` | Version de `ifelse()` qui garde le type de la colonne (dplyr). | `condition`, `true`, `false` | `if_else(surface > 300, NA_real_, surface)` |

1. Calculer le seuil haut `seuil_haut` : troisième quartile plus 1,5 fois l'écart interquartile de `surface_habitable_logement`. Combien de DPE dépassent ce seuil ?
<details>
<summary>Correction</summary>

```r
q3 <- quantile(dpe$surface_habitable_logement, 0.75, na.rm = TRUE)
seuil_haut <- q3 + 1.5 * IQR(dpe$surface_habitable_logement, na.rm = TRUE)
seuil_haut

sum(dpe$surface_habitable_logement > seuil_haut, na.rm = TRUE)
```
</details>

2. Créer `surface_valide`, égale à la surface si elle est inférieure ou égale au seuil, et `NA` sinon. On choisit de mettre `NA` plutôt que de supprimer la ligne, pour garder les autres informations du DPE. Comparer la moyenne avant et après.
<details>
<summary>Correction</summary>

```r
dpe <- dpe |>
  mutate(surface_valide = if_else(
    surface_habitable_logement > seuil_haut,
    NA_real_,
    surface_habitable_logement
  ))

round(mean(dpe$surface_habitable_logement, na.rm = TRUE), 1)
round(mean(dpe$surface_valide, na.rm = TRUE), 1)
# La moyenne baisse : les très grandes surfaces ne pèsent plus autant.
# Ce choix doit être noté : il dépend de la règle retenue, ce n'est pas une vérité.
```
</details>

## Exercice 6 – Contrôler la cohérence

Un contrôle de cohérence vérifie qu'une règle métier est respectée. Ici : une étiquette DPE appartient à `A` à `G`, une date de réception n'est pas dans le futur, une consommation n'est pas négative, une surface est strictement positive. On compte les violations pour chaque règle en une seule fois.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `summarise()` | Calcule une ou plusieurs valeurs résumées sur tout le tableau (voir chapitre 3). | `...` : `nom = expression` | `summarise(dpe, n_invalides = sum(...))` |
| `LETTERS` | Vecteur des lettres de l'alphabet majuscule. | | `LETTERS[1:7]` retourne `"A"` à `"G"` |
| `%in%` | Retourne `TRUE` si la valeur appartient à l'ensemble. | `x`, `table` | `"B" %in% LETTERS[1:7]` |

1. Compter, pour chaque règle, le nombre de lignes qui ne la respectent pas : étiquette invalide, date de réception postérieure à `date_ref`, consommation négative, surface nulle ou négative. Les `NA` ne comptent pas comme des violations.
<details>
<summary>Correction</summary>

```r
dpe |>
  summarise(
    etiquette_invalide = sum(!etiquette_dpe %in% LETTERS[1:7]),
    date_future = sum(date_reception > date_ref, na.rm = TRUE),
    conso_negative = sum(conso_5_usages_ep < 0, na.rm = TRUE),
    surface_nulle_ou_negative = sum(surface_habitable_logement <= 0, na.rm = TRUE)
  )
# Une règle qui ne renvoie aucune violation n'est pas une preuve : elle dit seulement que
# le contrôle ne détecte rien. Il faut aussi regarder les valeurs extrêmes (exercice 5).
```
</details>

2. Un contrôle qui échoue doit être visible. Afficher les DPE qui ont une étiquette invalide, sur le tableau `dpe_test` de l'exercice 3, en utilisant `filter()`. Que renvoie-t-il ?
<details>
<summary>Correction</summary>

```r
dpe_test |>
  filter(!etiquette_dpe %in% LETTERS[1:7]) |>
  nrow()
# [1] 0 : aucune étiquette invalide. Pour tester le contrôle, on peut modifier une étiquette
# dans une copie, par exemple dpe_test$etiquette_dpe[1] <- "Z", puis relancer le filtre.
```
</details>

## Liens utiles

- [Documentation de `as.Date()` et `format()`](https://stat.ethz.ch/R-manual/R-patched/library/base/html/as.Date.html)
- [Documentation de `grepl()` et des expressions régulières](https://stat.ethz.ch/R-manual/R-patched/library/base/html/grep.html)
- [Documentation de `case_when()`](https://dplyr.tidyverse.org/reference/case_when.html)
- [Données DPE de l'ADEME](https://data.ademe.fr/datasets/dpe03existant)
