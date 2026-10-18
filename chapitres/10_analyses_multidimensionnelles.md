# Chapitre 10 – Analyses multidimensionnelles : ACP, classification et régression

## Objectifs

Voici les objectifs de ce chapitre :
- [ ] résumer plusieurs variables quantitatives par une ACP avec `PCA()` de FactoMineR, et la lire avec factoextra
- [ ] regrouper des logements semblables par une classification ascendante hiérarchique (CAH) avec `hclust()`
- [ ] expliquer le besoin de chauffage par une régression linéaire simple, puis multiple, avec `lm()`
- [ ] prédire une valeur avec `predict()`, et vérifier la qualité du modèle

- [Chapitre 10 – Analyses multidimensionnelles : ACP, classification et régression](#chapitre-10--analyses-multidimensionnelles--acp-classification-et-régression)
  - [Objectifs](#objectifs)
  - [Préparation](#préparation)
  - [Exercice 1 – Choisir les variables et vérifier les données](#exercice-1--choisir-les-variables-et-vérifier-les-données)
    - [Mémo](#mémo)
  - [Exercice 2 – Faire une ACP avec FactoMineR](#exercice-2--faire-une-acp-avec-factominer)
    - [Mémo](#mémo-1)
  - [Exercice 3 – Lire l'ACP avec factoextra](#exercice-3--lire-lacp-avec-factoextra)
    - [Mémo](#mémo-2)
  - [Exercice 4 – Regrouper les logements par CAH](#exercice-4--regrouper-les-logements-par-cah)
    - [Mémo](#mémo-3)
  - [Exercice 5 – Prédire le besoin de chauffage avec `lm()`](#exercice-5--prédire-le-besoin-de-chauffage-avec-lm)
    - [Mémo](#mémo-4)
  - [Liens utiles](#liens-utiles)

## Préparation

Ce chapitre utilise deux packages, à installer une seule fois : **FactoMineR** calcule l'ACP, et **factoextra** la dessine. La CAH et la régression utilisent les fonctions de base de R.

```r
# install.packages(c("FactoMineR", "factoextra"))  # une seule fois
library(readr)
library(dplyr)
library(FactoMineR)
library(factoextra)

dpe <- read_csv("data/dpe_villeurbanne_10000.csv")
```

Les méthodes multidimensionnelles résument plusieurs variables à la fois. Elles supposent des données propres : on retire donc les lignes avec des valeurs manquantes, et on ne garde que les variables utiles.

## Exercice 1 – Choisir les variables et vérifier les données

On travaille sur cinq variables quantitatives : la surface, les déperditions des murs, le besoin de chauffage et deux consommations. L'étiquette DPE n'entre pas dans l'ACP : elle sert seulement à colorier les graphiques.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `setdiff()` | Donne les éléments d'un vecteur absents d'un autre. | `x`, `y` | `setdiff(variables, names(dpe))` |
| `colSums(is.na())` | Compte les valeurs manquantes par colonne (voir chapitre 5). | `x` | `colSums(is.na(dpe[, variables]))` |
| `na.omit()` | Retire les lignes qui contiennent une valeur manquante. | `object` | `na.omit(donnees)` |
| `all_of()` | Sélectionne des colonnes dont les noms sont stockés dans un vecteur (dplyr). | `x` | `select(all_of(variables))` |

1. Vérifier que les cinq variables existent dans le fichier, puis compter les valeurs manquantes de chacune.
<details>
<summary>Correction</summary>

```r
variables <- c(
  "surface_habitable_logement",
  "deperditions_murs",
  "besoin_chauffage",
  "conso_5_usages_ep",
  "conso_chauffage_ep"
)

setdiff(variables, names(dpe))   # character(0) : toutes les variables sont présentes
colSums(is.na(dpe[, variables]))
```
</details>

2. Créer `dpe_acp`, qui garde l'étiquette et les cinq variables, sans lignes manquantes. Combien de logements restent ?
<details>
<summary>Correction</summary>

```r
dpe_acp <- dpe |>
  select(etiquette_dpe, all_of(variables)) |>
  na.omit()

nrow(dpe_acp)
```
</details>

## Exercice 2 – Faire une ACP avec FactoMineR

L'**analyse en composantes principales** (ACP) remplace les cinq variables par quelques axes, appelés composantes. Le premier axe capte le plus d'information possible, le deuxième un peu moins, et ainsi de suite. Si quelques axes suffisent à résumer les données, on peut décrire le parc de logements avec peu de chiffres.

Les variables doivent être mesurées dans des unités différentes : on les **réduit** (`scale.unit = TRUE`) pour qu'elles pèsent le même poids.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `PCA()` | Calcule une ACP (FactoMineR). | `X` : les données, `scale.unit` : réduire les variables, `graph` : tracer ou non | `PCA(X, scale.unit = TRUE, graph = FALSE)` |
| `res$eig` | Tableau des valeurs propres, de la part de variance de chaque axe et de la part cumulée. | | `res_acp$eig` |

1. Calculer l'ACP des cinq variables avec `PCA()`, en désactivant les graphiques automatiques, et stocker le résultat dans `res_acp`. Afficher le tableau `res_acp$eig`.
<details>
<summary>Correction</summary>

```r
res_acp <- PCA(dpe_acp[, variables], scale.unit = TRUE, graph = FALSE)

res_acp$eig
```
</details>

## Exercice 3 – Lire l'ACP avec factoextra

**factoextra** transforme les résultats de FactoMineR en graphiques. Trois graphiques suffisent pour lire une ACP :
- le **diagramme des valeurs propres** : la part de variance de chaque axe ;
- le **cercle des variables** : deux variables proches sont fortement liées, et une variable proche du bord est bien représentée par les deux premiers axes ;
- le **nuage des individus** : chaque point est un logement, coloré par étiquette DPE.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `fviz_eig()` | Trace le diagramme des valeurs propres. | `X` : résultat de l'ACP, `addlabels` | `fviz_eig(res_acp, addlabels = TRUE)` |
| `fviz_pca_var()` | Trace le cercle des variables. | `X`, `col.var`, `repel` : éviter le chevauchement des noms | `fviz_pca_var(res_acp, repel = TRUE)` |
| `fviz_pca_ind()` | Trace le nuage des individus. | `X`, `col.ind` : couleur par individu, `label` : `"none"` pour masquer les noms | `fviz_pca_ind(res_acp, col.ind = etiquette, label = "none")` |

1. Tracer le diagramme des valeurs propres, puis le cercle des variables. Quelles variables vont dans le même sens ?
<details>
<summary>Correction</summary>

```r
fviz_eig(res_acp, addlabels = TRUE)
fviz_pca_var(res_acp, repel = TRUE)
# Les variables dont les flèches sont proches dans le même sens sont corrélées positivement.
# Des flèches opposées indiquent une corrélation négative.
```
</details>

2. Tracer le nuage des logements, coloré par étiquette DPE. Les étiquettes `A` et `G` se séparent-elles sur le premier plan ?
<details>
<summary>Correction</summary>

```r
fviz_pca_ind(
  res_acp,
  col.ind = dpe_acp$etiquette_dpe,
  label = "none",
  addEllipses = TRUE
)
# Sur le premier plan, les étiquettes se recouvrent en partie : l'ACP ne résume pas
# l'étiquette, qui dépend de plusieurs variables à la fois.
```
</details>

## Exercice 4 – Regrouper les logements par CAH

La **classification ascendante hiérarchique** (CAH) regroupe les logements qui se ressemblent. Au départ, chaque logement est seul. À chaque étape, on fusionne les deux groupes les plus proches, jusqu'à n'avoir plus qu'un seul groupe. Le dendrogramme montre l'ordre de ces fusions, et on le coupe pour obtenir un nombre de groupes choisi.

La méthode calcule une distance entre tous les couples de logements. Pour 10 000 logements, cela ferait environ 50 millions de distances : on travaille donc sur les 500 premières lignes. Le critère `ward.D2` fusionne les groupes qui augmentent le moins la variance à l'intérieur de chaque groupe.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `scale()` | Réduit les colonnes : moyenne 0, écart-type 1 (voir chapitre 4). | `x` | `scale(donnees)` |
| `dist()` | Calcule les distances entre toutes les lignes. | `x`, `method` : `"euclidean"` par défaut | `dist(donnees_cah)` |
| `hclust()` | Calcule la classification hiérarchique. | `d` : les distances, `method` : `"ward.D2"`, `"complete"`, etc. | `hclust(distances, method = "ward.D2")` |
| `cutree()` | Découpe le dendrogramme en un nombre de groupes `k`. | `tree`, `k` | `cutree(cah, k = 4)` |
| `fviz_dend()` | Trace le dendrogramme (factoextra). | `x`, `k` : groupes à colorier, `show_labels` | `fviz_dend(cah, k = 4, show_labels = FALSE)` |

1. Calculer la CAH sur les 500 premiers logements, réduits, avec le critère `ward.D2`. Tracer le dendrogramme avec quatre groupes coloriés.
<details>
<summary>Correction</summary>

```r
donnees_cah <- scale(dpe_acp[1:500, variables])

distances <- dist(donnees_cah)
cah <- hclust(distances, method = "ward.D2")

fviz_dend(cah, k = 4, show_labels = FALSE)
```
</details>

2. Découper l'arbre en quatre groupes avec `cutree()`, puis comparer les groupes aux étiquettes DPE avec `table()`.
<details>
<summary>Correction</summary>

```r
groupes <- cutree(cah, k = 4)

table(groupes, dpe_acp$etiquette_dpe[1:500])
# Les groupes ne correspondent pas exactement aux étiquettes : la CAH ne voit que
# les cinq variables, pas l'étiquette.
```
</details>

3. Décrire chaque groupe par la moyenne de ses variables. Quel groupe a le plus fort besoin de chauffage ?
<details>
<summary>Correction</summary>

```r
aggregate(
  dpe_acp[1:500, variables],
  by = list(groupe = groupes),
  FUN = mean
)
# Le groupe avec la plus grande moyenne de besoin_chauffage est celui des logements les plus énergivores.
```
</details>

## Exercice 5 – Prédire le besoin de chauffage avec `lm()`

Une **régression linéaire** cherche une droite qui passe au plus près des points. Avec une seule variable explicative, c'est une régression simple. Avec plusieurs, c'est une régression multiple : chaque coefficient mesure l'effet d'une variable, les autres restant fixes.

Le besoin de chauffage est exprimé dans la même unité pour tous les logements, mais un logement de 3300 m² pèse beaucoup sur la droite. On retire donc les surfaces au-dessus de 300 m², pour rester sur des logements courants.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `lm()` | Ajuste une régression linéaire. La formule `y ~ x1 + x2` indique la variable à expliquer (à gauche) et les explicatives (à droite). | `formula`, `data` | `lm(besoin_chauffage ~ surface_habitable_logement, data = dpe_lm)` |
| `summary()` | Affiche les coefficients, leur significativité et le R². | `object` | `summary(modele)` |
| `coef()` | Renvoie les coefficients du modèle. | `object` | `coef(modele)` |
| `predict()` | Calcule les valeurs prédites pour de nouvelles données. | `object`, `newdata` | `predict(modele, newdata = nouveaux)` |

1. Créer `dpe_lm` avec la surface, les déperditions des murs et le besoin de chauffage, sans lignes manquantes, et ne garder que les surfaces inférieures ou égales à 300 m². Ajuster la régression simple `besoin_chauffage ~ surface_habitable_logement`, puis lire la pente.
<details>
<summary>Correction</summary>

```r
dpe_lm <- dpe |>
  select(besoin_chauffage, surface_habitable_logement, deperditions_murs) |>
  na.omit() |>
  filter(surface_habitable_logement <= 300)

modele_simple <- lm(besoin_chauffage ~ surface_habitable_logement, data = dpe_lm)

coef(modele_simple)
# La pente indique de combien le besoin de chauffage augmente pour 1 m² de plus.
```
</details>

2. Ajuster la régression multiple `besoin_chauffage ~ surface_habitable_logement + deperditions_murs`. Comparer les R² des deux modèles avec `summary()`. Le second explique-t-il mieux le besoin de chauffage ?
<details>
<summary>Correction</summary>

```r
modele_multiple <- lm(
  besoin_chauffage ~ surface_habitable_logement + deperditions_murs,
  data = dpe_lm
)

summary(modele_simple)$r.squared
summary(modele_multiple)$r.squared
# Le R² mesure la part de la variation expliquée par le modèle (entre 0 et 1).
# Il augmente toujours quand on ajoute une variable : on regarde aussi si la nouvelle variable est significative.
summary(modele_multiple)
```
</details>

3. Prédire le besoin de chauffage d'un logement de 80 m² avec des déperditions égales à la médiane des données. Puis faire la même prédiction pour un logement de 150 m².
<details>
<summary>Correction</summary>

```r
nouveaux <- data.frame(
  surface_habitable_logement = c(80, 150),
  deperditions_murs = median(dpe_lm$deperditions_murs)
)

predict(modele_multiple, newdata = nouveaux)
# Une prédiction est une estimation moyenne : elle ne dit rien sur un logement précis.
```
</details>

4. Vérifier la qualité du modèle en traçant les résidus, c'est-à-dire l'écart entre la valeur observée et la valeur prédite. Un nuage sans forme nette, centré sur zéro, indique un modèle correct.
<details>
<summary>Correction</summary>

```r
plot(
  fitted(modele_multiple),
  resid(modele_multiple),
  xlab = "Valeur prédite",
  ylab = "Résidu",
  main = "Résidus du modèle multiple",
  pch = 16,
  col = rgb(0, 0, 0, 0.3)
)
abline(h = 0, col = "red")
# Une forme en entonnoir ou en courbe signale un modèle qui manque une part de la relation.
```
</details>

## Liens utiles

- [Documentation de FactoMineR](http://factominer.free.fr/)
- [Documentation de factoextra](https://rpkgs.datanovia.com/factoextra/)
- [Documentation de `hclust()`](https://stat.ethz.ch/R-manual/R-patched/library/stats/html/hclust.html)
- [Documentation de `lm()`](https://stat.ethz.ch/R-manual/R-patched/library/stats/html/lm.html)
- [Données DPE de l'ADEME](https://data.ademe.fr/datasets/dpe03existant)

