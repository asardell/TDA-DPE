# Chapitre 9 – Git et GitHub Desktop : sauvegarder et partager son travail

## Objectifs

Voici les objectifs de ce chapitre :
- [ ] comprendre à quoi servent un dépôt, un commit et une branche
- [ ] cloner un dépôt sur son ordinateur avec GitHub Desktop
- [ ] enregistrer une modification avec un commit, puis l'envoyer sur GitHub
- [ ] proposer une modification avec une pull request

- [Chapitre 9 – Git et GitHub Desktop : sauvegarder et partager son travail](#chapitre-9--git-et-github-desktop--sauvegarder-et-partager-son-travail)
  - [Objectifs](#objectifs)
  - [Prérequis](#prérequis)
  - [L'idée en une phrase](#lidée-en-une-phrase)
  - [Le vocabulaire](#le-vocabulaire)
  - [Pas à pas : le dépôt de test](#pas-à-pas--le-dépôt-de-test)
  - [Pour aller plus loin : le projet de la formation](#pour-aller-plus-loin--le-projet-de-la-formation)
  - [Liens utiles](#liens-utiles)

## Prérequis

- Un **compte GitHub**, créé sur [github.com](https://github.com/signup).
- **GitHub Desktop**, téléchargé sur [desktop.github.com](https://desktop.github.com/), installé et connecté à son compte (menu « File » ou « Fichier » > « Options », puis connexion).

Aucune ligne de commande n'est nécessaire dans ce chapitre : tout se fait avec la souris.

> **Visuel à insérer :** écran d'accueil de GitHub Desktop après la connexion.

## L'idée en une phrase

**Git** garde l'historique de tous les fichiers d'un projet, comme un carnet de recettes qui noterait chaque modification avec la date et un message. **GitHub** est le site où ce carnet est conservé en ligne, pour le partager ou le reprendre depuis un autre ordinateur. **GitHub Desktop** est l'application qui permet de faire tout cela avec des boutons.

## Le vocabulaire

| Terme | Ce que c'est | Où le trouver dans GitHub Desktop |
|-------|--------------|-----------------------------------|
| **Dépôt** (*repository*) | Le dossier du projet, avec tout son historique. | La liste de gauche « Current repository » |
| **Cloner** | Copier sur son ordinateur un dépôt qui est sur GitHub. | « File » > « Clone repository » |
| **Modification** | Un fichier ajouté, changé ou supprimé. | La liste « Changes » |
| **Commit** | Une sauvegarde nommée de ses modifications, avec un message. | Le bas de la colonne « Changes » : titre du commit, puis bouton « Commit to main » |
| **Push** | Envoyer ses commits sur GitHub. | Le bouton « Push origin » en haut |
| **Pull** | Récupérer sur son ordinateur les commits faits ailleurs. | Le bouton « Pull origin » en haut |
| **Branche** | Une copie parallèle du projet, pour essayer sans toucher à la version principale. | Le menu « Current branch » |
| **Pull request** | Une demande pour intégrer une branche dans la version principale, avec une discussion. | Le bouton « Create Pull Request » après un push |

Les libellés des boutons peuvent varier selon la langue de l'application. Les noms ci-dessus sont ceux de la version anglaise, avec leur traduction en français entre parenthèses quand elle existe.

## Pas à pas : le dépôt de test

Le dépôt de test de la formation est [git-cooking-workshop](https://github.com/asardell/git-cooking-workshop). Il sert à s'entraîner sans risque : rien de ce qui est fait ici n'affecte les autres participants, ni le projet de la formation.

1. **Cloner le dépôt.** Dans GitHub Desktop, « File » > « Clone repository », puis choisir l'onglet « URL » et coller l'adresse du dépôt de test. Choisir un dossier sur son ordinateur, puis valider.

   > **Visuel à insérer :** fenêtre de clonage avec l'adresse du dépôt de test.

2. **Faire une modification.** Ouvrir le dossier du dépôt avec l'explorateur de fichiers (bouton « Show in Explorer » ou « Ouvrir dans l'explorateur »), modifier un fichier texte, puis l'enregistrer. Revenir dans GitHub Desktop : la modification apparaît dans la liste « Changes ».

   > **Visuel à insérer :** la liste « Changes » avec un fichier modifié.

3. **Faire un commit.** Cocher le fichier, écrire un titre court dans le champ en bas à gauche (par exemple « Ajout de ma ligne »), puis cliquer sur « Commit to main ». La modification est maintenant enregistrée dans l'historique, mais seulement sur son ordinateur.

   > **Visuel à insérer :** le champ du titre du commit et le bouton « Commit to main ».

4. **Envoyer sur GitHub.** Cliquer sur « Push origin ». Le commit apparaît maintenant sur le site GitHub, dans l'onglet « Commits » du dépôt.

5. **Proposer une pull request (facultatif).** Créer une branche avec « Current branch » > « New branch », faire une modification et un commit sur cette branche, puis la publier. GitHub Desktop propose alors « Create Pull Request » : la demande s'ouvre sur le site GitHub, où le responsable du dépôt peut la lire, la commenter ou l'accepter.

   > **Visuel à insérer :** la page de pull request sur GitHub.

## Liens utiles

- [Site officiel de GitHub Desktop](https://desktop.github.com/)
- [Documentation de GitHub Desktop](https://docs.github.com/en/desktop)
- [Dépôt de test de la formation](https://github.com/asardell/git-cooking-workshop)
