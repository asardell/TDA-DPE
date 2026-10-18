# Chapitre 2 – Import et export de données

## Objectifs

Voici les objectifs de ce chapitre :
- [ ] lire un fichier CSV avec `read.csv()` et vérifier le résultat
- [ ] comprendre les paramètres qui changent la lecture (séparateur, décimale, encodage, valeurs manquantes)
- [ ] lire un fichier Excel avec `read_excel()`, et en écrire un avec `write_xlsx()`
- [ ] exporter un dataframe en CSV avec `write.table()`
- [ ] assembler des lignes de deux dataframes avec `rbind()`
- [ ] afficher le dossier de travail avec `getwd()` et le changer avec `setwd()`
- [ ] enregistrer des objets R avec `save()` et les recharger avec `load()`

- [Chapitre 2 – Import et export de données](#chapitre-2--import-et-export-de-données)
  - [Objectifs](#objectifs)
  - [Introduction : les fichiers plats](#introduction--les-fichiers-plats)
  - [Exercice 1 – Lire le CSV avec `read.csv()`](#exercice-1--lire-le-csv-avec-readcsv)
    - [Mémo](#mémo)
    - [Lire le fichier](#lire-le-fichier)
  - [Exercice 2 – Vérifier les types et les valeurs manquantes](#exercice-2--vérifier-les-types-et-les-valeurs-manquantes)
  - [Exercice 3 – Lire et écrire un fichier Excel avec `read_excel()` et `write_xlsx()`](#exercice-3--lire-et-écrire-un-fichier-excel-avec-read_excel-et-write_xlsx)
  - [Exercice 4 – Exporter avec `write.table()`](#exercice-4--exporter-avec-writetable)
  - [Exercice 5 – Assembler des lignes avec `rbind()`](#exercice-5--assembler-des-lignes-avec-rbind)
  - [Exercice 6 – Dossier de travail avec `getwd()` et `setwd()`](#exercice-6--dossier-de-travail-avec-getwd-et-setwd)
  - [Exercice 7 – Enregistrer et recharger avec `save()` et `load()`](#exercice-7--enregistrer-et-recharger-avec-save-et-load)
  - [Exercice 8 (facultatif) – Lire un CSV directement depuis une URL](#exercice-8-facultatif--lire-un-csv-directement-depuis-une-url)
  - [Liens utiles](#liens-utiles)

## Introduction : les fichiers plats

Un fichier **CSV** (*comma-separated values*) est un fichier texte dont chaque ligne est un enregistrement et dont les champs sont séparés par un caractère. Ce caractère dépend du logiciel qui a produit le fichier :

| Région ou logiciel | Séparateur de champs | Séparateur décimal |
|--------------------|----------------------|--------------------|
| Export de l'API ADEME (utilisé ici) | `,` | `.` |
| Excel en français | `;` | `,` |
| Standard international | `,` | `.` |

Trois erreurs classiques à l'import :
- **le mauvais séparateur** : toutes les colonnes se retrouvent dans une seule ;
- **le mauvais encodage** : les accents deviennent illisibles (`Ã©` au lieu de `é`) ;
- **les valeurs manquantes** : une cellule vide n'est pas toujours lue comme `NA`.

Le fichier de la formation, `data/dpe_villeurbanne_10000.csv`, utilise un séparateur `,`, un point décimal, un encodage UTF-8 et une première ligne d'en-tête. Il commence par un caractère invisible appelé BOM, qui peut perturber le nom de la première colonne.

## Exercice 1 – Lire le CSV avec `read.csv()`

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `read.csv()` | Lit un fichier CSV et retourne un `data.frame`. | `file` : chemin du fichier, `header` : première ligne = noms (TRUE par défaut), `sep` : séparateur (`","` par défaut), `dec` : séparateur décimal (`"."` par défaut), `fileEncoding` : encodage, `na.strings` : valeurs à lire comme `NA` | `read.csv("data/fichier.csv", fileEncoding = "UTF-8-BOM")` |
| `read.csv2()` | Variante de `read.csv()` pour les fichiers au format français (`;` et `,`). | | `read.csv2("data/fichier_fr.csv")` |

### Lire le fichier

1. Lire le fichier `data/dpe_villeurbanne_10000.csv` avec `read.csv()` et stocker le résultat dans `dpe_base`. Vérifier avec `class()` et `dim()`.
<details>
<summary>Correction</summary>

```r
dpe_base <- read.csv(
  "data/dpe_villeurbanne_10000.csv",
)

class(dpe_base)
dim(dpe_base)
```
</details>

2. Afficher les noms des colonnes. Le premier nom est-il bien `numero_dpe` ?
<details>
<summary>Correction</summary>

```r
colnames(dpe_base)
```
</details>

3. Afficher les accents de la colonne `type_energie_n1` (par exemple les 5 premières valeurs). Sont-ils correctement lus ?
<details>
<summary>Correction</summary>

```r
head(dpe_base$type_energie_n1, n = 5)
```
</details>

## Exercice 2 – Vérifier les types et les valeurs manquantes

Après un import, il faut toujours contrôler le **type** de chaque colonne : une colonne de nombres lue comme texte ne se calcule pas.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `class()` | Retourne le type d'un vecteur. | | `class(dpe_base$surface_habitable_logement)` |
| `is.na()` | Indique quelles valeurs sont manquantes. | | `is.na(dpe_base$annee_construction)` |
| `sum()` | Compte les `TRUE` (ici, les valeurs manquantes). | | `sum(is.na(dpe_base$annee_construction))` |
| `colSums()` | Somme par colonne d'une matrice ou d'un dataframe. | | `colSums(is.na(dpe_base))` |

1. Afficher le type des colonnes `surface_habitable_logement`, `etiquette_dpe` et `annee_construction`.
<details>
<summary>Correction</summary>

```r
class(dpe_base$surface_habitable_logement)  # "numeric"
class(dpe_base$etiquette_dpe)               # "character"
class(dpe_base$annee_construction)          # "integer" ou "numeric"
```
</details>

2. Compter le nombre de valeurs manquantes par colonne avec `colSums(is.na(...))`. Quelle colonne en contient le plus ?
<details>
<summary>Correction</summary>

```r
colSums(is.na(dpe_base))
# annee_construction contient le plus de valeurs manquantes.
```
</details>

3. La colonne `type_energie_n2` contient des cellules vides. Sont-elles lues comme `NA` ? Vérifier avec `sum(is.na(...))` puis avec `sum(... == "")`.
<details>
<summary>Correction</summary>

```r
sum(is.na(dpe_base$type_energie_n2))
sum(dpe_base$type_energie_n2 == "")
# Une cellule vide dans une colonne de texte est lue comme "" et non comme NA.
# Pour corriger à l'import : read.csv(..., na.strings = c("", "NA"))
```
</details>

4. Relire le fichier avec `na.strings = c("", "NA")` dans `dpe_na`, puis recompter les valeurs manquantes de `type_energie_n2`.
<details>
<summary>Correction</summary>

```r
dpe_na <- read.csv(
  "data/dpe_villeurbanne_10000.csv",
  na.strings = c("", "NA")
)

sum(is.na(dpe_na$type_energie_n2))
```
</details>

## Exercice 3 – Lire et écrire un fichier Excel avec `read_excel()` et `write_xlsx()`

Le package `readxl` lit les fichiers `.xlsx`, mais ne sait pas en écrire. Pour l'écriture, on utilise le package `writexl`, qui n'a pas besoin de Microsoft Excel installé.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `read_excel()` | Lit une feuille d'un fichier Excel. | `path` : chemin du fichier, `sheet` : nom ou numéro de la feuille (la première par défaut) | `readxl::read_excel(path = "data/fichier.xlsx", sheet = 1)` |
| `write_xlsx()` | Écrit un dataframe dans un fichier `.xlsx`. | `x` : le dataframe, `path` : chemin de sortie | `writexl::write_xlsx(x = df, path = "data/fichier.xlsx")` |

1. Installer les packages `readxl` et `writexl` si besoin, puis les charger.
<details>
<summary>Correction</summary>

```r
install.packages(c("readxl", "writexl"))

library(readxl)
library(writexl)
```
</details>

2. Créer un dataframe `dpe_apercu` contenant les 20 premières lignes de `dpe_base`, puis l'écrire dans `data/dpe_apercu.xlsx`.
<details>
<summary>Correction</summary>

```r
dpe_apercu <- dpe_base[1:20, ]
write_xlsx(x = dpe_apercu, path = "data/dpe_apercu.xlsx")
```
</details>

3. Relire le fichier avec `read_excel()` et vérifier ses dimensions.
<details>
<summary>Correction</summary>

```r
dpe_excel <- read_excel(path = "data/dpe_apercu.xlsx")
dim(dpe_excel)
# [1] 20 20
```
</details>

## Exercice 4 – Exporter avec `write.table()`

`write.table()` écrit un dataframe dans un fichier texte. Pour un export au format français, on précise `sep` et `dec`.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `write.table()` | Écrit un dataframe dans un fichier texte. | `x` : l'objet, `file` : chemin de sortie, `sep` : séparateur (" " par défaut), `dec` : séparateur décimal, `row.names` : écrire les noms de lignes (TRUE par défaut), `fileEncoding` : encodage | `write.table(df, file = "out.csv", sep = ";", row.names = FALSE)` |
| `write.csv()` | Variante de `write.table()` au format CSV standard (`,` et `.`). | | `write.csv(df, file = "out.csv", row.names = FALSE)` |

1. Sélectionner les lignes de `dpe_base` dont l'étiquette est `A`, `B` ou `C`, à l'aide de `dpe_base$etiquette_dpe %in% c("A", "B", "C")`, et stocker le résultat dans `dpe_abc`.
<details>
<summary>Correction</summary>

```r
dpe_abc <- dpe_base[dpe_base$etiquette_dpe %in% c("A", "B", "C"), ]
dim(dpe_abc)
```
</details>

2. Écrire `dpe_abc` dans `data/dpe_abc_fr.csv` avec un séparateur `;`, un séparateur décimal `,`, sans noms de lignes et en UTF-8.
<details>
<summary>Correction</summary>

```r
write.table(
  dpe_abc,
  file = "data/dpe_abc_fr.csv",
  sep = ";",
  dec = ",",
  row.names = FALSE,
  fileEncoding = "UTF-8"
)
```
</details>

## Exercice 5 – Assembler des lignes avec `rbind()`

`rbind()` empile des dataframes qui ont **les mêmes colonnes**, dans le même ordre. C'est la façon de « mettre bout à bout » deux extraits de données, par exemple deux fichiers mensuels.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `rbind()` | Empile les lignes de plusieurs dataframes ayant les mêmes colonnes. | `...` : les dataframes à empiler | `rbind(df1, df2)` |

1. Créer `partie_1` avec les 5 premières lignes de `dpe_base`, et `partie_2` avec les lignes 6 à 10.
<details>
<summary>Correction</summary>

```r
partie_1 <- dpe_base[1:5, ]
partie_2 <- dpe_base[6:10, ]
```
</details>

2. Les empiler avec `rbind()` dans `dix_premieres`, puis vérifier qu'il est identique à `dpe_base[1:10, ]`.
<details>
<summary>Correction</summary>

```r
dix_premieres <- rbind(partie_1, partie_2)
identical(dix_premieres, dpe_base[1:10, ])
# [1] TRUE
```
</details>

3. Tenter de combiner `dpe_base[1:5, ]` avec `dpe_base[1:5, c("numero_dpe", "etiquette_dpe")]`. Quel message d'erreur obtient-on ?
<details>
<summary>Correction</summary>

```r
rbind(dpe_base[1:5, ], dpe_base[1:5, c("numero_dpe", "etiquette_dpe")])
# Erreur : les colonnes ne correspondent pas (nombre de colonnes différent).
```
</details>

## Exercice 6 – Dossier de travail avec `getwd()` et `setwd()`

R lit et écrit les fichiers par rapport à un **dossier de travail**. Tous les chemins de cette formation, comme `"data/dpe_villeurbanne_10000.csv"`, supposent que ce dossier est la racine du dépôt TDA-DPE. Si le dossier change, les mêmes chemins ne fonctionnent plus.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `getwd()` | Affiche le dossier de travail courant. | aucun | `getwd()` |
| `setwd()` | Change le dossier de travail. Renvoie l'ancien dossier, de façon invisible. | `dir` : chemin du nouveau dossier | `setwd("data")` |
| `list.files()` | Liste les fichiers d'un dossier. | `path` : dossier, `pattern` : filtre sur le nom | `list.files("data")` |
| `file.exists()` | Indique si un fichier existe à ce chemin. | `file` : chemin du fichier | `file.exists("data/dpe_villeurbanne_10000.csv")` |

Sous Windows, un chemin écrit avec `\` doit s'écrire `/` ou `\\` : `setwd("C:/Users/Anthony/Documents")` ou `setwd("C:\\Users\\Anthony\\Documents")`. Dans un script partagé, on évite les chemins absolus : un chemin relatif comme `"data/..."` fonctionne sur tous les postes, à condition d'ouvrir le projet à la racine du dépôt.

1. Afficher le dossier de travail courant. Est-ce la racine du dépôt TDA-DPE ?
<details>
<summary>Correction</summary>

```r
getwd()
```
</details>

2. Vérifier que le fichier de la formation est trouvé avec `file.exists()`, puis lister le contenu du dossier `data` avec `list.files()`.
<details>
<summary>Correction</summary>

```r
file.exists("data/dpe_villeurbanne_10000.csv")  # [1] TRUE
list.files("data")
```
</details>

3. Passer dans le dossier `data` avec `setwd("data")`, puis relire le fichier avec le chemin `"data/dpe_villeurbanne_10000.csv"`. Que se passe-t-il ?
<details>
<summary>Correction</summary>

```r
ancien_dossier <- getwd()
setwd("data")

read.csv("data/dpe_villeurbanne_10000.csv")
# Erreur : le fichier est introuvable. Le chemin est maintenant relatif à data/,
# donc R cherche data/data/dpe_villeurbanne_10000.csv.
```
</details>

4. Revenir au dossier de départ avec la valeur sauvegardée dans `ancien_dossier`, puis vérifier avec `getwd()`.
<details>
<summary>Correction</summary>

```r
setwd(ancien_dossier)
getwd()
```
</details>

## Exercice 7 – Enregistrer et recharger avec `save()` et `load()`

`write.table()` et `write.csv()` enregistrent du texte : les types (facteurs, dates, nombres) ne sont pas conservés. `save()` enregistre les objets R tels quels, dans un fichier `.RData`. `load()` les recharge avec leurs noms d'origine. C'est pratique pour garder un résultat intermédiaire sans relancer tout l'import.

Le fichier `.RData` ne se lit qu'avec R. Pour transmettre des données à quelqu'un qui n'utilise pas R, on reste sur le CSV ou le fichier Excel.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `save()` | Enregistre un ou plusieurs objets dans un fichier `.RData`. | `...` : noms des objets (sans guillemets), `list` : noms des objets en texte, `file` : chemin du fichier | `save(dpe_base, dpe_abc, file = "data/sauvegarde.RData")` |
| `load()` | Recharge dans la session les objets d'un fichier `.RData`, avec leurs noms d'origine. Renvoie les noms chargés. | `file` : chemin du fichier | `load("data/sauvegarde.RData")` |
| `saveRDS()` / `readRDS()` | Variante pour un seul objet. Le nom n'est pas conservé : on le choisit à la lecture. | `object`, `file` | `dpe_abc <- readRDS("data/dpe_abc.rds")` |

1. Enregistrer `dpe_base` et `dpe_abc` dans `data/sauvegarde.RData`, puis vérifier que le fichier existe.
<details>
<summary>Correction</summary>

```r
save(dpe_base, dpe_abc, file = "data/sauvegarde.RData")
file.exists("data/sauvegarde.RData")
```
</details>

2. Supprimer les deux objets avec `rm()`, vérifier avec `exists()` qu'ils ont disparu, puis les recharger avec `load()`.
<details>
<summary>Correction</summary>

```r
rm(dpe_base, dpe_abc)
exists("dpe_abc")  # [1] FALSE

noms <- load("data/sauvegarde.RData")
noms
dim(dpe_abc)
```
</details>

3. Créer un objet `dpe_abc <- "brouillon"`, puis rappeler `load()`. Que devient `dpe_abc` ? Pourquoi faut-il être prudent avec `load()` ?
<details>
<summary>Correction</summary>

```r
dpe_abc <- "brouillon"
load("data/sauvegarde.RData")
dim(dpe_abc)
# load() remplace sans prévenir les objets qui portent le même nom.
```
</details>

## Exercice 8 (facultatif) – Lire un CSV directement depuis une URL

Quand une URL renvoie directement un fichier CSV, `read.csv()` peut la lire sans téléchargement préalable. La requête est longue : elle sert ici à montrer le principe. L'interrogation de l'API, avec ses paramètres et sa pagination, est traitée au chapitre 10.

1. Lire les 10 premières lignes d'un CSV distant. L'URL suivante renvoie les DPE de Villeurbanne avec `size=10000` :

```r
url <- "https://data.ademe.fr/data-fair/api/v1/datasets/dpe03existant/lines?q_fields=code_postal_ban&code_postal_ban_eq=69100&select=numero_dpe%2Cdate_reception_dpe%2Cetiquette_dpe%2Ctype_batiment%2Cannee_construction%2Cperiode_construction%2Ctype_installation_chauffage%2Csurface_habitable_logement%2Ccode_postal_ban%2Cdeperditions_murs%2Cbesoin_chauffage%2Cconso_5_usages_ep%2Cconso_chauffage_ep%2Cconso_ecs_ep%2Cconso_refroidissement_ep%2Cconso_eclairage_ep%2Cconso_auxiliaires_ep%2Ctype_energie_n1%2Ctype_energie_n2%2Ccout_total_5_usages%2Cidentifiant_ban&format=csv"
```

<details>
<summary>Correction</summary>

```r
dpe_url <- read.csv(url)
dim(dpe_url)
```
</details>

## Liens utiles

- [Documentation de `read.csv()` et `write.table()`](https://stat.ethz.ch/R-manual/R-patched/library/utils/html/read.table.html)
- [Package `readxl`](https://readxl.tidyverse.org/)
- [Package `writexl`](https://docs.ropensci.org/writexl/)
- [Données DPE de l'ADEME](https://data.ademe.fr/datasets/dpe03existant)
