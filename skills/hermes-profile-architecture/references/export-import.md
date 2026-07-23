# Export / Import

Two different systems for moving profile data:

| System | Format | Includes user data? | Use case |
|---|---|---|---|
| `export` / `import` | tar.gz archive | Yes (sessions, memories) | Local backup/restore, machine-to-machine transfer |
| `install` / `update` | git repo | No (clean distribution) | Publishing/sharing a profile as a package |

Source: `profile_distribution.py` module docstring, lines 12-16.

## Export

`hermes profile export <name> [-o <file>]` — source: `profiles.py:export_profile()`:

- Creates a `.tar.gz` archive via `shutil.make_archive`
- **Default profile** (root `~/.hermes`): uses an allow-list (`_DEFAULT_EXPORT_INCLUDE_ROOT`) of known Hermes artifacts to avoid exporting arbitrary user files that may coexist alongside HERMES_HOME (Docker/custom deployments). Credentials (`auth.json`, `.env`) and caches excluded.
- **Named profiles**: excludes `auth.json` and `.env` (credentials stay local). All other data included.
- Stages a filtered copy under a temp dir before archiving

### Default profile export allow-list

```python
_DEFAULT_EXPORT_INCLUDE_ROOT = frozenset({
    "config.yaml", "SOUL.md", "MEMORY.md", "USER.md", "todo.json",
    "system_prompt.md", "AGENTS.md", "CLAUDE.md", ".cursorrules",
    "skills", "cron", "scripts", "sessions",
    "plugins", "memories", "knowledge", "preferences",
})
```
Source: `profiles.py:67-78`.

## Import

`hermes profile import <archive> [--name <name>]` — source: `profiles.py:import_profile()`:

1. Inspects archive top-level directory to infer profile name (or use `--name`)
2. Can't import as `"default"` — that targets `~/.hermes` itself; must use `--name`
3. Safe extraction: no symlinks, no absolute paths, no `..` traversal (`_safe_extract_profile_archive`)
4. Extracts to temp dir, validates structure, then `shutil.move` into `~/.hermes/profiles/<name>/`
5. Pre-existing profile at target name → `FileExistsError`
