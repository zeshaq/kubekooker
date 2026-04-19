# ADR 0006: Port-per-concern adapters, not a unified "Protocol" interface

## Status
Accepted

## Context
The initial design called the Go adapter layer a "protocol runner" with the implication that there'd be a single `Protocol` interface every backend implements. The backends in scope are:

- Local `oc` CLI (subprocess, PTY, streamed output)
- OpenShift API (kube-apiserver, RESTful + long-polling watches)
- GitHub (REST + OAuth, rate-limited, paginated)
- SSH to remote hosts (persistent channel, sometimes PTY, sometimes exec)

These are fundamentally different shapes. A single `Protocol` interface would collapse to something like `Run(any) (any, error)` — an `interface{}` in a trench coat — with every caller needing to know which adapter they're talking to to construct the right `any`. Abstraction without subtraction.

## Decision
Reject the unified `Protocol` abstraction. Instead, define **one interface per real concern**, each small, typed, and testable:

```go
// Examples — final signatures during implementation
type ClusterAPI interface {
    Apply(ctx context.Context, kubeconfig *Kubeconfig, manifest []byte) error
    Watch(ctx context.Context, kubeconfig *Kubeconfig, resource ResourceRef) (<-chan WatchEvent, error)
    ...
}

type GitHost interface {
    CommitFiles(ctx context.Context, repo RepoRef, files []File, msg string) (CommitSHA, error)
    OpenPR(ctx context.Context, repo RepoRef, branch, title, body string) (PRRef, error)
    ...
}

type ShellExecutor interface {
    RunPTY(ctx context.Context, target ExecTarget, cmd []string, io PTYIO) error
    // target resolves to local subprocess OR SSH session
}

type FileStore interface { ... }  // encrypted-at-rest credential + kubeconfig storage
```

All adapters live in the `proto/` package, organized by port. Each port has:
- A real implementation.
- A fake implementation.
- A **shared compliance test suite** both must pass.

**Drop "plugin" language.** Adapters are compiled into the binary. Adding one is a code change + recompile, not a runtime plug-in. Go's runtime `plugin` package is brittle and Linux-only; the simpler answer is the right one.

## Consequences

**Good:**
- Each interface stays small and meaningful; no god-interface.
- Fakes are easy to write and keep honest via the shared compliance suite.
- Manager code is typed against purpose, not against `any`.
- New backends for existing concerns (e.g., GitLab as a `GitHost`) drop in cleanly.

**Bad:**
- More interfaces to learn than "one Protocol."
- Adding a genuinely new concern (e.g., Terraform) requires defining a new port, not just implementing an existing one.

**Mitigations:**
- Ports live in `proto/ports/` with consistent naming and doc comments.
- Each new adapter lands with its compliance-suite contribution in the same PR.
