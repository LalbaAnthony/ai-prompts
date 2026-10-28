# Portage d'un module de la boilerplate Atlas

## Module à porter

```

```

## Contexte

- **Atlas** est une boilerplate fullstack. Source de référence : `/var/www/html/demo-project` (appelée **SOURCE** ci-dessous).
  - `back/` : backend et backoffice Laravel.
  - `front/` : application Nuxt.
- Le repo courant (appelé **CIBLE**) dérive de SOURCE et reprend la même organisation `back/` et `front/`.
- Les deux ont divergé depuis :
  - **CIBLE** contient des adaptations propres au projet : logique métier, configuration, intégration avec d'autres composants du projet.
  - **SOURCE** contient des améliorations génériques du socle : nouvelles fonctionnalités et correctifs.

## Objectif

Porter dans CIBLE le module indiqué ci-dessus, avec ses améliorations venues de SOURCE, sans écraser les adaptations propres au projet présentes dans CIBLE.

Le module peut s'étendre sur `back/` et `front/`. Le point d'entrée fourni n'indique pas le côté concerné : le périmètre réel est déterminé à l'étape 1.

## Contraintes strictes

- SOURCE est entièrement en lecture seule, `back/` comme `front/` : aucune création, modification ou suppression de fichier.
- Aucun accès au git de SOURCE : pas de commande `git` dans SOURCE ni dans ses sous-dossiers, pas de lecture de `.git/`. La comparaison se fait uniquement sur le contenu des fichiers (lecture directe, `diff`).
- Aucun commit, aucun push dans le repo courant.
- Aucune modification de CIBLE avant validation du plan (étape 4).

## Méthode

### 1. Délimiter le module dans SOURCE

Le chemin fourni est un point d'entrée, pas le périmètre complet. Remonter toutes les dépendances du module, des deux côtés.

**Côté `back/` :**

- vues, composants Blade et partials ;
- routes web et API ;
- contrôleurs, form requests, API resources, policies, middlewares ;
- modèles, migrations, factories, seeders ;
- services, actions, jobs, events et listeners ;
- traductions, configuration, entrées de menu et permissions du backoffice ;
- assets (JS, CSS) ;
- tests ;
- packages Composer et NPM requis.

**Côté `front/` :**

- pages, layouts, composants ;
- composables, stores ;
- middlewares de route, plugins ;
- routes serveur Nitro (`server/`) ;
- types et clients API ;
- traductions ;
- modules et clés de `nuxt.config`, `runtimeConfig` ;
- assets ;
- tests ;
- packages NPM requis.

**Liens entre `back/` et `front/` :**

- endpoints API consommés par le front ;
- format des payloads et des ressources ;
- types partagés ou générés ;
- variables d'environnement des deux côtés (`.env.example`) ;
- comportements répartis entre les deux (exemple : redirections gérées par le back et appliquées par le front).

### 2. Comparer avec CIBLE

Pour chaque élément du périmètre, comparer avec son équivalent dans CIBLE (`back/` ↔ `back/`, `front/` ↔ `front/`) et le classer dans une de ces catégories :

- **Absent** de CIBLE.
- **Identique**.
- **Divergent**.

### 3. Qualifier chaque divergence

Pour chaque divergence, déterminer son origine :

- **Amélioration générique de SOURCE** : à intégrer.
- **Adaptation propre au projet dans CIBLE** : à préserver.
- **Ambiguë**, ou les deux à la fois : à soumettre à l'utilisateur.

Vérifier aussi la compatibilité du contrat entre back et front : un endpoint porté doit rester compatible avec le front de CIBLE, et inversement.

### 4. Présenter le plan avant toute écriture

Le plan contient :

- la liste des fichiers à créer, modifier ou ignorer, séparée en `back/` et `front/`, avec la justification de chaque choix ;
- les dépendances hors module (classes de base, helpers, composables partagés, packages, migrations touchant des tables existantes, modules Nuxt) ;
- les impacts sur le contrat API entre back et front ;
- les conflits et leur résolution proposée.

Faire valider ce plan via **AskUserQuestion**.

### 5. Implémenter

- Fusionner les fichiers divergents ; ne jamais les remplacer en bloc.
- Respecter les conventions de CIBLE de chaque côté : namespaces, nommage, structure, style, auto-imports Nuxt.
- Pour un schéma de base de données modifié : ne pas éditer une migration existante de CIBLE sans validation.
- Porter le back et le front de manière cohérente : jamais un côté dépendant d'un élément non porté de l'autre.

### 6. Vérifier

Exécuter les vérifications disponibles dans CIBLE.

**Côté `back/` :**

- `php artisan route:list` filtré sur le module ;
- lint et analyse statique, s'ils sont configurés ;
- tests liés au module ;
- build des assets, si le module en ajoute.

**Côté `front/` :**

- lint ;
- typecheck (`nuxi typecheck` ou script équivalent) ;
- tests liés au module ;
- build.

Corriger les erreurs introduites par le portage.

## Règles de décision

- En cas de conflit entre une amélioration de SOURCE et une adaptation de CIBLE, ne pas trancher seul.
- Toute dépendance hors du périmètre du module requiert une validation avant d'être portée.
- Toute modification du contrat API entre back et front requiert une validation.
- Au moindre doute, utiliser **AskUserQuestion** avant de continuer :
  - poser des questions fermées, avec options concrètes et conséquences explicites ;
  - regrouper les questions liées dans un même appel ;
  - itérer jusqu'à ce qu'aucun doute ne subsiste, sans limite de nombre de questions ;
  - ne jamais combler une incertitude par une supposition.

## Rapport final

- Fichiers créés et modifiés, séparés en `back/` et `front/`.
- Décisions prises et leur justification.
- Impacts sur le contrat API.
- Éléments volontairement non portés, avec la raison.
- Commandes à exécuter manuellement : migrations, installation de packages (Composer, NPM), variables d'environnement à ajouter, build, cache.