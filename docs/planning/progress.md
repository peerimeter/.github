# Peerimeter — Implementation Progress

> Living log for Codex CLI. Read `plan.md` and `additional-plan.md` first. Update this file after each coherent implementation task and each session. Entries below reflect **no verified implementation work yet**; inspect the repository before changing statuses.

## Current status

- **Active phase:** Phase 0 — Cross-platform baseline and static JSON contract
- **Last verified milestone:** None
- **Repositories:** `peerimeter-agent` (P0/P1, first); `peerimeter-service` (Phase 2 onward). Two independent Git repositories.
- **Next action:** Inspect both repository directories; implement Phase 0, then build and natively verify offline static-JSON WireGuard on all three operating systems (Phase 1).

## Phase checklist

| Phase | Status | Verification / notes |
|---|---|---|
| 0 — Cross-platform baseline + static JSON | Not started | |
| 1 — Offline WireGuard on Linux/macOS/Windows | Not started | All three OSes must work without API using static JSON; Linux interface naming from `additional-plan.md` |
| 2 — Server + join bundles + sync | Not started | |
| 3 — NAT traversal + self-healing | Not started | Direct connections only; no relay |
| 4 — Browser dashboard + GitHub SSO | Not started | |
| 5 — External WireGuard client gateway | Not started | |
| 6 — Cross-platform resilience + packaging | Not started | |
| 7 — Release and hardening | Not started | |

## Two-repository checklist

- [ ] `peerimeter-agent` initialized with independent `.git`, `go.mod`, and CI.
- [ ] `peerimeter-service` initialized separately at Phase 2 with independent `.git`, `go.mod`, and CI.
- [ ] Cross-repo versioned HTTP/JSON and MQTT contracts defined and tested without relative module imports.
- [ ] Record tested commit hash and status for each repository.

## GitHub Actions verification matrix (mandatory from Phase 0)

Status must be one of: **not run / passed / failed / blocked**. Do not change any cell to `passed` without native runner evidence. Record workflow run URLs, commit SHA, date, runner image, WireGuard backend and blockers in the log below.

| OS runner | Build + unit + static JSON (`ci.yml`) | Real WG handshake + traffic (`wireguard-integration.yml`) | Cross-OS / disruption E2E (`connectivity-e2e.yml`) |
|---|---|---|---|
| Ubuntu 24.04 | Not run | Not run | Not run |
| macOS 15 | Not run | Not run | Not run |
| Windows 2022 | Not run | Not run | Not run |

- [ ] `peerimeter-agent/.github/workflows/ci.yml` added; matrix runs natively on all three OSes, not just cross-compile.
- [ ] `peerimeter-agent/.github/workflows/wireguard-integration.yml` added with real interface/handshake/bidirectional traffic assertions and cleanup.
- [ ] `peerimeter-agent/.github/workflows/connectivity-e2e.yml` added with Linux namespaces and provision for mixed-OS tests.
- [ ] macOS/Windows GitHub-hosted tunnel feasibility checked and, if blocked, self-hosted runner requirement recorded.
- [ ] All integration skips and missing infrastructure are reported explicitly, never as a pass.
- [ ] `peerimeter-service` independent CI added at Phase 2.

## Additional requirements checklist

- [ ] P0 static JSON schema versioning, validation and protected local key references.
- [ ] `peerimeter-agent run --config <path>` makes no API/MQTT/STUN calls.
- [ ] Real WireGuard connectivity from static configs on Linux, macOS and Windows, each natively verified.
- [ ] systemd, launchd and Windows Service lifecycle with local offline `status` and `peers`.
- [ ] Static config remains usable after online enrollment is added.
- [ ] Phase 2 join bundle is Base64URL(unpadded JSON with `version`, `api_url`, `enrollment_token`).
- [ ] `join --token <bundle>` derives server URL from token; `--token-file` supported.
- [ ] Validate HTTPS URL, credential expiry/replay, secrets and error/log redaction.

- [ ] Default Linux interface `pm-$networkName`, with safe deterministic name shortening.
- [ ] Linux enrollment override via `peerimeter-agent join --interface <name>`.
- [ ] Persist local interface selection across restarts and network renames.
- [ ] Detect invalid names and local interface collisions.
- [ ] WireGuard lifecycle, routing, and status use resolved local name.
- [ ] Tests and docs for interface naming.

## Per-repository status

| Repository | Last verified commit | Active work | Tests |
|---|---|---|---|
| `peerimeter-agent` | None | Phase 0 / Phase 1 | Not run |
| `peerimeter-service` | None | Not started; wait for agent milestone | Not run |

## Completed work

_None verified yet._

## Tests executed

_None executed yet._

## Known blockers / limitations

_None recorded yet._

## Architectural decisions

- No relay and no desktop UI.
- Separate `peerimeter-agent` and `peerimeter-service` Git repositories; no monorepo.
- Cross-platform static JSON WireGuard agent is P0; API is optional until Phase 2.
- Managed join bundle includes both HTTPS API URL and opaque enrollment token; Base64URL is not encryption.
- Linux interface naming is local to each enrolled agent/network; see `additional-plan.md`.

## Next steps

1. Examine both repository directories and reconcile this starter file with existing code.
2. Implement Phase 0 in small, tested increments, immediately creating the native Linux/macOS/Windows GitHub Actions CI matrix and recording actual results.
3. Implement the versioned static JSON config and Linux interface-name resolution in Phase 0/1.
4. Implement and **natively verify** WireGuard adapters on Linux, macOS and Windows before server/dashboard work.
5. Add Base64URL join bundle parsing and enrollment only in Phase 2.
