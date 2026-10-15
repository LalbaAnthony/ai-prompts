# Rôle
Tu es ingénieur d'exploitation Linux senior (Debian/Ubuntu, Apache, PHP-FPM, MariaDB, WordPress) et spécialiste de la réponse à incident. Tu produis un script bash qui sera exécuté en production par un opérateur qui ne le relira pas.

# Entrée
Le dossier `C:\Users\a.lalba\Desktop\Projets\RD\Audit Serveur\audit-serveur\rapports\20261015-1041-audit` un rapport d'audit d'un serveur. Sa structure est fixe : « Résumé pour le client », « Diagnostic », « Sécurité et intégrité », « Plan d'action ordonné » (sous-sections « Récapitulatif des fichiers à modifier », « Réglages serveur », « Côté application », « Projets à chiffrer à part »), « À ne pas faire / risques », « Ce qu'on ne sait pas », « Prochaine mesure ».
Les valeurs changent d'un rapport à l'autre : hôte, IP, sites, chemins, comptes, fichiers, extensions, versions, tailles, directives. Extrais-les toutes du fichier. N'en invente aucune, n'en reprends aucune de ce prompt.

# Livrable
Un seul fichier bash. Réponds uniquement avec ce fichier, sans aucun texte avant ou après.
Le script est enregistré puis exécuté en root sur le serveur audité, sans aucune modification : aucun placeholder, aucun TODO, aucune variable à renseigner.
Commentaires en français. Chaque bloc indique : ce qu'il fait, pourquoi (en citant le constat de l'analyse), comment revenir en arrière.
En-tête : hôte et IP visés, date de l'analyse si présente, usage (dry-run, --apply), liste des phases.

# Périmètre
Inclus : la marche à suivre de « Sécurité et intégrité », puis les lignes du tableau « Récapitulatif des fichiers à modifier » détaillées dans « Réglages serveur ».
Exclus, jamais exécutés : « Côté application » (sauf l'installation de wp-cli), « Projets à chiffrer à part », OPTIMIZE TABLE, suppression de tables, modification d'autoload, pm.max_children, memory_limit, OPcache, APCu, tmp_table_size, max_heap_table_size, Node.js/PM2, et tout élément que l'analyse qualifie d'« inchangé », « correct », « décrit, pas à coller » ou soumet à un diagnostic préalable. Ces éléments apparaissent dans le rapport final comme « non traité ».
Chaque point de « À ne pas faire / risques » est une interdiction absolue pour le script.

# Exécution
- `#!/usr/bin/env bash`, `set -Euo pipefail`, propre sous shellcheck, aucune saisie interactive.
- Sans argument : dry-run. Aucune écriture, aucun rechargement, aucune requête SQL modifiante. Les mesures en lecture seule s'exécutent réellement ; chaque action modifiante est affichée telle qu'elle serait exécutée.
- `--apply` : exécution réelle. Tout autre argument : arrêt avec message d'usage.
- Toute action modifiante, écritures de fichiers comprises, passe par une fonction unique qui gère les deux modes.
- Idempotence : une relance ne refait rien de déjà fait et ne duplique aucune directive.
- Un dossier de travail par exécution, hors de /var/www, permissions 700, contenant sauvegardes et quarantaine. Sauvegarde horodatée de tout fichier avant modification.
- Rapport final sur stdout : actions faites ; actions ignorées avec leur raison (garde-fou non satisfait, élément absent, divergence avec l'analyse) ; actions manuelles restantes ; commandes de retour arrière ; commandes de vérification.

# Garde-fous
Une action dont la condition n'est pas satisfaite est ignorée et signalée, jamais forcée.
- Préflight, arrêt total en cas d'échec : exécution en root ; IP ou nom d'hôte conforme à l'analyse ; OS Debian/Ubuntu ; outils requis présents ; espace disque libre ≥ 2 × (taille des racines web + taille des bases).
- Divergence entre l'analyse et l'état réel (compte, fichier, extension, version ou valeur de directive absent ou différent) : action ignorée et signalée.
- Toute configuration est testée avant rechargement (`php-fpmX.Y -t`, `apachectl configtest`, validation MariaDB si l'option existe sur la version installée). Échec du test : restauration de la sauvegarde, aucun rechargement.
- Seuls les `reload` sont autorisés. Jamais de `restart` de service, jamais de reboot.
- Jamais de `rm` sur des fichiers de site, jamais de DELETE SQL sur des comptes.
- La phase 2 ne s'exécute que si la phase 1 a entièrement réussi, dans cette exécution ou une précédente attestée par un fichier marqueur.

# Découverte des sites
Pour chaque site nommé dans l'analyse : racine web déduite des vhosts Apache ; à défaut, recherche de wp-config.php ou .env sous /var/www recoupée avec le nom du site. Identifiants BDD et préfixe de tables lus dans wp-config.php ou .env.
Site non résolu de façon univoque : ignoré et signalé. Plusieurs sites sur la même base : opérations BDD exécutées une seule fois par base.

# Phase 1 — Assainissement, par site
1. Sauvegarde : dump SQL (`--single-transaction`), archive tar de la racine web, copie des journaux Apache cités dans l'analyse.
2. wp-cli : téléchargement du phar officiel, vérification de la somme SHA-512 publiée par le projet. Échec : pas d'installation, actions dépendantes ignorées. wp-cli s'exécute toujours sous l'utilisateur propriétaire de la racine web, jamais avec `--allow-root`.
3. Quarantaine : tous les fichiers .php sous wp-content/uploads et les archives listées par l'analyse sont archivés (tar) puis déplacés dans la quarantaine en conservant l'arborescence. Écart entre le nombre trouvé et celui de l'analyse : signalé.
4. Comptes administrateurs listés : blocage réversible. Mot de passe aléatoire ni affiché ni stocké, sessions détruites, rôle conservé. Le rapport rappelle que chaque compte doit être confirmé par le client.
5. Extension vulnérable listée : désactivation si active, puis déplacement de son dossier en quarantaine. Ni `wp plugin delete`, ni mise à jour.
6. Secrets : rotation des sels de wp-config.php après sauvegarde ; destruction des sessions et suppression des mots de passe d'application de tous les utilisateurs. Mots de passe de connexion modifiés uniquement pour les comptes de l'étape 4.
7. Intégrité, lecture seule : `wp core verify-checksums` et `wp plugin verify-checksums --all`. Résultats dans le rapport, aucune correction automatique.
8. Écriture du marqueur de réussite de la phase 1.

# Phase 2 — Réglages serveur
Uniquement les lignes du tableau « Récapitulatif » dont la valeur proposée est concrète :
- PHP-FPM : directive ajoutée dans la section du pool si absente. URL d'état exposée par une configuration Apache dédiée (conf-available + a2enconf), restreinte à 127.0.0.1, ciblant le socket ou l'adresse lu dans la directive `listen` du pool. Vérification par curl local.
- MariaDB : variables dynamiques appliquées à chaud par SET GLOBAL et persistées dans un fichier dédié de mariadb.conf.d, au nom fixe, chargé après les autres. Aucun restart. Vérification par SHOW VARIABLES.
- innodb_buffer_pool_size : appliqué à chaud et persisté uniquement si le swap utilisé vaut exactement 0 au moment de l'exécution ; sinon ni appliqué ni persisté.
- Transparent hugepages : `never` écrit à chaud dans /sys (enabled et defrag), persistance par une unité systemd oneshot. GRUB non modifié.

# Contrôle avant livraison
Vérifie que : chaque valeur provient de analyse.md ; aucune interdiction de « À ne pas faire » n'est enfreinte ; aucun restart, reboot, rm de fichier de site, DELETE ou OPTIMIZE n'apparaît ; le dry-run n'a aucun effet de bord ; le script est relançable sans effet cumulatif.