# Profile configuration files

Each profile has its own copies of these files:

| File | Purpose | Created by |
|---|---|---|
| `config.yaml` | All settings: model, providers, terminal, toolsets, curator, gateway | `create`, `--clone`, distribution |
| `.env` | API keys and secrets (0o600 permissions) | `create` (empty header), `--clone`, `backfill_profile_envs()` |
| `SOUL.md` | Agent personality (slot #1 in system prompt) | `create` (default template), `--clone`, distribution |
| `profile.yaml` | Profile metadata: description, description_auto flag | `describe` command, `--description` on create |
| `distribution.yaml` | Distribution manifest (present only if installed via `install`) | distribution |

## config.yaml

Deep-merged over `DEFAULT_CONFIG` at read time (`config.py:load_config()`). User-set keys preserved; absent keys get defaults. Schema version tracked via `_config_version` key. Migration triggered on `create --clone` and via `hermes update`.

- Source defaults: `config.py:933` (`DEFAULT_CONFIG`)
- Migration: `profiles.py:_migrate_profile_config_if_outdated()` (lines 186-215)

## SOUL.md

Seeded from `hermes_cli/default_soul.py::DEFAULT_SOUL_MD` on fresh create:
```
"You are Hermes Agent, an intelligent AI assistant created by Nous Research. ..."
```
Legacy templates (comment-scaffolding style, no actual persona text) are detected and auto-upgraded to `DEFAULT_SOUL_MD` on hermes update.

Source: `default_soul.py:3-7`, `_LEGACY_TEMPLATE_SOULS`.

## profile.yaml

Stores `description` (1-2 sentence role summary) and `description_auto` (bool). Written by `hermes profile describe` or by `--description` flag on `create`. Read by `list_profiles()` for kanban decomposer routing. Missing file → empty defaults; never an error.

Source: `profiles.py:read_profile_meta()` / `write_profile_meta()`.

## `.env` seeding

`create_profile()` seeds an empty `.env` with a header comment so the profile has its own credentials file from day one. Without it, profiles predating PR #44792 inherited API keys from the shell environment silently.

`backfill_profile_envs()` copies the default install's `.env` into each named profile that lacks one — preserves effective credentials they were already running with (they previously read root `.env` via process environment).

Both tighten mode to 0o600. Source: `profiles.py:backfill_profile_envs()`.
