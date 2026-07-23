# Wrappers, aliases, active profile & rename

## Wrapper scripts

Wrapper scripts live in `~/.local/bin/<alias>` (POSIX) or `.bat` (Windows). They invoke `hermes -p <profile> "$@"`.

Source: `profiles.py:create_wrapper_script()` (lines 149-173), `remove_wrapper_script()`.

### `create_wrapper_script(name, target=None)`

- Creates `~/.local/bin/<name>` → `exec hermes -p <profile> "$@"`
- `target` defaults to `name`; set explicitly when alias name differs from profile name (custom alias)
- POSIX: `chmod +x`; Windows: `.bat` file
- Returns the path to the created wrapper, or None on failure

### `remove_wrapper_script(name)`

- Verifies the file contains `"hermes -p"` before removing (safety check)
- Returns True if removed

### Alias collision detection

`check_alias_collision(name)` — source: `profiles.py:118-144`:

1. Validates alias name against `[a-z0-9][a-z0-9_-]{0,63}` regex
2. Checks against `_RESERVED_NAMES` (`hermes`, `default`, `test`, `tmp`, `root`, `sudo`)
3. Checks against `_HERMES_SUBCOMMANDS` (`chat`, `model`, `profile`, `skills`, `install`, etc.)
4. Runs `which`/`where` to detect existing binaries on PATH
5. Allows overwriting hermès's own wrappers (detected by reading file content)

### Reverse alias resolution

`build_alias_map()` — source: `profiles.py:207-237`:

Single-pass scan of `~/.local/bin/` reading only an 8KB head slice of each file (avoids reading large unrelated binaries like ffmpeg). Custom aliases (file name ≠ profile name) are preferred over profile-named wrappers. Used by `list_profiles()` to show the alias the user actually types.

`find_alias_for_profile(name)` — wrapper that calls `build_alias_map()` and reverse-lookups. Prefer `build_alias_map()` for bulk operations (O(N) vs O(N²)).

## Active profile (sticky default)

`hermes profile use <name>` — source: `profiles.py:get_active_profile()`, `set_active_profile()`:

- Stored in `~/.hermes/active_profile` as a one-line plaintext file
- `get_active_profile()` reads it; returns `"default"` if absent or empty
- `set_active_profile("default")` removes the file (reverts to root)
- `get_active_profile_name()` infers profile from HERMES_HOME (not from the marker file): returns `"default"`, the profile name, or `"custom"` for unrecognized paths

## Rename

`hermes profile rename <old> <new>` — source: `profiles.py:rename_profile()`:

1. Validates both names (not "default", no existing collision)
2. Stops gateway if running
3. Renames directory on disk
4. Migrates Honcho host blocks: `hermes_<old>` → `hermes_<new>`, preserving `aiPeer` identity (source: `_migrate_honcho_profile_host()`)
5. Removes old wrapper, creates new wrapper
6. Updates `active_profile` if it pointed to old name

## Custom alias

`hermes profile alias <name> --name <custom>` — creates a wrapper where the command name differs from the profile name. Example: `hermes profile alias coder --name code` → `~/.local/bin/code` executes `hermes -p coder`. Use `--remove` to delete.

Source: `profiles.py:create_wrapper_script()` with explicit `target` param.
