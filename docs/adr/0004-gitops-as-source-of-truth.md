# ADR 0004: GitOps as source of truth; kubekooker is a git-writer

## Status
Accepted

## Context
Cluster-management tools commonly fall into one of two shapes:

1. **Orchestrator**: the tool itself reconciles cluster state. Imperative `oc apply` on the tool's timeline. The tool's DB is authoritative.
2. **GitOps writer**: the tool commits manifests to a git repo; an in-cluster controller (ArgoCD / Flux) reconciles. Git + the cluster are authoritative. The tool's DB is a cache.

The orchestrator shape is simpler to build but produces an authority conflict: the tool disagrees with the cluster whenever someone edits outside the tool, whenever reconciliation fails, whenever a rollback happens through git. The GitOps shape inherits a battle-tested reconciliation loop and aligns with how production OpenShift clusters are operated.

## Decision
Kubekooker is a **git-writer**, not an orchestrator.

- Each kubekooker project is backed by a dedicated GitHub repo (the "project repo").
- Every cluster-facing change kubekooker makes is a commit to that repo.
- Reconciliation is performed by ArgoCD or Flux running in-cluster, outside kubekooker's control.
- Kubekooker's SQLite DB is a cache of observed state, never authoritative.

The one allowed exception is the user's **interactive terminal** (xterm.js) — direct `oc` commands bypass git by design, for debugging and ad-hoc use.

## Consequences

**Good:**
- Full audit trail in git (who committed what, when, why).
- Rollback is `git revert`.
- Cluster-side reconciliation is already solved by mature controllers.
- Works when kubekooker is offline.

**Bad:**
- Every UI action has a commit latency (push + webhook + reconciler loop).
- State drift is possible (ArgoCD says X, cluster has Y, kubekooker DB has Z) — need a refresh/sync model.
- Users must have ArgoCD or Flux installed; kubekooker does not install it for them in v1.

**Mitigations:**
- A cluster refresh job that reconciles kubekooker's DB with the observed cluster state on a schedule.
- Explicit "sync" UI action for users who don't want to wait for the scheduled refresh.
- Drift banners in the UI when DB and live state diverge.

## Open
Whether kubekooker commits directly to `main` of the project repo (fast UX) or always opens a PR (auditable + approval-gated). Default will be **PR-mode with auto-merge for trusted members** — GitOps-native answer. Revisit as a separate ADR when the auth model is settled.
