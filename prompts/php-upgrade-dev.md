# Task: Migrate this codebase between PHP versions

## Context
- Migration: 8.3 -> 8.5          <!-- the ONLY line to change between migrations: FROM -> TO -->
- Environment: Debian development server. You are running inside the project repository.
- Runtime status: the PHP runtime has already been switched to the target version. Application code and dependencies have NOT been updated.
- Application URL: <!-- <URL> -->
- Key routes to smoke-test: <!-- <ROUTES or "discover from router config"> -->
- Must still run on the source version: <!-- <YES/NO> -->

## Step 0 — Derive migration parameters (before anything else)
From the `Migration` line, compute and write at the top of the final report:
- `FROM` / `TO`: source and target versions (major.minor).
- `DIRECTION`: `upgrade` if TO > FROM, `downgrade` if TO < FROM. If equal, stop and report.
- `PATH`: every released minor version strictly after the lower version, up to and including the higher one, in order. Include major boundaries (e.g. 7.4 -> 8.1 gives 8.0, 8.1).
- `GUIDES`: for each version `X.Y` in `PATH`, `https://www.php.net/manual/en/migrationXY.php` (e.g. 8.0 -> `migration80.php`). If a guide does not exist (unreleased or RC version), use the `UPGRADING` file of the matching `php-src` branch instead. If neither exists, stop and report.
- `TO_ID`: `major*10000 + minor*100` (e.g. 8.5 -> `80500`). Used by static analysis tools.
- `BRANCH`: `chore/php-<TO>-migration`.
- `COMMIT_PREFIX`: `chore(php<TO>):`.
- `CONSTRAINT`: `^<TO>`, or `^<FROM> || ^<TO>` when the source version must still be supported. For a downgrade, use the lowest version.
- `INI_DIRS`: `/etc/php/<FROM>/` and `/etc/php/<TO>/`.
- `RECTOR_SET`, where `XY` is `TO` without the dot (e.g. 8.5 -> `85`):
  - upgrade: `->withPhpSets(phpXY: true)`
  - downgrade: `->withSets([\Rector\Set\ValueObject\DowngradeLevelSetList::DOWN_TO_PHP_XY])`

Every later step uses these values. No other step contains a hard-coded version.

## Goal (definition of done)
1. The application runs on `TO` (web and CLI) with no fatal errors and no warnings.
2. Zero deprecation notices originating from project code (excluding `vendor/`).
3. Functional behavior unchanged: tests and HTTP responses match the baseline.

## Scope
- In scope: project source code, `composer.json` / `composer.lock`, project config, tests, CI/Docker files referencing the PHP version.
- Out of scope: business logic changes, unrelated refactors, new features, style-only changes.
- Out of scope for upgrades: adopting syntax or APIs introduced in `PATH`, unless required for compatibility.
- Never edit `vendor/` directly.
- Never modify system packages or files outside the repository (apt, systemctl, /etc). List required system actions in the final report instead.

## Step 1 — Inventory (read-only)
1. Run `php -v`, `php -m` and `php --ini`. Confirm the CLI runs `TO` and list the loaded extensions.
2. Identify the SAPI serving <URL> (php<TO>-fpm, Apache mod_php, etc.) from the web server vhost config. Confirm it uses `TO`, not a leftover `FROM` pool or socket.
3. Diff the enabled extensions and custom ini directives between the two `INI_DIRS`. List the missing ones.
4. Detect the framework, entry points (web, console commands, cron scripts, workers), test suite, and static analysis tools (PHPUnit, PHPStan, Psalm).
5. Detect whether `rector/rector` is already a project dependency, and record its installed version and any existing `rector.php`.
6. Read `require.php` and `config.platform.php` in `composer.json`.

## Step 2 — Baseline
1. Check that the working tree is clean. If it is not, stop and report.
2. Create `BRANCH`.
3. Run the test suite and static analysis and record the results, so that pre-existing failures can later be told apart from new ones.
4. Record the HTTP status codes for <URL> and the key routes: `curl -sS -o /dev/null -w '%{http_code}\n' <URL>`.
5. Record the current PHP and web server error log entries. Find the log paths from the ini and vhost config.

## Step 3 — Build the migration checklist
1. Read every guide in `GUIDES`, in `PATH` order.
2. Extract every applicable item from these sections: backward-incompatible changes, deprecated features, removed or unbundled extensions, changed function signatures, removed or changed ini directives.
3. For a downgrade, extract instead every feature introduced in `PATH` (new syntax, functions, classes, constants, extensions). Each one used by the project must be replaced.
4. Search the codebase for each item and keep only those that match.
5. Write the resulting checklist into the report before editing. Mark each item with its source version.

## Step 4 — Dependencies
1. Run `composer check-platform-reqs` and `composer why-not php <TO>`.
2. Set `require.php` to `CONSTRAINT`. Align `config.platform.php` with `TO`.
3. Move each blocking package to the closest version that supports `TO`, using patch or minor releases only.
4. Do not apply major releases. List each one with its breaking changes.

## Step 5 — Code migration

### 5.1 Rector pass (mandatory)
Rector is required for this migration. Install it if it is missing.

Installation:
1. If the project already requires `rector/rector` and the installed version provides `RECTOR_SET`, use it as-is.
2. Otherwise, install an isolated copy without touching project dependencies:
   ```bash
   mkdir -p .tools/rector
   grep -qxF '.tools/' .git/info/exclude || echo '.tools/' >> .git/info/exclude
   composer require --dev rector/rector --working-dir=.tools/rector --no-interaction
   ```
3. Check that the installed version provides `RECTOR_SET`.
   - If it does not, run `composer update rector/rector --working-dir=.tools/rector`.
   - If it still does not, use the highest available set below `TO`, cover the missing versions manually in 5.2, and record the gap in the report.
4. Never add Rector to the project `composer.json` and never commit `.tools/`.

Configuration:
- Do not modify an existing project `rector.php`.
- Create `.tools/rector/rector.php`:
  ```php
  <?php

  declare(strict_types=1);

  use Rector\Config\RectorConfig;

  return RectorConfig::configure()
      // Project source dirs detected in Step 1 (absolute paths)
      ->withPaths([])
      // vendor/, generated code, caches, compiled templates
      ->withSkip([])
      // Project autoloader so Rector can resolve types
      ->withAutoloadPaths([dirname(__DIR__, 2) . '/vendor/autoload.php'])
      // Insert RECTOR_SET here (discovery pass only)
  ;
  ```

Execution, from the project root:
1. Discovery: run `.tools/rector/vendor/bin/rector process --config .tools/rector/rector.php --dry-run --output-format=json > .tools/rector/discovery.json`.
2. Classify each rule listed under `applied_rectors`:
   - Compatibility: it fixes a Step 3 checklist item (deprecation, removal, backward-incompatible change).
   - Modernization: it introduces optional newer syntax or APIs.
   - For a downgrade, every rule in the set counts as compatibility.
3. Apply:
   - Replace `RECTOR_SET` in the config with `->withRules([...])`, listing compatibility rules only.
   - Run the same command without `--dry-run`.
4. Review the full diff. Revert any hunk that changes behavior.
5. Commit with the message `COMMIT_PREFIX apply Rector compatibility rules`.
6. Record each discarded modernization rule, with its number of occurrences.

### 5.2 Manual pass
1. Apply the remaining Step 3 checklist items not resolved by Rector, in `PATH` order.
2. Follow these rules:
   - Fix root causes. Never suppress errors with `@`, `error_reporting()` changes, or ini tweaks.
   - If the source version must still be supported, use no syntax or API missing from the lowest version.
   - Make one commit per change category, with the message `COMMIT_PREFIX <description>`.

### 5.3 Cleanup
After Step 6 verification passes, delete `.tools/rector/` unless the project's own Rector was used.

## Step 6 — Verification
1. Lint every file: `find . -path ./vendor -prune -o -path ./.tools -prune -o -name '*.php' -print0 | xargs -0 -n1 -P4 php -l`. Expect zero errors.
2. Run with `error_reporting=E_ALL` and `log_errors=On`. Grep the logs for `Deprecated|Warning|Fatal` and attribute each entry to project code or `vendor/`.
3. Run the tests: no new failures compared with the baseline.
4. Run static analysis targeting `TO_ID` (e.g. PHPStan `phpVersion: TO_ID`): no new errors.
5. Smoke-test <URL> and the key routes over HTTP: status codes match the baseline and no new log entries appear. If `opcache.validate_timestamps=0`, list a reload of the `TO` FPM service as a required system action.
6. Execute the CLI entry points with `--help` or in dry-run mode.

## Stop conditions
Stop and report instead of guessing when:
- the `Migration` line is missing, malformed, or has FROM equal to TO;
- no guide or `UPGRADING` file exists for a version in `PATH`;
- the working tree is dirty at start;
- Rector installation or execution fails (network, Composer, autoload errors);
- a fix would change business behavior;
- a dependency requires a major upgrade or has no release compatible with `TO`;
- the code depends on an extension that is removed or unbundled in `PATH`;
- a system-level change is required to proceed.

## Final report (Markdown)
1. Derived parameters (Step 0 output).
2. Environment: versions, SAPI, extension and ini diff between `FROM` and `TO`.
3. Migration checklist: each item with its source version and status (fixed, not applicable, or blocked).
4. Rector:
   - version and source (project or isolated `.tools/`);
   - set used, and any gap versus `TO`;
   - rules applied with file counts;
   - modernization rules discarded with occurrence counts.
5. Code changes by category: file, before -> after, reason (guide entry).
6. Dependency changes: package, old -> new version.
7. Verification results compared with the baseline.
8. Remaining issues, including deprecations inside `vendor/`.
9. Required manual or system actions, with exact commands.