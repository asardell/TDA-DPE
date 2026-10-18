# Chapitre 8 – Automatisation sous Windows (projet de fin)

## Objectifs

Voici les objectifs de ce chapitre :
- [ ] transformer un script interactif en script autonome, lançable sans RStudio
- [ ] retrouver le programme `Rscript.exe` de R avec `R.home()`
- [ ] écrire un journal d'exécution avec la date et l'heure de chaque événement
- [ ] intercepter une erreur avec `tryCatch()` pour que le script ne s'arrête pas en silence
- [ ] interroger l'API ADEME avec un filtre sur la date de réception, et mettre à jour la base sans doublons
- [ ] planifier l'exécution hebdomadaire du script avec le Planificateur de tâches Windows

- [Chapitre 8 – Automatisation sous Windows (projet de fin)](#chapitre-8--automatisation-sous-windows-projet-de-fin)
  - [Objectifs](#objectifs)
  - [Préparation](#préparation)
  - [Exercice 1 – Créer un script autonome](#exercice-1--créer-un-script-autonome)
    - [Mémo](#mémo)
  - [Exercice 2 – Lancer le script avec `Rscript.exe`](#exercice-2--lancer-le-script-avec-rscriptexe)
    - [Mémo](#mémo-1)
  - [Exercice 3 – Écrire un journal](#exercice-3--écrire-un-journal)
    - [Mémo](#mémo-2)
  - [Exercice 4 – Gérer les erreurs](#exercice-4--gérer-les-erreurs)
    - [Mémo](#mémo-3)
  - [Exercice 5 – Interroger l'API sur les 7 derniers jours](#exercice-5--interroger-lapi-sur-les-7-derniers-jours)
    - [Mémo](#mémo-4)
  - [Exercice 6 – Planifier l'exécution hebdomadaire](#exercice-6--planifier-lexécution-hebdomadaire)
    - [Mémo](#mémo-5)
  - [Projet de fin](#projet-de-fin)
  - [Liens utiles](#liens-utiles)

## Préparation

Une tâche planifiée ne voit ni RStudio ni la session ouverte : elle lance R en arrière-plan, avec le même compte Windows, et lit le script du début à la fin. Le script doit donc être **autonome** : pas de `View()`, pas de graphique, pas de dépendance à un objet resté en mémoire.

Le projet de fin reprend le chapitre 7 : la fonction `lire_page()` sert à interroger l'API, et la table `dpe_api` de la base SQLite reçoit les nouveaux DPE chaque semaine.

On crée le dossier `scripts/` (versionné sur GitHub) et le dossier `logs/` (non versionné, comme `data/`).

```r
dir.create("scripts", showWarnings = FALSE)
dir.create("logs", showWarnings = FALSE)
```

## Exercice 1 – Créer un script autonome

Un script autonome commence par se placer dans le bon dossier, charge ses packages, puis fait son travail. Le dossier de travail doit être fixé dans le script, car la tâche planifiée ne l'hérite pas de RStudio (voir le chapitre 2).

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `setwd()` | Fixe le dossier de travail du script. | `dir` : racine du dépôt | `setwd("C:/Users/Anthony/Documents/GitHub/TDA-DPE")` |
| `suppressPackageStartupMessages()` | Masque les messages d'accueil des packages au lancement. | `expr` | `suppressPackageStartupMessages(library(DBI))` |
| `invisible()` | Évite d'afficher une valeur renvoyée. | `x` | `invisible(dbDisconnect(con))` |

1. Créer `scripts/mise_a_jour_dpe.R` avec l'en-tête suivant : le dossier de travail fixé, les packages chargés sans messages, et le chemin de la base défini dans une variable `base_sqlite`. Ce script ne fait rien d'autre pour l'instant.
<details>
<summary>Correction</summary>

```r
# scripts/mise_a_jour_dpe.R
# Met à jour la base SQLite des DPE de Villeurbanne à partir de l'API ADEME.

racine <- "C:/Users/Anthony/Documents/GitHub/TDA-DPE"   # à adapter sur chaque poste
setwd(racine)

suppressPackageStartupMessages({
  library(DBI)
  library(RSQLite)
  library(httr2)
  library(jsonlite)
  library(dplyr)
})

base_sqlite <- "data/dpe_villeurbanne.sqlite"

cat("Script chargé dans", getwd(), "\n")
```
</details>

2. Lancer le script depuis RStudio avec `source("scripts/mise_a_jour_dpe.R")`. Puis vérifier qu'il s'exécute sans rien laisser de la session : relancer R, et l'exécuter une seconde fois.
<details>
<summary>Correction</summary>

```r
source("scripts/mise_a_jour_dpe.R")
# Dans une nouvelle session, le script doit fonctionner sans rien d'autre.
```
</details>

## Exercice 2 – Lancer le script avec `Rscript.exe`

Le Planificateur de tâches n'ouvre pas RStudio : il appelle `Rscript.exe`, le programme qui exécute un script R sans interface. Son emplacement dépend de la version de R installée. `R.home()` renvoie le dossier d'installation de R, et son sous-dossier `bin` contient `Rscript.exe`.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `R.home()` | Renvoie le dossier d'installation de R. | `component` : `"bin"` pour le dossier des programmes | `R.home("bin")` |
| `file.exists()` | Teste si un fichier existe. | `file` | `file.exists(file.path(R.home("bin"), "Rscript.exe"))` |
| `file.path()` | Assemble des morceaux de chemin avec le bon séparateur. | `...` | `file.path(R.home("bin"), "Rscript.exe")` |

1. Trouver le chemin complet de `Rscript.exe` et vérifier qu'il existe.
<details>
<summary>Correction</summary>

```r
rscript <- file.path(R.home("bin"), "Rscript.exe")
rscript
file.exists(rscript)   # [1] TRUE
```
</details>

2. Lancer le script depuis un terminal (PowerShell), en utilisant le chemin trouvé. L'option `--vanilla` ignore les préférences et les fichiers `.Rprofile` de la session, pour que le lancement soit identique sur chaque poste.
<details>
<summary>Correction</summary>

```powershell
& "C:\Program Files\R\R-4.4.1\bin\Rscript.exe" --vanilla "C:\Users\Anthony\Documents\GitHub\TDA-DPE\scripts\mise_a_jour_dpe.R"
```

Remplacer le chemin de `Rscript.exe` par celui obtenu à la question 1 (la version de R peut différer). Le message « Script chargé dans ... » doit s'afficher.

:warning: le `&` est nécessaire pour *Windows PowerShell*
</details>

## Exercice 3 – Écrire un journal

Quand le script tourne la nuit, personne ne voit ce qui s'est passé. Un journal (*log*) garde une ligne par événement, avec la date et l'heure. On l'écrit dans `logs/mise_a_jour.log` avec `cat(..., append = TRUE)`, pour ajouter à la suite sans effacer les exécutions précédentes.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `Sys.time()` | Retourne la date et l'heure du moment (voir chapitre 5). | | `Sys.time()` |
| `format()` | Met la date et l'heure en texte. | `x`, `format` | `format(Sys.time(), "%Y-%m-%d %H:%M:%S")` |
| `cat()` | Affiche ou écrit du texte. `file` : fichier de sortie, `append = TRUE` : ajoute à la fin. | `...`, `file`, `append` | `cat("texte\n", file = "logs/x.log", append = TRUE)` |
| `dir.create()` | Crée un dossier. `showWarnings = FALSE` : pas de message si le dossier existe déjà. | `path`, `showWarnings` | `dir.create("logs", showWarnings = FALSE)` |

1. Écrire la fonction `journaliser(message)` dans `mise_a_jour_dpe.R`: elle ajoute au fichier `logs/mise_a_jour.log` une ligne contenant l'heure et le message, et l'affiche aussi à l'écran. Ajouter les appels au début et à la fin du script.
<details>
<summary>Correction</summary>

```r
journaliser <- function(message, fichier = "logs/mise_a_jour.log") {
  dir.create(dirname(fichier), showWarnings = FALSE)
  ligne <- paste(format(Sys.time(), "%Y-%m-%d %H:%M:%S"), "-", message)
  cat(ligne, "\n", file = fichier, append = TRUE)
  cat(ligne, "\n")
  invisible(ligne)
}

journaliser("Début de la mise à jour")
# ... le travail du script ...
journaliser("Fin de la mise à jour")
```
</details>

2. Lancer le script deux fois, puis ouvrir `logs/mise_a_jour.log` : les deux exécutions doivent apparaître l'une après l'autre.
<details>
<summary>Correction</summary>

```r
source("scripts/mise_a_jour_dpe.R")
source("scripts/mise_a_jour_dpe.R")
readLines("logs/mise_a_jour.log")
```
</details>

## Exercice 4 – Gérer les erreurs

Si l'API ne répond pas, un script sans gestion d'erreur s'arrête, et le journal ne dit rien. `tryCatch()` exécute un bloc de code et capture l'erreur : on peut alors la journaliser, puis décider de la suite. Le code de sortie du script doit aussi signaler l'échec, pour que le Planificateur le voie.

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `tryCatch()` | Exécute une expression et capte les erreurs. | `expr`, `error` : fonction appelée en cas d'erreur | `tryCatch(log(-1), error = function(e) "échec")` |
| `conditionMessage()` | Donne le message d'une erreur capturée. | `c` : l'erreur | `conditionMessage(e)` |
| `stop()` | Déclenche une erreur avec un message. | `...` : le message | `stop("API indisponible")` |
| `quit()` | Arrête R avec un code de sortie. `status = 1` signale un échec. | `save`, `status` | `quit(save = "no", status = 1)` |

1. Entourer le travail principal du script avec `tryCatch()`. En cas d'erreur, journaliser le message puis arrêter R avec `status = 1`. Tester en remplaçant temporairement l'adresse de l'API par une adresse fausse.
<details>
<summary>Correction</summary>

```r
resultat <- tryCatch(
  {
    # ... le travail du script (exercice 5) ...
    journaliser("Mise à jour réussie")
    0
  },
  error = function(e) {
    journaliser(paste("Erreur :", conditionMessage(e)))
    1
  }
)

quit(save = "no", status = resultat)
```

Pour tester, mettre une adresse inexistante dans `url_api` : le journal doit contenir une ligne `Erreur : ...` et le script doit se terminer avec le code 1.
</details>

2. Ajouter une vérification avant l'appel : si le fichier de la base n'existe pas, lever une erreur avec `stop()` plutôt que de créer une base vide sans prévenir.
<details>
<summary>Correction</summary>

```r
if (!file.exists(base_sqlite)) {
  stop("La base ", base_sqlite, " est introuvable : vérifier le chemin.")
}
```

Le message est alors capté par `tryCatch()` et écrit dans le journal.
</details>

## Exercice 5 – Interroger l'API sur les 7 derniers jours

La tâche ne retélécharge pas toute la base : elle demande à l'API seulement les DPE reçus pendant les 7 derniers jours, en filtrant sur `date_reception_dpe`. Les nouveaux DPE sont ajoutés à la base, sans doublons, grâce à `numero_dpe`.

Le filtre utilise deux paramètres : `date_reception_dpe_gte` (date supérieure ou égale) et `date_reception_dpe_lte` (date inférieure ou égale). L'URL correspondante ressemble à `...&date_reception_dpe_gte=2026-01-01&date_reception_dpe_lte=2026-01-07`. Une page de 10 000 lignes suffit pour une semaine : le script journalise un avertissement si l'API annonce une page suivante.

Le script doit contenir la fonction `lire_page()` (chapitre 7, exercice 5, question 3) et `journaliser()` (exercice 3).

### Mémo

| Nom de la commande | Description | Argument Pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `Sys.Date()` | Retourne la date du jour (voir chapitre 5). | | `Sys.Date()` |
| `format()` | Met une date en texte `AAAA-MM-JJ`, le format attendu par l'API. | `x`, `format` | `format(Sys.Date() - 7, "%Y-%m-%d")` |
| `req_url_query()` | Ajoute le filtre sur la date à la requête. | paramètres nommés | `date_reception_dpe_gte = "2026-10-11", date_reception_dpe_lte = "2026-10-18"` |
| `dbConnect()` / `dbDisconnect()` | Ouvre et ferme la connexion à la base (chapitre 6). | `drv`, `dbname` | `dbConnect(RSQLite::SQLite(), base_sqlite)` |
| `dbExistsTable()` | Teste si la table existe déjà dans la base. | `conn`, `name` | `dbExistsTable(con, "dpe_api")` |
| `dbGetQuery()` | Lit les numéros de DPE déjà présents dans la base. | `conn`, `statement` | `dbGetQuery(con, "SELECT numero_dpe FROM dpe_api")` |
| `filter()` avec `!` | Garde les lignes qui ne sont pas dans un ensemble. | `!numero_dpe %in% deja` | `filter(!numero_dpe %in% deja)` |
| `dbAppendTable()` | Ajoute les nouvelles lignes à la table. | `conn`, `name`, `value` | `dbAppendTable(con, "dpe_api", nouveaux)` |

1. Construire la requête qui demande les DPE de Villeurbanne reçus depuis il y a 7 jours, puis les lire avec `lire_page()`. Stocker le résultat dans `telecharges`, et journaliser le nombre de DPE reçus.
<details>
<summary>Correction</summary>

```r
url_api <- "https://data.ademe.fr/data-fair/api/v1/datasets/dpe03existant/lines"
colonnes <- c(
  "numero_dpe", "date_reception_dpe", "etiquette_dpe",
  "surface_habitable_logement", "conso_5_usages_ep",
  "type_energie_n1", "identifiant_ban"
)

date_debut <- format(Sys.Date() - 7, "%Y-%m-%d")
date_fin <- format(Sys.Date(), "%Y-%m-%d")

requete <- request(url_api) |>
  req_url_query(
    q_fields = "code_postal_ban",
    code_postal_ban_eq = 69100,
    date_reception_dpe_gte = date_debut,
    date_reception_dpe_lte = date_fin,
    select = paste(colonnes, collapse = ","),
    size = 10000
  )

page <- lire_page(requete$url)
telecharges <- page$donnees

if (!is.null(page$suivante)) {
  journaliser("Attention : plus de 10 000 DPE sur la semaine, certains DPE sont manquants")
}

journaliser(paste(nrow(telecharges), "DPE reçus depuis le", date_debut))
```
</details>

2. Ajouter l'étape d'écriture : lire les numéros déjà présents dans `dpe_api`, garder les DPE nouveaux, les ajouter à la table, puis journaliser le nombre de DPE ajoutés. Si la table n'existe pas encore, la créer avec tous les DPE.
<details>
<summary>Correction</summary>

```r
con <- dbConnect(RSQLite::SQLite(), base_sqlite)

if (dbExistsTable(con, "dpe_api")) {
  deja <- dbGetQuery(con, "SELECT numero_dpe FROM dpe_api")$numero_dpe
  nouveaux <- telecharges |>
    filter(!numero_dpe %in% deja)

  if (nrow(nouveaux) > 0) {
    dbAppendTable(con, "dpe_api", nouveaux)
  }
} else {
  nouveaux <- telecharges
  dbWriteTable(con, "dpe_api", telecharges)
}

dbDisconnect(con)
journaliser(paste(nrow(nouveaux), "nouveaux DPE ajoutés"))
```
</details>

## Exercice 6 – Planifier l'exécution hebdomadaire

Le Planificateur de tâches Windows lance une commande à une heure donnée. Ici, la commande est `Rscript.exe` avec le script en argument, chaque lundi à 6 h 00.

Dans l'interface graphique (Planificateur de tâches, « Créer une tâche ») :
- **Général** : nommer la tâche `MiseAJourDPE`. Cocher « Exécuter même si l'utilisateur n'est pas connecté » si le poste doit lancer la tâche sans session ouverte, ce qui demande le mot de passe du compte.
- **Déclencheurs** : nouveau déclencheur, hebdomadaire, le lundi, à 6 h 00.
- **Actions** : programme `C:\Program Files\R\R-4.4.1\bin\Rscript.exe` (le chemin de l'exercice 2), arguments `--vanilla "C:\Users\Anthony\Documents\GitHub\TDA-DPE\scripts\mise_a_jour_dpe.R"`.

1. Créer la tâche avec l'interface graphique, puis la lancer immédiatement (clic droit, « Exécuter »). Vérifier dans `logs/mise_a_jour.log` que la dernière ligne vient de la tâche.
<details>
<summary>Correction</summary>

```text
Dans le Planificateur de tâches : Bibliothèque du Planificateur > MiseAJourDPE > clic droit > Exécuter.
Puis ouvrir logs/mise_a_jour.log : une nouvelle série de lignes doit apparaître,
avec l'heure du lancement.
```
</details>

## Liens utiles

- [Documentation de `Rscript`](https://cran.r-project.org/doc/manuals/r-release/R-intro.html)
- [Documentation de `tryCatch()`](https://stat.ethz.ch/R-manual/R-patched/library/base/html/conditions.html)
- [Référence de la commande `schtasks`](https://learn.microsoft.com/windows-server/administration/windows-commands/schtasks-create)
- [Documentation de httr2](https://httr2.r-lib.org/)
