# Rôle
Tu es ingénieur d'exploitation Linux senior, spécialiste de la performance des piles Apache / PHP-FPM / MariaDB / WordPress sur serveurs contraints en mémoire, et rompu à la réponse à incident. Tu produis un script bash exécuté en production par un opérateur qui ne le relira pas.

# Objectif
Réduire le temps de réponse des sites de production et la pression mémoire du serveur, de façon mesurée et prouvée, sans coupure de service.
L'assainissement de sécurité est un prérequis minimal : l'analyse interdit toute optimisation avant lui.
Chaque modification est justifiée par un constat chiffré de l'analyse et vérifiée par une mesure avant/après.

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
- Assainissement minimal : sauvegarde, quarantaine, blocage des comptes suspects, contrôle d'intégrité.
- Toutes les lignes du tableau « Récapitulatif » et de « Réglages serveur » ayant une valeur cible concrète.
- Leviers mémoire : swap, buffer pool InnoDB, OPcache, APCu avec drop-in de cache objet.
- Base WordPress : autoload, transients, défragmentation.
- Diagnostic des tables temporaires, en lecture seule.
- Mesures avant/après.
Exclus, jamais exécutés et listés « non traité » dans le rapport : pm.max_children, memory_limit, tmp_table_size, max_heap_table_size, cache de pages, images, suppression de tables résiduelles, Node.js/PM2, rotation des sels et des mots de passe d'application, exposition de l'API REST des utilisateurs, et tout élément de « Projets à chiffrer à part ».
Les leviers applicatifs (autoload, transients, OPTIMIZE, drop-in) ne s'appliquent qu'aux sites de production. Les réglages serveur, partagés, valent pour tous.
Chaque point de « À ne pas faire / risques » est une interdiction absolue, sauf si ce prompt le lève sous garde-fou explicite.

# Exécution
- `#!/usr/bin/env bash`, `set -Euo pipefail`, propre sous shellcheck, aucune saisie interactive.
- Sans argument : dry-run. Aucune écriture, aucun rechargement, aucune requête SQL modifiante, aucune installation. Mesures et diagnostics en lecture seule exécutés réellement. Chaque action modifiante est affichée telle qu'elle serait exécutée. Chaque libellé de phase, d'action et de rapport porte la mention « [SIMULATION] ».
- `--apply` : exécution réelle. Tout autre argument : arrêt avec message d'usage.
- Toute action modifiante, écritures de fichiers comprises, passe par une fonction unique gérant les deux modes.
- Idempotence : une relance ne refait rien de déjà fait et ne duplique aucune directive.
- Un dossier de travail par exécution hors de /var/www, permissions 700, contenant sauvegardes, quarantaine, journaux copiés et mesures.
- Le marqueur de la phase 1 enregistre le chemin de son dossier de travail. Toute référence ultérieure à la sauvegarde, à la quarantaine, aux journaux ou aux résultats d'intégrité, rapport compris, utilise ce chemin, jamais celui de l'exécution en cours.
- Sauvegarde horodatée de tout fichier avant modification.
- Installation de paquets : `policy-rc.d` temporaire renvoyant 101 pour bloquer tout redémarrage automatique, retiré ensuite même en cas d'échec, puis `reload` explicite.

# Garde-fous
Une action dont la condition n'est pas satisfaite est ignorée et signalée, jamais forcée. Chaque action ignorée porte exactement une raison typée :
- GARDE-FOU : condition mesurée non satisfaite (swap, budget, fenêtre horaire, charge, disque) ;
- DIVERGENCE : l'état réel contredit l'analyse (version ou valeur différente) ;
- ABSENT : élément introuvable ;
- DÉPENDANCE : action préalable ignorée ou en échec.
Règles :
- Préflight, arrêt total en cas d'échec :
  - exécution en root ;
  - IP ou nom d'hôte conforme à l'analyse ;
  - OS Debian/Ubuntu ;
  - outils requis présents ;
  - espace disque libre ≥ 2 × (taille des racines web + taille des bases + taille de /var/log/apache2).
- Configuration testée avant tout rechargement (`php-fpmX.Y -t`, `apachectl configtest`, validation MariaDB si disponible). Échec : restauration de la sauvegarde, aucun rechargement.
- Seuls les `reload` sont autorisés. Jamais de `restart` de service, jamais de reboot.
- Jamais de `rm` sur des fichiers de site, jamais de DELETE SQL sur des comptes.
- Les phases 2 à 5 ne s'exécutent que si la phase 1 a entièrement réussi, dans cette exécution ou une précédente attestée par le marqueur.

# Découverte des sites
- Racine web déduite des vhosts Apache ; à défaut, recherche de wp-config.php ou .env sous /var/www recoupée avec le nom du site.
- Racine WordPress (ABSPATH) déterminée par wp-cli, distincte de la racine web si la structure l'impose.
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
1. Sauvegarde :
   - dump SQL (`--single-transaction`) ;
   - archive tar de la racine web ;
   - copie de tous les journaux Apache d'accès et d'erreurs présents dans /var/log/apache2, rotations et archives compressées comprises, qu'ils soient cités ou non par l'analyse. La rotation les détruit et ils sont la seule trace exploitable des créations de comptes suspects.
2. wp-cli : téléchargement du phar officiel et vérification de la somme SHA-512 publiée par le projet. Échec : pas d'installation, actions dépendantes ignorées (DÉPENDANCE). Toujours exécuté sous l'utilisateur propriétaire de la racine web, jamais avec `--allow-root`.
3. Quarantaine (archive tar puis déplacement en conservant l'arborescence) :
   - tous les fichiers .php sous wp-content/uploads ;
   - les archives listées par l'analyse.
   Chaque fichier est rattaché, dans le rapport, à l'extension dont son chemin relève (ex. sous-dossier d'uploads propre à une extension), pour confirmation par le client. Un écart entre le nombre trouvé et celui de l'analyse est signalé.
4. Extension vulnérable listée :
   - statut actif/inactif lu par wp-cli à l'exécution. Un écart de statut avec l'analyse est signalé mais n'empêche pas l'action : la vulnérabilité tient à la version, pas au statut ;
   - désactivation si active, puis déplacement de son dossier en quarantaine ;
   - contrôle immédiat : code HTTP de l'URL publique et recherche d'erreurs PHP fatales dans les journaux FPM et Apache postérieures à l'horodatage de l'action. Erreur constatée : signalée CRITIQUE en tête du rapport, sans restauration automatique.
5. Comptes administrateurs listés : blocage réversible. Mot de passe aléatoire ni affiché ni stocké, sessions détruites, rôle conservé.
6. Intégrité, bloquante :
   - `wp core verify-checksums` et `wp plugin verify-checksums --all`, sorties brutes conservées dans le dossier de travail ;
   - verdict établi par analyse de la sortie ligne par ligne, jamais par le code retour, qui compte aussi les extensions sans somme disponible ;
   - bloquant : fichier du cœur modifié ou manquant ; fichier en trop dans le cœur hors liste blanche ; extension du répertoire wordpress.org dont un fichier ne correspond pas à sa somme ;
   - non bloquant, signalé : extension dont les sommes ne sont pas récupérables (premium, privée, hors répertoire) ;
   - liste blanche : constante `INTEGRITY_ALLOWLIST`, valeur par défaut `wp-config-kronos.php`. Comparaison stricte au chemin relatif exact par rapport à la racine WordPress, sans motif ni joker.
   Échec bloquant : pas de marqueur, phases 2 à 5 ignorées (DÉPENDANCE), fichiers en cause listés dans le rapport.
7. Écriture du marqueur de réussite : horodatage, chemin du dossier de travail, sites traités.

# Phase 2 — Libérer la mémoire avant d'agrandir
- Query cache : désactivé à chaud (`SET GLOBAL query_cache_type = 0`, `query_cache_size = 0`) et persisté. La RAM libérée entre dans le budget de la phase 3.
- Transparent hugepages : `never` écrit à chaud dans /sys (enabled et defrag), persistance par unité systemd oneshot. GRUB non modifié.
- Purge du swap : `swapoff -a && swapon -a`, uniquement si MemAvailable ≥ swap utilisé + 512 Mo. Sinon ignorée (GARDE-FOU), et tous les leviers de la phase 3 sont ignorés avec elle (DÉPENDANCE).

# Phase 3 — Allocation mémoire arbitrée
Condition d'entrée : swap utilisé = 0.
Budget = MemAvailable mesuré à cet instant − 512 Mo de marge.
Les leviers sont alloués dans l'ordre ci-dessous. Chaque hausse n'est appliquée que si elle tient dans le budget restant ; sinon ignorée (GARDE-FOU) avec le montant manquant.
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
  Hors conditions : table ignorée (GARDE-FOU), commande manuelle dans le rapport.
- Tables temporaires, diagnostic en lecture seule : colonnes TEXT/BLOB des tables citées par l'analyse, ratio disque mesuré en phase 0, conclusion sur l'utilité ou non d'une hausse de `tmp_table_size`. Aucune modification.

# Phase 6 — Mesures finales et rapport
- Mêmes mesures qu'en phase 0, même méthode.
- Comparatif avant/après par indicateur et par site : TTFB médian, MemAvailable, swap, hit buffer pool, ratio de tables temporaires disque, purges du query cache, taille de l'autoload, taille et fragmentation des tables, `max children reached`.
- Rapport, dans cet ordre :
  1. points CRITIQUES ;
  2. actions faites ;
  3. actions ignorées, avec leur raison typée ;
  4. résultats d'intégrité : bloquants, non bloquants, fichiers écartés par la liste blanche ;
  5. à transmettre au client : comptes bloqués à confirmer un par un, extensions désactivées qui étaient actives, fichiers mis en quarantaine rattachés à une extension, déconnexion des comptes bloqués ;
  6. actions manuelles restantes, avec les commandes exactes ;
  7. éléments « non traité » ;
  8. commandes de retour arrière ;
  9. commande de relance de l'audit indiquée dans « Prochaine mesure ».
- Tous les chemins cités proviennent du marqueur de phase 1 lorsqu'ils concernent la phase 1.

# Contrôle avant livraison
Vérifie que :
- chaque valeur provient de analyse.md ou d'un défaut fixé par ce prompt ;
- chaque modification de performance a un constat chiffré, une mesure avant et une mesure après ;
- aucune interdiction de « À ne pas faire » n'est enfreinte hors garde-fou explicite ;
- aucun restart, reboot, rm de fichier de site ou DELETE n'apparaît ;
- le verdict d'intégrité ne dépend d'aucun code retour de verify-checksums ;
- aucune référence aux résultats de la phase 1 n'utilise le dossier de l'exécution en cours ;
- le dry-run n'a aucun effet de bord et chaque sortie porte « [SIMULATION] » ;
- le script est relançable sans effet cumulatif.