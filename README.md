# git-hooks

Personal Git hooks by [Scott Robinson](https://github.com/ssx), managed via Git's native `core.hooksPath` mechanism.

Licensed under the [MIT License](LICENSE).

---

## Setup

Point Git at this directory globally so every repository on your machine uses these hooks automatically:

```bash
git config --global core.hooksPath /path/to/git-hooks
```

Or per-repository:

```bash
git config core.hooksPath /path/to/git-hooks
```

---

## Structure

```
git-hooks/
  commit-msg                       # Validates commit message format
  pre-commit                       # Runner: discovers checks from pre-commit.d/
  pre-commit.d/
    all/                           # Runs on every staged file
      01-conflict-markers
    php/                           # Runs on .php and .phtml files (see EXT_MAP)
      01-syntax
      02-no-die
      03-no-var-dump
      04-no-print-r
```

---

## Hooks

### `commit-msg` — Conventional Commits

Validates that every commit message follows the [Conventional Commits](https://www.conventionalcommits.org/) specification.

**Format:** `<type>[optional scope][optional !]: <description>`

| Type | Purpose |
|------|---------|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Formatting, no logic change |
| `refactor` | Code restructure, no behaviour change |
| `perf` | Performance improvement |
| `test` | Adding or updating tests |
| `build` | Build system or dependency changes |
| `ci` | CI configuration changes |
| `chore` | Maintenance tasks |
| `revert` | Reverts a previous commit |

**Examples:**

```
feat: add user authentication
fix(auth): handle expired tokens correctly
feat!: remove deprecated v1 API endpoints
docs(readme): update installation instructions
chore(deps): bump lodash to 4.17.21
```

Merge commits and auto-generated revert commits are skipped automatically.

---

### `pre-commit` — File quality checks

A runner that discovers and executes check scripts from `pre-commit.d/<scope>/` against staged files. Checks are only run against the **staged version** of each file, not the working tree.

---

## Checks

### All files

These run against every staged file regardless of type. Binary files are skipped automatically.

| Script | What it checks |
|--------|---------------|
| `all/01-conflict-markers` | Blocks commits containing unresolved Git merge conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`) |

### PHP / PHTML

These run against `.php` and `.phtml` files (controlled by `EXT_MAP` in `pre-commit`).

| Script | What it checks |
|--------|---------------|
| `php/01-syntax` | Validates PHP syntax using `php -l`. Skipped gracefully if `php` is not in `$PATH` |
| `php/02-no-die` | Blocks `die()` calls |
| `php/03-no-var-dump` | Blocks `var_dump()` calls |
| `php/04-no-print-r` | Blocks `print_r()` calls |

---

## Configuration

### Extension → scope mapping

The `EXT_MAP` array in `pre-commit` controls which file extensions are checked by which scope directory. Files not listed only run `all/` checks.

```bash
EXT_MAP=(
  "php:php"
  "phtml:php"     # .phtml treated as PHP
# "ts:js"         # example: TypeScript runs js/ checks
# "scss:css"      # example: SCSS runs css/ checks
)
```

To add a new mapping, append a `"extension:scope"` entry to the array.

---

## Extending

### Adding a check to an existing scope

1. Create an executable script in `pre-commit.d/<scope>/`
2. Name it `NN-description` — the `NN` prefix controls run order
3. Follow the check interface:

```bash
#!/usr/bin/env bash
# ─────────────────────────────────────────────────────────────────────────────
# Repository : https://github.com/ssx/git-hooks
# Author     : Scott Robinson (https://github.com/ssx)
# License    : MIT — see LICENSE for details
# ─────────────────────────────────────────────────────────────────────────────
#
# Check: description of what this checks
# Scope: <scope>

tmp="$1"   # temp file containing the staged file content
src="$2"   # original repo-relative path — use this in error output

if matches=$(grep -nE 'your-pattern' "$tmp" 2>/dev/null); then
  printf '%s\n' "$matches" | sed "s|^|        $src:|"
  exit 1
fi

exit 0
```

### Adding a new scope

1. Add an entry to `EXT_MAP` in `pre-commit`:
   ```bash
   "js:js"
   "ts:js"   # TypeScript shares the js scope
   ```
2. Create the directory: `pre-commit.d/js/`
3. Drop executable check scripts into it

The runner picks up new scopes and scripts automatically — no changes to `pre-commit` needed beyond the `EXT_MAP` entry.
