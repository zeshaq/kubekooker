# ADR 0002: SQLite for state storage

## Status
Accepted

## Context
Kubekooker's daemon needs to persist projects, clusters, kubeconfigs (encrypted), operations, monitors, events, and audit entries. It runs as a single-user, single-host daemon — no multi-tenant database is required. Truth for cluster state lives in git + the cluster itself; kubekooker's DB is a cache + workflow store.

## Decision
SQLite as the sole state store. Single `.db` file under the daemon's data directory. WAL mode enabled for concurrent read/write from monitor goroutines.

Preferred driver: `modernc.org/sqlite` (pure Go, no CGo) unless performance demands `mattn/go-sqlite3`.

Migrations via `golang-migrate`, `goose`, or `atlas` — exact tool TBD but picked before v0.1.

## Consequences

**Good:**
- Zero-ops deployment — no separate database process.
- Single file makes backup/restore trivial.
- Excellent local performance.

**Bad:**
- Single-writer serialization can bottleneck under high monitor-event volume.
- No network-accessible DB for future multi-host setups.

**Mitigations:**
- WAL mode + batched event writes in monitor code paths.
- Per-monitor in-memory ring buffer with periodic flush if event rate strains the DB.
- Ring-buffered retention policy for monitor events (non-sticky events pruned after N events or T hours).

## Related

- [ADR-0005](0005-operations-monitors-watch-until.md) — event volume from monitors drives the storage-strategy concern.
