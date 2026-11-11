# Peerimeter — Implementation Progress

> Living log for Codex CLI. Read `plan.md` and `additional-plan.md` first. Update this file after each coherent implementation task and each session. Entries below record verified work and explicitly separate native backend feasibility from agent tunnel integration.

## Current status

- **Active phase:** Phase 1 — Native offline agent runtime and local status/service lifecycle (not started); Phase 0 complete
- **Last verified milestone:** Phase 0 static-config/build/unit/CLI baseline passed natively with Go 1.27.4 on all three OSes; real WireGuard backend feasibility also passed (2026-11-11).
- **Repositories:** `peerimeter-agent` (P0/P1, first); `peerimeter-service` (Phase 2 onward). Two independent Git repositories.
- **Next action:** Implement Phase 1 native agent adapters/IPC and service lifecycle to make run/status/peers operational; preserve the tested static validation and key contract.

## Phase checklist

| Phase | Status | Verification / notes |
|---|---|---|
| 0 — Cross-platform baseline + static JSON | Complete | Independent agent repo, strict config/key handling, CLI contracts/validation, testable lifecycle/interfaces, native CI and backend feasibility verified on all three; operational run/status/peers explicitly belong to Phase 1 |
| 1 — Offline WireGuard on Linux/macOS/Windows | Not started | All three OSes must work without API using static JSON; Linux interface naming from `additional-plan.md` |
| 2 — Server + join bundles + sync | Not started | |
| 3 — NAT traversal + self-healing | Not started | Direct connections only; no relay |
| 4 — Browser dashboard + GitHub SSO | Not started | |
| 5 — External WireGuard client gateway | Not started | |
| 6 — Cross-platform resilience + packaging | Not started | |
| 7 — Release and hardening | Not started | |

## Two-repository checklist

- [x] `peerimeter-agent` initialized with independent `.git`, `go.mod`, and CI.
- [ ] `peerimeter-service` initialized separately at Phase 2 with independent `.git`, `go.mod`, and CI.
- [ ] Cross-repo versioned HTTP/JSON and MQTT contracts defined and tested without relative module imports.
- [ ] Record tested commit hash and status for each repository.

## GitHub Actions verification matrix (mandatory from Phase 0)

Status must be one of: **not run / passed / failed / blocked**. Do not change any cell to `passed` without native runner evidence. Record workflow run URLs, commit SHA, date, runner image, WireGuard backend and blockers in the log below.

| OS runner | Build + unit + static JSON (`ci.yml`) | Real WG handshake + traffic (`wireguard-integration.yml`) | Cross-OS / disruption E2E (`connectivity-e2e.yml`) |
|---|---|---|---|
| Ubuntu 24.04 | passed (`ef0abfc`) | passed — backend feasibility (`ebb4a1f`), agent integration not run | not run |
| macOS 15 | passed (`ef0abfc`) | passed — backend feasibility (`ebb4a1f`), agent integration not run | not run |
| Windows 2022 | passed (`ef0abfc`) | passed — backend feasibility (`ebb4a1f`), agent integration not run | not run |

- [x] `peerimeter-agent/.github/workflows/ci.yml` added; matrix runs natively on all three OSes, not just cross-compile.
- [ ] `peerimeter-agent/.github/workflows/wireguard-integration.yml` added with real interface/handshake/bidirectional traffic assertions and cleanup.
- [ ] `peerimeter-agent/.github/workflows/connectivity-e2e.yml` added with Linux namespaces and provision for mixed-OS tests.
- [x] macOS/Windows GitHub-hosted tunnel feasibility checked: both passed on the exact target images; no self-hosted fallback currently needed.
- [x] All integration skips and missing infrastructure are reported explicitly, never as a pass; agent E2E remains unavailable and manual E2E workflow fails with a blocked explanation.
- [ ] `peerimeter-service` independent CI added at Phase 2.

## Additional requirements checklist

- [x] P0 static JSON schema versioning, validation and protected local key references.
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
| `peerimeter-agent` | `ef0abfcf23c385355f383eeda69509eb33dc2962` (native baseline); `ebb4a1f` (backend feasibility) | Phase 0 complete; Phase 1 runtime/IPC next | Native CI and backend feasibility each passed on all three OSes |
| `peerimeter-service` | None | Not started; wait for agent milestone | Not run |

## Completed work

- Created [peerimeter-agent](https://github.com/peerimeter/peerimeter-agent) as an independent repository. No service code/repository created ahead of Phase 2.
- Implemented versioned strict static JSON parsing/validation, two public templates, secure fresh example generation, private key handling, Linux naming/ownership helpers, platform interfaces and foreground cleanup lifecycle tests.
- Added all three workflow files. `run`, `status` and `peers` explicitly report unavailable until the native agent/IPC implementation in Phase 1; no fake running tunnel is exposed.
- Go **1.27.4** is required in the plan, modules and native CI. Agent self-update is a Phase 7 task.
- Backend feasibility uses real official WireGuard implementations, independent of the unfinished agent.


## Tests executed

- [Initial native CI, `ebb4a1f`](https://github.com/peerimeter/peerimeter-agent/actions/runs/42942950063): Go 1.27.4 build/vet/race/unit/CLI passed on Ubuntu/macOS; Windows failed at the version/format shell gate before tests. Follow-up `ef0abfc` enforces LF checkout, normalizes CR in version output, adds format diagnostics and macOS extended-ACL tests.
- [Native backend feasibility, `ebb4a1f`](https://github.com/peerimeter/peerimeter-agent/actions/runs/42942949983): all three passed actual interface/route creation, authenticated handshake, bidirectional traffic, restart and teardown. Evidence artifacts include SHA, image/version, architecture, timestamp and run URL.
- Runners: Ubuntu 24.04 x64 image `20261101.360.1` (kernel WG); macOS 15 ARM64 image `20261104.0514.1` (native utun/wireguard-go); Windows 2022 x64 image `20261101.353.1` (WireGuardNT, official signed MSI test prerequisite).
- [Follow-up native CI, `ef0abfcf23c385355f383eeda69509eb33dc2962`](https://github.com/peerimeter/peerimeter-agent/actions/runs/42943244505): **passed on all three OSes**, including exact Go 1.27.4, formatting, vet, race/unit tests, build and protected example generation/validation. Windows first-run gate failure resolved; native macOS extended-ACL and Windows DACL security tests passed.
- Local Go 1.27.4 toolchain segfaults in this Termux/Nix ARM64 environment, including `go version`; local Go 1.26.4 with an external temporary manifest passed unit/vet/build/CLI checks as diagnostics only. Workflow actionlint and Bash syntax passed. No local result is represented as Go 1.27.4 verification.


## Known blockers / limitations

- Phase 1 agent backends, real `run`/local `status`/`peers`, service managers, persisted local interface selection and actual OS collision detection are not implemented.
- The backend test MSI is not the required offline Windows-agent installer; bundling the signed WireGuardNT component remains Phase 1 work.
- Connectivity E2E, NAT/IP-change recovery and mixed-OS agent links are not run; separate hosted runner jobs do not share a LAN.
- The privileged workflow currently verifies backend feasibility; the stronger agent integration and E2E checklist items remain open.


## Architectural decisions

- No relay and no desktop UI.
- Separate `peerimeter-agent` and `peerimeter-service` Git repositories; no monorepo.
- Cross-platform static JSON WireGuard agent is P0; API is optional until Phase 2.
- Managed join bundle includes both HTTPS API URL and opaque enrollment token; Base64URL is not encryption.
- Linux interface naming is local to each enrolled agent/network; see `additional-plan.md`.

## Next steps

1. Implement the Phase 1 native agent/IPC adapters and make run/status/peers operational on all three OSes.
2. Keep the now-passing native Linux/macOS/Windows Go 1.27.4 CI matrix green and record actual results per tested code SHA.
3. Reuse the verified static JSON contract and Linux name resolver; add persisted local selection and actual OS collision detection in Phase 1.
4. Implement and **natively verify** WireGuard adapters on Linux, macOS and Windows before server/dashboard work.
5. Add Base64URL join bundle parsing and enrollment only in Phase 2.

## Session log — 2026-11-11

- Agent repository created with first commit `ebb4a1f`; all three real backend
  probes passed. Follow-up `ef0abfc` adds macOS extended ACL protection and fixes
  cross-platform line endings; all three native baseline jobs passed.
- Both repositories' progress logs distinguish tested code SHA from later
  documentation-only commits. Phase 0 is not declared fully complete while
  operational run/status/peers depend on unimplemented Phase 1 native/IPC adapters.
- Go pin and future self-update requirements were committed/pushed separately in
  planning commits `3c9a31d` and `025d12c`.
- Added Phase 1 requirements for Linux/macOS `install.sh` and Windows `install.ps1`: latest stable by default, explicit version and installation directory, matching verified release artifacts and native installer tests. Added README presentation/navigation and platform-specific installation examples as a Phase 1 task; neither installer implementation nor README reformatting is claimed complete.

## Phase boundary clarification — 2026-11-11

The earlier session left Phase 0 open because operational run/status/peers require
Phase 1 native/IPC backends. That scheduling dependency is now corrected in
plan.md and additional-plan.md: Phase 0 owns command/argument contracts, functional
offline validation and testable lifecycle/adapter interfaces; Phase 1 owns working
tunnel/service operation and observed local status/peers. Prior session notes
above describe the earlier checklist wording and are superseded by this entry.

Phase 0 is complete against the clarified criteria, supported by native CI at
`ef0abfcf23c385355f383eeda69509eb33dc2962` and backend feasibility at
`ebb4a1f880e39e2ca1003d9551c185a76ee81069` (run links above). No implementation or
test result changed in this documentation correction. Phase 1 remains not started;
there is no claim of working agent tunnels, IPC or service management.
