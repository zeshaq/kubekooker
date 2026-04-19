# ADR 0001: Monorepo with domain-based layout

## Status
Accepted

## Context
Kubekooker has three distinct concerns: a React frontend, a Go application core, and a Go adapter layer that talks to `oc`, OpenShift API, GitHub, and SSH. Development is planned around parallel Claude Code sessions in git worktrees (up to 3 at a time). Worktrees require a single shared repo.

## Decision
Single monorepo. Top-level layout:

```
gui/        React + Vite + xterm.js
manager/    Go application core (workflows, state, HTTP/WS/SSE API)
proto/      Go adapters (oc, OpenShift API, GitHub, SSH)
shared/     cross-domain artifacts (OpenAPI schema, running docs)
deploy/     systemd unit, install scripts
docs/       ADRs + non-goals
.github/    issue templates, workflows
.claude/    subagent defs and task-lock files
```

Each domain has a long-lived worktree branch (`worktree/gui`, `worktree/manager`, `worktree/proto`); feature branches fork off them; PRs merge into `main`.

## Consequences

**Good:**
- Single history, single schema, atomic cross-domain changes possible.
- Worktrees work naturally (one repo, multiple checkouts).
- CI can use path filters to run only affected pipelines.

**Bad:**
- Cannot release domains on independent cadences.
- Larger clone; first-time setup needs worktree hygiene.

**Mitigations:**
- Path-filtered CI per domain.
- Shared contracts live in `shared/` — changes there trigger all domain pipelines.
