# Task locks

Decentralized task-claiming for the multi-agent worktree pattern.

When an agent (Claude Code session, or human) starts work on an issue, it commits a lock file here named `ISSUE-<number>.lock`. The file records who claimed it and when.

## Lock file format

Plain text:

```
issue: 42
claimed-by: claude-gui-session-a
claimed-at: 2026-04-19T14:30:00Z
branch: gui/42-kubeconfig-dropdown
```

## Rules

- One lock file per active issue.
- Before claiming, check for an existing lock — if present, do not start.
- Delete the lock file in the same PR that closes the issue.
- Stale locks (claimer hasn't pushed in 48h) are fair game; leave a comment on the issue noting reclaim.

## Why a file and not a GitHub label?

Labels race; committed files are atomic under PR review. This mechanism was chosen specifically to survive concurrent agents who can't see each other's work in real time.
