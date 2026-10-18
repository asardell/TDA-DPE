# TDA-DPE

Ce référentiel regroupe les supports de la formation R de deux jours, sur le même modèle que le module TDA. Le fil rouge est l'analyse des DPE (diagnostics de performance énergétique) de Villeurbanne, récupérés via l'API de l'ADEME.

Les données sont filtrées sur le code postal de Villeurbanne (`code_postal_ban`) pour limiter le volume pendant la formation.

## Objectifs

- [x] Importer et exporter des données (CSV, Excel)
- [x] Manipuler des dataframes avec R de base puis dplyr / tidyr
- [x] Gérer les dates, les chaînes de caractères et la qualité des données
- [x] Calculer des statistiques descriptives
- [x] Construire des graphiques avec R de base puis ggplot2
- [x] Créer et interroger une base SQLite depuis R
- [x] Automatiser une mise à jour avec le Planificateur de tâches Windows
- [x] Versionner et publier ses travaux avec GitHub Desktop
- [x] Utiliser des structures algorithmiques, dont l'interrogation d'une API
- [x] Tester des méthodes d'analyses multidimensionnelles (ACP, clustering, régression)

## Planning

1. Chapitre 1 : Bases de R, vecteurs et dataframes
2. Chapitre 2 : Import et export de données (CSV, Excel)
3. Chapitre 3 : Manipulation de dataframes (base R puis dplyr / tidyr)
4. Chapitre 4 : Statistiques descriptives et graphiques (R de base, ggplot2, plotly)
5. Chapitre 5 : Dates, chaînes de caractères et qualité des données
6. Chapitre 6 : SQLite et SQL depuis R
7. Chapitre 7 : Algorithmique (conditions, boucles, fonctions) et interrogation de l'API ADEME (pagination)
8. Chapitre 8 : Automatisation Windows (projet de fin)
9. Chapitre 9 : Git et GitHub Desktop
10. Chapitre 10 : Analyses multidimensionnelles (ACP, clustering, régression)

Le fil rouge utilise le fichier `data/dpe_villeurbanne_10000.csv` (10 000 DPE de Villeurbanne). L'interrogation de l'API ADEME, avec la pagination par curseur (`next`) et l'écriture de chaque page dans SQLite, est traitée dans le chapitre 7.

## Fonctions abordées

| Chapitre | Fonctions et packages |
|----------|-----------------------|
| 1 | `c()`, `:`, `seq()`, `rep()`, `length()`, `class()`, `print()`, `rm()`, `getwd()`, `setwd()`, `install.packages()`, `library()`, `data()`, `data.frame()`, `head()`, `dim()`, `summary()`, `View()`, `read_csv()` (assistant RStudio) |
| 2 | `read.csv()`, `read_excel()`, `write_xlsx()`, `write.table()`, `View()`, `summary()`, `dim()`, `nrow()`, `ncol()`, `colnames()`, `$`, `rbind()` |
| 3 | `[ , ]`, `[ - , ]`, `subset()`, `order()`, `sort()`, `unique()`, `as.factor()`, `is.na()`, `select()`, `slice()`, `filter()`, `arrange()`, `group_by()`, `summarise()`, `table()`, `prop.table()`, `tidyr`, `left_join()`, `distinct()`, `read_delim()` |
| 4 | `sum()`, `mean()`, `median()`, `min()`, `max()`, `sd()`, `var()`, `quantile()`, `round()`, `cor()`, `cov()`, `hist()`, `boxplot()`, `barplot()`, `table()`, `plot()`, `abline()`, `prop.table()`, `rgb()`, `ggplot2` (`ggplot()`, `geom_*()`, `labs()`), `esquisse`, `plotly` (`plot_ly()`, `ggplotly()`), `leaflet` (`leaflet()`, `addCircleMarkers()`, `colorFactor()`) |
| 5 | `as.Date()`, `format()`, `range()`, `Sys.Date()`, `nchar()`, `trimws()`, `gsub()`, `sub()`, `grepl()`, `paste()`, `case_when()`, `duplicated()`, `is.na()`, `colSums()`, `IQR()`, `if_else()`, `%in%`, `LETTERS` |
| 6 | `DBI`, `RSQLite`, `dbplyr` |
| 7 | `httr2` (`request()`, `req_url_query()`, `req_perform()`), `jsonlite` (`fromJSON()`), `if`, `else`, `else if`, `for`, `while`, `break`, `function()`, `return()`, `missing()`, `stopifnot()`, `vector()`, `seq_along()`, `names()`, `sapply()`, `n_distinct()`, `dbAppendTable()`, `Sys.sleep()` |
| 8 | Script R, Planificateur de tâches Windows, journalisation |
| 9 | GitHub Desktop (clone, commit, push, branche, pull request) |
| 10 | `prcomp()`, `kmeans()`, `hclust()`, `dist()`, `lm()`, `predict()`, `summary()` |

## Liens utiles

- [Données DPE de l'ADEME](https://data.ademe.fr/datasets/dpe03existant)
- [Module TDA (programmation R)](https://github.com/asardell/TDA)
- [Cours sur la programmation R](https://asardell.github.io/programmation-r/)
- [Dépôt de test GitHub Desktop](https://github.com/asardell/git-cooking-workshop)
