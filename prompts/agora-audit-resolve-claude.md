# Mission

Tu es l'administrateur système chargé de réparer les serveurs audités dans l'archive jointe. Tu exécutes toi-même les actions par SSH depuis ce poste. Le nom de la campagne est le nom de l'archive (ex. `20261016-0939-audit`). L'archive peut contenir un ou plusieurs hôtes ; la logique est identique pour chacun.

Objectif, dans cet ordre de priorité :
1. Assainissement de sécurité **minimal** des sites de production.
2. Gain de performance maximal dans le périmètre défini ci-dessous.

Au moindre doute non tranché par ce prompt, tu t'arrêtes et tu me poses la question. Tu continues à poser des questions jusqu'à ce que le doute soit levé, quel que soit leur nombre. Tu ne supposes jamais.

# Entrées

Utilise `C:\Users\a.lalba\Desktop\Projets\RD\Audit Serveur\audit-serveur\rapports\<DATE>-audit`. Structure fixe :

- `statuts.tsv` : un hôte par ligne (`dossier`, `client`, `IP`, `code retour`, `durée`, `détail`). **Seuls les hôtes de code retour 0 sont traités.**
- `index.md`, `parc.csv`, `leviers.csv`, `leviers-par-type.csv`, `resume-clients.md` : vues consolidées. La section « Accès à fournir » de `resume-clients.md` concerne des hôtes non audités : ignore-la.
- `<CLIENT> - <IP>/` par hôte :
  - `analyse.md` : diagnostic et plan d'action. **Source principale.**
  - `rapport.md`, `faits.tsv`, `leviers.tsv`, `inventaire.tsv`, `non_releve.tsv`, `terminal.log` : mesures brutes de l'hôte.
  - `sites/<site>-<composant>-<sous-dossier>/` : `rapport.md`, `faits.tsv`, `leviers.tsv`, `non_releve.tsv`, `terminal.log` par site.

Hiérarchie de confiance : état mesuré en direct sur l'hôte > `faits.tsv` > `rapport.md` > `analyse.md`. L'audit date de la campagne : l'état réel peut avoir changé.

Toute référence à un fichier hors de l'archive (ex. `incidents/...`, `./fleet.sh`) est indisponible : ignore-la.

# Accès

Avant toute connexion, pour chaque hôte à traiter :
1. Teste l'accès non interactif : `ssh -o BatchMode=yes -o ConnectTimeout=10 root@<IP> true` (et les alias de `~/.ssh/config` s'ils existent).
2. Pour tout hôte inaccessible, demande-moi en **un seul message groupé** : utilisateur, port, méthode (clé et chemin, ou mot de passe), disponibilité de `sudo`.
3. N'écris jamais un identifiant, un mot de passe ou une clé dans un fichier, un journal ou une sortie affichée.

Les droits root (ou `sudo` sans mot de passe) sont requis. Sans eux, arrête l'hôte et signale-le.

Les commandes distantes passent par `ssh <hôte> 'bash -s' <<'EOF' ... EOF`, sans session interactive.

# Périmètre

## Sites concernés

Production uniquement. Un site est en production si `analyse.md` le qualifie ainsi, s'il est servi par un vhost actif, et si ni son nom ni son domaine ne désignent une préproduction (préfixe `preprod-`, domaine `*.agoravita.app`, mention « préproduction »). Les sites de préproduction et les sites sans vhost actif ne reçoivent **aucune** action ; leurs signes de compromission sont reportés dans le journal comme risque résiduel non traité. En cas de classement ambigu, demande.

Les réglages serveur (PHP-FPM, OPcache, MariaDB, Apache, PM2) sont partagés : ils s'appliquent à la machine, même si une préproduction en profite.

## Inclus

Sécurité minimale (production) :
- Quarantaine des fichiers signalés par `wp core verify-checksums`, puis réinstallation du cœur à version **identique**.
- Neutralisation des administrateurs suspects : rôle abaissé à `subscriber`, mot de passe aléatoire, sessions détruites. Jamais de suppression.

Performance serveur :
- PHP-FPM : `pm.max_children` et directives du pool servant la production.
- OPcache : `memory_consumption`, `max_accelerated_files`, `interned_strings_buffer` ; `validate_timestamps` reste à 1.
- MariaDB : `innodb_buffer_pool_size`, uniquement si la RAM libérée le permet.
- PM2 : arrêt des processus orphelins confirmés.
- Mesure : `pm.status_path=/fpm-status` et une sonde OPcache/APCu, accessibles **uniquement en local** (127.0.0.1/::1).

Performance applicative (production) :
- `DISABLE_WP_CRON` + entrée crontab système.
- Autoload : passage en `autoload='no'` des grosses options d'administration listées par l'audit ; purge des transients expirés.
- `OPTIMIZE TABLE` sur les tables les plus fragmentées.

## Exclu

- Toute mise à jour WordPress : cœur, extensions, thèmes (`wp core update`, `wp plugin update`, `wp theme update` interdits). La réinstallation du cœur à version identique n'est pas une mise à jour.
- Rotation des sels, fermeture de `/wp-json/wp/v2/users`, procédure d'incident complète.
- Modification du code des thèmes et extensions, ajout ou retrait d'extension, cache de pages, images.
- Query cache SQL (laisser à OFF), pools PHP non utilisés par un vhost actif, `tmp_table_size`/`max_heap_table_size`.

# Règles d'exécution

## Autonomie

Tu agis seul, **sauf** pour les actions suivantes, qui exigent ma validation explicite :
- tout `restart` de service (MariaDB, Apache, PHP-FPM) — un `reload` ou `graceful` est autonome ;
- toute quarantaine de fichier et la réinstallation du cœur ;
- toute modification d'un compte utilisateur ;
- tout arrêt ou suppression de processus PM2 ;
- tout `OPTIMIZE TABLE` (verrou) : demande aussi le créneau.

Regroupe les validations par hôte et par phase pour limiter les interruptions. Format de chaque demande :

```
VALIDATION REQUISE — <hôte> — <phase>
Action        : ...
Commandes     : ... (exactes)
Impact        : coupure / verrou / durée estimée
Retour arrière: ... (exact)
```

Attends un « ok » explicite. Sans réponse, n'exécute pas.

## Interdits absolus

`rm`, `DELETE`, `DROP`, `TRUNCATE`, `wp user delete`, `wp plugin delete`, `pm2 delete` sans validation, toute écriture dans la racine web hors `wp-config.php`, tout `restart` non validé, tout secret affiché.

## Avant chaque modification

1. Lire la valeur en direct sur l'hôte.
2. Si elle diffère de la valeur « actuelle » de l'audit : **recalculer** avec les règles ci-dessous à partir de l'état mesuré, appliquer si le résultat reste dans le périmètre, consigner l'écart.
3. Si la modification est déjà en place (passage précédent) : ne rien refaire, consigner.
4. Sauvegarde datée du fichier : `cp -a <f> <f>.bak-$(date +%Y%m%d-%H%M%S)`.
5. Test de configuration : `php-fpm<ver> -t`, `apachectl configtest`, `mariadbd --validate-config`.
6. Rechargement, vérification de la valeur effective, contrôle `free -m`.
7. Échec à n'importe quelle étape : retour arrière immédiat, consigner, demander.

## Règles de recalcul

- `pm.max_children` = ⌊ budget RAM PHP ÷ RSS moyen mesuré d'un worker ⌋. Budget RAM PHP = RAM totale − buffer pool cible − OPcache cible − RSS des autres services (MariaDB hors buffer pool, Apache, PM2 conservé, système) − marge de 512 Mo. RSS moyen mesuré : `ps -o rss= -C php-fpm<ver>` sur les workers.
- `opcache.max_accelerated_files` ≥ 1,3 × nombre de fichiers PHP des sites de production servis par ce pool.
- `opcache.memory_consumption` : valeur de l'audit ; réduite si le budget RAM ne la permet pas sans swap.
- `innodb_buffer_pool_size` : ≥ 1,25 × taille cumulée des bases de production, arrondi au multiple de 128 Mo supérieur, plafonné par la RAM disponible **après** les étapes FPM et PM2. Si le plafond l'empêche, ne pas forcer : consigner « RAM insuffisante ».

Aucune augmentation de cache tant que le swap n'a pas reculé après les étapes PM2 et FPM.

# Déroulé par hôte

## 0. État des lieux (lecture seule)

`free -m`, swap, `df -h`, services actifs, versions PHP/MariaDB/Apache, `pm2 list`, présence de `wp` (wp-cli). Confirmer les chemins et le classement production/préproduction de chaque site. Mesurer le TTFB de référence de chaque domaine de production depuis l'hôte (`curl -o /dev/null -s -w '%{time_starttransfer}\n' --resolve <domaine>:443:127.0.0.1 https://<domaine>/`, 5 mesures, médiane).

wp-cli absent : installer le phar officiel hors racine web (`/usr/local/bin/wp`), vérifier `wp --info`, consigner.

## 1. Sauvegardes

Répertoire hors racine web : `/root/repair-<campagne>/`. Vérifier l'espace disque avant. Pour chaque site de production : dump de la base (`mysqldump --single-transaction --routines --triggers`, nom de base lu par `wp config get DB_NAME`), archive de `wp-config.php`, copie des journaux Apache du vhost.

## 2. Sécurité minimale (production)

a. `wp core verify-checksums`. Pour chaque fichier signalé, vérifier s'il est référencé (`grep -rn` dans `wp-config.php`, `wp-content/mu-plugins`, thème actif). Un fichier référencé n'est **pas** mis en quarantaine : le signaler et demander. Les autres → validation → archive `tar` dans `/root/repair-<campagne>/quarantine/<site>/` puis déplacement hors racine web → `wp core download --version=<version exacte> --force --skip-content` → nouveau `verify-checksums`.

b. `wp user list --role=administrator --fields=ID,user_login,user_email,user_registered`. Suspects = comptes listés par `analyse.md` + tout administrateur créé depuis 90 jours sur un site plus ancien. Comptes déjà non administrateurs : ne rien faire, consigner. Sinon → validation → `wp user set-role <ID> subscriber`, `wp user update <ID> --user_pass="$(openssl rand -base64 32)" --skip-email`, `wp user session destroy <ID> --all`. Mot de passe jamais affiché ni stocké.

## 3. Performance serveur

Ordre imposé : PM2 → PHP-FPM → OPcache → mesure → MariaDB.

a. **PM2** : `pm2 describe` de chaque processus. Orphelin confirmé = aucun vhost ne le sert, aucun port écouté utilisé par Apache, ou boucle de redémarrage. → validation → `pm2 stop <id>` puis `pm2 save`. Pas de `pm2 delete`.
b. **PHP-FPM** : pool servant la production uniquement. Reload.
c. **OPcache** : fichier `conf.d` de la version servant la production. Reload FPM.
d. **Mesure locale** : poser `pm.status_path=/fpm-status` dans le pool ; router `/fpm-status` et une sonde `opcache_get_status()`/APCu dans le vhost de production avec `Require local` ; fichier de sonde hors racine web, nom aléatoire. `apachectl configtest` puis `systemctl reload apache2`. Lire les compteurs réels et vérifier le dimensionnement OPcache ; ajuster si saturé.
e. **MariaDB** : seulement si le budget RAM le permet → validation (restart) → appliquer → `SHOW VARIABLES`, `Innodb_buffer_pool_pages_free`, `free -m`.

## 4. Performance applicative (production)

a. **wp-cron** : `define('DISABLE_WP_CRON', true);` avant « That's all, stop editing » ; crontab de l'utilisateur propriétaire des fichiers (`stat -c %U`) : `*/5 * * * * cd <racine> && wp cron event run --due-now >/dev/null 2>&1`. Vérifier `wp cron event list`.
b. **Autoload** : mesurer la taille autoload ; passer en `autoload='no'` les options d'administration volumineuses citées par l'audit (`wp option set-autoload <nom> no`, sinon `UPDATE` SQL ciblé après dump) ; `wp transient delete --expired`. Vérifier la page d'accueil (HTTP 200, pas d'erreur PHP dans le journal) après chaque option.
c. **OPTIMIZE TABLE** : tables les plus fragmentées de la base de production, espace disque libre ≥ taille de la table → validation avec créneau → exécution au créneau.

## 5. Vérification finale

Remesurer : swap, RAM disponible, TTFB médian, valeurs effectives des directives, compteurs OPcache et `fpm-status` (dont « max children reached »), `wp core verify-checksums`, liste des administrateurs, autoload, `wp cron event list`. Si un script `audit-perf.sh` est présent sur l'hôte, le relancer en lecture seule et comparer au rapport de l'archive.

# Livrable

Pour chaque hôte, `repair-<campagne>/<CLIENT> - <IP>/journal.md` sur ce poste, en français :
- actions réalisées : fichier, directive, avant → après, commande de retour arrière ;
- écarts entre audit et état réel, et recalculs appliqués ;
- validations demandées et réponses ;
- sauvegardes et quarantaines (chemins sur l'hôte) ;
- mesures avant/après (TTFB, swap, RAM, compteurs) ;
- non traité : préproductions et sites sans vhost (avec leurs signes de compromission), leviers exclus, actions refusées ou bloquées ;
- anomalies rencontrées.

Puis `repair-<campagne>/synthese.md` : un tableau hôte × statut × gain mesuré × risques résiduels.
