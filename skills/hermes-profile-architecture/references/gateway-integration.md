# Gateway integration

## Per-profile gateway

Each profile can run its own gateway process (messaging server). The gateway is supervised via:
- **systemd** (Linux host): per-profile user service
- **launchd** (macOS host): per-profile plist
- **s6** (container): per-profile service slot under s6-supervise
- **Windows**: task-based or service

## Gateway multiplexing

`profiles_to_serve(multiplex)` — source: `profiles.py:profiles_to_serve()`, lines 196-225.

This is the single chokepoint for "which profiles does the gateway handle":

| `multiplex` | Returns |
|---|---|
| `False` (default) | Exactly one entry — the active profile. Name is `"default"` or the named profile id. |
| `True` | Every valid profile (default + all named), each paired with its own HERMES_HOME. |

Lightweight: directory scan + name validation only — no config reads, gateway probes, or skill counts.

## Service lifecycle on delete

When a profile is deleted (source: `profiles.py:delete_profile()`):

1. Stops the gateway process via `gateway.pid` + `terminate_pid()` (graceful → force after 10s)
2. Stops other profile-bound backends (Desktop-spawned `serve`/`dashboard`) via `_profile_bound_backend_pids()` — these hold SQLite connections that block `rmtree`
3. Disables and removes systemd/launchd service (prevents auto-restart)
4. If container/s6: unregisters the s6 service slot

### Backend PID detection

`_profile_bound_backend_pids(canon, profile_dir)` — source: `profiles.py` ~lines 454-487:

- Uses psutil to find current-user processes
- Checks for backend subcommand tokens (`serve`, `dashboard`, `gateway`)
- Bound to profile by `--profile`/`-p` flag in argv or by `HERMES_HOME` env var
- Never kills the calling process or its ancestors

## s6 container path

In container deployments, `_maybe_register_gateway_service()` registers profile gateways as runtime s6 services on create. Each supervised gateway loads its own HERMES_HOME and binds the port from `gateway/config.py` — defaulting to 8642. Two profiles with default port need distinct `API_SERVER_PORT` in `.env`.

Source: `profiles.py:_maybe_register_gateway_service()`, `_maybe_unregister_gateway_service()`.
