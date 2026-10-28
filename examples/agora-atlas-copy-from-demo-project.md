# Portage d'un module de la boilerplate Atlas

## Module à porter

```

```

## Contexte

- **Atlas** est une boilerplate Laravel (backend + backoffice). Source de référence : `/var/www/html/temporary-ouranos` (appelée **SOURCE** ci-dessous).
- Le dossier `back/` du repo courant (appelé **CIBLE**) dérive de SOURCE.
- Les deux ont divergé depuis :
  - **CIBLE** contient des adaptations propres au projet : logique métier, configuration, intégration avec d'autres composants du projet.
  - **SOURCE** contient des améliorations génériques du socle : nouvelles fonctionnalités et correctifs.

## Objectif

Porter dans CIBLE le module indiqué ci-dessus, avec ses améliorations venues de SOURCE, sans écraser les adaptations propres au projet présentes dans CIBLE.

## Contraintes strictes

- SOURCE est en lecture seule : aucune création, modification ou suppression de fichier.
- Aucun accès au git de SOURCE : pas de commande `git` dans ce dossier, pas de lecture de `.git/`. La comparaison se fait uniquement sur le contenu des fichiers (lecture directe, `diff`).
- Aucun commit, aucun push dans le repo courant.
- Aucune modification de CIBLE avant validation du plan (étape 4).

## Méthode

### 1. Délimiter le module dans SOURCE

Le chemin fourni est un point d'entrée, pas le périmètre complet. Remonter toutes les dépendances du module :

- vues, composants Blade et partials ;
- routes ;
- contrôleurs, form requests, policies, middlewares ;
- modèles, migrations, factories, seeders ;
- services, actions, jobs, events et listeners ;
- traductions, configuration, entrées de menu et permissions du backoffice ;
- assets (JS, CSS) ;
- tests ;
- packages Composer et NPM requis.

### 2. Comparer avec CIBLE

Classer chaque élément du périmètre dans une de ces catégories :

- **Absent** de CIBLE.
- **Identique**.
- **Divergent**.

### 3. Qualifier chaque divergence

Pour chaque divergence, déterminer son origine :

- **Amélioration générique de SOURCE** : à intégrer.
- **Adaptation propre au projet dans CIBLE** : à préserver.
- **Ambiguë**, ou les deux à la fois : à soumettre à l'utilisateur.

### 4. Présenter le plan avant toute écriture

Le plan contient :

- la liste des fichiers à créer, modifier ou ignorer, avec la justification de chaque choix ;
- les dépendances hors module (classes de base partagées, helpers, packages, migrations touchant des tables existantes) ;
- les conflits et leur résolution proposée.

Faire valider ce plan via **AskUserQuestion**.

### 5. Implémenter

- Fusionner les fichiers divergents ; ne jamais les remplacer en bloc.
- Respecter les conventions de CIBLE : namespaces, nommage, structure, style.
- Pour un schéma de base de données modifié : ne pas éditer une migration existante de CIBLE sans validation.

### 6. Vérifier

Exécuter les vérifications disponibles dans CIBLE :

- `php artisan route:list` filtré sur le module ;
- lint et analyse statique, s'ils sont configurés ;
- tests liés au module ;
- build des assets, si le module en ajoute.

Corriger les erreurs introduites par le portage.

## Règles de décision

- En cas de conflit entre une amélioration de SOURCE et une adaptation de CIBLE, ne pas trancher seul.
- Toute dépendance hors du périmètre du module requiert une validation avant d'être portée.
- Au moindre doute, utiliser **AskUserQuestion** avant de continuer :
  - poser des questions fermées, avec options concrètes et conséquences explicites ;
  - regrouper les questions liées dans un même appel ;
  - itérer jusqu'à ce qu'aucun doute ne subsiste, sans limite de nombre de questions ;
  - ne jamais combler une incertitude par une supposition.

## Rapport final

- Fichiers créés et modifiés.
- Décisions prises et leur justification.
- Éléments volontairement non portés, avec la raison.
- Commandes à exécuter manuellement (migrations, installation de packages, build, cache).