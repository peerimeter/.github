# Peerimeter — Implementation Progress

> Living log for Codex CLI. Read `plan.md` and `additional-plan.md` first. Update this file after each coherent implementation task and each session. Entries below record verified work and explicitly separate native backend feasibility from agent tunnel integration.

## Current status

- **Active phase:** ✅ Phase 1 complete; ⬜ Phase 2 not started
- **Last verified milestone:** Native offline agent, service/IPC, installers, packages and independent review verified for Phase 1 (2026-11-11); exact commits/runs below.
- **Repositories:** `peerimeter-agent` (P0/P1, first); `peerimeter-service` (Phase 2 onward). Two independent Git repositories.
- **Next action:** Choose the first published agent release or begin Phase 2 when requested. No service/control-plane work has started.

## Phase checklist

**Phase status:** ✅ Complete · 🔄 In progress · ⬜ Not started · ⛔ Blocked.
A passed test alone does not mean the phase is complete; Phase 1 and later also require the independent review gate.

| Phase | Status | Verification / notes |
|---|---|---|
| 0 — Cross-platform baseline + static JSON | ✅ Complete | Independent agent repo, strict config/key handling, CLI contracts/validation, testable lifecycle/interfaces, native CI and backend feasibility verified on all three; operational run/status/peers explicitly belong to Phase 1 |
| 1 — Offline WireGuard on Linux/macOS/Windows | ✅ Complete | Native runtime, directory separation, migration, complete uninstall, CLI presentation, installers and independent review passed at `091bd40` |
| 2 — Server + join bundles + sync | ⬜ Not started | |
| 3 — NAT traversal + self-healing | ⬜ Not started | Direct connections only; no relay |
| 4 — Browser dashboard + GitHub SSO | ⬜ Not started | |
| 5 — External WireGuard client gateway | ⬜ Not started | |
| 6 — Cross-platform resilience + packaging | ⬜ Not started | |
| 7 — Release and hardening | ⬜ Not started | |

## Two-repository checklist

- [x] `peerimeter-agent` initialized with independent `.git`, `go.mod`, and CI.
- [ ] `peerimeter-service` initialized separately at Phase 2 with independent `.git`, `go.mod`, and CI.
- [ ] Cross-repo versioned HTTP/JSON and MQTT contracts defined and tested without relative module imports.
- [ ] Record tested commit hash and status for each repository.

## GitHub Actions verification matrix (mandatory from Phase 0)

Status must be one of: **not run / passed / failed / blocked**. Do not change any cell to `passed` without native runner evidence. Record workflow run URLs, commit SHA, date, runner image, WireGuard backend and blockers in the log below.

| OS runner | Build + unit + static JSON (`ci.yml`) | Real WG handshake + traffic (`wireguard-integration.yml`) | Cross-OS / disruption E2E (`connectivity-e2e.yml`) |
|---|---|---|---|
| Ubuntu 24.04 | passed (`4246c6b`) | passed — actual agent (`04648b9`) | not run |
| macOS 15 | passed (`4246c6b`) | passed — actual agent (`04648b9`) | not run |
| Windows 2022 | passed (`4246c6b`) | passed — actual agent (`04648b9`) | not run |

- [x] `peerimeter-agent/.github/workflows/ci.yml` added; matrix runs natively on all three OSes, not just cross-compile.
- [x] `peerimeter-agent/.github/workflows/wireguard-integration.yml` now runs real agent handshake/traffic, restart, cleanup, IPC/operator and native service lifecycle on all three OSes.
- [ ] `peerimeter-agent/.github/workflows/connectivity-e2e.yml` added with Linux namespaces and provision for mixed-OS tests.
- [x] macOS/Windows GitHub-hosted tunnel feasibility checked: both passed on the exact target images; no self-hosted fallback currently needed.
- [x] All integration skips and missing infrastructure are reported explicitly, never as a pass; agent E2E remains unavailable and manual E2E workflow fails with a blocked explanation.
- [ ] `peerimeter-service` independent CI added at Phase 2.

## Additional requirements checklist

- [x] P0 static JSON schema versioning, validation and protected local key references.
- [x] `peerimeter-agent run --config <path>` makes no API/MQTT/STUN calls.
- [x] Real WireGuard connectivity from static configs on Linux, macOS and Windows, each natively verified.
- [x] systemd, launchd and Windows Service lifecycle with local offline `status` and `peers`.
- [ ] Static config remains usable after online enrollment is added.
- [ ] Phase 2 join bundle is Base64URL(unpadded JSON with `version`, `api_url`, `enrollment_token`).
- [ ] `join --token <bundle>` derives server URL from token; `--token-file` supported.
- [ ] Validate HTTPS URL, credential expiry/replay, secrets and error/log redaction.

- [x] Default Linux interface `pm-$networkName`, with safe deterministic name shortening.
- [ ] Linux enrollment override via `peerimeter-agent join --interface <name>`.
- [x] Persist local interface selection across restarts and network renames.
- [x] Detect invalid names and local interface collisions.
- [x] WireGuard lifecycle, routing, and status use resolved local name.
- [x] Tests and docs for interface naming.

## Per-repository status

| Repository | Last verified commit | Active work | Tests |
|---|---|---|---|
| `peerimeter-agent` | `4246c6bea040bfa41a0bd506f0df501310a20367` (unit/packages/review); `04648b9` (actual agent integration, same runtime source) | 🔄 Phase 1 in progress | Native unit, actual agent/service integration and installers passed on all three OSes; independent review passed |
| `peerimeter-service` | None | Not started; wait for agent milestone | Not run |

## Completed work

- Created [peerimeter-agent](https://github.com/peerimeter/peerimeter-agent) as an independent repository. No service code/repository created ahead of Phase 2.
- Implemented versioned strict static JSON parsing/validation, two public templates, secure fresh example generation, private key handling, Linux naming/ownership helpers, platform interfaces and foreground cleanup lifecycle tests.
- Functional `run`, local `status`/`peers`, safe `sync`/`stop`, protected operator IPC, and native service lifecycle are implemented and tested. Cobra owns CLI parsing/help/completion. An additional package workflow builds release archives and verifies installers.
- Go **1.27.4** is required in the plan, modules and native CI. Agent self-update is a Phase 7 task.
- Backend feasibility uses real official WireGuard implementations, independent of the unfinished agent.


## Tests executed

- [Initial native CI, `ebb4a1f`](https://github.com/peerimeter/peerimeter-agent/actions/runs/42942950063): Go 1.27.4 build/vet/race/unit/CLI passed on Ubuntu/macOS; Windows failed at the version/format shell gate before tests. Follow-up `ef0abfc` enforces LF checkout, normalizes CR in version output, adds format diagnostics and macOS extended-ACL tests.
- [Native backend feasibility, `ebb4a1f`](https://github.com/peerimeter/peerimeter-agent/actions/runs/42942949983): all three passed actual interface/route creation, authenticated handshake, bidirectional traffic, restart and teardown. Evidence artifacts include SHA, image/version, architecture, timestamp and run URL.
- Runners: Ubuntu 24.04 x64 image `20261101.360.1` (kernel WG); macOS 15 ARM64 image `20261104.0514.1` (native utun/wireguard-go); Windows 2022 x64 image `20261101.353.1` (WireGuardNT, official signed MSI test prerequisite).
- [Follow-up native CI, `ef0abfcf23c385355f383eeda69509eb33dc2962`](https://github.com/peerimeter/peerimeter-agent/actions/runs/42943244505): **passed on all three OSes**, including exact Go 1.27.4, formatting, vet, race/unit tests, build and protected example generation/validation. Windows first-run gate failure resolved; native macOS extended-ACL and Windows DACL security tests passed.
- Local Go 1.27.4 toolchain segfaults in this Termux/Nix ARM64 environment, including `go version`; local Go 1.26.4 with an external temporary manifest passed unit/vet/build/CLI checks as diagnostics only. Workflow actionlint and Bash syntax passed. No local result is represented as Go 1.27.4 verification.


## Known blockers / limitations

- No blocking Phase 1 code/review findings remain. Mixed-OS VM agent pairs,
  NAT/IP-change recovery and E2E disruption scenarios are **not run**; the E2E
  workflow still explicitly reports blocked until suitable topology exists.
- Linux integration uses two actual agents in separate namespaces; macOS and
  Windows each use an actual native agent against an independent userspace peer.
- Linux amd64, macOS arm64 and Windows amd64 are natively verified. Linux arm64,
  macOS amd64 and Windows arm64 packages cross-build; their runtime is unverified.
- No public GitHub release has been published. Latest/pinned URL resolution uses
  explicit fixtures in native installer tests; real packaged binaries are also
  installed and executed. Live default downloads require publishing a release.
- One named native system service is currently supported per machine. Changing
  macOS/Windows interface addresses requires explicit stop/start. No self-updater
  exists yet (Phase 7). The service repository remains unstarted.

## Architectural decisions

- No relay and no desktop UI.
- Separate `peerimeter-agent` and `peerimeter-service` Git repositories; no monorepo.
- Cross-platform static JSON WireGuard agent is P0; API is optional until Phase 2.
- Managed join bundle includes both HTTPS API URL and opaque enrollment token; Base64URL is not encryption.
- Linux interface naming is local to each enrolled agent/network; see `additional-plan.md`.

## Next steps

1. Publish a first agent release when requested so live `latest` installation is available.
2. Begin Phase 2 control-plane/API/enrollment work only when requested; retain the offline agent contract.
3. Add reachable mixed-OS VM tests and later NAT/disruption recovery tests without treating cross-builds as runtime proof.

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

## Phase 1 review gate — requested 2026-11-11

Added a mandatory independent agent review before closing the ongoing Phase 1
and every subsequent phase. A different model is optional; record the actual
reviewer/model, reviewed SHA, findings, fixes and final outcome. Blocking findings
must be resolved and affected tests rerun. Phase 1 review is pending; the phase
is not complete. Native runtime implementation and its CI are in progress.

## CLI argument library — requested 2026-11-11

Use Cobra/pflag for the agent CLI. Pinned stable `spf13/cobra v1.10.2`, verified
against its official release. Phase 1 migrates command/argument handling to Cobra,
adds contextual help and shell completion, and preserves redacted JSON errors.
Native verification of the migration is pending.

## Verified Phase 1 core evidence — 2026-11-11 (expanded phase remains open)

- [Native unit/build/CLI](https://github.com/peerimeter/peerimeter-agent/actions/runs/42948951660): all three passed at `4246c6bea040bfa41a0bd506f0df501310a20367`, exact Go 1.27.4, vet/race/unit, static config/key permissions, Cobra help/completion.
- [Actual agent integration](https://github.com/peerimeter/peerimeter-agent/actions/runs/42948600470): all three passed at `04648b92b7bdc3635fdf12b56d2729667130538a`. Real authenticated handshake and traffic, in-place peer changes, invalid reload preservation, graceful restart/cleanup, systemd/launchd/SCM lifecycle and protected config/key preservation. Linux also verifies SIGKILL recovery, UID-based delegated control and authorization even after socket permission widening; Windows verifies a real nonadmin operator and private-key read denial.
- [Packages/installers](https://github.com/peerimeter/peerimeter-agent/actions/runs/42948951604): all six OS/architecture artifacts built at `4246c6b`; native Linux/macOS/Windows installer checks passed. Actual Unix archive installation and installed binary version/toolchain verified; actual Windows ZIP includes signed pinned WireGuardNT with no separate GUI/MSI. Tests cover latest/pinned URL fixtures, offline archives, custom/spaced paths, invalid checksums, preservation and Windows rollback. No public release was published.
- `4246c6b` differs from `04648b9` only by the extra Unix actual-package test step; runtime sources are identical. Runner images: Ubuntu 24.04 x64 `20261101.360.1`; macOS 15 arm64 `20261104.0514.1`; Windows 2022 x64 `20261101.353.1` (recorded in evidence artifacts).
- Independent reviewer `/root/phase1_review` **passed** final code at `4246c6bea040bfa41a0bd506f0df501310a20367`. Parent model was inherited; concrete model identifier unavailable. No blocking findings remain. Four findings fixed and rechecked: failed install retry/reinstall, delegated nonadmin IPC, Cobra/service script arguments, and concurrent installer snapshot rollback. The latter has a barrier-controlled regression test that failed before the lock fix and passed afterward. Review is independent of platform implementation agents.
- Prior failures are preserved in agent progress.md: Linux alias initialization, Windows FILE_ADD_* constants, SYSTEM metadata cleanup ACLs, stale service script flags and Python Windows UTF-8 decoding; all resolved in the successful runs above.
- ✅ Phase 0 complete; 🔄 Phase 1 in progress; ⬜ Phase 2 not started. Historical “Phase 1 pending” entries above are superseded by this dated completion record.

## 🔄 Phase 1 scope extension — plan commit a022780

The current plan adds mandatory Unix config/state/runtime directory separation,
safe migration and a complete top-level uninstall with dry-run, installation
inventory and native removal tests. These were not part of the implementation
reviewed at 4246c6b and are not complete. The independent review caught the overly
broad checklist marking before it was committed. Phase 1 remains in progress;
all earlier core test passes remain valid evidence, not full-phase completion.

## Additional Phase 1 presentation requirements — 2026-11-11

🔄 Pending verification/implementation: centralized Linux prefix constant,
command-per-file Cobra organization, branded/color-aware version output with
`--json`, and wgshow-style peer rendering from the explicitly pinned MIT example
in additional-plan.md A10. All remain within the ongoing Phase 1 review gate.

## 🔄 Expanded Phase 1 closing review — 2026-11-11

Fixes were committed and pushed in agent `8f247070fb6c4ff571c0de58b8c85196f4d1d9da`.
[Native CI](https://github.com/peerimeter/peerimeter-agent/actions/runs/42998147614)
passed on Linux, macOS and Windows with exact Go 1.27.4 (format, vet, race/unit,
build and CLI smoke). This includes directory ownership, branded CLI and peer
presentation tests; full-phase completion still requires integration and review.

Independent reviewer `/root/phase1_review` reviewed that exact commit; inherited
parent model, concrete model identifier unavailable. Configuration ownership,
provisional migration tracking, runtime identity and Windows deletion locking
passed code review. One blocking finding remains: uninstall must detect a native
service pointing to its executable even when a pre-inventory upgrade has no
recorded service profile. The fix and regression are in progress.

[Integration](https://github.com/peerimeter/peerimeter-agent/actions/runs/42998147627)
and [packages/installers](https://github.com/peerimeter/peerimeter-agent/actions/runs/42998147753)
failed. Follow-up fixes address legacy migration planning before ownership claim,
Windows helper hashing without an unavailable PowerShell cmdlet, default macOS
resource-ancestor fixture permissions and an outdated Unix symlink test. Native
verification and a review of the final corrected commit are pending. Phase 1
remains 🔄 in progress; Phase 2 remains ⬜ not started.

## ✅ Phase 1 accepted — 2026-11-11

Final tested and independently reviewed implementation:
`091bd40d115c7371e1c0f4cd0f18fb8f3ab71ca2`. Subsequent completion records and
README status updates change documentation only.

| Gate | Linux amd64 | macOS arm64 | Windows amd64 |
| --- | --- | --- | --- |
| [Native Go 1.27.4 CI](https://github.com/peerimeter/peerimeter-agent/actions/runs/43002033268) — format, vet, race/unit, build and CLI | passed | passed | passed |
| [Actual WireGuard agent integration](https://github.com/peerimeter/peerimeter-agent/actions/runs/43002033188) | passed | passed | passed |
| [Native packages and installers](https://github.com/peerimeter/peerimeter-agent/actions/runs/43002033328) | passed | passed | passed |

All six Linux/macOS/Windows amd64/arm64 packages built successfully. Native
receipts report the exact final SHA and images Ubuntu 24.04 x64
`20261101.360.1`, macOS 15 arm64 `20261104.0514.1`, Windows 2022 x64
`20261101.353.1`.

Expanded acceptance includes default/custom/spaced config/state/runtime paths,
Unix reboot-style runtime recreation and live migration from actual `4246c6b`
legacy services, ownership-bound configuration/runtime claims, protected
operator access, read-only removal previews, live full removal, changed-file
refusal, retries/idempotency and unrelated/source credential preservation.
Windows additionally verifies signed bundled WireGuardNT, standard ProgramData
ACLs, custom ProgramData program paths and bounded executable self-removal.
All four agent READMEs have consistent navigation, headings, tables, code blocks
and verified local links; stale backend probe descriptions were corrected.
Cobra file organization, centralized prefix, branded version/color/JSON and
refactored public wgshow peer presentation with MIT attribution are complete.

Independent reviewer `/root/phase1_review` **passed code review and final phase
acceptance** at the exact SHA above after inspecting native receipts and Windows
logs. The reviewer inherited the parent model; a concrete model identifier is
unavailable. All blocking findings are resolved: cross-installation config/runtime
ownership, removal/startup exclusion and Windows lock deletion, unhealthy-state
shutdown, provisional migration tracking, untracked native service detection and
ambiguous registration refusal. Native follow-up also fixed asynchronous legacy
shutdown, explicit private test fixtures, PowerShell helper hashing and narrow
standard shared-root ACL handling. No blocking or deferred code findings remain.

Limits remain explicit: Linux tests two actual agents in isolated namespaces;
macOS/Windows test the native agent against an independent userspace WireGuard
peer. Mixed-OS VM links and NAT/network-disruption recovery are **not run**;
other packaged architectures are cross-build only. No public release was
published, so live latest-release downloads remain unverified (URL resolution is
fixture-tested with actual native packages). These limits belong to the later
network/resilience/release work and are not claims of verified connectivity.

✅ Phase 0 complete · ✅ Phase 1 complete · ⬜ Phase 2 not started.

## Phase 1 add-on — STUN command

Added a requirement for a standalone Cobra STUN command with selected or OS-assigned UDP port, observable mapped endpoint, cleanup and native tests on all three platforms. Implementation remains pending; this does not change the Phase 1 completion record for the preceding accepted scope.

