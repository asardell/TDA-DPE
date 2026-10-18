# Chapitre 1 – Les bases de R, les vecteurs et les dataframes

## Objectifs

Voici les objectifs de ce chapitre :
- [ ] comprendre ce qu'est R et comment il se situe par rapport à Python
- [ ] prendre en main RStudio, les scripts et les commentaires
- [ ] créer et manipuler des objets et des vecteurs
- [ ] importer un fichier CSV avec l'assistant de RStudio
- [ ] calculer des statistiques descriptives simples sur les colonnes d'un dataframe
- [ ] créer un dataframe à partir de vecteurs

- [Chapitre 1 – Les bases de R, les vecteurs et les dataframes](#chapitre-1--les-bases-de-r-les-vecteurs-et-les-dataframes)
  - [Objectifs](#objectifs)
  - [Introduction : R et Python](#introduction--r-et-python)
  - [Exercice 1 – Les objets et les vecteurs](#exercice-1--les-objets-et-les-vecteurs)
    - [Mémo](#mémo)
    - [Les fonctions `c()`, `rm()`, `print()`](#les-fonctions-c-rm-print)
  - [Exercice 2 – Fonctions usuelles sur les vecteurs](#exercice-2--fonctions-usuelles-sur-les-vecteurs)
    - [Mémo](#mémo-1)
    - [Les fonctions `class()`, `seq()`, `rep()`, `length()`](#les-fonctions-class-seq-rep-length)
    - [Accéder aux éléments d'un vecteur](#accéder-aux-éléments-dun-vecteur)
  - [Exercice 3 – Statistiques descriptives et simulation](#exercice-3--statistiques-descriptives-et-simulation)
    - [Mémo](#mémo-2)
    - [Les fonctions `runif()`, `mean()`, `median()`, `min()`, `max()`](#les-fonctions-runif-mean-median-min-max)
    - [Les fonctions `rnorm()`, `sd()`, `var()`, `hist()`, `quantile()`](#les-fonctions-rnorm-sd-var-hist-quantile)
    - [Les fonctions `sum()`, `round()`](#les-fonctions-sum-round)
    - [Les fonctions `sample()`, `table()`, `prop.table()`, `unique()`, `sort()`](#les-fonctions-sample-table-proptable-unique-sort)
  - [Exercice 4 – Importer les DPE de Villeurbanne](#exercice-4--importer-les-dpe-de-villeurbanne)
    - [Mémo](#mémo-3)
    - [Importer le fichier avec l'assistant](#importer-le-fichier-avec-lassistant)
  - [Exercice 5 – Explorer le dataframe et ses colonnes](#exercice-5--explorer-le-dataframe-et-ses-colonnes)
    - [Explorer la structure](#explorer-la-structure)
    - [Explorer une colonne](#explorer-une-colonne)
    - [Statistiques sur une colonne](#statistiques-sur-une-colonne)
  - [Exercice 6 – Créer un dataframe à partir de vecteurs](#exercice-6--créer-un-dataframe-à-partir-de-vecteurs)
  - [Liens utiles](#liens-utiles)

## Introduction : R et Python

**R** est un langage conçu pour la statistique et l'analyse de données. Il a été créé pour manipuler des données tabulaires et produire des analyses et des graphiques. **Python** est un langage généraliste qui s'est fortement développé en data science grâce à des bibliothèques comme `pandas`, `numpy` ou `scikit-learn`. Les deux langages font aujourd'hui sensiblement la même chose sur les données : les concepts (vecteurs, tableaux, filtres, agrégations) se retrouvent dans les deux.

Dans ce module, on utilise **RStudio**, un environnement de développement qui regroupe :
- un éditeur de scripts (en haut à gauche) ;
- une console où s'exécutent les commandes (en bas à gauche) ;
- l'environnement, qui liste les objets créés (en haut à droite) ;
- les fichiers, les graphiques et l'aide (en bas à droite).

Quelques différences utiles à connaître quand on vient de Python :

| Sujet | R | Python |
|-------|---|--------|
| Affectation | `x <- 5` (ou `x = 5`) | `x = 5` |
| Position du premier élément | 1 | 0 |
| Liste de valeurs | vecteur, créé avec `c()` | liste `[]` ou tableau `numpy` |
| Tableau de données | `data.frame` (ou `tibble`) | `pd.DataFrame` |
| Accéder à une colonne | `df$colonne` | `df["colonne"]` |
| Valeur manquante | `NA` | `NaN`, `None`, `pd.NA` |
| Installer un package | `install.packages("nom")` | `pip install nom` |
| Charger un package | `library(nom)` | `import nom` |
| Aide sur une fonction | `?mean` | `help(mean)` |

Deux points à retenir dès le début :
- **R est sensible à la casse** : `Surface` et `surface` sont deux objets différents.
- **Les opérations sont vectorisées** : `surfaces * 2` multiplie chaque élément du vecteur, sans boucle.

Les exercices 1 à 3 utilisent des vecteurs créés à la main. À partir de l'exercice 4, on travaille sur les **DPE (diagnostics de performance énergétique) de l'ADEME pour Villeurbanne**, qui serviront de fil rouge pour toute la formation.

## Exercice 1 – Les objets et les vecteurs

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `<-` | Affecte une valeur à un objet. | | `surface <- 65` |
| `c()` | Concatène des valeurs ou des vecteurs pour créer un nouveau vecteur. | | `etiquettes <- c("A", "B", "C")` |
| `rm()` | Supprime des objets de l'environnement. | `list = ls()` supprime tous les objets | `rm(surface)` |
| `print()` | Affiche un objet dans la console. | | `print(surface)` |

### Les fonctions `c()`, `rm()`, `print()`

1. Créer un objet `surface_1` avec la valeur `65` et un objet `surface_2` avec la valeur `42.5`.
<details>
<summary>Correction</summary>

```r
surface_1 <- 65
surface_2 <- 42.5
```
</details>

2. Additionner les deux surfaces et stocker le résultat dans un objet `surface_totale`. Afficher cet objet avec `print()`.
<details>
<summary>Correction</summary>

```r
surface_totale <- surface_1 + surface_2
print(surface_totale)
```
</details>

3. Créer un objet `Surface_1` avec la valeur `80`. Que remarque-t-on sur `surface_1` et `Surface_1` ?
<details>
<summary>Correction</summary>

```r
Surface_1 <- 80
print(surface_1)
print(Surface_1)
# R est sensible à la casse : ce sont deux objets différents
```
</details>

4. Créer un vecteur `etiquettes` qui contient les lettres `"A"`, `"C"`, `"E"`, `"G"` et `"D"`, en utilisant `c()`.
<details>
<summary>Correction</summary>

```r
etiquettes <- c("A", "C", "E", "G", "D")
print(etiquettes)
```
</details>

5. Supprimer tous les objets créés dans cet exercice avec `rm()`.
<details>
<summary>Correction</summary>

```r
rm(surface_1, surface_2, Surface_1, surface_totale)
# Pour supprimer tous les objets d'un coup : rm(list = ls())
```
</details>

## Exercice 2 – Fonctions usuelles sur les vecteurs

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `:` | Génère une séquence avec un pas de 1. | | `8:15` |
| `seq()` | Génère une séquence de nombres. | `from`, `to`, `by` | `seq(from = 0, to = 100, by = 20)` |
| `rep()` | Répète les éléments d'un vecteur. | `times` : nombre de répétitions | `rep("E", times = 3)` |
| `length()` | Retourne la longueur d'un vecteur. | | `length(surfaces)` |
| `class()` | Retourne le type d'un objet. | | `class(surfaces)` |
| `[ ]` | Accède aux éléments d'un vecteur. | | `surfaces[4]` |
| `[-1]` | Accède à tous les éléments sauf ceux précisés. | | `surfaces[-4]` |

### Les fonctions `class()`, `seq()`, `rep()`, `length()`

On utilise le vecteur suivant, qui contient les surfaces habitables de 5 logements fictifs :

```r
surfaces <- c(65, 42.5, 88, 120, 51)
```

1. Afficher le type de `surfaces` et le type de `etiquettes` avec `class()`. Quelle différence observe-t-on ?
<details>
<summary>Correction</summary>

```r
etiquettes <- c("A", "C", "E", "G", "D")
class(surfaces)    # "numeric" : des nombres
class(etiquettes)  # "character" : du texte
```
</details>

2. Générer la séquence des nombres de 0 à 100 avec un pas de 20, avec `seq()`. Puis générer la séquence de 8 à 15 avec `:`.
<details>
<summary>Correction</summary>

```r
seq(from = 0, to = 100, by = 20)
# [1]   0  20  40  60  80 100
8:15
# [1]  8  9 10 11 12 13 14 15
```
</details>

3. Répéter la lettre `"E"` trois fois avec `rep()`. Calculer ensuite le nombre de logements dans le vecteur `surfaces` avec `length()`.
<details>
<summary>Correction</summary>

```r
rep("E", times = 3)
# [1] "E" "E" "E"
length(surfaces)
# [1] 5
```
</details>

4. (Facultatif) Créer un vecteur `numeros_logements` contenant les nombres de 1 à 5, puis un vecteur `etiquettes_repetees` qui répète 2 fois chaque étiquette de `etiquettes`.
<details>
<summary>Correction</summary>

```r
numeros_logements <- 1:5
etiquettes_repetees <- rep(etiquettes, times = 2)
```
</details>

### Accéder aux éléments d'un vecteur

Les positions commencent à **1** en R (contrairement à Python où elles commencent à 0).

5. Afficher le deuxième élément de `surfaces`, puis les éléments 1 et 3.
<details>
<summary>Correction</summary>

```r
surfaces[2]
# [1] 42.5
surfaces[c(1, 3)]
# [1] 65 88
```
</details>

6. Afficher tous les éléments de `surfaces` sauf le premier.
<details>
<summary>Correction</summary>

```r
surfaces[-1]
# [1]  42.5  88.0 120.0  51.0
```
</details>

## Exercice 3 – Statistiques descriptives et simulation

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `runif()` | Génère des valeurs suivant une loi uniforme. | `n`, `min`, `max` | `runif(20, min = 30, max = 150)` |
| `rnorm()` | Génère des valeurs suivant une loi normale. | `n`, `mean`, `sd` | `rnorm(20, mean = 250, sd = 60)` |
| `set.seed()` | Fixe la graine aléatoire pour obtenir les mêmes résultats. | `seed` | `set.seed(42)` |
| `mean()` | Calcule la moyenne. | `na.rm` (FALSE par défaut) | `mean(surfaces)` |
| `median()` | Calcule la médiane. | `na.rm` (FALSE par défaut) | `median(surfaces)` |
| `min()`, `max()` | Retournent la valeur minimale ou maximale. | `na.rm` (FALSE par défaut) | `min(surfaces)` |
| `sd()`, `var()` | Calculent l'écart type et la variance. | `na.rm` (FALSE par défaut) | `sd(surfaces)` |
| `quantile()` | Calcule les quantiles. | `probs`, `na.rm` | `quantile(surfaces, probs = c(0.25, 0.5, 0.75))` |
| `hist()` | Trace un histogramme. | `x`, `breaks` | `hist(surfaces, breaks = 10)` |
| `sum()` | Calcule la somme. | `na.rm` (FALSE par défaut) | `sum(surfaces)` |
| `round()` | Arrondit les nombres. | `digits` (0 par défaut) | `round(3.14159, digits = 2)` |
| `sample()` | Tire un échantillon aléatoire. | `x`, `size`, `replace` | `sample(etiquettes, size = 10, replace = TRUE)` |
| `table()` | Construit une table de fréquences. | `x` | `table(etiquettes)` |
| `prop.table()` | Convertit des effectifs en proportions. | `x` | `prop.table(table(etiquettes))` |
| `unique()` | Retourne les valeurs uniques. | `x` | `unique(etiquettes)` |
| `sort()` | Trie les éléments d'un vecteur. | `x`, `decreasing` (FALSE par défaut) | `sort(surfaces, decreasing = TRUE)` |

### Les fonctions `runif()`, `mean()`, `median()`, `min()`, `max()`

Les données sont simulées : on fixe la graine avec `set.seed()` pour obtenir les mêmes valeurs à chaque exécution.

1. Simuler les surfaces de 20 logements suivant une loi uniforme entre 30 et 150 m², et stocker le résultat dans `surfaces_simulees`.
<details>
<summary>Correction</summary>

```r
set.seed(42)
surfaces_simulees <- runif(20, min = 30, max = 150)
```
</details>

2. Calculer la moyenne, la médiane, le minimum et le maximum de `surfaces_simulees`.
<details>
<summary>Correction</summary>

```r
mean(surfaces_simulees)
median(surfaces_simulees)
min(surfaces_simulees)
max(surfaces_simulees)
```
</details>

### Les fonctions `rnorm()`, `sd()`, `var()`, `hist()`, `quantile()`

3. Simuler les consommations de 20 logements suivant une loi normale de moyenne 250 et d'écart type 60, dans un vecteur `conso`. Calculer l'écart type et la variance.
<details>
<summary>Correction</summary>

```r
conso <- rnorm(20, mean = 250, sd = 60)
sd(conso)
var(conso)
```
</details>

4. Calculer les quartiles de `conso` avec `quantile()`, puis tracer son histogramme avec `hist()`.
<details>
<summary>Correction</summary>

```r
quantile(conso, probs = c(0.25, 0.5, 0.75))
hist(conso, breaks = 10)
```
</details>

### Les fonctions `sum()`, `round()`

5. Calculer la somme de `surfacconsoes_simulees`, puis sa moyenne arrondie à une décimale.
<details>
<summary>Correction</summary>

```r
sum(conso)
round(mean(conso), digits = 1)
```
</details>

### Les fonctions `sample()`, `table()`, `prop.table()`, `unique()`, `sort()`

6. Tirer au hasard 10 étiquettes parmi `"A"`, `"B"`, `"C"`, `"D"`, `"E"`, `"F"`, `"G"` avec remise, et stocker le résultat dans `echantillon_dpe`.
<details>
<summary>Correction</summary>

```r
set.seed(42)
echantillon_dpe <- sample(c("A", "B", "C", "D", "E", "F", "G"), size = 10, replace = TRUE)
```
</details>

7. Afficher les étiquettes uniques de `echantillon_dpe`, leurs effectifs avec `table()`, puis leurs proportions avec `prop.table()`.
<details>
<summary>Correction</summary>

```r
unique(echantillon_dpe)
table(echantillon_dpe)
prop.table(table(echantillon_dpe))
```
</details>

8. Trier `surfaces_simulees` par ordre décroissant avec `sort()`.
<details>
<summary>Correction</summary>

```r
sort(surfaces_simulees, decreasing = TRUE)
```
</details>

## Exercice 4 – Importer les DPE de Villeurbanne

Le fichier `dpe_villeurbanne_10000.csv` se trouve dans le dossier `data/` du dépôt. Il contient **10 000 DPE** de Villeurbanne (code postal 69100) et **20 colonnes**, issues de l'API de l'ADEME.

### Mémo

| Élément | Description |
|---------|-------------|
| Menu `File > Import Dataset > From Text (readr)` | Ouvre l'assistant d'import d'un fichier texte ou CSV. |
| `read_csv()` | Fonction générée par l'assistant pour lire un CSV (package `readr`). |

### Importer le fichier avec l'assistant

1. Dans RStudio, ouvrir le menu **File > Import Dataset > From Text (base)**.
2. Choisir le fichier `data/dpe_villeurbanne_10000.csv`.
3. Vérifier l'aperçu : les colonnes doivent être séparées par des virgules et les accents doivent s'afficher correctement.
4. Conserver le nom proposé (`dpe_villeurbanne_10000`) et cliquer sur **Import**.
5. Dans la console, RStudio affiche le code qu'il a exécuté. Copier ce code dans votre script pour le conserver.

<details>
<summary>Correction (code généré par l'assistant, à peu près équivalent)</summary>

```r
dpe_villeurbanne_10000 <- read_csv("data/dpe_villeurbanne_10000.csv")
```
</details>

## Exercice 5 – Explorer le dataframe et ses colonnes

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `$` | Accède à une colonne d'un dataframe, sous forme de vecteur. | | `dpe$etiquette_dpe` |
| `[ , ]` | Accède à certaines lignes et/ou colonnes. | `lignes`, `colonnes` | `dpe[2, c("etiquette_dpe", "type_energie_n1")]` |
| `dim()` | Retourne le nombre de lignes et de colonnes. | | `dim(dpe)` |
| `nrow()`, `ncol()` | Retournent le nombre de lignes ou de colonnes. | | `nrow(dpe)` |
| `colnames()` | Retourne les noms des colonnes. | | `colnames(dpe)` |
| `head()` | Affiche les premières lignes. | `n` (6 par défaut) | `head(dpe, n = 3)` |
| `summary()` | Produit un résumé statistique de chaque colonne. | | `summary(dpe)` |
| `View()` | Ouvre le dataframe dans une fenêtre de visualisation. | | `View(dpe)` |
| `class()` | Retourne le type d'un objet. | | `class(dpe)` |
| `unique()` | Retourne les valeurs uniques d'un vecteur. | | `unique(dpe$etiquette_dpe)` |
| `table()` | Construit une table de fréquences. | | `table(dpe$etiquette_dpe)` |
| `prop.table()` | Convertit des effectifs en proportions. | | `prop.table(table(dpe$etiquette_dpe))` |
| `sort()` | Trie les éléments d'un vecteur. | `decreasing` (FALSE par défaut) | `sort(x, decreasing = TRUE)` |

Dans les exercices suivants, on crée un raccourci pour le dataframe importé :

```r
dpe <- dpe_villeurbanne_10000
```

### Explorer la structure

1. Afficher les dimensions du dataframe, son nombre de lignes, son nombre de colonnes, puis les noms des colonnes.
<details>
<summary>Correction</summary>

```r
dim(dpe)
nrow(dpe)
ncol(dpe)
colnames(dpe)
```
</details>

2. Afficher les 3 premières lignes, puis le type de l'objet `dpe`.
<details>
<summary>Correction</summary>

```r
head(dpe, n = 3)
class(dpe)
```
</details>

3. Produire un résumé avec `summary()`, puis ouvrir le dataframe avec `View()`. Repérer la colonne `annee_construction` : que représente le nombre de valeurs manquantes affiché ?
<details>
<summary>Correction</summary>

```r
summary(dpe)
View(dpe)
# Les valeurs manquantes (NA) sont des DPE pour lesquels l'année de construction n'est pas renseignée.
```
</details>

### Explorer une colonne

Une colonne se récupère avec `$` et devient un **vecteur**.

4. Afficher la colonne `etiquette_dpe`, ses valeurs uniques, puis sa répartition en effectifs et en proportions.
<details>
<summary>Correction</summary>

```r
dpe$etiquette_dpe
unique(dpe$etiquette_dpe)
table(dpe$etiquette_dpe)
prop.table(table(dpe$etiquette_dpe))
```
</details>

5. Afficher les effectifs de `type_energie_n1`, du plus fréquent au moins fréquent.
<details>
<summary>Correction</summary>

```r
sort(table(dpe$type_energie_n1), decreasing = TRUE)
```
</details>

6. Afficher les 10 premières valeurs de `surface_habitable_logement`, puis la valeur du 5e logement.
<details>
<summary>Correction</summary>

```r
dpe$surface_habitable_logement[1:10]
dpe$surface_habitable_logement[5]
```
</details>

### Statistiques sur une colonne

7. Calculer la moyenne, la médiane, le minimum et le maximum de `surface_habitable_logement`. Que se passe-t-il à cause des valeurs manquantes ? Recommencer avec `na.rm = TRUE`.
<details>
<summary>Correction</summary>

```r
mean(dpe$surface_habitable_logement)
# [1] NA

mean(dpe$surface_habitable_logement, na.rm = TRUE)
median(dpe$surface_habitable_logement, na.rm = TRUE)
min(dpe$surface_habitable_logement, na.rm = TRUE)
max(dpe$surface_habitable_logement, na.rm = TRUE)
```
</details>

8. Calculer l'écart type, la variance, la somme et les quartiles de `surface_habitable_logement`, puis arrondir l'écart type à une décimale.
<details>
<summary>Correction</summary>

```r
sd(dpe$surface_habitable_logement, na.rm = TRUE)
var(dpe$surface_habitable_logement, na.rm = TRUE)
sum(dpe$surface_habitable_logement, na.rm = TRUE)
quantile(dpe$surface_habitable_logement, probs = c(0.25, 0.5, 0.75), na.rm = TRUE)
round(sd(dpe$surface_habitable_logement, na.rm = TRUE), digits = 1)
```
</details>

9. Tracer l'histogramme de `surface_habitable_logement`. Que remarque-t-on sur la forme de la distribution ?
<details>
<summary>Correction</summary>

```r
hist(dpe$surface_habitable_logement, breaks = 20)
# La distribution est concentrée sur les petites surfaces, avec quelques valeurs extrêmes :
# on y reviendra au chapitre 6.
```
</details>

10. Afficher les étiquettes DPE triées, sans doublon.
<details>
<summary>Correction</summary>

```r
sort(unique(dpe$etiquette_dpe))
```
</details>

## Exercice 6 – Créer un dataframe à partir de vecteurs

Un dataframe est un tableau dont chaque colonne est un vecteur **de même longueur**. En Python, on construit un `DataFrame` à partir d'un dictionnaire de listes ; en R, on utilise `data.frame()`.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `data.frame()` | Crée un tableau à partir de vecteurs de même longueur. | Un argument par colonne, `nom = vecteur` | `data.frame(a = 1:3, b = c("x", "y", "z"))` |

1. Créer trois vecteurs `etiquette`, `surface` et `energie` contenant les 10 premières valeurs des colonnes `etiquette_dpe`, `surface_habitable_logement` et `type_energie_n1`.
<details>
<summary>Correction</summary>

```r
etiquette <- dpe$etiquette_dpe[1:10]
surface <- dpe$surface_habitable_logement[1:10]
energie <- dpe$type_energie_n1[1:10]
```
</details>

2. Créer le dataframe `dpe_extrait` à partir de ces trois vecteurs, puis l'afficher et vérifier ses dimensions.
<details>
<summary>Correction</summary>

```r
dpe_extrait <- data.frame(
  etiquette_dpe = etiquette,
  surface_habitable_logement = surface,
  type_energie_n1 = energie
)

dpe_extrait
dim(dpe_extrait)
```
</details>

*Équivalent Python :*
```python
import pandas as pd

dpe_extrait = pd.DataFrame({
    "etiquette_dpe": etiquette,
    "surface_habitable_logement": surface,
    "type_energie_n1": energie,
})
```

3. Vérifier le type de `dpe_extrait` avec `class()`
  
<details>
<summary>Correction</summary>

```r
class(dpe_extrait)  # "data.frame"
```
</details>

## Liens utiles

- [Données DPE de l'ADEME](https://data.ademe.fr/datasets/dpe03existant)
- [Cours sur la programmation R](https://asardell.github.io/programmation-r/)
- [Documentation de R (manuels officiels)](https://cran.r-project.org/manuals.html)
