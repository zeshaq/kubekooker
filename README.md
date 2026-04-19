# kubekooker

GitOps-based OpenShift cluster management.

**Status:** design phase. No runnable code yet.

## Overview

Kubekooker manages OpenShift clusters. Clusters are grouped into **projects**, and every project is backed by its own GitHub repo. All cluster operations are GitOps-native: kubekooker writes manifests to the project's repo; an in-cluster reconciler (ArgoCD / Flux) applies them.

Each project exposes an xterm.js-based terminal with a kubeconfig dropdown for running `oc` commands against the project's clusters.

## Architecture

Three domains, developed in parallel worktrees:

| Domain | Stack | Role |
|---|---|---|
| `gui/` | React + Vite + xterm.js | Browser UI |
| `manager/` | Go | Application core — workflows, state, API, orchestration |
| `proto/` | Go | Adapters to external systems: `oc`, OpenShift API, GitHub, SSH |

Shared contracts live in `shared/`. Local state in SQLite. Runs as a systemd daemon on a Linux server; UI connects via SSH tunnel.

## Design docs

- [Architecture Decision Records](docs/adr/) — the `ADR-000N` series captures every locked architectural choice.
- [Non-goals](docs/NON_GOALS.md) — what kubekooker explicitly will not do.

## Repo layout

```
kubekooker/
├── gui/                   # React + Vite + xterm.js
├── manager/               # Go application core
├── proto/                 # Go adapters (oc, OpenShift API, GitHub, SSH)
├── shared/
│   ├── schema/            # OpenAPI schema + generated types
│   └── docs/              # cross-domain running docs (e.g., failed approaches)
├── deploy/                # systemd unit, install scripts, dev compose
├── docs/
│   ├── adr/               # architecture decision records
│   └── NON_GOALS.md
├── .github/
│   └── ISSUE_TEMPLATE/    # feature, bug, spike, agent-task, qa-finding
└── .claude/
    ├── agents/            # subagent definitions (review, verification)
    └── tasks/             # text-file locks for decentralized task claiming
```

## Development pattern

Multi-agent worktree pattern — up to 3 parallel Claude Code sessions, one per domain. Coordination via issue acceptance criteria, shared OpenAPI schema, and task-lock files. See [ADR-0001](docs/adr/0001-monorepo-domain-layout.md).
