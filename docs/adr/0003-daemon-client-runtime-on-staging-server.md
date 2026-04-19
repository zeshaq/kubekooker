# ADR 0003: Daemon + client runtime on a Linux staging server

## Status
Accepted

## Context
Two hard requirements collided:

1. Monitors (cluster health, ArgoCD sync, log tailing, git drift) run for hours and must survive the UI closing.
2. Operations like VM provisioning and cluster installs run against infrastructure; privileged remote execution (SSH to servers) is required.

Running on the developer's laptop fails requirement 1 (closing the window kills the process) and complicates requirement 2 (laptop sleeps, disconnects, changes networks).

## Decision
Kubekooker runs as a **systemd service on a dedicated Linux staging server**. The user's laptop is used only for writing code and pushing to GitHub.

- Staging server: `dl385` (ssh `ze@dl385`, passwordless sudo).
- Daemon packaging: system-unit under `/etc/systemd/system/kubekooker.service`, running as a dedicated `kubekooker` user.
- Daemon binds API to `127.0.0.1:PORT` on the server.
- Browser UI access via **SSH tunnel** — `ssh -L PORT:localhost:PORT ze@dl385` — user opens `http://localhost:PORT` locally.
- Auth between UI and daemon: bearer token written to a mode-0600 file on the server, copied once to the laptop for the UI.

## Consequences

**Good:**
- Monitors outlive UI sessions cleanly.
- No TLS cert management needed for v1 (tunnel carries auth).
- Daemon restart via standard `systemctl` flows.
- Minimal external attack surface — nothing listening on public ports.

**Bad:**
- Laptop offline ⇒ no UI access (daemon still runs; catch up on reconnect).
- Developer must run the tunnel; not a double-click app experience.
- Single point of failure: if the server is down, so is kubekooker.

**Mitigations:**
- Auto-resume monitors on daemon start (see ADR-0005).
- UI reconnects automatically via SSE `Last-Event-ID` after tunnel reconnects.
- v2 option to expose behind Caddy/Traefik + TLS for non-tunnel access.

## Superseded decisions
This supersedes an earlier "local-first on laptop" decision and any macOS-specific runtime plans (launchd, Keychain, Tauri tray).
