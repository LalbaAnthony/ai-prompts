Apply the "disable MySQL strict mode" fix to this project's SQL connection classes.

## Context

This project is built on Ample, Agoravita's in-house PHP framework. Each project embeds its own diverged copy of the core, so read the local files before editing and never assume they match the reference below line for line.

The code was written for MySQL 5's non-strict mode. It omits NOT NULL columns that have no default from INSERTs (e.g. `INSERT INTO actualite__base (created_at) VALUES (NOW())`), writes `''` into INT/DATE/DATETIME columns, and inserts `''` into AUTO_INCREMENT ids (`lib/avgc`, `lib/avmedias`). MySQL 5.7+ and MariaDB 10.2.4+ enable STRICT_TRANS_TABLES by default, which turns these into fatal errors: 1364 "Field ... doesn't have a default value", 1366 "Incorrect integer value", 1292 "Incorrect date value". The fix sets the session sql_mode right after the connection opens, in every SQL connection class, so the whole project, vendored libs included, gets the legacy behaviour back.

## The change

Same statement and same comment everywhere:
- SQL: `SET SESSION sql_mode = 'NO_ENGINE_SUBSTITUTION'` (the MySQL 5.6 default; valid on MySQL 5.x/8.x and MariaDB).
- Comment: copy it verbatim from the diffs below (French, like the rest of these files).

Reference implementation.

PDO class (`class.sql-pdo.php`), inside the connection `try`, after the last `setAttribute()`:

```diff
             $this->link->setAttribute(PDO::ATTR_STRINGIFY_FETCHES, true);
+            // Le code suppose le mode non strict de MySQL 5 (INSERT sans colonnes NOT NULL, '' dans des colonnes INT/DATE) : on désactive le mode strict, actif par défaut depuis MySQL 5.7 et MariaDB 10.2.4
+            if ($this->type == 'mysql') {
+                $this->link->exec("SET SESSION sql_mode = 'NO_ENGINE_SUBSTITUTION'");
+            }
         } catch (Exception $e) {
```

mysqli class (`class.sql-mysql.php`), right after `SET NAMES`, inside the `if ($this->link)` block:

```diff
             $this->result = mysqli_query($this->link, "SET NAMES utf8");
+            // Le code suppose le mode non strict de MySQL 5 (INSERT sans colonnes NOT NULL, '' dans des colonnes INT/DATE) : on désactive le mode strict, actif par défaut depuis MySQL 5.7 et MariaDB 10.2.4
+            $this->query("SET SESSION sql_mode = 'NO_ENGINE_SUBSTITUTION'");
         }
```

## Steps

1. Run `git status` and leave unrelated changes alone.
2. List the candidates: `git ls-files | grep -E '(^|/)class\.sql-[^/]*\.php$'`. Also list every connection point: `git grep -n -E "new PDO|mysqli_connect|new mysqli|mysql_p?connect" -- '*.php'`.
3. Check whether it is already handled: `git grep -n -i sql_mode`. If sql_mode is already set anywhere (PHP or config), change nothing and report it.
4. Read each candidate file in full, then:
   - PDO class: insert inside the connection `try`, after `ERRMODE_EXCEPTION` and any other `setAttribute()`, so that a failure lands in the existing `catch`. Guard on MySQL: use `$this->type == 'mysql'` if the class has a type property filled from `DB_TYPE`, otherwise `$this->link->getAttribute(PDO::ATTR_DRIVER_NAME) == 'mysql'`. If `ERRMODE_EXCEPTION` is not set before that point, check whether `exec()` returns `false` and handle that the same way as the class's connection errors.
   - mysqli class, or legacy `mysql_*` class (PHP 5, not migrated): insert after `SET NAMES`, in the same "connection succeeded" block, through `$this->query(...)`. First confirm that `query()` in that file reports the error and dies on failure; if it does not, add explicit handling that mirrors the class's own error output. The statement must never fail silently.
   - PostgreSQL-only class, or a file that opens no connection (e.g. `class.sql-gen.php`, a query builder): skip it.
   - File under `lib/` or another vendored directory: do not modify it; report it, and whether it is loaded at runtime.
5. Preserve each file's indentation (tabs vs spaces, width), line endings (check CRLF with `git ls-files --eol <file>`) and encoding (`file <file>`). If a file is not UTF-8 (e.g. ISO-8859-1), use this ASCII-only comment instead so that no mixed encoding gets in: `// Le code suppose le mode non strict de MySQL 5 (INSERT sans colonnes NOT NULL, '' dans des colonnes INT/DATE) : on desactive le mode strict, actif par defaut depuis MySQL 5.7 et MariaDB 10.2.4`
6. Change nothing else: no refactoring and no other fixes, even obvious ones; list those in the report instead.
7. Lint every modified file: `php -d error_reporting=-1 -d display_errors=1 -l <file>`. If it fails, lint the committed version (`git show HEAD:<file> | php -l`) to tell a pre-existing failure from one you introduced.
8. Do not connect to the database and do not commit.

## Report

- Each candidate file: modified or skipped, and why.
- Connection points from step 2 that bypass these classes; they stay in strict mode.
- Lint results.
- The full `git diff`.
- What you could not verify. Suggest this runtime check to the user: create a record in the back-office (e.g. a news item), then run `SELECT @@SESSION.sql_mode` through `Sql::getInstance()->getValeur(...)`; it must return `NO_ENGINE_SUBSTITUTION`.
