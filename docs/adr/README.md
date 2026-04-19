# Architecture Decision Records

Every locked architectural choice gets an ADR. Format is Michael Nygard style:

```
# ADR NNNN: Title

## Status
Proposed | Accepted | Superseded by ADR-XXXX

## Context
What's the problem, constraint, or forcing function?

## Decision
What we decided.

## Consequences
Good and bad. Be honest about tradeoffs.
```

## Rules

- ADRs are **append-only**. To change a decision, write a new ADR that supersedes the old one and update the old one's `Status`.
- Number them `NNNN-short-slug.md`, zero-padded to 4 digits.
- Commit ADRs in the same PR as the code that implements them (or first, for wide-blast-radius decisions like repo layout).
- Keep them short — under 40 lines where possible. If an ADR needs a diagram or 200 lines of context, something is wrong.

## Why ADRs and not just the memory files?

Memory files (`~/.claude/projects/...`) are invisible to agents running in a fresh worktree. ADRs in the repo are the single source of truth that every agent, human, or reviewer can read cold.

## Index

- [ADR-0001: Monorepo with domain-based layout](0001-monorepo-domain-layout.md)
- [ADR-0002: SQLite for state storage](0002-sqlite-for-state-storage.md)
- [ADR-0003: Daemon + client runtime on a Linux staging server](0003-daemon-client-runtime-on-staging-server.md)
- [ADR-0004: GitOps as source of truth; kubekooker is a git-writer](0004-gitops-as-source-of-truth.md)
- [ADR-0005: Operations, Monitors, and Watch-until as separate concepts](0005-operations-monitors-watch-until.md)
- [ADR-0006: Port-per-concern adapters, not a unified "Protocol" interface](0006-port-per-concern-adapters.md)
- [ADR-0007: gui↔manager via REST + WebSocket + SSE with OpenAPI-first schema](0007-rest-websocket-sse-openapi-first.md)
