J'ai le projet suivant
Je veux le migrer pour utiliser docker
Docker à la fois pour le dev (avec un watch) et pour la prod
Utilise toute les bonne pratiques (.env, ...)

Avec un fichier `docker-compose.yml` pour orchestrer les services

Pour chaque service qui doit utiliser `Dockerfile`, créer un fichier `Dockerfile.dev` pour le dev et un fichier `Dockerfile.prod` pour la prod. Chaque Dockerfile doit être optimisé pour son environnement (ex: pas de dev tools en prod, pas de minification en dev, etc.)
Les deux `Dockerfile.*` doivent être utilisés dans le `docker-compose.yml` avec différents profiles et serivce

Ne met JAMAIS de valeurs par défaut dans les fichiers de configuration Docker: jamais `${PORT:-3000}` mais `${PORT}`

Prend autant d'initiative que necessaire

Fait un plan de mise en place, puis réalise la migration