---
name: hermes-profile-architecture
description: >
  Comprehensive skill for every aspect of Hermes Agent profiles — the
  multi-profile isolation model, profile creation and lifecycle, HERMES_HOME
  resolution, CLI subcommands, config.yaml and SOUL.md, wrapper aliases, sticky
  active profiles, profile descriptions and the kanban orchestrator, export/import,
  rename, alias management, profile-scoped auth and credential shadowing, profile
  distributions (install/update/info), gateway multiplexing, and the s6 container
  service model. Load whenever the user asks about profiles, multi-profile setups,
  profile internals, or any `hermes profile <subcommand>`. Source-grounded with
  file:line citations into ~/Local/docs/hermes-agent/.
compatibility: Local clone of hermes-agent at ~/Local/docs/hermes-agent/
---

# Hermes Profile Architecture

Reference skill covering the **full Hermes profile system** — how profiles work,
how they relate to HERMES_HOME, the CLI surface, distribution packaging, gateway
multiplexing, and every subcommand under `hermes profile`.

## When to load this skill

| User wants… | Load when… |
|---|---|
| Profile model | Understanding what a profile IS, how HERMES_HOME scopes it |
| Create | `hermes profile create`, with `--clone`, `--clone-all`, or `--no-skills` |
| Use | `hermes -p <name>`, wrapper aliases (`coder chat`), sticky default |
| Manage | `list`, `show`, `delete`, `rename`, `alias`, `describe` |
| Package | `export` / `import` (tar.gz backup/restore) |
| Distribute | `install` / `update` / `info` (git-based distribution) |
| Configure | `config.yaml`, `SOUL.md`, `profile.yaml`, `.env` |
| Auth | Credential shadowing, how auth falls back from profile to global |
| Troubleshoot | Gateway conflicts, profile-not-found, alias collisions |
| Integrate | Gateway multiplexing, s6 services, kanban routing by description |

Do NOT load for: general Hermes architecture, memory providers, curator logic,
auto-triggers (use `hermes-dev-kit`), or `distribution.yaml` authoring details
(use `hermes-profile-distributions`).

## Core concept

A **profile** is a fully independent HERMES_HOME directory with its own
config.yaml, .env, memory, sessions, skills, gateway, cron, and logs — an
isolated agent identity.

```
~/.hermes/                       ← "default" profile (backward-compatible root)
├── config.yaml   SOUL.md   .env   skills/   memories/   sessions/
└── profiles/                    ← named profiles
    ├── coder/   config.yaml   SOUL.md   .env   ...
    └── researcher/   config.yaml   SOUL.md   .env   ...
```

**Rules:** `default` = `~/.hermes` itself (source: `profiles.py:14-16`).
Named profiles at `~/.hermes/profiles/<name>/` (source: `profiles.py:13`).
Names: `[a-z0-9][a-z0-9_-]{0,63}` (source: `profiles.py:25`). Reserved:
`hermes`, `default`, `test`, `tmp`, `root`, `sudo` + all hermes subcommands
(source: `profiles.py:83-97`).

## CLI surface

| Subcommand | Action | Source |
|---|---|---|
| `list` | List all profiles | `profiles.py:list_profiles()` |
| `use <name>` | Set sticky active profile | `profiles.py:set_active_profile()` |
| `create <name>` | Create a new profile | `profiles.py:create_profile()` |
| `delete <name>` | Delete profile + alias + service | `profiles.py:delete_profile()` |
| `show <name>` | Show profile details | `profiles.py:ProfileInfo` |
| `describe <name>` | Read/set/auto-generate description | `profile_describer.py:describe_profile()` |
| `alias <name>` | Manage wrapper script | `profiles.py:create_wrapper_script()` |
| `rename <old> <new>` | Rename profile | `profiles.py:rename_profile()` |
| `export <name>` | Export to tar.gz | `profiles.py:export_profile()` |
| `import <archive>` | Import from tar.gz | `profiles.py:import_profile()` |
| `install <source>` | Install distribution from git/local | `profile_distribution.py:install_distribution()` |
| `update <name>` | Re-pull distribution | `profile_distribution.py:update_distribution()` |
| `info <name>` | Show distribution manifest | `profile_distribution.py:describe_distribution()` |

Parser details: `subcommands/profile.py`.

## How to use this skill (progressive disclosure)

1. **Read this page** — understand the model and which CLI command does what.
2. **Follow the intent map below** — pick the reference that matches your task.
3. **Deep-dive into a reference** when you need details (source citations, flags, edge cases).

### Intent → reference map

| You want to… | Open this reference |
|---|---|
| Create, clone, seed skills, or delete a profile | [`references/creation-lifecycle.md`](references/creation-lifecycle.md) |
| Understand `-p` / `--profile` flag and HERMES_HOME resolution | [`references/cli-internals.md`](references/cli-internals.md) |
| Configure `config.yaml`, `SOUL.md`, `profile.yaml`, `.env` | [`references/config-files.md`](references/config-files.md) |
| Understand how auth falls back from profile to global | [`references/auth-model.md`](references/auth-model.md) |
| Create/manage wrapper aliases, rename, or set active profile | [`references/wrappers-aliases.md`](references/wrappers-aliases.md) |
| Export/import a profile as tar.gz | [`references/export-import.md`](references/export-import.md) |
| Install/update/info a git-based distribution | *(see `hermes-profile-distributions` skill)* |
| Understand gateway multiplexing and per-profile services | [`references/gateway-integration.md`](references/gateway-integration.md) |
| Auto-generate descriptions, kanban routing by role | [`references/descriptions-kanban.md`](references/descriptions-kanban.md) |
| Browse directory structure, reserved names, file locations | [`references/reference-tables.md`](references/reference-tables.md) |

## Primary source files

| File | What it contains |
|---|---|
| `hermes_cli/profiles.py` | All CRUD, path helpers, wrappers, export/import, rename, active profile |
| `hermes_cli/profile_distribution.py` | Distribution manifest, install/update/info, env template, git staging |
| `hermes_cli/profile_describer.py` | LLM-based description autogeneration, skill collection |
| `hermes_cli/subcommands/profile.py` | CLI argument parsers for every profile subcommand |
| `hermes_cli/_parser.py` | `-p`/`--profile` pre-argparse flag |
| `hermes_cli/auth.py` | Credential shadowing, profile vs global auth |
| `hermes_cli/config.py` | `DEFAULT_CONFIG`, `load_config()`, migration |
| `hermes_cli/default_soul.py` | `DEFAULT_SOUL_MD` template |

All citations reference `~/Local/docs/hermes-agent/`.

## Related skills

- [`hermes-profile-distributions`](../hermes-profile-distributions/SKILL.md) —
  distribution.yaml authoring, security audit, 5 use cases
- [`hermes-dev-kit`](../hermes-dev-kit/SKILL.md) — general Hermes architecture,
  memory providers, curator, auto-triggers
