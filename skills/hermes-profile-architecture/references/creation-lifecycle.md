# Profile creation & lifecycle

Source: `hermes_cli/profiles.py:create_profile()`, `delete_profile()`, `seed_profile_skills()`.

## `hermes profile create <name>`

| Flag | Effect |
|---|---|
| *(none)* | Fresh min profile: bootstrap dirs + empty `.env` + default `SOUL.md` + bundled skills seeded |
| `--clone` | Copy `config.yaml`, `.env` (0o600), `SOUL.md`, `skills/`, `MEMORY.md`, `USER.md` from active profile |
| `--clone-all` | Full copytree of active profile (excludes infra + session history) |
| `--clone-from <src>` | Source profile override instead of active; implies `--clone` unless `--clone-all` |
| `--no-alias` | Skip wrapper script creation |
| `--no-skills` | Empty profile, writes `.no-bundled-skills` marker so `hermes update` skips skill re-seed |
| `--description <text>` | Persisted to `profile.yaml` |

### Clone semantics — what's included/excluded

`--clone` copies: `config.yaml`, `.env` (tightened to 0o600), `SOUL.md`, all installed `skills/`, `memories/MEMORY.md`, `memories/USER.md` (source: `_CLONE_CONFIG_FILES`, `_CLONE_SUBDIR_FILES`).

`--clone-all` additionally copies everything except:
- **Infrastructure** (only when cloning from `default`): `hermes-agent/`, `.worktrees/`, `profiles/`, `bin/`, `node_modules/` (source: `_CLONE_ALL_DEFAULT_EXCLUDE_ROOT`)
- **Session history** (any source): `state.db*`, `sessions/`, `backups/`, `state-snapshots/`, `checkpoints/` (source: `_CLONE_ALL_HISTORY_EXCLUDE_ROOT`)
- **Runtime files**: `gateway.pid`, `gateway_state.json`, `processes.json` (source: `_CLONE_ALL_STRIP`)

### Fresh profile seeding

1. Bootstrap dirs: `memories/`, `sessions/`, `skills/`, `skins/`, `logs/`, `plans/`, `workspace/`, `cron/`, `home/` (source: `_PROFILE_DIRS`)
2. Empty `.env` with header comment (source: `create_profile` ~line 297)
3. Default `SOUL.md` from `hermes_cli/default_soul.py::DEFAULT_SOUL_MD`
4. Bundled skills via `seed_profile_skills()` subprocess, unless `--no-skills`
5. Config migration if cloned config predates schema version (`_migrate_profile_config_if_outdated`)
6. If container/s6: registers gateway service slot (`_maybe_register_gateway_service`)
7. Persists `--description` to `profile.yaml` if provided

### Bootstrap directories

```python
_PROFILE_DIRS = [
    "memories", "sessions", "skills", "skins", "logs",
    "plans", "workspace", "cron", "home",
]
```
Source: `profiles.py:28-41`.

### Opt-out marker

`.no-bundled-skills` at profile root: `hermes profile create --no-skills` writes it; `has_bundled_skills_opt_out()` checks it; `seed_profile_skills()` and `hermes update`'s all-profile sync loop skip that profile. Delete the marker to re-enable.

## Delete

`hermes profile delete <name>` — source: `profiles.py:delete_profile()`:

1. Shows summary (model, skills, distribution info), asks confirmation (unless `--yes`)
2. Disables systemd/launchd service (prevents auto-restart)
3. Stops gateway process via `gateway.pid` + `terminate_pid()`
4. Stops other profile-bound backends (serve/dashboard) via `_stop_profile_backends()` — these hold SQLite connections that would otherwise cause `ENOTEMPTY` on rmdir
5. Removes wrapper alias from `~/.local/bin/`
6. Removes profile directory with `_rmtree_with_retry()` (3 attempts with backoff for transient `ENOTEMPTY`)
7. Clears `active_profile` if it pointed to this profile
8. If container/s6: unregisters gateway service slot
