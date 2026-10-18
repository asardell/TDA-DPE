# Chapitre 6 – SQLite et SQL depuis R

## Objectifs

Voici les objectifs de ce chapitre :
- [ ] créer une base SQLite et y écrire un dataframe avec `dbWriteTable()`
- [ ] lister les tables et leurs colonnes, puis lire une table avec `dbReadTable()`
- [ ] interroger la base avec du SQL (`SELECT`, `WHERE`, `GROUP BY`, `ORDER BY`) via `dbGetQuery()`
- [ ] passer des valeurs à une requête avec des paramètres
- [ ] créer une table à partir d'une requête avec `dbExecute()`
- [ ] écrire la même requête avec dplyr grâce à dbplyr, et voir le SQL généré avec `show_query()`

- [Chapitre 6 – SQLite et SQL depuis R](#chapitre-6--sqlite-et-sql-depuis-r)
  - [Objectifs](#objectifs)
  - [Préparation](#préparation)
  - [Exercice 1 – Créer une base SQLite](#exercice-1--créer-une-base-sqlite)
    - [Mémo](#mémo)
  - [Exercice 2 – Lire des lignes avec `SELECT`](#exercice-2--lire-des-lignes-avec-select)
    - [Mémo](#mémo-1)
  - [Exercice 3 – Agréger avec `GROUP BY`](#exercice-3--agréger-avec-group-by)
    - [Mémo](#mémo-2)
  - [Exercice 4 – Passer des paramètres à une requête](#exercice-4--passer-des-paramètres-à-une-requête)
    - [Mémo](#mémo-3)
  - [Exercice 5 – Créer une table avec `CREATE TABLE`](#exercice-5--créer-une-table-avec-create-table)
    - [Mémo](#mémo-4)
  - [Exercice 6 – Écrire la requête avec dbplyr](#exercice-6--écrire-la-requête-avec-dbplyr)
    - [Mémo](#mémo-5)
  - [Liens utiles](#liens-utiles)

## Préparation

Une base SQLite est un simple fichier, sans serveur à installer. R s'y connecte avec le package **DBI** (l'interface commune aux bases de données) et le pilote **RSQLite**. Le package **dbplyr** traduit les verbes de dplyr en SQL. Ces trois packages s'installent une seule fois :

```r
# install.packages(c("DBI", "RSQLite", "dbplyr"))  # une seule fois
```

On charge les données du fil rouge et on ouvre la connexion. `dbConnect()` crée le fichier s'il n'existe pas encore. Le fichier se trouve dans `data/`, qui n'est pas versionné sur GitHub.

```r
library(readr)
library(dplyr)
library(DBI)
library(RSQLite)

dpe <- read_csv("data/dpe_villeurbanne_10000.csv")

# SQLite ne connaît pas les dates de R : on les stocke au format texte AAAA-MM-JJ,
# qui se trie et se compare correctement en SQL.
dpe_sql <- dpe |>
  mutate(date_reception_dpe = as.character(date_reception_dpe))

con <- dbConnect(RSQLite::SQLite(), "data/dpe_villeurbanne.sqlite")
```

## Exercice 1 – Créer une base SQLite

Une base contient des **tables**. Une table a un nom, des colonnes et des lignes : c'est l'équivalent d'un dataframe. `dbWriteTable()` écrit un dataframe dans la base comme une table.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `dbConnect()` | Ouvre une connexion à la base. | `drv` : le pilote, `dbname` : chemin du fichier | `dbConnect(RSQLite::SQLite(), "data/base.sqlite")` |
| `dbWriteTable()` | Écrit un dataframe dans une table. | `conn`, `name` : nom de la table, `value` : le dataframe, `overwrite` : remplace la table si elle existe | `dbWriteTable(con, "dpe", dpe_sql, overwrite = TRUE)` |
| `dbListTables()` | Liste les tables de la base. | `conn` | `dbListTables(con)` |
| `dbListFields()` | Liste les colonnes d'une table. | `conn`, `name` | `dbListFields(con, "dpe")` |

1. Écrire `dpe_sql` dans la table `dpe` avec `overwrite = TRUE`, puis vérifier avec `dbListTables()` que la table existe.
<details>
<summary>Correction</summary>

```r
dbWriteTable(con, "dpe", dpe_sql, overwrite = TRUE)
dbListTables(con)
```
</details>

2. Lister les colonnes de la table `dpe`. Combien y en a-t-il ?
<details>
<summary>Correction</summary>

```r
champs <- dbListFields(con, "dpe")
champs
length(champs)
```
</details>

> **Note : consulter la base avec DB Browser for SQLite**
>
> Une fois le fichier `data/dpe_villeurbanne.sqlite` créé, on peut le consulter avec **DB Browser for SQLite**, [un logiciel gratuit](https://sqlitebrowser.org/dl/) : « Ouvrir une base de données », puis choisir le fichier. L'onglet « Parcourir les données » affiche les lignes de la table `dpe`, et l'onglet « Exécuter le SQL » permet de tester les requêtes de ce chapitre.
>
> DB Browser n'affiche pas les changements en direct. Après une écriture depuis R (`dbWriteTable()`, `dbExecute()`), il faut rouvrir le fichier ou actualiser la vue. Évite aussi de modifier les données dans DB Browser pendant que R travaille sur la même base. Dans RStudio, l'onglet **Connections** affiche les tables de la connexion ouverte avec `dbConnect()`.

## Exercice 2 – Lire des lignes avec `SELECT`

`dbGetQuery()` envoie une requête SQL à la base et renvoie le résultat sous forme de dataframe. La requête la plus courante est `SELECT` : elle choisit des colonnes (`SELECT`), des lignes (`WHERE`), puis trie (`ORDER BY`) et limite le nombre de lignes (`LIMIT`).

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `dbGetQuery()` | Exécute une requête `SELECT` et renvoie un dataframe. | `conn`, `statement` : la requête SQL, `params` : valeurs (voir exercice 4) | `dbGetQuery(con, "SELECT * FROM dpe LIMIT 5")` |
| `SELECT` | Choisit les colonnes à afficher. `*` désigne toutes les colonnes. | colonnes séparées par des virgules | `SELECT etiquette_dpe, surface_habitable_logement` |
| `WHERE` | Garde les lignes qui respectent une condition. | `=`, `<>`, `>`, `<`, `IN (...)`, `AND`, `OR` | `WHERE surface_habitable_logement > 100` |
| `ORDER BY` | Trie les lignes. `DESC` : ordre décroissant. | colonne, `ASC` / `DESC` | `ORDER BY surface_habitable_logement DESC` |
| `LIMIT` | Ne garde que les premières lignes. | nombre de lignes | `LIMIT 10` |

1. Afficher les 5 premières lignes de `identifiant_ban`, `etiquette_dpe` et `surface_habitable_logement`.
<details>
<summary>Correction</summary>

```r
dbGetQuery(
  con,
  "SELECT identifiant_ban, etiquette_dpe, surface_habitable_logement
   FROM dpe
   LIMIT 5"
)
```
</details>

2. Afficher les 10 logements de plus de 100 m² ayant la plus grande surface, avec leur étiquette et leur surface.
<details>
<summary>Correction</summary>

```r
dbGetQuery(
  con,
  "SELECT etiquette_dpe, surface_habitable_logement
   FROM dpe
   WHERE surface_habitable_logement > 100
   ORDER BY surface_habitable_logement DESC
   LIMIT 10"
)
```
</details>

## Exercice 3 – Agréger avec `GROUP BY`

`GROUP BY` regroupe les lignes qui partagent une même valeur, puis calcule un indicateur par groupe. Les fonctions d'agrégation sont `COUNT()`, `AVG()`, `SUM()`, `MIN()` et `MAX()`. C'est l'équivalent de `group_by()` et `summarise()` de dplyr.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `COUNT()` | Compte les lignes. `COUNT(*)` compte toutes les lignes. | colonne ou `*` | `COUNT(*) AS nb` |
| `AVG()` | Moyenne, en ignorant les valeurs manquantes. | colonne | `AVG(conso_5_usages_ep)` |
| `ROUND()` | Arrondit à un nombre de décimales. | valeur, `décimales` | `ROUND(AVG(conso_5_usages_ep), 0)` |
| `GROUP BY` | Regroupe les lignes par valeur d'une ou plusieurs colonnes. | colonnes | `GROUP BY etiquette_dpe` |
| `HAVING` | Filtre les groupes après l'agrégation (`WHERE` filtre avant). | condition sur un agrégat | `HAVING COUNT(*) >= 500` |
| `AS` | Donne un nom à une colonne calculée. | nom | `COUNT(*) AS nb` |

1. Pour chaque étiquette DPE, compter les logements (`nb`) et calculer la consommation moyenne arrondie à l'unité (`conso_moy`). Trier les étiquettes de `A` à `G`.
<details>
<summary>Correction</summary>

```r
dbGetQuery(
  con,
  "SELECT etiquette_dpe,
          COUNT(*) AS nb,
          ROUND(AVG(conso_5_usages_ep), 0) AS conso_moy
   FROM dpe
   GROUP BY etiquette_dpe
   ORDER BY etiquette_dpe"
)
```
</details>

2. Garder uniquement les étiquettes qui comptent au moins 500 logements, avec `HAVING`. Pourquoi ne peut-on pas écrire ce filtre dans `WHERE` ?
<details>
<summary>Correction</summary>

```r
dbGetQuery(
  con,
  "SELECT etiquette_dpe, COUNT(*) AS nb
   FROM dpe
   GROUP BY etiquette_dpe
   HAVING COUNT(*) >= 500
   ORDER BY nb DESC"
)
# WHERE s'applique aux lignes avant le regroupement, donc avant que COUNT(*) existe.
# HAVING s'applique aux groupes, une fois le COUNT(*) calculé.
```
</details>

## Exercice 4 – Passer des paramètres à une requête

Quand la valeur à chercher vient de R (une variable, une boucle), on ne colle pas du texte dans la requête. On écrit un `?` à la place de la valeur, et on donne la valeur dans `params`. C'est plus sûr et évite les erreurs de guillemets.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `?` | Marque l'emplacement d'une valeur dans la requête. | une valeur par `?`, dans l'ordre | `WHERE etiquette_dpe = ?` |
| `params` | Liste les valeurs à remplacer les `?`. | liste R, dans l'ordre des `?` | `params = list("G")` |

1. Compter les logements de l'étiquette `G` avec un paramètre, en écrivant la valeur `"G"` dans `params`.
<details>
<summary>Correction</summary>

```r
dbGetQuery(
  con,
  "SELECT COUNT(*) AS nb FROM dpe WHERE etiquette_dpe = ?",
  params = list("G")
)
```
</details>

2. Calculer le nombre de logements pour chaque étiquette de `A` à `G` avec une seule requête répétée. Utiliser `sapply()` sur `LETTERS[1:7]`.
<details>
<summary>Correction</summary>

```r
effectifs_db <- sapply(LETTERS[1:7], function(lettre) {
  dbGetQuery(
    con,
    "SELECT COUNT(*) AS nb FROM dpe WHERE etiquette_dpe = ?",
    params = list(lettre)
  )$nb
})
effectifs_db
```
</details>

## Exercice 5 – Créer une table avec `CREATE TABLE`

`dbExecute()` exécute une requête qui modifie la base (`CREATE`, `INSERT`, `UPDATE`, `DELETE`) sans renvoyer de résultat. Il renvoie seulement le nombre de lignes concernées. Ici, on crée une table `passoires` à partir d'une requête `SELECT`, sans passer les données dans R.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `dbExecute()` | Exécute une requête qui modifie la base. | `conn`, `statement` | `dbExecute(con, "DROP TABLE IF EXISTS passoires")` |
| `CREATE TABLE ... AS SELECT` | Crée une table avec le résultat d'une requête. | nom de la table, requête `SELECT` | `CREATE TABLE passoires AS SELECT * FROM dpe WHERE ...` |
| `DROP TABLE IF EXISTS` | Supprime une table si elle existe. Évite l'erreur quand on relance le code. | nom de la table | `DROP TABLE IF EXISTS passoires` |
| `IN (...)` | Teste si une valeur fait partie d'une liste. | liste de valeurs | `WHERE etiquette_dpe IN ('E', 'F', 'G')` |

1. Créer la table `passoires` avec les logements étiquetés `E`, `F` ou `G`. Supprimer d'abord la table si elle existe déjà, puis vérifier avec `dbListTables()`.
<details>
<summary>Correction</summary>

```r
dbExecute(con, "DROP TABLE IF EXISTS passoires")
dbExecute(
  con,
  "CREATE TABLE passoires AS
   SELECT * FROM dpe
   WHERE etiquette_dpe IN ('E', 'F', 'G')"
)
dbListTables(con)
```
</details>

2. Compter les lignes de `passoires` en SQL, puis vérifier le résultat avec R : `sum(dpe$etiquette_dpe %in% c("E", "F", "G"))`. Les deux nombres doivent être identiques.
<details>
<summary>Correction</summary>

```r
dbGetQuery(con, "SELECT COUNT(*) AS nb FROM passoires")
sum(dpe$etiquette_dpe %in% c("E", "F", "G"))
```
</details>

## Exercice 6 – Écrire la requête avec dbplyr

**dbplyr** permet d'écrire la requête avec les verbes de dplyr, comme sur un dataframe. `tbl()` ne télécharge rien : il crée une référence vers la table. Les étapes s'enchaînent en SQL, et les données ne reviennent dans R qu'à la fin, avec `collect()`. `show_query()` affiche le SQL généré.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `tbl()` | Crée une référence vers une table de la base. | `src` : la connexion, `from` : nom de la table | `tbl(con, "dpe")` |
| `filter()`, `group_by()`, `summarise()`, `arrange()` | Verbes dplyr traduits en SQL. | comme dans dplyr | `filter(etiquette_dpe %in% c("F", "G"))` |
| `collect()` | Télécharge le résultat dans R sous forme de dataframe. | aucun | `collect()` |
| `show_query()` | Affiche le SQL généré par dbplyr. | aucun | `show_query()` |

1. Avec `tbl(con, "dpe")`, garder les étiquettes `F` et `G`, puis compter les logements et calculer la consommation moyenne par `type_energie_n1`. Trier du plus au moins nombreux, et terminer par `collect()`.
<details>
<summary>Correction</summary>

```r
dpe_db <- tbl(con, "dpe")

dpe_db |>
  filter(etiquette_dpe %in% c("F", "G")) |>
  group_by(type_energie_n1) |>
  summarise(
    nb = n(),
    conso_moy = mean(conso_5_usages_ep, na.rm = TRUE)
  ) |>
  arrange(desc(nb)) |>
  collect()
```
</details>

2. Afficher le SQL généré par la requête de la question 1. Quel mot SQL correspond à `group_by()` ?
<details>
<summary>Correction</summary>

```r
dpe_db |>
  filter(etiquette_dpe %in% c("F", "G")) |>
  group_by(type_energie_n1) |>
  summarise(nb = n()) |>
  show_query()
# Le mot SQL correspondant à group_by() est GROUP BY.
```
</details>

3. Refaire le calcul du nombre de logements `F` et `G` sur le dataframe `dpe` avec dplyr, et comparer le résultat à la requête SQL. Puis fermer la connexion avec `dbDisconnect()`.
<details>
<summary>Correction</summary>

```r
dpe |>
  filter(etiquette_dpe %in% c("F", "G")) |>
  count(type_energie_n1)

dpe_db |>
  filter(etiquette_dpe %in% c("F", "G")) |>
  count(type_energie_n1) |>
  collect()

dbDisconnect(con)
```
</details>

> **Note : connexion avec identifiant et mot de passe**
>
> SQLite n'a pas de comptes utilisateurs. Le fichier est protégé par les droits du dossier où il se trouve, et RSQLite ne le chiffre pas : un mot de passe n'ajouterait donc pas de sécurité ici. Un identifiant et un mot de passe ont un sens avec une base serveur, comme PostgreSQL ou MariaDB. Exemple indicatif, non exécuté dans ce chapitre (package `RPostgres`) :
>
> ```r
> con <- DBI::dbConnect(
>   RPostgres::Postgres(),
>   host = "localhost",
>   dbname = "dpe",
>   user = "anthony",
>   password = keyring::key_get("dpe_postgres", "anthony")
> )
> ```
>
> Le mot de passe ne doit pas être écrit dans le script. `keyring::key_get()` le lit dans le gestionnaire d'identifiants de Windows, et `rstudioapi::askForPassword()` le demande à l'ouverture de la session.

## Liens utiles

- [DB Browser for SQLite](https://sqlitebrowser.org/)
- [Documentation de DBI](https://dbi.r-dbi.org/)
- [Documentation de RSQLite](https://rsqlite.r-dbi.org/)
- [Documentation de dbplyr](https://dbplyr.tidyverse.org/)
- [Référence du langage SQL de SQLite](https://www.sqlite.org/lang.html)
- [Données DPE de l'ADEME](https://data.ademe.fr/datasets/dpe03existant)
