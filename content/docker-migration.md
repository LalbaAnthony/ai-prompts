J'ai le projet suivant
Je veux le migrer pour utiliser docker
Docker à la fois pour le dev (avec un watch) et pour la prod
Utilise toute les bonne pratiques (.env, ...)
Les cron doivent être gérés dans le container app
Pas de SSL en dev
La version de PHP doit être 8.3, elle doit être configurable via une variable d'environnement

Déplace tout le code source dans un dossier `src` à la racine du projet

Oublie grunt, gruntfile, scss ... ce sont des reliquas du projet d'origine ne le supprime pas mais ne les prends pas en compte (en d'autre termes, MINIFY de la config sera toujours à FALSE)

Prend autant d'initiative que necessaire

Fait un plan de mise en place