# CLI internals: `-p` / `--profile` flag

Source: `hermes_cli/_parser.py:16-22`, `profiles.py:resolve_profile_env()`.

## Pre-argparse consumption

The `-p` / `--profile` flag is consumed **before argparse runs**. It sets `HERMES_HOME` and strips itself from `sys.argv` so downstream modules see the correct HERMES_HOME from the start.

Source: `_parser.py:16-22` (`PRE_ARGPARSE_INHERITED_FLAGS`).

## Resolution chain

1. `--profile <name>` / `-p <name>` is parsed from raw `sys.argv`
2. Calls `profiles.py:resolve_profile_env(name)` — returns the `HERMES_HOME` path string
3. Sets `os.environ["HERMES_HOME"]` to that path
4. Strips the flag pair from `sys.argv`
5. Normal `argparse` runs with the remaining args

Source: `profiles.py:resolve_profile_env()`, lines 995-1007.

## Without `-p`

HERMES_HOME defaults to `~/.hermes` (the "default" profile). The active_profile file can also set a sticky default — see [`wrappers-aliases.md`](wrappers-aliases.md).

## HERMES_HOME inference

`get_active_profile_name()` infers the current profile name from the HERMES_HOME environment variable:
- Returns `"default"` if HERMES_HOME is not set or points to `~/.hermes`
- Returns the profile name if HERMES_HOME points into `~/.hermes/profiles/<name>/`
- Returns `"custom"` if HERMES_HOME is set to an unrecognized path

Source: `profiles.py:get_active_profile_name()`, lines ~530-545.
