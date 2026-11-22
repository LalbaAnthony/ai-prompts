# Mission

Tu as accès en lecture à plusieurs dépôts. Ils sont tous issus d'un même fork d'origine puis ont divergé. Objectif : produire un `CLAUDE.md` générique, en français, copiable tel quel dans chacun de ces dépôts, contenant uniquement ce qui est commun à la majorité d'entre eux. Chaque dépôt ajoutera ensuite son contenu spécifique à la main, sous ce fichier.

Dépôts à analyser :
- <CHEMIN_REPO_1>
- <CHEMIN_REPO_2>
- <CHEMIN_REPO_N>

# Contraintes absolues

- Lecture seule sur les dépôts : aucune écriture, aucun `install`, `build`, `test`, migration, ni commande modifiant l'état (fichiers, lockfiles, git). Commandes autorisées : lecture de fichiers, `ls`, `find`, `grep`, `git log`, `git show`, `git ls-files`, `git branch`, `git remote -v`.
- Les livrables s'écrivent uniquement dans le répertoire courant (celui de lancement).
- Aucune affirmation non vérifiée dans le code. Pas de bonnes pratiques génériques ajoutées par défaut : chaque règle du `CLAUDE.md` doit découler d'un constat dans les dépôts.
- Aucune valeur secrète. Variables d'environnement : noms uniquement.
- En cas d'ambiguïté (équivalence douteuse entre deux implémentations, comptage de majorité incertain, convention implicite contradictoire), arrête-toi et pose la question via AskUserQuestion avant de trancher.

# Méthode

## Phase 1 — Inventaire par dépôt

Lance un sous-agent par dépôt, en parallèle. Chaque sous-agent produit un inventaire factuel et structuré couvrant :

1. Identité : premier commit (`git log --reverse --format='%H %ad %s' | head -1`), remotes, branche par défaut.
2. Stack : langages, frameworks, versions (fichiers de version, manifestes, lockfiles, Dockerfile, CI), gestionnaire de paquets.
3. Commandes : scripts déclarés (package.json, Makefile, justfile, pyproject, composer, Taskfile, etc.) avec leur commande exacte.
4. Arborescence : structure des dossiers jusqu'à 3 niveaux, rôle de chaque dossier racine, points d'entrée.
5. Architecture : couches, découpage en modules, flux de données, patterns récurrents (injection de dépendances, repository, services, stores, routing, middleware), communication entre composants.
6. Conventions de code : nommage (fichiers, classes, fonctions, variables), organisation des imports, typage, gestion d'erreurs, logging, configuration lint/format (règles effectives, pas seulement la présence de l'outil).
7. Tests : framework, emplacement, nommage, type (unitaires, intégration, e2e), fixtures/mocks, commande d'exécution.
8. Git et workflow : format des messages de commit (déduit des 100 derniers commits), stratégie de branches, hooks (husky, pre-commit, lefthook), templates de PR.
9. CI/CD : pipelines, étapes, conditions de blocage.
10. Configuration : fichiers de config, variables d'environnement attendues (`.env.example`, lecture dans le code).
11. Invariants et pièges : fichiers générés à ne pas éditer, ordre d'opérations requis, dépendances entre modules, code legacy signalé.

Chaque élément est relevé avec son chemin source.

## Phase 2 — Matrice de comparaison

Construis une matrice élément × dépôt. Règle de sélection :
- Un élément est commun s'il est présent de manière identique ou fonctionnellement équivalente dans strictement plus de la moitié des dépôts.
- Les dépôts minoritaires sur cet élément sont ignorés pour le `CLAUDE.md` et consignés dans le rapport de divergences.
- Versions : retiens le niveau de précision le plus fin partagé par la majorité (ex. `Node 20.x` si la majorité concorde sur la majeure mais pas sur la mineure ; rien si aucune majorité sur la majeure).
- Chemins et noms : retiens uniquement ceux identiques dans la majorité. Aucun nom de dépôt, de produit ou de client dans le `CLAUDE.md`.

## Phase 3 — Rédaction de `./CLAUDE.md`

Langue : français. Style : impératif, factuel, dense. Niveau : exhaustif, architecture détaillée incluse. Commandes dans des blocs de code, exactes et copiables.

Sections, dans cet ordre (omets une section si aucun contenu majoritaire) :

1. Vue d'ensemble — nature du projet, rôle fonctionnel commun.
2. Stack et versions.
3. Prérequis et installation.
4. Commandes — dev, build, lint, format, test, autres scripts communs.
5. Arborescence — arbre commenté des dossiers communs.
6. Architecture — couches, modules, responsabilités, flux de données, patterns, règles de dépendance entre couches.
7. Conventions de code — nommage, imports, typage, erreurs, logging, style imposé par le linter.
8. Tests — organisation, conventions, exécution.
9. Git et workflow — format de commit, branches, hooks, CI.
10. Configuration et environnement — fichiers, noms de variables, rôle.
11. Invariants et pièges — ce qui casse si on l'ignore, fichiers générés, interdits.

Le fichier se termine sur la dernière section de contenu : aucun séparateur, aucun placeholder, aucune section réservée au contenu spécifique, aucune mention de son caractère générique.

## Phase 4 — Rédaction de `./DIVERGENCES.md`

Langue : français. Structure :

1. Matrice synthétique : tableau élément × dépôt, avec taux d'adoption, pour chaque élément évalué (retenus et exclus).
2. Une section par dépôt, contenant :
   - Contradictions : règles du `CLAUDE.md` commun que ce dépôt ne respecte pas (à surcharger obligatoirement dans son contenu spécifique). Section prioritaire, en tête.
   - Spécificités : éléments propres à ce dépôt, absents du commun.
   - Éléments exclus faute de majorité, et la variante de ce dépôt.
   Chaque point avec son chemin source.

## Phase 5 — Vérification

Avant de terminer :
- Relis chaque affirmation du `CLAUDE.md` et confirme qu'elle est vérifiée dans la majorité des dépôts. Supprime toute affirmation non confirmée.
- Confirme que chaque contradiction relevée en phase 2 apparaît dans `DIVERGENCES.md` pour le dépôt concerné.
- Confirme l'absence de noms de dépôts, de valeurs secrètes et de chemins absolus dans le `CLAUDE.md`.

Sortie terminale finale : nombre de dépôts analysés, nombre d'éléments retenus, nombre d'éléments exclus, nombre de contradictions par dépôt.