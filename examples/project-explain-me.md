Je découvre ce projet et je dois pouvoir l'utiliser puis le modifier. Analyse le dépôt et explique-le-moi. Ne modifie aucun fichier.

## Méthode
1. Explore avant de conclure : README, docs, fichiers de dépendances et de build (package.json, pyproject.toml, Cargo.toml, go.mod, Makefile, Dockerfile, etc.), configuration (.env.example, fichiers de config), points d'entrée, structure des dossiers, tests, CI, et `git log` récent.
2. Lis le code des modules centraux, pas seulement leurs noms.
3. Distingue ce que tu as vérifié dans le code de ce que tu déduis. Signale les zones non lues ou ambiguës.
4. Cite tes sources sous forme `chemin/fichier:ligne`.

## Livrable attendu
1. **Finalité** : quel problème le projet résout, pour qui, dans quel contexte. Trois phrases maximum.
2. **Stack** : langages, frameworks, dépendances clés et leur rôle.
3. **Architecture** : arborescence commentée des dossiers importants, responsabilités de chaque module, relations entre eux.
4. **Flux d'exécution** : ce qui se passe du lancement au résultat, étape par étape, à partir du point d'entrée réel. Ajoute un diagramme Mermaid si le flux le justifie.
5. **Données** : entrées, sorties, formats, stockage, services ou API externes.
6. **Mise en route** : prérequis, installation, configuration (variables d'environnement, secrets), commandes pour lancer, tester, builder. Vérifie que ces commandes existent réellement dans le projet.
7. **Points de modification** : où intervenir pour les changements les plus probables (ajouter une fonctionnalité, changer un comportement, modifier la config), conventions de code à respecter, et ce qu'un changement risque de casser.
8. **Points d'attention** : dette technique, code mort, TODO, incohérences, dépendances obsolètes, absence de tests, risques de sécurité.
9. **Zones d'incertitude** : ce que tu n'as pas pu déterminer et pourquoi.

## Contraintes
- Réponds en français.
- Va du général au détail.
- Pas de paraphrase du README : confronte-le au code et signale les écarts.
- Si le projet est volumineux, couvre d'abord le cœur fonctionnel et indique explicitement les parties laissées de côté.