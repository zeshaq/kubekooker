# Non-goals

What kubekooker explicitly will **not** do. This list prevents scope creep — especially from agents that might otherwise "helpfully" add features beyond what was asked.

Adding to this list is cheap; removing from it requires an ADR.

## v1 non-goals

- **Multi-tenancy.** Single-user, single-daemon. No concept of organizations, teams, or user isolation.
- **User authentication / RBAC.** Inter-process bearer token only. If it's on the network tunnel, you own it.
- **SaaS / hosted offering.** Kubekooker runs on the user's own server. No cloud control plane.
- **Windows runtime.** Linux server only (specifically `dl385` for the primary author's setup). The laptop-side tooling is macOS-first but not Windows-hostile.
- **Generic Kubernetes.** OpenShift-specific. Uses Operators, Routes, OperatorHub, SCCs. Vanilla k8s is not a supported target.
- **Cluster installation.** Kubekooker manages existing clusters. It does not install OpenShift from bare metal or from an installer payload. Bring your own cluster.
- **GitOps controller installation.** Kubekooker assumes ArgoCD or Flux is already running in each managed cluster. It does not install them for you.
- **Electron.** Browser UI served by the daemon (or a lightweight native wrapper if ever needed). No Electron binary shipped.
- **Runtime plugins.** Adapters are compiled in. Adding an adapter is a code change + rebuild, not a runtime drop-in. See ADR-0006.
- **Drive a GitOps controller directly.** Kubekooker commits to git. ArgoCD/Flux reconciles. Kubekooker does not manipulate Application CRs or GitOps-controller state directly.
- **Replace `oc` or `kubectl`.** The terminal is a thin PTY proxy. Kubekooker does not parse or synthesize `oc` behavior; it just runs the binary.
- **Support for legacy OpenShift 3.x.** OpenShift 4.x+ only.

## Things that may become goals later (but aren't now)

- Multi-user web UI (would require RBAC).
- Non-SSH-tunnel access (Caddy/Traefik + Let's Encrypt for TLS-public ports).
- GitLab as a `GitHost` adapter.
- Terraform / Ansible / Helm as additional port concerns.
- Scheduled operation execution (cron-like).

When any of these move from the second list to an active goal, write an ADR.
