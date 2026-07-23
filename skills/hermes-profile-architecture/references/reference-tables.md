# Reference tables

## Profile directory structure (named profile)

```
~/.hermes/profiles/<name>/
├── config.yaml                 # personal config overrides
├── distribution.yaml           # distribution manifest (if installed)
├── SOUL.md                     # personality
├── profile.yaml                # description metadata
├── .env                        # API keys (0o600)
├── .env.EXAMPLE                # template for .env (distribution)
├── .no-bundled-skills          # opt-out marker (--no-skills)
├── auth.json                   # credential pool
├── gateway.pid                 # running gateway PID
├── gateway_state.json          # gateway runtime state
├── memories/
│   ├── MEMORY.md
│   └── USER.md
├── sessions/                   # per-session transcripts
├── skills/                     # installed skills (bundled + user-installed)
├── cron/                       # scheduled job definitions
├── mcp.json                    # MCP server definitions
├── logs/                       # gateway logs
├── plans/                      # workspace plans
├── workspace/                  # file workspace
├── home/                       # subprocess HOME (containers)
├── local/                      # user customization override namespace
├── cache/                      # regenerable caches
├── checkpoints/                # session checkpoint data
├── sandboxes/                  # sandbox environments
├── backups/                    # hermes backup archives
├── state.db*                   # SQLite session store
├── image_cache/
├── audio_cache/
├── document_cache/
├── browser_screenshots/
└── processes.json
```

## Reserved names

| Name | Reason |
|---|---|
| `default` | Built-in root profile alias — can't create or import as this |
| `hermes` | Binary name collision |
| `test`, `tmp` | Common temp words |
| `root`, `sudo` | System identity collision |
| All hermes subcommands | CLI parse ambiguity — `chat`, `model`, `gateway`, `setup`, `profile`, `skills`, `tools`, `mcp`, `sessions`, `update`, `uninstall`, `plugins`, `honcho`, `acp`, `login`, `logout`, `status`, `cron`, `doctor`, `dump`, `config`, `pairing`, `insights`, `version`, `whatsapp` |

Source: `_RESERVED_NAMES` + `_HERMES_SUBCOMMANDS` in `profiles.py:83-97`.

## Important file locations

| Item | Path |
|---|---|
| Default profile (root) | `~/.hermes/` |
| Named profile | `~/.hermes/profiles/<name>/` |
| Profiles parent dir | `~/.hermes/profiles/` |
| Active profile marker | `~/.hermes/active_profile` |
| Staging for active_profile write | `~/.hermes/active_profile.tmp` |
| Wrapper scripts | `~/.local/bin/<alias>` |
| Wrapper dir detection | `~/.local/bin` in PATH |
| Canonical HERMES_HOME | `~/.hermes` (overrideable via env, Docker) |

## Bootstrap directories (created on `hermes profile create`)

```
memories/    sessions/    skills/    skins/    logs/
plans/       workspace/   cron/      home/
```

Source: `_PROFILE_DIRS` in `profiles.py:28-41`.

## Distribution-owned files (replaced on every `update`)

`SOUL.md`, `config.yaml`, `mcp.json`, `skills/`, `cron/`, `distribution.yaml`

`config.yaml` preserved on update unless `--force-config`.

Source: `profile_distribution.py:DEFAULT_DIST_OWNED`, lines 44-49.
