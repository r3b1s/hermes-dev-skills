# Auth model: credential shadowing

Source: `hermes_cli/auth.py`, lines 917-1553.

Profiles have **per-provider credential shadowing** from the global root. This means a profile can use providers authenticated at the global level without re-entering keys, while still having its own credential pool for overrides.

## How it works

- **Read**: `_load_provider_state()` first looks in the profile's own credential pool (inside the profile's `auth.json`). If the profile has **zero entries** for a given provider, it falls back to the global-root `auth.json` for that provider. (source: `auth.py:1269-1274`, `auth.py:1435-1445`)
- **Write**: always goes to the profile's own `auth.json`. Never writes to global. (source: `auth.py:1416`)
- **Write-through to global**: the `target_path` parameter enables profile-to-global writes when needed (e.g., `nous` provider that should be shared across profiles). (source: `auth.py:1088`)
- **Profile state fully shadows** the global state once the profile has ANY entries for a provider — on the next write, profile entries always win. (source: `auth.py:1411-1413`)

## File resolution

In profile mode, `auth.json` at the profile root is the primary auth store. The global `~/.hermes/auth.json` is additionally read as a per-provider fallback.

Source: `auth.py:917-936` (`get_global_auth_store_if_profile()`).

## Key behaviors

- A malformed global auth store must not break profile reads (source: `auth.py:978-979`)
- Profile writes that touch a provider also update the global store for that provider (write-through), keeping both in sync (source: `auth.py:1088`)
- Reads within a profile first check per-provider shadower, then fall back (source: `auth.py:1435-1445`)
- Lock scoping: profile vs global auth stores share credentials but not reentrancy (source: `auth.py:1020`)
