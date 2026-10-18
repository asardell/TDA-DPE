# Chapitre 4 – Statistiques descriptives et graphiques

## Objectifs

Voici les objectifs de ce chapitre :
- [ ] résumer une variable quantitative avec `sum()`, `mean()`, `median()`, `min()`, `max()` et `round()`
- [ ] mesurer la dispersion avec `sd()`, `var()` et `quantile()`
- [ ] mesurer le lien entre deux variables avec `cor()` et `cov()`
- [ ] représenter une variable : histogramme (`hist()`), boîte à moustaches (`boxplot()`), diagramme en barres en effectifs puis en pourcentage (`barplot()`)
- [ ] représenter deux variables : boîtes à moustaches par modalité, barres croisées, nuage de points (`plot()`)
- [ ] refaire les graphiques avec ggplot2, puis les rendre interactifs avec plotly

- [Chapitre 4 – Statistiques descriptives et graphiques](#chapitre-4--statistiques-descriptives-et-graphiques)
  - [Objectifs](#objectifs)
  - [Préparation](#préparation)
  - [Exercice 1 – Résumer une variable](#exercice-1--résumer-une-variable)
    - [Mémo](#mémo)
  - [Exercice 2 – Mesurer la dispersion](#exercice-2--mesurer-la-dispersion)
    - [Mémo](#mémo-1)
  - [Exercice 3 – Mesurer un lien entre deux variables](#exercice-3--mesurer-un-lien-entre-deux-variables)
    - [Mémo](#mémo-2)
  - [Partie 1 – Graphiques pour une variable](#partie-1--graphiques-pour-une-variable)
  - [Exercice 4 – Histogramme avec `hist()`](#exercice-4--histogramme-avec-hist)
    - [Mémo](#mémo-3)
  - [Exercice 5 – Boîte à moustaches avec `boxplot()`](#exercice-5--boîte-à-moustaches-avec-boxplot)
    - [Mémo](#mémo-4)
  - [Exercice 6 – Diagramme en barres des effectifs](#exercice-6--diagramme-en-barres-des-effectifs)
    - [Mémo](#mémo-5)
  - [Exercice 7 – Diagramme en barres des pourcentages](#exercice-7--diagramme-en-barres-des-pourcentages)
  - [Partie 2 – Graphiques pour deux variables](#partie-2--graphiques-pour-deux-variables)
  - [Exercice 8 – Boîtes à moustaches par modalité](#exercice-8--boîtes-à-moustaches-par-modalité)
    - [Mémo](#mémo-6)
  - [Exercice 9 – Diagramme en barres croisé](#exercice-9--diagramme-en-barres-croisé)
    - [Mémo](#mémo-7)
  - [Exercice 10 – Nuage de points avec `plot()`](#exercice-10--nuage-de-points-avec-plot)
    - [Mémo](#mémo-8)
  - [Exercice 11 – Les mêmes graphiques avec ggplot2](#exercice-11--les-mêmes-graphiques-avec-ggplot2)
    - [Mémo](#mémo-9)
  - [Exercice 12 – Graphique interactif avec plotly](#exercice-12--graphique-interactif-avec-plotly)
    - [Mémo](#mémo-10)
  - [Liens utiles](#liens-utiles)

## Préparation

On relit le fichier et on isole la surface habitable, qui sert de fil rouge à ce chapitre. Les valeurs manquantes sont fréquentes dans les DPE : la plupart des fonctions de ce chapitre demandent de les ignorer avec `na.rm = TRUE`.

```r
library(readr)

dpe <- read_csv("data/dpe_villeurbanne_10000.csv")
surface <- dpe$surface_habitable_logement

# Une couleur par étiquette DPE, dans l'ordre A à G (même palette que la carte leaflet)
couleurs_dpe <- c("darkgreen", "green", "yellowgreen", "yellow", "orange", "red", "darkred")
```

## Exercice 1 – Résumer une variable

Une statistique descriptive résume une série de valeurs par un seul nombre. La moyenne est sensible aux valeurs extrêmes, la médiane ne l'est pas : comparer les deux est déjà un premier diagnostic.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `sum()` | Somme des valeurs. | `x`, `na.rm` | `sum(surface, na.rm = TRUE)` |
| `mean()` | Moyenne arithmétique. | `x`, `na.rm` | `mean(surface, na.rm = TRUE)` |
| `median()` | Valeur qui partage la série en deux moitiés. | `x`, `na.rm` | `median(surface, na.rm = TRUE)` |
| `min()` / `max()` | Plus petite et plus grande valeur. | `x`, `na.rm` | `max(surface, na.rm = TRUE)` |
| `round()` | Arrondit à un nombre de décimales. | `x`, `digits` | `round(mean(surface, na.rm = TRUE), 1)` |
| `summary()` | Affiche plusieurs indicateurs d'un coup. | `object` | `summary(surface)` |

1. Compter le nombre de logements dont la surface est renseignée, avec `sum()` et `!is.na()`.
<details>
<summary>Correction</summary>

```r
sum(!is.na(surface))
```
</details>

2. Calculer la moyenne et la médiane de la surface, arrondies à 1 décimale. Laquelle est la plus élevée ?
<details>
<summary>Correction</summary>

```r
moyenne <- round(mean(surface, na.rm = TRUE), 1)
mediane <- round(median(surface, na.rm = TRUE), 1)
moyenne
mediane
# La moyenne est plus élevée que la médiane : quelques très grandes surfaces
# tirent la moyenne vers le haut.
```
</details>

3. Afficher la plus petite et la plus grande surface, puis tous les indicateurs avec `summary()`.
<details>
<summary>Correction</summary>

```r
min(surface, na.rm = TRUE)
max(surface, na.rm = TRUE)
summary(surface)
```
</details>

## Exercice 2 – Mesurer la dispersion

La moyenne seule ne dit pas si les valeurs sont proches les unes des autres. L'écart-type (`sd()`) et la variance (`var()`) mesurent la dispersion autour de la moyenne. Les quantiles découpent la série en parts de même effectif.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `sd()` | Écart-type. | `x`, `na.rm` | `sd(surface, na.rm = TRUE)` |
| `var()` | Variance, carré de l'écart-type. | `x`, `na.rm` | `var(surface, na.rm = TRUE)` |
| `quantile()` | Valeurs qui découpent la série selon des probabilités. | `x`, `probs`, `na.rm` | `quantile(surface, probs = c(0.1, 0.9), na.rm = TRUE)` |

1. Calculer l'écart-type et la variance de la surface, arrondis à 1 décimale.
<details>
<summary>Correction</summary>

```r
round(sd(surface, na.rm = TRUE), 1)
round(var(surface, na.rm = TRUE), 1)
# La variance est égale à l'écart-type au carré.
```
</details>

2. Calculer les déciles 1, 5 et 9 (`probs = c(0.1, 0.5, 0.9)`). Que représente la valeur pour `0.5` ?
<details>
<summary>Correction</summary>

```r
quantile(surface, probs = c(0.1, 0.5, 0.9), na.rm = TRUE)
# La valeur pour 0.5 est la médiane.
```
</details>

3. Calculer le coefficient de variation, égal à l'écart-type divisé par la moyenne. Il permet de comparer des dispersions entre variables d'échelles différentes.
<details>
<summary>Correction</summary>

```r
cv <- sd(surface, na.rm = TRUE) / mean(surface, na.rm = TRUE)
round(cv, 2)
```
</details>

## Exercice 3 – Mesurer un lien entre deux variables

La **covariance** (`cov()`) et le **coefficient de corrélation** (`cor()`) mesurent le lien entre deux variables quantitatives. La corrélation est comprise entre -1 et 1 : elle ne dépend pas de l'unité, contrairement à la covariance.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `cov()` | Covariance entre deux variables. | `x`, `y`, `use` | `cov(x, y, use = "complete.obs")` |
| `cor()` | Corrélation de Pearson, entre -1 et 1. | `x`, `y`, `use`, `method` | `cor(x, y, use = "complete.obs")` |

`use = "complete.obs"` ignore les lignes où l'une des deux valeurs est manquante.

1. Calculer la covariance entre `surface_habitable_logement` et `conso_5_usages_ep`.
<details>
<summary>Correction</summary>

```r
cov(dpe$surface_habitable_logement, dpe$conso_5_usages_ep, use = "complete.obs")
```
</details>

2. Calculer la corrélation entre les mêmes variables, arrondie à 2 décimales. Le lien est-il fort, faible, positif ou négatif ?
<details>
<summary>Correction</summary>

```r
round(
  cor(dpe$surface_habitable_logement, dpe$conso_5_usages_ep, use = "complete.obs"),
  2
)
# Une corrélation proche de 0 indique un lien faible.
```
</details>

3. Calculer la corrélation de la surface avec elle-même. Que vaut-elle ? Pourquoi ce résultat est-il logique ?
<details>
<summary>Correction</summary>

```r
cor(surface, surface, use = "complete.obs")
# [1] 1 : une variable est toujours parfaitement corrélée à elle-même.
```
</details>

## Partie 1 – Graphiques pour une variable

Un graphique pour une variable montre sa distribution. Pour une variable quantitative, on utilise l'histogramme et la boîte à moustaches. Pour une variable qualitative, on utilise le diagramme en barres, en effectifs ou en pourcentages.

## Exercice 4 – Histogramme avec `hist()`

Un histogramme découpe les valeurs en intervalles (les classes) et compte les effectifs de chacun. Il montre la forme de la distribution : où se concentrent les valeurs, et si la série est dissymétrique.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `hist()` | Trace un histogramme. | `x`, `breaks` : nombre de classes, `main` : titre, `xlab` / `ylab` : axes, `col` : couleur | `hist(surface, breaks = 30, col = "lightblue")` |
| `abline()` | Ajoute une droite verticale (`v`) ou horizontale (`h`) sur un graphique existant. | `v`, `h`, `col`, `lwd`, `lty` | `abline(v = 100, col = "red")` |

1. Tracer l'histogramme de `surface` avec 30 classes, un titre et des libellés d'axes.
<details>
<summary>Correction</summary>

```r
hist(
  surface,
  breaks = 30,
  main = "Surface habitable des logements de Villeurbanne",
  xlab = "Surface habitable (m²)",
  ylab = "Nombre de logements",
  col = "lightblue"
)
```
</details>

2. Ajouter sur le même graphique la moyenne (trait rouge plein) et la médiane (trait vert en pointillés). Laquelle est à droite de l'autre ?
<details>
<summary>Correction</summary>

```r
hist(surface, breaks = 30, col = "lightblue", main = "Surface habitable", xlab = "m²")
abline(v = mean(surface, na.rm = TRUE), col = "red", lwd = 2)
abline(v = median(surface, na.rm = TRUE), col = "darkgreen", lwd = 2, lty = 2)
# La moyenne est à droite de la médiane : la distribution est étirée vers les grandes surfaces.
# La barre isolée très loin à droite correspond à la valeur d'environ 3300 m², traitée au chapitre 5.
```
</details>

## Exercice 5 – Boîte à moustaches avec `boxplot()`

Une boîte à moustaches résume une série par ses quartiles : la boîte va du premier au troisième quartile, le trait au milieu est la médiane, et les points isolés sont des valeurs atypiques.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `boxplot()` | Trace une boîte à moustaches pour une variable. | `x`, `main`, `ylab`, `col` | `boxplot(surface, col = "lightblue")` |

1. Tracer la boîte à moustaches de `surface`, avec un titre et un libellé d'axe.
<details>
<summary>Correction</summary>

```r
boxplot(
  surface,
  main = "Surface habitable des logements",
  ylab = "Surface habitable (m²)",
  col = "lightblue"
)
# Les points isolés en haut sont les valeurs atypiques, dont celle d'environ 3300 m².
```
</details>

## Exercice 6 – Diagramme en barres des effectifs

Pour une variable qualitative, on représente les effectifs par des barres. `barplot()` prend un tableau d'effectifs produit par `table()`.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `table()` | Compte les effectifs de chaque modalité. | `...` : une ou deux variables | `table(dpe$etiquette_dpe)` |
| `barplot()` | Trace des barres à partir de valeurs ou d'un tableau. | `height` : effectifs, `main`, `xlab` / `ylab`, `col` | `barplot(table(dpe$etiquette_dpe))` |

1. Tracer le diagramme en barres des effectifs par étiquette, dans un tableau `effectifs`.
<details>
<summary>Correction</summary>

```r
effectifs <- table(dpe$etiquette_dpe)

barplot(
  effectifs,
  main = "Nombre de DPE par étiquette",
  xlab = "Étiquette DPE",
  ylab = "Nombre de DPE",
  col = couleurs_dpe
)
```
</details>

## Exercice 7 – Diagramme en barres des pourcentages

Les effectifs ne sont pas comparables entre groupes de tailles différentes : on passe en pourcentages. `prop.table()` transforme un tableau d'effectifs en proportions, dont la somme vaut 1.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `prop.table()` | Transforme un tableau d'effectifs en proportions. | `x`, `margin` (1 ou 2 pour un tableau croisé) | `prop.table(effectifs)` |

1. Tracer le diagramme en barres des pourcentages par étiquette : multiplier les proportions par 100 et ajouter un titre `"Part des DPE par étiquette (%)"`. Quelle étiquette est la plus représentée ?
<details>
<summary>Correction</summary>

```r
parts <- round(prop.table(effectifs) * 100, 1)
barplot(
  parts,
  main = "Part des DPE par étiquette (%)",
  xlab = "Étiquette DPE",
  ylab = "Part (%)",
  col = couleurs_dpe
)
names(which.max(parts))  # l'étiquette la plus représentée
```
</details>

## Partie 2 – Graphiques pour deux variables

Un graphique pour deux variables montre leur lien. On croise une variable qualitative et une variable quantitative (boîtes par modalité), ou deux variables qualitatives (barres croisées), ou deux variables quantitatives (nuage de points).

## Exercice 8 – Boîtes à moustaches par modalité

Avec la formule `y ~ x`, `boxplot()` trace une boîte par modalité de `x`. On compare ainsi la distribution de `y` entre les groupes.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `boxplot()` avec une formule | Une boîte par modalité de la variable qualitative. | `formula` : `y ~ x`, `data`, `main`, `xlab` / `ylab`, `col`, `outline` : `FALSE` masque les points atypiques | `boxplot(conso_5_usages_ep ~ etiquette_dpe, data = dpe)` |

1. Tracer la boîte à moustaches de `conso_5_usages_ep` pour chaque étiquette `A` à `G`.
<details>
<summary>Correction</summary>

```r
boxplot(
  conso_5_usages_ep ~ etiquette_dpe,
  data = dpe,
  main = "Consommation par étiquette DPE",
  xlab = "Étiquette DPE",
  ylab = "Consommation (kWh/m²/an)",
  col = couleurs_dpe
)
```
</details>

2. Même graphique sans les points atypiques mais les moustaches restent calculées sur toutes les données.
<details>
<summary>Correction</summary>

```r
boxplot(
  conso_5_usages_ep ~ etiquette_dpe,
  data = dpe,
  outline = FALSE,
  main = "Consommation par étiquette DPE (sans points atypiques)",
  xlab = "Étiquette DPE",
  ylab = "Consommation (kWh/m²/an)",
  col = couleurs_dpe
)
```
</details>

## Exercice 9 – Diagramme en barres croisé

Un tableau croisé compte les effectifs pour chaque couple de modalités. Ici, on croise le type d'énergie (`type_energie_n1`) et l'étiquette DPE.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `table()` à deux variables | Tableau croisé des effectifs. | `x`, `y` | `table(dpe$type_energie_n1, dpe$etiquette_dpe)` |
| `barplot()` sur un tableau croisé | Une barre par colonne. Les segments de chaque barre sont les lignes du tableau. `beside = TRUE` place les barres côte à côte ; sans cet argument, elles sont empilées. | `beside`, `legend.text`, `col` : une couleur par ligne | `barplot(tab, col = couleurs_dpe, legend.text = TRUE)` |
| `las` | Oriente les étiquettes des axes : `2` les rend perpendiculaires à l'axe. | `las` : 0 à 3 | `barplot(tab, las = 2)` |
| `par()` | Règle la mise en page : `mar` marges (dans l'ordre bas, gauche, haut, droite), `mfrow` nombre de graphiques par fenêtre. À rétablir après usage. | `mar`, `mfrow` | `par(mar = c(10, 4, 4, 9))` en barres verticales ; `par(mar = c(5, 10, 4, 9))` avec `horiz = TRUE` |

1. Construire le tableau croisé `tab_energie` entre `etiquette_dpe` (en lignes) et `type_energie_n1` (en colonnes). Tracer le diagramme en barres empilées en effectifs : une barre par type d'énergie, coupée par étiquette, avec la couleur de chaque étiquette et une légende. Pivoter les noms de l'axe x avec `las = 2`.
<details>
<summary>Correction</summary>

```r
tab_energie <- table(dpe$etiquette_dpe, dpe$type_energie_n1)

par(mar = c(5, 13, 4, 9))   # marges : place pour les noms à gauche (horiz = TRUE) et la légende à droite

barplot(
  tab_energie,
  col = couleurs_dpe,
  legend.text = TRUE,
  args.legend = list(x = "topright", inset = c(-0.2, 0), bty = "n", xpd = TRUE),
  las = 2,                 # noms de l'axe perpendiculaires à l'axe
  cex.names = 0.8,         # noms un peu plus petits pour qu'ils tiennent dans la marge
  main = "Effectifs par type d'énergie et étiquette DPE",
  xlab = "Nombre de DPE",  # avec horiz = TRUE, la valeur est sur l'axe horizontal
  horiz = TRUE
)

par(mar = c(5, 4, 4, 2) + 0.1)   # rétablir les marges par défaut
```
</details>

2. Même graphique en pourcentages
<details>
<summary>Correction</summary>

```r
par(mar = c(5, 13, 4, 9))   # marges : place pour les noms à gauche (horiz = TRUE) et la légende à droite

barplot(
  prop.table(tab_energie, margin = 2) * 100,
  col = couleurs_dpe,
  legend.text = TRUE,
  args.legend = list(x = "topright", inset = c(-0.2, 0), bty = "n", xpd = TRUE),
  las = 2,                 # noms de l'axe perpendiculaires à l'axe
  cex.names = 0.8,         # noms un peu plus petits pour qu'ils tiennent dans la marge
  main = "Part de chaque étiquette par type d'énergie (%)",
  xlab = "Part (%)",       # la valeur est sur l'axe horizontal avec horiz = TRUE
  horiz = TRUE
)

par(mar = c(5, 4, 4, 2) + 0.1)   # rétablir les marges par défaut
```
</details>

## Exercice 10 – Nuage de points avec `plot()`

Un nuage de points place chaque logement selon sa surface (en abscisse) et sa consommation (en ordonnée). La corrélation de l'exercice 3 se lit dans la forme du nuage.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `plot()` | Trace un nuage de points entre deux variables. | `x`, `y`, `pch` : forme des points, `col`, `xlim` : bornes de l'axe x | `plot(x, y, pch = 16)` |
| `rgb()` | Crée une couleur ; le 4e argument est la transparence. | `red`, `green`, `blue`, `alpha` (entre 0 et 1) | `rgb(0, 0, 0, 0.3)` |

1. Tracer le nuage de points entre `surface` et `conso_5_usages_ep`, avec des points semi-transparents, un titre contenant la corrélation arrondie à 2 décimales.
<details>
<summary>Correction</summary>

```r
plot(
  dpe$surface_habitable_logement,
  dpe$conso_5_usages_ep,
  pch = 16,
  col = rgb(0, 0, 0, 0.3),
  xlab = "Surface habitable (m²)",
  ylab = "Consommation (kWh/m²/an)",
  main = paste(
    "Corrélation :",
    round(cor(dpe$surface_habitable_logement, dpe$conso_5_usages_ep, use = "complete.obs"), 2)
  )
)
```
</details>

1. Le nuage est écrasé par les valeurs atypiques.
<details>
<summary>Correction</summary>

```r
plot(
  dpe$surface_habitable_logement,
  dpe$conso_5_usages_ep,
  pch = 16,
  col = rgb(0, 0, 0, 0.3),
  xlim = c(0, 200),
  ylim = c(0, 50000),
  xlab = "Surface habitable (m²)",
  ylab = "Consommation (kWh/m²/an)",
  main = "Surface et consommation (surfaces jusqu'à 200 m²)"
)
```
</details>

## Exercice 11 – Les mêmes graphiques avec ggplot2

ggplot2 construit un graphique par couches : on indique les données et les variables avec `aes()`, puis on ajoute une forme géométrique avec `geom_*()`. Les graphiques se combinent avec `+`. Le package `esquisse` permet aussi de construire un graphique à la souris, et de récupérer le code généré (`esquisse::esquisser(dpe)`).

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `ggplot()` | Initialise le graphique avec les données. | `data`, `mapping` (via `aes()`) | `ggplot(dpe, aes(x = surface_habitable_logement))` |
| `aes()` | Associe les variables aux axes, aux couleurs, etc. | `x`, `y`, `fill`, `colour` | `aes(x = etiquette_dpe, y = conso_5_usages_ep)` |
| `geom_histogram()` | Ajoute un histogramme. | `bins`, `fill`, `colour` | `geom_histogram(bins = 30)` |
| `geom_boxplot()` | Ajoute des boîtes à moustaches. | `fill`, `outlier.shape` : `NA` masque les points atypiques | `geom_boxplot(outlier.shape = NA)` |
| `geom_point()` | Ajoute des points (nuage de points). | `alpha`, `colour`, `size` | `geom_point(alpha = 0.3)` |
| `geom_bar()` | Ajoute des barres qui comptent les effectifs. `position = "fill"` ramène chaque barre à 100 %. | `position`, `fill` | `geom_bar(position = "fill")` |
| `labs()` | Ajoute titre et libellés. | `title`, `x`, `y` | `labs(title = "Titre", x = "Axe x")` |
| `coord_cartesian()` | Zoome sur une plage d'axe sans retirer de données (les statistiques des boîtes sont calculées sur toutes les données). | `xlim`, `ylim` | `coord_cartesian(ylim = c(0, 50000))` |

1. Refaire l'histogramme de l'exercice 4 avec `geom_histogram()`, avec une ligne verticale pour la moyenne (`geom_vline()`).
<details>
<summary>Correction</summary>

```r
#install.packages("ggplot2")
library(ggplot2)

ggplot(dpe, aes(x = surface_habitable_logement)) +
  geom_histogram(bins = 30, fill = "lightblue", colour = "white") +
  geom_vline(xintercept = mean(surface, na.rm = TRUE), colour = "red") +
  labs(
    title = "Surface habitable des logements de Villeurbanne",
    x = "Surface habitable (m²)",
    y = "Nombre de logements"
  )
```
</details>

2. Refaire la boîte à moustaches par étiquette de l'exercice 8 avec `geom_boxplot()`.
<details>
<summary>Correction</summary>

```r
# outlier.shape = NA masque les points atypiques ; les moustaches restent calculées sur toutes les données
ggplot(dpe, aes(x = etiquette_dpe, y = conso_5_usages_ep)) +
  geom_boxplot(aes(fill = etiquette_dpe), outlier.shape = NA) +
  scale_fill_manual(values = couleurs_dpe, guide = "none") +
  coord_cartesian(ylim = c(0, 50000)) +   # zoom sur l'axe y sans retirer de données
  labs(
    title = "Consommation par étiquette DPE (sans points atypiques)",
    x = "Étiquette DPE",
    y = "Consommation (kWh/m²/an)"
  )
```
</details>

3. Refaire le nuage de points de l'exercice 10 avec `geom_point()`, et l'enregistrer dans un objet `p_nuage` (il servira à l'exercice 12).
<details>
<summary>Correction</summary>

```r
p_nuage <- ggplot(dpe, aes(x = surface_habitable_logement, y = conso_5_usages_ep)) +
  geom_point(alpha = 0.3) +
  coord_cartesian(xlim = c(0, 200), ylim = c(0, 50000)) +   # zoom : une partie des valeurs extrêmes sort du cadre
  labs(
    title = "Surface et consommation",
    x = "Surface habitable (m²)",
    y = "Consommation (kWh/m²/an)"
  )

p_nuage
```
</details>

4. Refaire le diagramme en barres croisé de l'exercice 9 en pourcentages avec `geom_bar(position = "fill")`, en mettant `etiquette_dpe` en abscisse et `type_energie_n1` en couleur (`fill`). Puis ouvrir `esquisse` sur `dpe` et comparer le code généré.
<details>
<summary>Correction</summary>

```r
ggplot(dpe, aes(x = etiquette_dpe, fill = type_energie_n1)) +
  geom_bar(position = "fill") +
  labs(
    title = "Part de chaque type d'énergie par étiquette DPE",
    x = "Étiquette DPE",
    y = "Part",
    fill = "Type d'énergie"
  )
```
</details>

5. Interface graphique avec `esquisse`

```r
#install.packages("esquisse")
library(esquisse)
esquisse::esquisser(dpe)
```

## Exercice 12 – Graphique interactif avec plotly

Un graphique **plotly** est interactif : au survol, il affiche la valeur exacte, et l'on peut zoomer ou masquer une série. Il s'enregistre aussi en page HTML, pratique pour partager un résultat. `ggplotly()` transforme un graphique ggplot2 existant, et `plot_ly()` construit directement un graphique plotly.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `ggplotly()` | Transforme un graphique ggplot2 en graphique interactif. | `p` : le graphique ggplot2 | `ggplotly(p)` |
| `plot_ly()` | Crée un graphique plotly. | `data`, `x`, `y`, `type` : `"scatter"`, `"histogram"`, `"box"`, `"bar"` | `plot_ly(dpe, x = ~surface_habitable_logement, type = "histogram")` |
| `~` | Désigne une colonne du dataframe dans `plot_ly()`. | | `~surface_habitable_logement` |
| `htmlwidgets::saveWidget()` | Enregistre un graphique plotly en page HTML. | `widget`, `file` | `saveWidget(p, "graphique.html")` |

1. Installer le package une seule fois avec `install.packages("plotly")`, puis le charger. Transformer `p_nuage` (exercice 11.3) avec `ggplotly()`. Survoler un point : quelles valeurs apparaissent ?
<details>
<summary>Correction</summary>

```r
# install.packages("plotly")  # une seule fois
library(plotly)

ggplotly(p_nuage)
# Au survol : la surface et la consommation du logement.
```
</details>

2. Construire directement avec `plot_ly()` un histogramme interactif de `surface_habitable_logement`, avec 30 classes. Zoomer sur la partie gauche du graphique : que se passe-t-il ?
<details>
<summary>Correction</summary>

```r
plot_ly(
  dpe,
  x = ~surface_habitable_logement,
  type = "histogram",
  nbinsx = 30
) |>
  layout(
    title = "Surface habitable des logements de Villeurbanne",
    xaxis = list(title = "Surface habitable (m²)"),
    yaxis = list(title = "Nombre de logements")
  )
# Le zoom ne recalcule pas les classes : on zoome sur les barres existantes.
```
</details>

3. Enregistrer l'histogramme interactif en page HTML avec `htmlwidgets::saveWidget()`, sous le nom `histogramme_surface.html`.
<details>
<summary>Correction</summary>

```r
histo <- plot_ly(dpe, x = ~surface_habitable_logement, type = "histogram", nbinsx = 30)
htmlwidgets::saveWidget(histo, "histogramme_surface.html")
```
</details>

## Exercice 13 – Pour aller plus loin : une carte avec leaflet

Cet exercice est facultatif. **leaflet** affiche des points sur un fond de carte interactif : on peut zoomer, se déplacer et cliquer sur un point. Il utilise les coordonnées GPS obtenues par la jointure de l'exercice 8.4 du chapitre 3 (`dpe_gps`). Si cet objet n'existe plus dans ta session, relance le code de cet exercice.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `leaflet()` | Initialise la carte avec les données. | `data` | `leaflet(dpe_carte)` |
| `addProviderTiles()` | Ajoute un fond de carte. | `provider` : `providers$OpenStreetMap` (sans clé d'API) | `addProviderTiles(providers$OpenStreetMap)` |
| `addCircleMarkers()` | Ajoute un point rond par ligne. | `lng`, `lat`, `radius`, `color`, `popup`, `fillOpacity`, `stroke` | `addCircleMarkers(lng = ~lon, lat = ~lat, radius = 3)` |
| `colorFactor()` | Crée une fonction qui associe une couleur à chaque modalité. | `palette`, `domain` | `colorFactor(c("green", "red"), domain = c("A", "G"))` |
| `addLegend()` | Ajoute une légende. | `pal`, `values`, `title` | `addLegend(pal = pal, values = ~etiquette_dpe)` |

1. Installer leaflet une seule fois, puis garder les DPE qui ont des coordonnées dans `dpe_carte`, et afficher un point par DPE sur un fond OpenStreetMap.
<details>
<summary>Correction</summary>

```r
# install.packages("leaflet")  # une seule fois
library(leaflet)

dpe_carte <- dpe_gps |>
  filter(!is.na(lon), !is.na(lat))

leaflet(dpe_carte) |>
  addProviderTiles(providers$OpenStreetMap) |>
  addCircleMarkers(lng = ~lon, lat = ~lat, radius = 3, stroke = FALSE, fillOpacity = 0.6)
# Plusieurs DPE partagent la même adresse : leurs points se superposent.
```
</details>

2. Colorer les points selon l'étiquette DPE avec `colorFactor()`, ajouter une légende, et afficher au clic l'étiquette et la surface.
<details>
<summary>Correction</summary>

```r
pal <- colorFactor(
  palette = c("darkgreen", "green", "yellowgreen", "yellow", "orange", "red", "darkred"),
  domain = c("A", "B", "C", "D", "E", "F", "G")
)

leaflet(dpe_carte) |>
  addProviderTiles(providers$OpenStreetMap) |>
  addCircleMarkers(
    lng = ~lon,
    lat = ~lat,
    radius = 3,
    stroke = FALSE,
    fillOpacity = 0.7,
    color = ~pal(etiquette_dpe),
    popup = ~paste0("Étiquette : ", etiquette_dpe, "<br>Surface : ", surface_habitable_logement, " m²")
  ) |>
  addLegend(pal = pal, values = ~etiquette_dpe, title = "Étiquette DPE")
```
</details>

3. Ne garder que les passoires (étiquettes `E`, `F`, `G`) et compter les points restants. Où sont-elles concentrées sur la carte ?
<details>
<summary>Correction</summary>

```r
passoires <- dpe_carte |>
  filter(etiquette_dpe %in% c("E", "F", "G"))

nrow(passoires)

leaflet(passoires) |>
  addProviderTiles(providers$OpenStreetMap) |>
  addCircleMarkers(
    lng = ~lon,
    lat = ~lat,
    radius = 3,
    stroke = FALSE,
    fillOpacity = 0.6,
    color = "red"
  )
# Si les points se superposent trop, ajouter clusterOptions = markerClusterOptions() dans addCircleMarkers().
```
</details>

## Liens utiles

- [Documentation de leaflet pour R](https://rstudio.github.io/leaflet/)
- [Base Adresse Nationale (téléchargement par département)](https://adresse.data.gouv.fr/data/ban/adresses/latest/csv-with-ids)
- [Documentation de ggplot2](https://ggplot2.tidyverse.org/)
- [Documentation de plotly pour R](https://plotly.com/r/)
- [Cheatsheet ggplot2](https://posit.co/resources/cheatsheets/)
- [Données DPE de l'ADEME](https://data.ademe.fr/datasets/dpe03existant)
