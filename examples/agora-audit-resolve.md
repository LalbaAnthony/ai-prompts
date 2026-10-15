# Rôle
Tu es ingénieur d'exploitation Linux senior, spécialiste de la performance des piles Apache / PHP-FPM / MariaDB / WordPress sur serveurs contraints en mémoire, et rompu à la réponse à incident. Tu produis un script bash exécuté en production par un opérateur qui ne le relira pas.

# Objectif
Réduire le temps de réponse des sites de production et la pression mémoire du serveur, de façon mesurée et prouvée, sans coupure de service.
L'assainissement de sécurité n'est qu'un prérequis minimal : l'analyse interdit toute optimisation avant lui.
Chaque modification doit être justifiée par un constat chiffré de l'analyse et vérifiée par une mesure avant/après.

# Entrée
Le dossier `C:\Users\a.lalba\Desktop\Projets\RD\Audit Serveur\audit-serveur\rapports\20261015-1041-audit` contient un rapport d'audit d'un serveur. La structure de `analyse.md` est fixe : « Résumé pour le client », « Diagnostic », « Sécurité et intégrité », « Plan d'action ordonné » (sous-sections « Récapitulatif des fichiers à modifier », « Réglages serveur », « Côté application », « Projets à chiffrer à part »), « À ne pas faire / risques », « Ce qu'on ne sait pas », « Prochaine mesure ».
Les valeurs changent d'un rapport à l'autre : hôte, IP, sites et leur rôle (production, préproduction, sans vhost), chemins, comptes, fichiers, extensions, versions, tailles, taux, directives, valeurs cibles. Extrais-les toutes du fichier. N'en invente aucune et n'en reprends aucune de ce prompt, hormis les valeurs par défaut explicitement fixées ci-dessous.

# Livrable
Un seul fichier bash. Réponds uniquement avec ce fichier, sans aucun texte avant ou après.
Le script est enregistré puis exécuté en root sur le serveur audité, sans aucune modification : aucun placeholder, aucun TODO, aucune variable à renseigner.
Commentaires en français. Chaque bloc indique ce qu'il fait, le constat chiffré de l'analyse qui le justifie, le gain attendu et le retour arrière.
En-tête : hôte et IP visés, usage (dry-run, --apply), phases, constantes réglables avec leur valeur par défaut.

# Périmètre
Inclus :
- Assainissement minimal : sauvegarde, quarantaine, blocage des comptes suspects.
- Toutes les lignes du tableau « Récapitulatif » et de « Réglages serveur » ayant une valeur cible concrète.
- Leviers mémoire : swap, buffer pool InnoDB, OPcache, APCu avec drop-in de cache objet.
- Base WordPress : autoload, transients, défragmentation.
- Diagnostic des tables temporaires, en lecture seule.
- Mesures avant/après.
Exclus, jamais exécutés et listés « non traité » dans le rapport : pm.max_children, memory_limit, tmp_table_size, max_heap_table_size, cache de pages, images, suppression de tables résiduelles, Node.js/PM2, rotation des sels et des mots de passe d'application, et tout élément de « Projets à chiffrer à part ».
Les leviers applicatifs (autoload, transients, OPTIMIZE, drop-in) ne s'appliquent qu'aux sites de production. Les réglages serveur, partagés, valent pour tous.
Chaque point de « À ne pas faire / risques » est une interdiction absolue, sauf si ce prompt le lève sous garde-fou explicite.

# Exécution
- `#!/usr/bin/env bash`, `set -Euo pipefail`, propre sous shellcheck, aucune saisie interactive.
- Sans argument : dry-run. Aucune écriture, aucun rechargement, aucune requête SQL modifiante, aucune installation. Mesures et diagnostics en lecture seule exécutés réellement. Chaque action modifiante est affichée telle qu'elle serait exécutée.
- `--apply` : exécution réelle. Tout autre argument : arrêt avec message d'usage.
- Toute action modifiante, écritures de fichiers comprises, passe par une fonction unique gérant les deux modes.
- Idempotence : une relance ne refait rien de déjà fait et ne duplique aucune directive.
- Un dossier de travail par exécution hors de /var/www, permissions 700, contenant sauvegardes, quarantaine et mesures.
- Sauvegarde horodatée de tout fichier avant modification.
- Installation de paquets : `policy-rc.d` temporaire renvoyant 101 pour bloquer tout redémarrage automatique, retiré ensuite même en cas d'échec, puis `reload` explicite.

# Garde-fous
Une action dont la condition n'est pas satisfaite est ignorée et signalée, jamais forcée.
- Préflight, arrêt total en cas d'échec :
  - exécution en root ;
  - IP ou nom d'hôte conforme à l'analyse ;
  - OS Debian/Ubuntu ;
  - outils requis présents ;
  - espace disque libre ≥ 2 × (taille des racines web + taille des bases).
- Divergence analyse / état réel (élément absent, version ou valeur différente) : action ignorée et signalée.
- Configuration testée avant tout rechargement (`php-fpmX.Y -t`, `apachectl configtest`, validation MariaDB si disponible). Échec : restauration de la sauvegarde, aucun rechargement.
- Seuls les `reload` sont autorisés. Jamais de `restart` de service, jamais de reboot.
- Jamais de `rm` sur des fichiers de site, jamais de DELETE SQL sur des comptes.
- Les phases 2 à 5 ne s'exécutent que si la phase 1 a entièrement réussi, dans cette exécution ou une précédente attestée par un fichier marqueur.

# Découverte des sites
- Racine web déduite des vhosts Apache ; à défaut, recherche de wp-config.php ou .env sous /var/www recoupée avec le nom du site.
- Identifiants BDD et préfixe de tables lus dans wp-config.php ou .env.
- Rôle du site (production, préproduction, sans vhost) et URL publique repris de l'analyse.
- Site non résolu de façon univoque : ignoré et signalé.
- Base partagée entre plusieurs sites : opérations BDD exécutées une seule fois par base.

# Phase 0 — Mesures initiales (lecture seule, aussi en dry-run)
- Mémoire : MemAvailable, swap utilisé, top 10 des processus par VmSwap. L'analyse exige d'identifier le consommateur du swap avant tout arbitrage.
- TTFB : médiane sur 10 requêtes curl vers l'URL publique de chaque site de production (`time_starttransfer`).
- MariaDB : compteurs cumulatifs mesurés en delta sur une fenêtre fixe de 60 s. Les ratios cumulés depuis le démarrage ne mesurent pas l'effet d'un changement.
  - Taux de hit du buffer pool (`Innodb_buffer_pool_read_requests` / `Innodb_buffer_pool_reads`) ;
  - `Created_tmp_disk_tables` / `Created_tmp_tables` ;
  - `Qcache_lowmem_prunes` ;
  - `Threads_running`.
- Par site de production : taille totale de l'autoload, taille des tables citées comme fragmentées.
- Tout est enregistré dans le dossier de travail pour le comparatif final.

# Phase 1 — Assainissement minimal (prérequis), par site
1. Sauvegarde : dump SQL (`--single-transaction`), archive tar de la racine web, copie des journaux Apache cités par l'analyse.
2. wp-cli : téléchargement du phar officiel et vérification de la somme SHA-512 publiée par le projet. Échec : pas d'installation, actions dépendantes ignorées. Toujours exécuté sous l'utilisateur propriétaire de la racine web, jamais avec `--allow-root`.
3. Quarantaine (archive tar puis déplacement en conservant l'arborescence) :
   - tous les fichiers .php sous wp-content/uploads ;
   - les archives listées par l'analyse ;
   - le dossier de l'extension vulnérable, désactivée au préalable si active.
   Un écart entre le nombre de fichiers trouvés et celui de l'analyse est signalé.
4. Comptes administrateurs listés : blocage réversible. Mot de passe aléatoire ni affiché ni stocké, sessions détruites, rôle conservé. Le rapport rappelle que chaque compte doit être confirmé par le client.
5. Écriture du marqueur de réussite.

# Phase 2 — Libérer la mémoire avant d'agrandir
- Query cache : désactivé à chaud (`SET GLOBAL query_cache_type = 0`, `query_cache_size = 0`) et persisté. La RAM libérée entre dans le budget de la phase 3.
- Transparent hugepages : `never` écrit à chaud dans /sys (enabled et defrag), persistance par unité systemd oneshot. GRUB non modifié.
- Purge du swap : `swapoff -a && swapon -a`, uniquement si MemAvailable ≥ swap utilisé + 512 Mo. Sinon ignorée, et tous les leviers de la phase 3 sont ignorés avec elle.

# Phase 3 — Allocation mémoire arbitrée
Condition d'entrée : swap utilisé = 0.
Budget = MemAvailable mesuré à cet instant − 512 Mo de marge.
Les leviers sont alloués dans l'ordre ci-dessous. Chaque hausse n'est appliquée que si elle tient dans le budget restant ; sinon ignorée avec le montant manquant.
1. `innodb_buffer_pool_size` vers la cible de l'analyse : `SET GLOBAL` à chaud (redimensionnement en ligne) et persistance.
2. OPcache vers la cible de l'analyse : `opcache.memory_consumption`, et `opcache.max_accelerated_files` ≥ nombre de fichiers PHP relevé par l'analyse. Écrit dans un fichier dédié du conf.d FPM, puis reload php-fpm.
3. APCu, taille par défaut `apc.shm_size = 32M` :
   - installation du paquet de l'extension pour la version PHP de l'analyse, puis reload php-fpm ;
   - par site de production sans object-cache.php existant : drop-in fourni par une extension du répertoire officiel wordpress.org, intégrité vérifiée par `wp plugin verify-checksums` ;
   - `WP_CACHE_KEY_SALT` unique par site, posé par `wp config set` ;
   - vérification que le cache objet est actif (`wp cache type`).
Persistance MariaDB : fichier dédié de mariadb.conf.d, nom fixe, chargé après les autres. Un levier ignoré n'est ni appliqué à chaud ni persisté.

# Phase 4 — Visibilité PHP-FPM
- `pm.status_path` ajouté au pool si absent.
- URL d'état exposée par une configuration Apache dédiée (conf-available + a2enconf), restreinte à 127.0.0.1, ciblant le socket ou l'adresse lu dans la directive `listen` du pool.
- Vérification par curl local ; relevé de `max children reached`.

# Phase 5 — Base WordPress (sites de production uniquement)
- Transients : suppression des transients expirés et des transients nommés par l'analyse (WordPress les régénère à la demande).
- Autoload : passage en autoload désactivé, via wp-cli, des options nommées par l'analyse comme lourdes. Fonctionnellement sûr : une option non autochargée reste lue à la demande.
- Défragmentation : `OPTIMIZE TABLE` uniquement sur les tables citées comme fragmentées, une par une. Conditions vérifiées avant chaque table :
  - heure locale dans la fenêtre (constantes `OPTIMIZE_START=01`, `OPTIMIZE_END=05`) ;
  - `Threads_running` ≤ `OPTIMIZE_MAX_RUNNING=2` ;
  - espace disque libre ≥ 2 × taille de la table.
  Hors conditions : table ignorée, commande manuelle dans le rapport.
- Tables temporaires, diagnostic en lecture seule : colonnes TEXT/BLOB des tables citées par l'analyse, ratio disque mesuré en phase 0, conclusion sur l'utilité ou non d'une hausse de `tmp_table_size`. Aucune modification.

# Phase 6 — Mesures finales et rapport
- Mêmes mesures qu'en phase 0, même méthode.
- Comparatif avant/après par indicateur et par site : TTFB médian, MemAvailable, swap, hit buffer pool, ratio de tables temporaires disque, purges du query cache, taille de l'autoload, taille et fragmentation des tables, `max children reached`.
- Puis :
  - actions faites ;
  - actions ignorées avec leur raison ;
  - actions manuelles restantes, dont la confirmation client des comptes bloqués ;
  - éléments « non traité » ;
  - commandes de retour arrière ;
  - commande de relance de l'audit indiquée dans « Prochaine mesure ».

# Contrôle avant livraison
Vérifie que :
- chaque valeur provient de analyse.md ou d'un défaut fixé par ce prompt ;
- chaque modification de performance a un constat chiffré, une mesure avant et une mesure après ;
- aucune interdiction de « À ne pas faire » n'est enfreinte hors garde-fou explicite ;
- aucun restart, reboot, rm de fichier de site ou DELETE n'apparaît ;
- le dry-run n'a aucun effet de bord ;
- le script est relançable sans effet cumulatif.