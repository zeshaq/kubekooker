# ADR 0005: Operations, Monitors, and Watch-until as separate concepts

## Status
Accepted

## Context
Kubekooker runs three shapes of long-lived backend work:

1. **Finite tasks** — install an operator, download a bundle, provision a VM. Have a defined end; progress is meaningful (% or step-count).
2. **Persistent monitors** — cluster health, ArgoCD sync state, pod-log tail, git-drift detection. No expected end; emit events for hours.
3. **Bounded waits** — `oc wait` for a condition with a timeout. Persistent-like runtime, but with an exit condition.

Conflating these into one "Task" concept fails three ways:
- A monitor modeled as `kind=monitor, status=running` forever bloats event tables unbounded.
- A finite operation given a worker-pool slot that never returns starves the pool.
- "Interrupted on restart" is wrong semantics for monitors (they should auto-resume) and right for operations.

## Decision
Three first-class, sibling concepts sharing a single persistence table (with a `category` discriminator) but different lifecycle rules:

| | **Operation** | **Monitor** | **Watch-until** |
|---|---|---|---|
| Ends? | Yes, finite | No — user-stopped | Yes, on condition or timeout |
| On daemon restart | Marked `interrupted` | **Auto-resumes** | Auto-resumes |
| Runtime slot | Bounded worker pool | Dedicated goroutine | Dedicated goroutine |
| Event retention | Full history (capped) | Ring buffer (+ sticky alerts exempt) | Full history |
| On external I/O failure | Fails the op | Reconnects with backoff | Reconnects with backoff |

All three:
- Share a common event schema (monotonic `seq`, typed `kind`, sticky bit).
- Stream to the UI over the same SSE mechanism with `Last-Event-ID` resume.
- Honor the same cancellation API (`DELETE /runs/{id}`).
- Appear in one unified "Activity" view in the UI, split visually.

**Alerts** are a distinct event `kind`, sticky (exempt from ring-buffer pruning), and bubble through a separate SSE feed for global notification rendering.

**Per-cluster serialization** is the default concurrency rail — one op per cluster at a time unless a run is explicitly flagged `parallel-safe`. Monitors are one per cluster per monitor-kind (no fan-out).

**Parent/child** operations form a tree (`install-operator` parent → `download-bundle`, `apply-subscription`, `wait-csv-ready` children), tracked via `parent_id`.

## Consequences

**Good:**
- Correct semantics for each shape; no accidental bloat or starvation.
- Single unified streaming + cancellation API across all three.
- Scales to many simultaneous monitors without pressuring the operation pool.

**Bad:**
- More concepts for contributors to learn up front than "one task table."
- Event pruning logic must be careful to respect the sticky bit.

**Mitigations:**
- ADR + table above is the single-source explanation for new contributors.
- Ring-buffer janitor goroutine is small (one function per monitor).

## Related
- [ADR-0002](0002-sqlite-for-state-storage.md) — event volume drives the storage-strategy concern for monitors.
- [ADR-0007](0007-rest-websocket-sse-openapi-first.md) — SSE is the transport for event streams.
