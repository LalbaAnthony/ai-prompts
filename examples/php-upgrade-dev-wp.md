# Task: Migrate this WordPress codebase between PHP versions

## Context
- Migration: 8.3 -> 8.5          <!-- the ONLY line to change between migrations: FROM -> TO -->
- Environment: Debian development server. You are running inside the project repository.
- Runtime status: the PHP runtime has already been switched to the target version. WordPress core, plugins, themes, application code and dependencies have NOT been updated.
- Site URL: <!-- <URL> -->
- Key routes to smoke-test: <!-- <ROUTES or "discover from permalinks and sitemap"> -->
- Project-owned code: <!-- <PATHS or "detect"> e.g. wp-content/themes/acme, wp-content/plugins/acme-*, wp-content/mu-plugins/acme.php -->
- Must still run on the source version: <!-- <YES/NO> -->
- Database changes allowed (core, plugin and theme upgrade routines): <!-- <YES/NO> -->

## Step 0 — Derive migration parameters (before anything else)
From the `Migration` line, compute and write at the top of the final report:
- `FROM` / `TO`: source and target versions (major.minor).
- `DIRECTION`: `upgrade` if TO > FROM, `downgrade` if TO < FROM. If equal, stop and report.
- `PATH`: every released minor version strictly after the lower version, up to and including the higher one, in order. Include major boundaries (e.g. 7.4 -> 8.1 gives 8.0, 8.1).
- `GUIDES`: for each version `X.Y` in `PATH`, `https://www.php.net/manual/en/migrationXY.php` (e.g. 8.0 -> `migration80.php`). If a guide does not exist (unreleased or RC version), use the `UPGRADING` file of the matching `php-src` branch instead. If neither exists, stop and report.
- `TO_ID`: `major*10000 + minor*100` (e.g. 8.5 -> `80500`). Used by static analysis tools.
- `TEST_VERSION`: `<TO>-`, or `<lowest>-<highest>` of FROM and TO when the source version must still be supported. Used by PHPCompatibility.
- `BRANCH`: `chore/php-<TO>-migration`.
- `COMMIT_PREFIX`: `chore(php<TO>):`.
- `CONSTRAINT`: `^<TO>`, or `^<FROM> || ^<TO>` when the source version must still be supported. For a downgrade, use the lowest version.
- `INI_DIRS`: `/etc/php/<FROM>/` and `/etc/php/<TO>/`.
- `RECTOR_SET`, where `XY` is `TO` without the dot (e.g. 8.5 -> `85`):
  - upgrade: `->withPhpSets(phpXY: true)`
  - downgrade: `->withSets([\Rector\Set\ValueObject\DowngradeLevelSetList::DOWN_TO_PHP_XY])`

Every later step uses these values. No other step contains a hard-coded version.

## Goal (definition of done)
1. The site runs on `TO` (front end, wp-admin, REST API, AJAX, WP-CLI, cron) with no fatal errors and no warnings.
2. Zero PHP deprecation notices originating from project-owned code, including notices raised inside core or third-party code but caused by a project call site.
3. Functional behavior unchanged: tests and HTTP responses match the baseline.

## Code ownership (applies to every step)
Every PHP file belongs to exactly one class:
- `project`: themes, plugins, mu-plugins and configuration written for this site (`wp-config.php`, Bedrock `config/`, the paths in Context `Project-owned code`). The only code that may be edited.
- `core`: WordPress core (`wp-admin/`, `wp-includes/`, root `wp-*.php` except `wp-config.php`, or the core directory of a Bedrock-style layout). Never edit.
- `third-party`: plugins, themes, mu-plugins and drop-ins (`object-cache.php`, `advanced-cache.php`, `db.php`, etc.) from wordpress.org, commercial vendors or marketplaces, plus every `vendor/` directory. Never edit. Update only as described in Step 4.

When ownership is uncertain, classify as `third-party` and list the item in the report.

## Scope
- In scope: `project` code, every project `composer.json` / `composer.lock` (root and per plugin/theme), `Requires PHP` headers of project plugins and themes, tests, CI/Docker files referencing the PHP version, and core/third-party updates under the rules of Step 4.
- Out of scope: business logic changes, unrelated refactors, new features, style-only changes, content, options and settings changes.
- Out of scope for upgrades: adopting syntax or APIs introduced in `PATH`, unless required for compatibility.
- Never edit `core` or `third-party` files, even to silence a notice.
- Never modify system packages or files outside the repository (apt, systemctl, /etc). Exceptions: `TOOLS_DIR` (Step 1), and the temporary debug setup of Step 2 when `wp-config.php` or the mu-plugins directory of the same WordPress install lives outside the repository; restore those in Step 7. List every other required system action in the final report.
- Database: only read-only WP-CLI listing commands, `wp db export`, `wp cache flush`, and, only when `Database changes allowed` is YES, the upgrade routines triggered by Step 4 updates.

## Step 1 — Inventory (read-only)
1. Run `php -v`, `php -m` and `php --ini`. Confirm the CLI runs `TO` and list the loaded extensions.
2. Determine `TOOLS_DIR`:
   - `.tools/` when the repository root is not inside the web document root (from the vhost config). Add it to `.git/info/exclude`: `grep -qxF '.tools/' .git/info/exclude || echo '.tools/' >> .git/info/exclude`.
   - otherwise a directory created with `mktemp -d`, so that tools, logs and the database dump are never web-accessible.
   Record the absolute path. Never commit `TOOLS_DIR`.
3. WP-CLI: if `wp` is not on `PATH`, download the phar into `TOOLS_DIR` (`curl -sSLo <TOOLS_DIR>/wp-cli.phar https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar`) and use `php <TOOLS_DIR>/wp-cli.phar` wherever this prompt says `wp`. Run `wp --info` and confirm it uses the `TO` binary. Run WP-CLI as the owner of the WordPress files; use `--allow-root` only if root owns them.
4. Identify the SAPI serving <URL> (php<TO>-fpm, Apache mod_php, etc.) from the web server vhost config. Confirm it uses `TO`, not a leftover `FROM` pool or socket.
5. Diff the enabled extensions and custom ini directives between the two `INI_DIRS`. List the missing ones.
6. Detect the layout: classic install (`wp-config.php` at root or one level up), Bedrock-style (`web/wp`, `web/app`, `config/application.php`), repository limited to `wp-content`, single plugin, or single theme. Record `wp eval 'echo ABSPATH, PHP_EOL, WP_CONTENT_DIR, PHP_EOL;'` and whether core, plugins and themes are tracked in Git, installed by Composer, or neither.
7. WordPress inventory:
   - `wp core version --extra` and `wp core verify-checksums`.
   - `wp plugin list --fields=name,status,version,update_version` (includes `must-use` and `dropin` entries) and `wp plugin verify-checksums --all`.
   - `wp theme list --fields=name,status,version,update_version`, including parent/child relationships.
   - Multisite: `wp site list --fields=blog_id,url` if `wp core is-installed --network` succeeds.
8. Classify every plugin, theme, mu-plugin and drop-in per Code ownership. Signals for `third-party`: wordpress.org checksums match, installed via `wpackagist` or a vendor Composer repository, an `Update URI` or update library pointing to a vendor, a commercial vendor header. Classify as `project` only when listed in Context or clearly authored for this site (no external update source, own prefix, development history in this repository).
9. Record the `Requires PHP`, `Requires at least` and `Tested up to` headers of every plugin and theme (main file, `readme.txt`, `style.css`).
10. Detect entry points of project code: templates, REST routes (`register_rest_route`), AJAX handlers (`wp_ajax_`, `wp_ajax_nopriv_`), WP-CLI commands (`WP_CLI::add_command`), cron hooks (`wp cron event list`), system crontab entries calling `wp-cron.php` or WP-CLI, background workers. Detect the test suites (PHPUnit with `wp-phpunit` or the WordPress test library, Pest, wp-browser) and static analysis tools (PHPStan with WordPress stubs, Psalm, PHPCS with WordPress or PHPCompatibility standards).
11. Detect whether `rector/rector` is already a project dependency, and record its installed version and any existing `rector.php`.
12. Read `require.php` and `config.platform.php` in every project `composer.json`. If there is none, record "no Composer".
13. Record the debug and cache configuration: `WP_DEBUG`, `WP_DEBUG_LOG`, `WP_DEBUG_DISPLAY`, `SCRIPT_DEBUG`, `WP_ENVIRONMENT_TYPE`, `DISABLE_WP_CRON` (`wp config get <NAME>`, or `.env` / `config/environments/*` for Bedrock), the object-cache drop-in, the page-cache plugin and its purge command, and the OPcache settings of the `TO` SAPI.

## Step 2 — Baseline
1. Check that the working tree is clean. If it is not, stop and report.
2. Create `BRANCH`.
3. Export the database: `wp db export <TOOLS_DIR>/backup/baseline.sql`. It contains personal data: never commit it, never copy it elsewhere.
4. Enable debug logging temporarily, using the project's own mechanism (`wp config set` for `wp-config.php`, `.env` or environment config for Bedrock): `WP_DEBUG` true, `WP_DEBUG_LOG` set to `<TOOLS_DIR>/logs/debug.log`, `WP_DEBUG_DISPLAY` false. Record the original values. Never commit these changes.
5. Flush caches so requests reach PHP: `wp cache flush` and the page-cache plugin's purge command if any.
6. Run the test suite and static analysis and record the results, so that pre-existing failures can later be told apart from new ones. If the WordPress test suite requires creating a database or installing system packages, do not do it: record the suite as unavailable and list the requirement.
7. Record the HTTP status codes for <URL> and the key routes, bypassing page caches: `curl -sS -o /dev/null -w '%{http_code}\n' -H 'Cache-Control: no-cache' '<ROUTE>?nocache=<timestamp>'`. When discovering routes, include at least:
   - front page, one published post and one page (`wp post list --post_type=post,page --post_status=publish --posts_per_page=3 --field=url`), one archive, a search (`/?s=test`), a 404;
   - `/wp-login.php`, `/wp-admin/` (redirect expected), `/wp-json/`, `/feed/`, the sitemap (`/wp-sitemap.xml` or the SEO plugin's);
   - every GET REST route registered by project code;
   - key pages of major active plugins (e.g. shop, cart and account pages for WooCommerce);
   - for multisite, the front page of every site.
8. The baseline already runs on `TO` with unmigrated code. For each route returning 5xx, look up its status before the runtime switch in the web server access logs; that pre-switch status becomes the expected result. If it cannot be found, expect 2xx/3xx.
9. Record the current entries of the PHP error log, the web server error log and `debug.log`. Find the log paths from the ini and vhost config. Record `wp cron event list`.

## Step 3 — Build the migration checklist
1. Read every guide in `GUIDES`, in `PATH` order.
2. Extract every applicable item from these sections: backward-incompatible changes, deprecated features, removed or unbundled extensions, changed function signatures, removed or changed ini directives.
3. For a downgrade, extract instead every feature introduced in `PATH` (new syntax, functions, classes, constants, extensions). Each one used by project code must be replaced.
4. Run PHPCompatibility, installed in isolation:
   ```bash
   mkdir -p <TOOLS_DIR>/phpcs
   echo '{"config":{"allow-plugins":{"dealerdirect/phpcodesniffer-composer-installer":true}}}' > <TOOLS_DIR>/phpcs/composer.json
   composer require --dev phpcompatibility/phpcompatibility-wp --working-dir=<TOOLS_DIR>/phpcs --no-interaction -W
   <TOOLS_DIR>/phpcs/vendor/bin/phpcs -p -s --standard=PHPCompatibilityWP --runtime-set testVersion <TEST_VERSION> \
     --extensions=php --ignore='*/vendor/*,*/node_modules/*' --report=json <PROJECT_PATHS> > <TOOLS_DIR>/phpcs/project.json
   ```
   - Check from its changelog that the installed release covers `TO`. If it does not, try the latest development release of `phpcompatibility/php-compatibility`; if it still does not, record the gap and rely on the guides and Rector for the missing versions.
   - Run the same command on third-party plugin and theme paths into `third-party.json`. These findings are informative only and feed Step 4.
5. Search project code for each guide item and keep only those that match. Merge with the PHPCompatibility findings.
6. Write the resulting checklist into the report before editing. Mark each item with its source version and its origin (guide, PHPCompatibility, or both).

## Step 4 — Core, plugins, themes and Composer dependencies
Allowed updates:
- Core: maintenance releases (`x.y.Z`) within the installed branch only. A new branch (`x.Y`) is a major release.
- Plugins and themes: releases that keep the first version number, whose changelog announces no breaking change and no data migration, and whose `Requires at least` does not exceed the installed core version. Anything else is a major release.
- Composer packages: patch or minor releases only.

Update only what is incompatible with `TO` or emits `TO` notices. Do not update compatible components.

1. Core: check the installed version against https://make.wordpress.org/core/handbook/references/php-compatibility-and-wordpress-versions/.
   - If the installed branch does not support `TO`, apply its latest maintenance release (`wp core update --minor`, or `composer update` of the core package within its constraint for Bedrock-style layouts).
   - If support for `TO` still requires a new branch, do not apply it: stop and report the required branch.
   - If core is not managed by this repository, list the update as a system action.
2. Third-party plugins and themes: for each one, decide compatibility with `TO` from its `Requires PHP` header, changelog, `third-party.json` and baseline log entries. Update incompatible ones to the closest allowed release: `wp plugin update <slug> --version=<version>` (or `--patch` / `--minor`), `wp theme update <slug> --version=<version>`, or `composer update <package> --with-dependencies` when installed by Composer. Run `--dry-run` first where supported. List license-gated premium updates as manual actions.
3. If an update requires a database upgrade (`wp core update-db`, plugin or theme upgrade routines) and `Database changes allowed` is NO, revert that update and report it.
4. Commit each tracked update separately: `COMMIT_PREFIX update <slug> <old> -> <new>`.
5. For each project `composer.json`:
   - Run `composer check-platform-reqs` and `composer why-not php <TO>`.
   - Set `require.php` to `CONSTRAINT`. Align `config.platform.php` with `TO`.
   - Move each blocking package to the closest version that supports `TO`, using patch or minor releases only.
6. Project plugin and theme headers: set `Requires PHP` (main file, `readme.txt`, `style.css`) to the lowest version allowed by `CONSTRAINT` when the current header is higher than that version, or when the source version is no longer supported. Otherwise leave it unchanged.
7. Do not apply major releases. List each one with its breaking changes, required WordPress version and data migrations.

## Step 5 — Code migration (project code only)

### 5.1 Rector pass (mandatory)
Rector is required for this migration. Install it if it is missing.

Installation:
1. If the project already requires `rector/rector` and the installed version provides `RECTOR_SET`, use it as-is.
2. Otherwise, install an isolated copy without touching project dependencies, together with WordPress stubs so Rector can resolve core types:
   ```bash
   mkdir -p <TOOLS_DIR>/rector
   composer require --dev rector/rector php-stubs/wordpress-stubs --working-dir=<TOOLS_DIR>/rector --no-interaction
   ```
   Add stubs for major plugins the project code extends when they exist (e.g. `php-stubs/woocommerce-stubs`, `php-stubs/acf-pro-stubs`).
3. Check that the installed version provides `RECTOR_SET`.
   - If it does not, run `composer update rector/rector --working-dir=<TOOLS_DIR>/rector`.
   - If it still does not, use the highest available set below `TO`, cover the missing versions manually in 5.2, and record the gap in the report.
4. Never add Rector or the stubs to a project `composer.json` and never commit `TOOLS_DIR`.

Configuration:
- Do not modify an existing project `rector.php`.
- Create `<TOOLS_DIR>/rector/rector.php`:
  ```php
  <?php

  declare(strict_types=1);

  use Rector\Config\RectorConfig;

  return RectorConfig::configure()
      // Project-owned plugin, theme and mu-plugin dirs detected in Step 1 (absolute paths)
      ->withPaths([])
      // vendor/, node_modules/, build output, caches, compiled templates
      ->withSkip([])
      // Existing vendor/autoload.php files of project plugins/themes (absolute paths)
      ->withAutoloadPaths([])
      // WordPress and plugin stubs; never bootstrap wp-load.php (it boots WordPress and connects to the database)
      ->withBootstrapFiles([__DIR__ . '/vendor/php-stubs/wordpress-stubs/wordpress-stubs.php'])
      // Insert RECTOR_SET here (discovery pass only)
  ;
  ```

Execution, from the project root:
1. Discovery: run `<TOOLS_DIR>/rector/vendor/bin/rector process --config <TOOLS_DIR>/rector/rector.php --dry-run --output-format=json > <TOOLS_DIR>/rector/discovery.json`.
2. Classify each rule listed under `applied_rectors`:
   - Compatibility: it fixes a Step 3 checklist item (deprecation, removal, backward-incompatible change).
   - Modernization: it introduces optional newer syntax or APIs.
   - For a downgrade, every rule in the set counts as compatibility.
3. Apply:
   - Replace `RECTOR_SET` in the config with `->withRules([...])`, listing compatibility rules only.
   - Run the same command without `--dry-run`.
4. Review the full diff. Revert any hunk that changes behavior. Check with particular care hunks touching hook callback signatures (parameter count must match `$accepted_args` in `add_action` / `add_filter`), `$wpdb` queries, serialized or stored data, and template files mixing PHP and HTML.
5. Commit with the message `COMMIT_PREFIX apply Rector compatibility rules`.
6. Record each discarded modernization rule, with its number of occurrences.

### 5.2 Manual pass
1. Apply the remaining Step 3 checklist items not resolved by Rector, in `PATH` order.
2. Follow these rules:
   - Fix root causes. Never suppress errors with `@`, `error_reporting()` changes, ini tweaks, or WordPress filters such as `deprecated_function_trigger_error` and `doing_it_wrong_trigger_error`.
   - Prefer real fixes over opt-in attributes or shims that only silence a notice. When one is unavoidable, record why.
   - When a notice fires in core or third-party code because of a project call site (e.g. an invalid argument passed to a core function), fix the call site.
   - Use no WordPress API newer than the `Requires at least` header of the file's plugin or theme.
   - If the source version must still be supported, use no syntax or API missing from the lowest version.
   - Make one commit per change category, with the message `COMMIT_PREFIX <description>`.

## Step 6 — Verification
1. Lint project code and expect zero errors:
   ```bash
   find <PROJECT_PATHS> \( -name vendor -o -name node_modules \) -prune -o -name '*.php' -print0 \
     | xargs -0 -n1 -P4 php -l | grep -v '^No syntax errors'
   ```
   Run the same on third-party paths. A parse error there is a blocker; report it.
2. Logs, with the Step 2 debug setup active (`WP_DEBUG` true makes WordPress use `E_ALL`):
   - Add a temporary mu-plugin to attribute notices raised inside core or third-party code. Add its path to `.git/info/exclude`; never commit it:
     ```php
     <?php
     // Temporary migration helper: logs a backtrace for each notice. Never commit.
     set_error_handler(function ($errno, $errstr, $errfile, $errline) {
         if ($errno & (E_DEPRECATED | E_USER_DEPRECATED | E_WARNING | E_USER_WARNING | E_NOTICE | E_USER_NOTICE)) {
             $frames = array();
             foreach (array_slice(debug_backtrace(DEBUG_BACKTRACE_IGNORE_ARGS), 1, 15) as $frame) {
                 $frames[] = (isset($frame['file']) ? $frame['file'] : '?') . ':' . (isset($frame['line']) ? $frame['line'] : '?');
             }
             error_log(sprintf("[%d] %s at %s:%d\n  %s\n", $errno, $errstr, $errfile, $errline, implode("\n  ", $frames)), 3, '<TOOLS_DIR>/logs/trace.log');
         }
         return false; // keep PHP and WordPress default handling
     });
     ```
   - Exercise every Step 2 route and CLI entry point, then grep the PHP error log, the web server error log, `debug.log` and `trace.log` for `Deprecated|Warning|Fatal|Notice`.
   - Attribute each entry to `project`, `core` or `third-party`. An entry whose trace passes through a project call site counts as `project`.
   - Keep WordPress API notices (`_deprecated_function`, `_deprecated_argument`, `_doing_it_wrong`, messages citing a WordPress version) separate from PHP engine notices. They are out of scope unless introduced by a Step 4 update; report them.
3. Run the tests: no new failures compared with the baseline.
4. Run static analysis targeting `TO_ID` (e.g. PHPStan `phpVersion: TO_ID` with WordPress stubs): no new errors. Re-run PHPCompatibility on project paths with `TEST_VERSION`: zero errors.
5. Smoke-test over HTTP: flush caches as in Step 2, then request <URL> and every Step 2 route on every site. Status codes match the expected baseline and no new log entries appear. List the key admin screens (dashboard, editor, project settings pages) as manual checks in the report. If `opcache.validate_timestamps=0`, list a reload of the `TO` FPM service as a required system action.
6. CLI: `wp eval 'echo "ok", PHP_EOL;'` boots cleanly. Run each project WP-CLI command with `wp help <command>`, and with `--dry-run` where supported. `wp cron event list` succeeds. Do not run cron events (side effects such as emails or payments). Lint and run with `--help` any system cron script.
7. Run `wp core verify-checksums` and `wp plugin verify-checksums --all`: core and third-party code are unchanged apart from Step 4 updates.

## Step 7 — Cleanup (after Step 6 passes)
1. Delete the temporary mu-plugin and its `.git/info/exclude` entry.
2. Restore the debug constants to their recorded values. Confirm `git status` shows no change to `wp-config.php`, `.env` or environment config.
3. Delete `<TOOLS_DIR>/rector/` (unless the project's own Rector was used) and `<TOOLS_DIR>/phpcs/`. Keep the database dump and the logs; report their paths and list their deletion as a manual action.
4. Flush caches as in Step 2.

## Stop conditions
Stop and report instead of guessing when:
- the `Migration` line is missing, malformed, or has FROM equal to TO;
- no guide or `UPGRADING` file exists for a version in `PATH`;
- the working tree is dirty at start;
- WP-CLI cannot run on `TO`;
- `TOOLS_DIR` cannot be placed outside the web document root;
- `wp core verify-checksums` reports modified core files, or `wp plugin verify-checksums` reports a modified third-party plugin that would need an update (the update would erase the local patch);
- core needs a new branch to support `TO`;
- a third-party plugin or theme is incompatible with `TO` and has no allowed release, or its update is license-gated;
- an update requires a database upgrade and `Database changes allowed` is NO;
- Rector or PHPCompatibility installation or execution fails (network, Composer, autoload errors);
- a fix would change business behavior;
- a Composer dependency requires a major upgrade or has no release compatible with `TO`;
- the code depends on an extension that is removed or unbundled in `PATH`;
- a system-level change is required to proceed.

## Final report (Markdown)
1. Derived parameters (Step 0 output) and `TOOLS_DIR`.
2. Environment: PHP and WP-CLI versions, SAPI, extension and ini diff between `FROM` and `TO`, WordPress layout, core version and its compatibility status for `TO`, multisite sites.
3. Component inventory: every plugin, theme, mu-plugin and drop-in with ownership class, status, version, `Requires PHP`, and compatibility verdict for `TO`.
4. Migration checklist: each item with its source version, origin, and status (fixed, not applicable, or blocked).
5. Rector:
   - version and source (project or isolated `TOOLS_DIR`);
   - set used, and any gap versus `TO`;
   - rules applied with file counts;
   - modernization rules discarded with occurrence counts.
6. PHPCompatibility: version, coverage gap versus `TO`, findings in project code (before -> after) and in third-party code.
7. Code changes by category: file, before -> after, reason (guide entry).
8. Updates: core, plugins, themes and Composer packages, old -> new version; major releases not applied, with their breaking changes.
9. Verification results compared with the baseline: HTTP status per route and site, tests, static analysis, log entries by ownership class.
10. Remaining issues: PHP notices inside core and third-party code, WordPress API notices, admin screens to check manually.
11. Required manual or system actions, with exact commands (FPM reload, WP-CLI installation, premium updates, core branch upgrade, deletion of the database dump and logs).