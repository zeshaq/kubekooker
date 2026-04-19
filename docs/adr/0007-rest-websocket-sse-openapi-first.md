# ADR 0007: gui↔manager via REST + WebSocket + SSE with OpenAPI-first schema

## Status
Accepted

## Context
The gui↔manager boundary carries three traffic shapes:

1. **CRUD** on projects, clusters, kubeconfigs, credentials. Typical request/response.
2. **Interactive terminal** (xterm.js) — bidirectional byte stream, low latency, binary frames for ANSI.
3. **Operation / monitor event streams** — server-pushed, resumable, structured events.

No single transport fits all three. The frontend is developed in a separate worktree from the backend; multiple agents will work in parallel — contract drift is a real risk.

The `proto` layer is internal to the daemon. The browser must not talk to it directly — credentials and privileged executors must stay one hop away from the client.

## Decision

**Transport per shape:**

| Shape | Transport | Why |
|---|---|---|
| CRUD | HTTP + JSON (REST) | Simple, cacheable, matches request/response |
| Terminal | WebSocket (binary frames) | Bidirectional, low-latency, fits xterm.js |
| Event streams | Server-Sent Events (SSE) | Unidirectional, native `Last-Event-ID` resume, HTTP-native |

**Boundary rule:** the React client communicates only with `manager`. `proto` is manager's private dependency, invoked via Go interfaces (ADR-0006). The daemon is a single Go binary in v1; manager and proto share a process.

**Schema contract:** OpenAPI 3 in `shared/schema/openapi.yaml` is the source of truth for the REST surface.
- Go server stubs generated with `oapi-codegen`.
- TS client generated with `openapi-typescript` + `openapi-fetch`.
- Generated code is committed to `shared/schema/generated/` so agents can read types without running codegen.
- CI fails if the schema changes without regenerated artifacts.

WebSocket and SSE messages use JSON payloads where possible and reference schema types from the OpenAPI components.

**Auth:** bearer token (the daemon's local secret — see ADR-0003) on REST, WS, and SSE alike.

**Skip:** gRPC-Web / Connect-Web for v1. Their typing benefits don't offset the proxy/HTTP2 complexity for a local-first single-binary daemon. Revisit if the API surface balloons.

## Consequences

**Good:**
- Each transport does one job well.
- OpenAPI-first gives the multi-worktree team a single artifact both gui and manager read.
- SSE resumability handles UI reconnects after tunnel drops without custom logic.

**Bad:**
- Three transports means three client code paths in the browser.
- Schema-drift CI must be strict or contracts silently rot.

**Mitigations:**
- A thin `gui/api/` layer wraps all three transports behind a typed TS facade.
- Contract tests (consumer-driven examples in `shared/contract/`) catch behavior drift that schema checks miss.

## Related
- [ADR-0006](0006-port-per-concern-adapters.md) — the internal manager↔proto interfaces.
