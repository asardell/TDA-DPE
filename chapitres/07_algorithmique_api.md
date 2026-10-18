# Chapitre 7 – Algorithmique et interrogation de l'API ADEME

## Objectifs

Voici les objectifs de ce chapitre :
- [ ] tester une condition avec `if`, `else if` et `else`
- [ ] répéter une action avec `for`, et arrêter une boucle avec `break`
- [ ] créer une fonction avec des paramètres obligatoires et des paramètres optionnels
- [ ] appliquer une même règle à toutes les colonnes d'un tableau avec une boucle
- [ ] interroger une page de l'API ADEME avec `httr2` et lire sa réponse avec `jsonlite`
- [ ] parcourir toutes les pages de l'API avec `while` et écrire chaque page dans la base SQLite

- [Chapitre 7 – Algorithmique et interrogation de l'API ADEME](#chapitre-7--algorithmique-et-interrogation-de-lapi-ademe)
  - [Objectifs](#objectifs)
  - [Préparation](#préparation)
  - [Exercice 1 – Tester une condition avec `if` et `else`](#exercice-1--tester-une-condition-avec-if-et-else)
    - [Mémo](#mémo)
  - [Exercice 2 – Répéter avec `for`](#exercice-2--répéter-avec-for)
    - [Mémo](#mémo-1)
  - [Exercice 3 – Créer une fonction](#exercice-3--créer-une-fonction)
    - [Mémo](#mémo-2)
  - [Exercice 4 – Graphiques pour toutes les colonnes](#exercice-4--graphiques-pour-toutes-les-colonnes)
    - [Mémo](#mémo-3)
  - [Exercice 5 – Interroger une page de l'API](#exercice-5--interroger-une-page-de-lapi)
    - [Mémo](#mémo-4)
  - [Exercice 6 – Télécharger toutes les pages dans SQLite avec `while`](#exercice-6--télécharger-toutes-les-pages-dans-sqlite-avec-while)
    - [Mémo](#mémo-5)
  - [Liens utiles](#liens-utiles)

## Préparation

Ce chapitre vient avant l'automatisation : les boucles et les fonctions servent ensuite à écrire le script qui met à jour la base. On utilise les packages `httr2` (requêtes HTTP) et `jsonlite` (lecture du JSON), à installer une seule fois.

```r
# install.packages(c("httr2", "jsonlite"))  # une seule fois
library(readr)
library(dplyr)
library(DBI)
library(RSQLite)
library(httr2)
library(jsonlite)

dpe <- read_csv("data/dpe_villeurbanne_10000.csv")
```

## Exercice 1 – Tester une condition avec `if` et `else`

Une condition permet de faire un choix. `if` exécute un bloc de code si la condition est vraie, `else` exécute l'autre bloc sinon, et `else if` ajoute un cas intermédiaire. La condition doit être **une seule valeur** `TRUE` ou `FALSE`.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `if` | Exécute un bloc si la condition est vraie. | condition entre parenthèses | `if (x > 0) print("positif")` |
| `else` | Exécute un bloc si la condition est fausse. | | `else print("négatif ou nul")` |
| `else if` | Ajoute un cas quand la première condition est fausse. | condition | `else if (x == 0) print("nul")` |
| `%in%` | Teste si une valeur appartient à un ensemble. | `x`, `table` | `"F" %in% c("E", "F", "G")` |

1. Pour le premier DPE du fichier, afficher « passoire énergétique » si son étiquette est `E`, `F` ou `G`, et « pas une passoire » sinon.
<details>
<summary>Correction</summary>

```r
etiquette <- dpe$etiquette_dpe[1]

if (etiquette %in% c("E", "F", "G")) {
  print("passoire énergétique")
} else {
  print("pas une passoire")
}
```
</details>

2. Avec `else if`, classer une étiquette dans trois catégories : `A` ou `B` pour « performant », `C` ou `D` pour « moyen », `E` à `G` pour « passoire ». Tester avec `etiquette <- "D"`, puis avec `etiquette <- "G"`.
<details>
<summary>Correction</summary>

```r
etiquette <- "D"

if (etiquette %in% c("A", "B")) {
  categorie <- "performant"
} else if (etiquette %in% c("C", "D")) {
  categorie <- "moyen"
} else {
  categorie <- "passoire"
}

categorie
```
</details>

3. Essayer la même condition sur toute la colonne : `if (dpe$etiquette_dpe %in% c("E", "F", "G")) print("passoire")`. Quelle erreur obtient-on ? Quelle fonction de dplyr (chapitre 5) permet de faire ce test sur chaque ligne ?
<details>
<summary>Correction</summary>

```r
if (dpe$etiquette_dpe %in% c("E", "F", "G")) print("passoire")
# Erreur : the condition has length > 1
# if attend une seule valeur. Pour un test sur chaque ligne, on utilise if_else() ou case_when().
dpe |>
  mutate(passoire = if_else(etiquette_dpe %in% c("E", "F", "G"), "oui", "non")) |>
  count(passoire)
```
</details>

## Exercice 2 – Répéter avec `for`

Une boucle `for` répète le même bloc pour chaque élément d'une suite. Quand on stocke des résultats, on prépare d'abord le conteneur avec `vector()`, puis on remplit sa case `i` à chaque tour.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `for` | Répète un bloc pour chaque élément d'une suite. | `variable in suite` | `for (lettre in LETTERS[1:7]) print(lettre)` |
| `seq_along()` | Crée la suite des indices `1, 2, ..., n` d'un vecteur. | `x` | `seq_along(c("A", "B", "C"))` |
| `vector()` | Crée un vecteur vide de longueur donnée, pour le remplir ensuite. | `mode` : `"integer"`, `"character"`, `"list"`, `length` | `vector("integer", 7)` |
| `break` | Arrête la boucle immédiatement. | | `if (...) break` |

1. Afficher le nombre de DPE de chaque étiquette de `A` à `G`, avec `cat()`.
<details>
<summary>Correction</summary>

```r
for (lettre in LETTERS[1:7]) {
  n <- sum(dpe$etiquette_dpe == lettre, na.rm = TRUE)
  cat(lettre, ":", n, "\n")
}
```
</details>

2. Stocker ces effectifs dans un vecteur nommé `resultat`, rempli dans la boucle avec `seq_along()`. Comparer avec `table(dpe$etiquette_dpe)`.
<details>
<summary>Correction</summary>

```r
lettres <- LETTERS[1:7]

resultat <- vector("integer", length(lettres))
names(resultat) <- lettres

for (i in seq_along(lettres)) {
  resultat[i] <- sum(dpe$etiquette_dpe == lettres[i], na.rm = TRUE)
}

resultat
table(dpe$etiquette_dpe)  # mêmes effectifs
```
</details>


## Exercice 3 – Créer une fonction

Une fonction regroupe un calcul sous un nom, pour le réutiliser sans le recopier. Les paramètres sans valeur par défaut sont **obligatoires** : si on ne les donne pas, R renvoie une erreur. Les paramètres avec une valeur par défaut (`digits = 1`) sont **optionnels**.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `function()` | Crée une fonction et la stocke dans un objet. | paramètres, puis le corps entre accolades | `carre <- function(x) x^2` |
| `return()` | Renvoie une valeur à la sortie de la fonction. Sans `return()`, la dernière valeur calculée est renvoyée. | `value` | `return(moyenne)` |
| `stopifnot()` | Arrête la fonction avec un message si une condition est fausse. | une ou plusieurs conditions | `stopifnot(is.numeric(x))` |
| `missing()` | Indique si un paramètre n'a pas été fourni. | nom du paramètre | `missing(valeur)` |

1. Créer `resume_variable(x, digits = 1)`, qui renvoie la moyenne et la médiane de `x` arrondies à `digits` décimales. Tester sur `dpe$surface_habitable_logement`.
<details>
<summary>Correction</summary>

```r
resume_variable <- function(x, digits = 1) {
  stopifnot(is.numeric(x))

  c(
    moyenne = round(mean(x, na.rm = TRUE), digits),
    mediane = round(median(x, na.rm = TRUE), digits)
  )
}

resume_variable(dpe$surface_habitable_logement)
```
</details>

2. Appeler `resume_variable()` sans argument, puis avec `dpe$etiquette_dpe`. Quels messages obtient-on ? Pourquoi `digits` n'est-il pas obligatoire ?
<details>
<summary>Correction</summary>

```r
resume_variable()
# Erreur : argument "x" is missing, with no default
# x n'a pas de valeur par défaut : il est obligatoire.

resume_variable(dpe$etiquette_dpe)
# Erreur : is.numeric(x) is not TRUE
# digits a une valeur par défaut (1) : on peut l'omettre.

resume_variable(dpe$surface_habitable_logement, digits = 2)
```
</details>

## Exercice 4 – Graphiques pour toutes les colonnes

Une même règle peut s'appliquer à toutes les colonnes d'un tableau :
- une colonne **numérique** : une boîte à moustaches, sans les points atypiques (`outline = FALSE`) ;
- une colonne **qualitative** avec 20 modalités uniques ou moins : un diagramme en barres des pourcentages ;
- les autres colonnes : pas de graphique, avec un message.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `n_distinct()` | Compte les valeurs différentes d'une colonne (dplyr). | `x`, `na.rm` | `n_distinct(dpe$etiquette_dpe, na.rm = TRUE)` |
| `sapply()` | Applique une fonction à chaque colonne d'un tableau. | `X`, `FUN` | `sapply(dpe, n_distinct)` |
| `boxplot()` | Boîte à moustaches. `outline = FALSE` masque les points atypiques. | `x`, `outline`, `main` | `boxplot(x, outline = FALSE)` |
| `names()` | Retourne les noms des colonnes d'un tableau. | `x` | `for (nom in names(dpe))` |

1. Écrire `graphique_colonne(x, nom, seuil = 20)` : boîte à moustaches sans points atypiques si `x` est numérique ; sinon, si `x` a au plus `seuil` modalités, diagramme en barres des pourcentages ; sinon, un message qui indique que le graphique est ignoré.
<details>
<summary>Correction</summary>

```r
graphique_colonne <- function(x, nom, seuil = 20) {
  if (is.numeric(x)) {
    boxplot(x, outline = FALSE, main = nom, col = "lightblue")
  } else if (n_distinct(x, na.rm = TRUE) <= seuil) {
    pourcentages <- round(prop.table(table(x)) * 100, 1)
    barplot(pourcentages, main = nom, ylab = "Part (%)", las = 2)
  } else {
    message(nom, " : ", n_distinct(x, na.rm = TRUE), " modalités, graphique ignoré")
  }
}
```
</details>

2. Appliquer `graphique_colonne()` à toutes les colonnes de `dpe` avec une boucle `for` sur `names(dpe)`. Combien de colonnes ont plus de 20 modalités ?
<details>
<summary>Correction</summary>

```r
for (nom in names(dpe)) {
  graphique_colonne(dpe[[nom]], nom)
}

modalites <- sapply(dpe, function(x) n_distinct(x, na.rm = TRUE))
sum(modalites > 20)
# sum(modalites > 20) compte aussi les colonnes numériques, qui sont toujours tracées.
# Les colonnes ignorées sont donc les qualitatives avec plus de 20 modalités.
```
</details>

## Exercice 5 – Interroger une page de l'API

L'API ADEME renvoie les données par **pages**. La requête se fait avec `request()`, les paramètres avec `req_url_query()`, puis `req_perform()` l'envoie. La réponse est du JSON : `fromJSON()` le transforme en liste R, avec la liste des DPE dans `results`, le nombre total dans `total` et l'adresse de la page suivante dans `next`.

Les appels sont publics, sans clé d'API. Pour rester poli avec le serveur, on fait une pause entre deux appels.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `request()` | Prépare une requête HTTP vers une URL. | `url` | `request("https://data.ademe.fr/...")` |
| `req_url_query()` | Ajoute des paramètres à l'URL. | paramètres nommés | `req_url_query(size = 10)` |
| `req_perform()` | Envoie la requête et reçoit la réponse. | `req` | `req_perform(req)` |
| `resp_body_string()` | Renvoie le contenu de la réponse sous forme de texte. | `resp` | `resp_body_string(reponse)` |
| `fromJSON()` | Transforme du JSON en liste ou en tableau R. | `txt` | `fromJSON(texte)` |
| `$url` | Donne l'URL complète d'une requête, avec ses paramètres. | | `requete$url` |

1. Préparer la requête vers `https://data.ademe.fr/data-fair/api/v1/datasets/dpe03existant/lines` avec les paramètres `q_fields = "code_postal_ban"`, `code_postal_ban_eq = 69100`, `select` (les colonnes voulues, séparées par des virgules) et `size = 10`. Envoyer la requête et afficher le nom des éléments de la réponse.
<details>
<summary>Correction</summary>

```r
library(httr2)
library(jsonlite)
url_api <- "https://data.ademe.fr/data-fair/api/v1/datasets/dpe03existant/lines"

colonnes <- c(
  "numero_dpe", "date_reception_dpe", "etiquette_dpe",
  "surface_habitable_logement", "conso_5_usages_ep",
  "type_energie_n1", "identifiant_ban"
)

requete <- request(url_api) |>
  req_url_query(
    q_fields = "code_postal_ban",
    code_postal_ban_eq = 69100,
    select = paste(colonnes, collapse = ","),
    size = 10
  )

reponse <- req_perform(requete)
page <- fromJSON(resp_body_string(reponse))

names(page)
# Les noms incluent "total", "results" et "next".
```
</details>

2. Afficher `page$total` et le nombre de lignes de `page$results`. Refaire la requête avec `size = 10000` : combien de lignes reçoit-on ? Pourquoi ce paramètre est-il important ?
<details>
<summary>Correction</summary>

```r
page$total
nrow(page$results)

requete_10000 <- request(url_api) |>
  req_url_query(
    q_fields = "code_postal_ban",
    code_postal_ban_eq = 69100,
    select = paste(colonnes, collapse = ","),
    size = 10000
  )

page_10000 <- fromJSON(resp_body_string(req_perform(requete_10000)))
nrow(page_10000$results)
# Avec size = 10000, une page contient 10 000 lignes au lieu de 10.
# Il reste à parcourir les pages suivantes pour obtenir tous les DPE : c'est l'objet de l'exercice 6.
```
</details>

3. Écrire la fonction `lire_page(url)`, qui envoie la requête vers `url` et renvoie une liste de trois éléments : `donnees` (le tableau des DPE), `total` (le nombre total annoncé par l'API) et `suivante` (l'adresse de la page suivante, ou `NULL` sur la dernière page). Tester avec `url <- requete_10000$url`.
<details>
<summary>Correction</summary>

```r
lire_page <- function(url) {
  reponse <- request(url) |> req_perform()
  page <- fromJSON(resp_body_string(reponse))

  list(
    donnees = as_tibble(page$results),
    total = page$total,
    suivante = page[["next"]]   # next est un mot réservé de R : on utilise [["next"]]
  )
}

url <- requete_10000$url
test <- lire_page(url)
nrow(test$donnees)
test$total
is.null(test$suivante)  # FALSE : il y a d'autres pages
```
</details>

## Exercice 6 – Télécharger toutes les pages dans SQLite avec `while`

Une boucle `while` répète un bloc tant que la condition est vraie. Ici, on continue tant qu'il existe une page suivante : `url` devient `NULL` sur la dernière page, et la boucle s'arrête. Chaque page est écrite dans la base SQLite dès qu'elle est reçue, et non à la fin. Ainsi, si une page échoue, les précédentes sont déjà enregistrées.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `while` | Répète un bloc tant que la condition est vraie. | condition | `while (!is.null(url)) { ... }` |
| `is.null()` | Teste si une valeur est `NULL`. | `x` | `is.null(page[["next"]])` |
| `dbWriteTable()` | Crée la table, ou la remplace avec `overwrite = TRUE`. | `conn`, `name`, `value`, `overwrite` | `dbWriteTable(con, "dpe_api", donnees, overwrite = TRUE)` |
| `dbAppendTable()` | Ajoute des lignes à une table existante. Les colonnes doivent correspondre. | `conn`, `name`, `value` | `dbAppendTable(con, "dpe_api", donnees)` |
| `Sys.sleep()` | Met le script en pause, en secondes. | `time` | `Sys.sleep(1)` |

1. Test : parcourir seulement les deux premières pages avec `while`, en affichant le nombre de lignes de chaque page. Ne pas écrire encore dans la base.
<details>
<summary>Correction</summary>

```r
url <- requete_10000$url
nb_pages <- 0

while (!is.null(url) && nb_pages < 2) {   # limite de test : on arrête après deux pages
  page <- lire_page(url)
  nb_pages <- nb_pages + 1
  cat("Page", nb_pages, ":", nrow(page$donnees), "lignes\n")
  url <- page$suivante
  Sys.sleep(1)   # pause d'une seconde entre deux appels
}
```
</details>

2. Parcourir toutes les pages, et écrire chaque page dans la table `dpe_api` de `data/dpe_villeurbanne.sqlite`. La première page crée la table, les suivantes y ajoutent leurs lignes.
<details>
<summary>Correction</summary>

```r
con <- dbConnect(RSQLite::SQLite(), "data/dpe_villeurbanne.sqlite")

url <- requete_10000$url
nb_pages <- 0

while (!is.null(url)) {
  page <- lire_page(url)
  nb_pages <- nb_pages + 1

  if (nb_pages == 1) {
    dbWriteTable(con, "dpe_api", page$donnees, overwrite = TRUE)   # crée la table
  } else {
    dbAppendTable(con, "dpe_api", page$donnees)                    # ajoute les lignes
  }

  cat("Page", nb_pages, "écrite :", nrow(page$donnees), "lignes\n")
  url <- page$suivante
  Sys.sleep(1)
}
```
</details>

3. Vérifier que la base contient autant de lignes que l'API l'annonce (`total`), et qu'aucun numéro de DPE n'est présent deux fois. Puis fermer la connexion.
<details>
<summary>Correction</summary>

```r
dbGetQuery(
  con,
  "SELECT COUNT(*) AS nb, COUNT(DISTINCT numero_dpe) AS nb_distincts FROM dpe_api"
)

dbDisconnect(con)
# Si nb et nb_distincts diffèrent, une page a été écrite deux fois ou une page a été sautée.
```
</details>

## Liens utiles

- [Documentation de httr2](https://httr2.r-lib.org/)
- [Documentation de jsonlite](https://jeroen.r-universe.dev/jsonlite)
- [Référence de l'API ADEME (data-fair)](https://data.ademe.fr/datasets/dpe03existant)
- [Documentation de `while` et `for` dans R](https://stat.ethz.ch/R-manual/R-patched/library/base/html/Control.html)
