# Peerimeter — Implementation Progress

> Living log for Codex CLI. Read `plan.md` and `additional-plan.md` first. Update this file after each coherent implementation task and each session. Entries below record verified work and explicitly separate native backend feasibility from agent tunnel integration.

## Current status

- **Active work:** ✅ Phases 0–2 complete; 🔄 Phases 3–5 in progress.
- **Last verified milestone:** Agent PRs [#68](https://github.com/peerimeter/peerimeter-agent/pull/68), [#69](https://github.com/peerimeter/peerimeter-agent/pull/69), [#70](https://github.com/peerimeter/peerimeter-agent/pull/70), and [#71](https://github.com/peerimeter/peerimeter-agent/pull/71) merged on 2026-11-17. The agent `config` CLI command, including `pull/push`, is removed; managed `sync` refreshes this node's registration, fetches peer records, and applies them to the running agent config. PR #71 fixed the intermittent macOS full-uninstall failure caused by a runtime marker disappearing during extended-permission inspection. PR #71's native build/tests, WireGuard integration, packages, action-lock, and CodeRabbit checks passed on Linux, macOS and Windows (workflow run [44194689880](https://github.com/peerimeter/peerimeter-agent/actions/runs/44194689880)); the follow-up `main` run [44195585597](https://github.com/peerimeter/peerimeter-agent/actions/runs/44195585597) also passed every job, including macOS integration. Service main `b18eda3` and dashboard main `b2179eb` also pass their latest hosted CI. The dashboard PR #1 and service PR #10 remain merged.
- **Repositories:** `peerimeter-agent` (Phases 0–2, managed join and single-running-network sync); `peerimeter-service` (Phase 3 API and node registration); `peerimeter-dashboard` (Phase 5 API-key UI). Three independent Git repositories plus this shared `.github` documentation/configuration repository.
- **Next action:** Finish the PostgreSQL backend review and run its integration suite on a dedicated database; then continue concurrent multi-network service behavior, scoped/revocable credentials and durable enrollment. Phase 5 GitHub SSO and production-hosting documentation remain future work.

The phase numbering below supersedes historical entries: agent self-update is now
Phase 2, and the former control-plane through release-hardening phases move to
Phases 3–8. Older dated logs retain their original phase numbers for history.
The former agent-root `progress.md` is being consolidated into this shared
tracker; current agent documentation links here rather than maintaining a
duplicate progress file.

## Phase checklist

**Phase status:** ✅ Complete · 🔄 In progress · ⬜ Not started · ⛔ Blocked.
A passed test alone does not mean the phase is complete; Phase 1 and later also require the independent review gate.

| Phase | Status | Verification / notes |
|---|---|---|
| 0 — Cross-platform baseline + static JSON | ✅ Complete | Independent agent repo, strict config/key handling, CLI contracts/validation, testable lifecycle/interfaces, native CI and backend feasibility verified on all three; operational run/status/`wg show` belong to Phase 1 |
| 1 — Offline WireGuard on Linux/macOS/Windows | ✅ Complete | Native runtime, directory separation, migration, complete uninstall, CLI presentation, installers and independent review passed at `091bd40` |
| 2 — Agent self-update | ✅ Complete | Inventory-aware HTTPS/SHA256SUMS update, confirmation/`--yes`, rollback/recovery, progress reporting; native CI and review passed in agent PRs #37–#39 |
| 3 — Server + join bundles + sync | 🔄 In progress | Service API, join bundles, managed agent join and network/node peer exchange are implemented. Agent PRs #68–#70 make sync target the running configuration, refresh the node's public-key/underlay registration, then fetch and apply peers; `--network-id` defaults only when one joined network exists and is required when multiple are saved. The flag does not provide concurrent network runtimes: sync verifies the selected network matches the active agent. The agent `config` CLI command was removed in PR #69; sync does not upload peer config. The current `enrollment_token` is a deterministic HMAC-derived bearer credential scoped to one network; one-time, expiring and revocable device credentials remain future work. PostgreSQL dual-backend code, PostgreSQL 18.7 Alpine Compose service and updated deployment docs are in service PR [#16](https://github.com/peerimeter/peerimeter-service/pull/16); independent review of `2f5d64e` found no new findings. The latest Compose/docs commit is `7d49add`; its CI and CodeRabbit rerun is in progress. PostgreSQL integration tests are wired to CI's isolated database; a separate Supabase-hosted run remains pending. Concurrent network service management, durable agent credentials, background/service-driven reconciliation, MQTT and host-file DNS remain outstanding |
| 4 — NAT traversal + self-healing | 🔄 In progress | User-prioritized increment adds primary-interface underlay reporting and best-effort direct UDP discovery; no relay. Native LAN/NAT traversal and network-change recovery remain unverified |
| 5 — Browser dashboard + GitHub SSO | 🔄 In progress | Dashboard PR #1 provides API-key sign-in, network create/list, peer list and metadata editing, network-scoped join commands, `VITE_SERVICE_URL`, Playwright tests, and `test:ui`. GitHub SSO remains a later increment. |
| 6 — External WireGuard client gateway | ⬜ Not started | |
| 7 — Cross-platform resilience + packaging | ⬜ Not started | |
| 8 — Release and hardening | ⬜ Not started | |

## Repository checklist

- [x] `peerimeter-agent` initialized with independent `.git`, `go.mod`, and CI.
- [x] `peerimeter-service` initialized separately at Phase 3 with independent `.git`, `go.mod`, and CI workflow (initial API merged in PR #1).
- [x] `peerimeter-dashboard` initialized separately at Phase 5 with React/Vite, browser tests and CI; PR #1 is merged.
- [ ] Cross-repo versioned HTTP/JSON and MQTT contracts defined and tested without relative module imports.
- [ ] Record tested commit hash and status for each repository.

## GitHub Actions verification matrix (mandatory from Phase 0)

Status must be one of: **not run / passed / failed / blocked**. Do not change any cell to `passed` without native runner evidence. Record workflow run URLs, commit SHA, date, runner image, WireGuard backend and blockers in the log below.

| OS runner | Native build/unit/CLI (`pipeline.yml:test`) | Real agent traffic/service (`pipeline.yml:integration`) | Native package/install (`pipeline.yml:package`) | Mixed-OS/disruption (`connectivity-e2e.yml`) |
|---|---|---|---|---|
| Ubuntu 24.04 | passed (`4246c6b`) | passed — actual agent (`04648b9`) | passed (`4246c6b`) | blocked — no mixed-OS topology |
| macOS 15 | passed (`4246c6b`) | passed — actual agent (`04648b9`) | passed (`4246c6b`) | blocked — no mixed-OS topology |
| Windows 2022 | passed (`4246c6b`) | passed — actual agent (`04648b9`) | passed (`4246c6b`) | blocked — no mixed-OS topology |

- [x] `peerimeter-agent/.github/workflows/pipeline.yml` runs native build/unit/CLI tests, real agent integration and package/installer gates; its aggregate `check` job gates PR completion.
- [x] The pipeline's integration matrix runs real agent handshake/traffic, restart, cleanup, IPC/operator and native service lifecycle on all three OSes.
- [x] `peerimeter-agent/.github/workflows/connectivity-e2e.yml` exists as a manual entry point and reports a blocked result until a mixed-OS/NAT topology is provisioned; no connectivity pass is claimed.
- [x] macOS/Windows GitHub-hosted tunnel feasibility checked: both passed on the exact target images; no self-hosted fallback currently needed.
- [x] All integration skips and missing infrastructure are reported explicitly, never as a pass; agent E2E remains unavailable and manual E2E workflow fails with a blocked explanation.
- [x] `peerimeter-service` independent CI added at Phase 3; PR #1 `verify`, `action-lock` and aggregate `check` passed on hosted Ubuntu at `eb96f89` (run `43834468132`).

## Additional requirements checklist

- [ ] Phase 3: managed host-file DNS entries from peer names/network names using a configurable template and maintained Go hosts-file library; entries must map to the peer's WireGuard tunnel address(es), preserve unrelated content and reconcile lifecycle changes cross-platform.
- [ ] Phase 4: separately discover/exchange same-LAN agents' internal/private-underlay addresses and use reachable ones as peer endpoints; this is distinct from host-file DNS entries, which resolve names to WireGuard tunnel addresses.

- [x] P0 static JSON schema versioning, validation and protected local key references.
- [x] `peerimeter-agent run --config <path>` makes no API/MQTT/STUN calls.
- [x] Real WireGuard connectivity from static configs on Linux, macOS and Windows, each natively verified.
- [x] systemd, launchd and Windows Service lifecycle with local offline `status` and `wg show`.
- [ ] Static config remains usable after online enrollment is added.
- [ ] Phase 3 join bundle is Base64URL(unpadded JSON with `version`, `api_url`, `enrollment_token`).
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
| `peerimeter-agent` | `abe8ff1` (main; PR #71 merge; sync PR head `e8507bc`) | 🔄 Managed join, registration refresh and peer reconciliation | PR #71 CI run [44194689880](https://github.com/peerimeter/peerimeter-agent/actions/runs/44194689880) passed build/tests, integration and package jobs on Linux, macOS and Windows; action-lock and CodeRabbit passed. It fixed a macOS uninstall race where a runtime marker could disappear between metadata and extended-permission checks. Post-merge CI [44195585597](https://github.com/peerimeter/peerimeter-agent/actions/runs/44195585597) passed all jobs, including macOS integration. |
| `peerimeter-service` | `7d49add` (PR #16; based on main `b18eda3`) | 🔄 PostgreSQL storage and existing network API | Local Go 1.27.3 unit tests, vet, build, action-lock verification and diff checks pass. CI is rerunning the PostgreSQL integration test against its isolated PostgreSQL 18.7 Alpine service. Independent review found no new issue at `2f5d64e`; the Compose/docs commit is not yet independently reviewed. Auto-merge is enabled with rebase; checks and CodeRabbit are pending for `7d49add`. |
| `peerimeter-dashboard` | `b2179eb` (main) | 🔄 API-key dashboard UI | Latest hosted `check` run [44132896150](https://github.com/peerimeter/peerimeter-dashboard/actions/runs/44132896150) passed. |

## Completed work

- Created [peerimeter-agent](https://github.com/peerimeter/peerimeter-agent) as an independent repository. No service code/repository created ahead of Phase 3.
- Implemented versioned strict static JSON parsing/validation, two public templates, secure fresh example generation, private key handling, Linux naming/ownership helpers, platform interfaces and foreground cleanup lifecycle tests.
- Functional `run`, local `status`/`wg show`, safe `sync`/`stop`, protected operator IPC, and native service lifecycle are implemented and tested. Cobra owns CLI parsing/help/completion. The package workflow builds release archives and verifies installers.
- Go **1.27.4** is required in the plan, modules and native CI. Agent self-update is complete as Phase 2.
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
- Agent release [v0.17.0](https://github.com/peerimeter/peerimeter-agent/releases/tag/v0.17.0)
  was published on 2026-11-15 with Linux/macOS/Windows packages and SHA256SUMS.
  Latest/pinned URL resolution also uses explicit fixtures in native installer
  tests; real packaged binaries are installed and executed.
- One named native system service is currently supported per machine. Changing
  macOS/Windows interface addresses requires explicit stop/start. Self-update
  is complete; the service repository and Phase 3 control-plane work remain
  unstarted.

## Architectural decisions

- No relay and no desktop UI.
- Separate `peerimeter-agent`, `peerimeter-service` and `peerimeter-dashboard` Git repositories; no monorepo.
- Cross-platform static JSON WireGuard agent is P0; API is optional until Phase 3.
- Managed join bundle includes both HTTPS API URL and opaque enrollment token; Base64URL is not encryption.
- Linux interface naming is local to each enrolled agent/network; see `additional-plan.md`.

## Next steps

1. Complete dashboard PR #1 and service PR #10 checks and reviews, then update this tracker with the merged commits.
2. Continue managed enrollment and runtime reconciliation while retaining offline static WireGuard operation.
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
- Historical agent failures recorded in its former progress tracker included Linux alias initialization, Windows FILE_ADD_* constants, SYSTEM metadata cleanup ACLs, stale service script flags and Python Windows UTF-8 decoding; all were resolved in the successful runs above.
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


## 🔄 Phase 1 STUN add-on implementation

The agent now has a standalone `stun [local-udp-port]` command: explicit port or
OS-selected port 0, configurable STUN server, mapped endpoint JSON, response
validation, and socket cleanup. Diagnostic local and cross-platform builds pass;
review, native per-OS verification and live STUN behavior remain pending.


## Planned Phase 3 requirement — one agent, multiple networks (renumbered 2026-11-15)

Expanded the earlier A1 interface-naming requirement into explicit concurrent
multi-network management by one agent service. Phase 3 includes independent
network state/lifecycle, CLI selection/overview, conflict validation and two-network
native acceptance tests; details are in additional-plan.md A12. This is planned
work and does not claim that multi-network service management is implemented.

## Agent implementation review — 2026-11-14

Reviewed the current sibling `peerimeter-agent` tree at implementation commit
`0c258558099a3fe1258806d9819806b2c66023e6` (with later documentation/CI-name
commits at `360a42e`). The accepted Phase 1 core remains complete; managed
enrollment, MQTT sync and multi-network commands remain Phase 3 work.

Since the original Phase 1 acceptance, the CLI gained a live `status --watch`
view with an interval option and JSON-lines snapshots, responsive terminal
tables, interactive STUN progress/cancellation, refined `wg` / `wg show` output,
color/output controls, and installer progress reporting. The current CLI has no
standalone `peers`, `join`, or `networks` command. STUN is implemented as a
mapping probe, but native checks and live behavior for that add-on remain pending.

The agent CI has consolidated native unit/build checks, actual agent integration,
packages/installers and an aggregate PR `check` into `pipeline.yml`. The separate
`connectivity-e2e.yml` is a manual blocked placeholder until mixed-OS/NAT test
infrastructure is provisioned. The sibling progress log reports local checks
for the later CLI changes, but no fresh native pipeline run was available for
that reviewed snapshot; the native pass cells above refer only to their recorded
historical SHAs and must not be read as verification of `0c25855`.


## Self-update implementation — 2026-11-15

The user moved agent self-update forward to Phase 2. The feature is complete and
merged in [agent PR #37](https://github.com/peerimeter/peerimeter-agent/pull/37),
[PR #38](https://github.com/peerimeter/peerimeter-agent/pull/38) and
[PR #39](https://github.com/peerimeter/peerimeter-agent/pull/39). The engine
uses the installation inventory to select program files, verifies release
packages against SHA256SUMS over HTTPS, journals replacement and can restore the
previous installation after failed readiness or interruption. It preserves
installation data and profile directories and leaves the independently managed
Windows WireGuardNT DLL unchanged. SHA256SUMS shares the HTTPS release trust
channel; it is not a detached signature.

PR #37 merged at `7e3cb8b80587a123097b272c1c81989097a5841e` with native updater
integration. PR #38 merged at `2b4fac8891bf4320f24a8b79a75eab24bd4ea0de` with
interactive confirmation, `--yes`, the `self-update` command name and its
release-check E2E endpoint fix. Its native run
[`43735889325`](https://github.com/peerimeter/peerimeter-agent/actions/runs/43735889325)
passed build/tests, installed-binary E2E, WireGuard integration and all six
package targets on Linux, macOS and Windows. CodeRabbit reported no actionable
comments after the endpoint fix.

PR #39 merged at `8a6fb0d151e728c3ce9abace0413371271d4005e`, adding progress
reporting for release lookup, package download/verification, replacement,
service readiness and rollback. Its native run
[`43745890445`](https://github.com/peerimeter/peerimeter-agent/actions/runs/43745890445)
passed all platform build/test and WireGuard integration jobs, all six package
targets, the action-lock gate and aggregate `check`; CodeRabbit reported no
actionable comments. On a TTY the command animates the current operation on
stderr; with redirected output it prints step start/completion lines. JSON
results remain on stdout.

Local Go 1.27.3 ARM64 validation with `CGO_ENABLED=0` passed the updater
integration tests, `go test ./...`, `go vet ./...` and a production build. The
installed-binary E2E test runs in native CI with its opt-in environment enabled;
it skips by default in a local `go test ./...` invocation. These results do not
provide mixed-OS/NAT connectivity evidence or constitute a public release; those
remain outside the completed self-update scope.

## Phase 3 service start — 2026-11-16

### PostgreSQL storage increment — 2026-11-18 (in progress)

The service work branch `feat/postgresql-storage` adds a selectable `file` or
`postgres` backend. PostgreSQL uses embedded Goose SQL migrations and GORM with
JSONB for network metadata/config, node configs and registrations. Network
creation serializes CIDR-overlap checks, and registration locks its network row
while allocating an address. `PEERIMETER_IMPORT_FILE` imports the complete
legacy JSON/YAML file store into an empty database in one transaction; the
source files remain untouched. The file backend remains the default.

Local checks with Go 1.27.3: `go test ./...`, `go vet ./...`, `go build ./...`
and `git diff --check` passed. `gh actions-lock --verify` passed. The local
toolchain is Go 1.27.3, below the required Go 1.27.4 pin. Local race testing
could not run because the environment has no GCC. Service CI now runs the
PostgreSQL integration test against a dedicated PostgreSQL 17.6 container
pinned by image digest.

Independent Codex review agent `/root/postgres_review` reviewed commit
`39a2f830af3f7696816199f6053a8f499252b66f`; it found two issues in an earlier
working diff (import/write race and test cleanup ordering), verified both fixes
at this SHA, and reported no remaining finding. The exact model identifier was
not exposed by the review tooling. PR [#16](https://github.com/peerimeter/peerimeter-service/pull/16)
then received four CodeRabbit findings, all fixed in commit
`2f5d64e1e91297b6cfb4757c65a42c951abb361c`. The independent reviewer checked
that exact SHA and found no new issue. The changes validate the import source
before PostgreSQL startup, assert snapshot deep-copy behavior, use Goose's
PostgreSQL session migration lock, and silence GORM SQL logging. PR #16 is open
with rebase auto-merge enabled; latest-head CI and CodeRabbit are running.

The shared plan/progress PR [#38](https://github.com/peerimeter/.github/pull/38)
merged, then CodeRabbit identified two stale registration-storage statements.
The corrected wording is in follow-up PR
[#39](https://github.com/peerimeter/.github/pull/39), which merged on 2026-11-18. A subsequent documentation audit corrected the workspace guidance pointer and refreshed this storage status in PR #40.

Supabase CLI is authenticated and sees only the existing `netguard-prod` and
`netguard-staging` projects; neither is used for tests. The requested
`peerimeter-test` project creation is awaiting the user's choice on the possible
Micro compute charge. A separate Supabase-hosted integration run remains
unverified; this increment does not complete Phase 3.

### User-prioritized automatic discovery increment — started 2026-11-16

The requested scope combines managed node registration/peer exchange from
Phase 3 with direct UDP hole punching and primary-interface-only address
reporting from Phase 4. Relay/TURN remains out of scope. Current work branches
are `feat/agent-discovery` in `peerimeter-service` and `peerimeter-agent`, plus
`docs/agent-discovery-phase` here. The first service slice stores node documents
as independent mode-0600 YAML files and adds a versioned peer-list route; the
agent service API client can decode that peer-list response. Service unit tests
and vet pass with `CGO_ENABLED=0`; the agent service API package tests pass.

The user specified that the WireGuard address pool comes from a service network
resource created through the API. The service now has network `POST`/`GET` and
list routes for private IPv4 CIDRs, a node-registration route that allocates an
unused tunnel address atomically, and peer document listing. The agent model
accepts optional `network.cidr`; its API client can read network metadata and
register a node. A standard-library helper selects the interface carrying the
default route and chooses a peer LAN address only when the reported CIDRs
overlap. RFC1918 CIDR overlap alone does not prove that two nodes share a
reachable LAN; when an overlapping but unreachable LAN candidate is selected,
the agent currently has no automatic public-endpoint fallback.

The agent now has a foreground `join` command that generates/stores a protected
key, reports the selected underlay and an initial STUN mapping, registers the
node, builds its config from the service-allocated address and peer list, then
polls/reconciles peers every 15 seconds. It uses direct WireGuard handshakes
and keepalive only; there is no TURN/relay. Full agent and service unit checks
pass with `CGO_ENABLED=0`; Windows amd64 and macOS arm64 agent cross-builds
pass. Local Go is 1.27.3, while CI pins 1.27.4. Native tunnel/NAT integration
could not run in this Termux environment because it has no `sudo` or root
network privileges.

An independent diff review found and the implementation fixed unsafe path IDs
in node-file persistence and incomplete peer-list response validation. The
follow-up review confirmed those fixes; a direct YAML dependency issue was
corrected with `go mod tidy`. Regression tests cover invalid IDs, legacy
node-file migration, and malformed peer documents. The reviewed object was the
uncommitted working diff, so no reviewed commit SHA is recorded.

An independent review of service and agent code at the committed heads
`79593f91c0739a339beea8215eb96b57d25defac` and
`df9d8bba9aed113312a772e572715bc45e700f52` found no blocking issue. It confirmed
the config/registration storage separation, serialized non-overlapping CIDR
allocation, join key handling and best-effort direct-only traversal claims.
CodeRabbit on agent PR #51 then identified five additional correctness issues:
the refresh worker could race shutdown; a selected endpoint could conflict
with tunnel routes; generated config was written before complete validation;
CIDRs could extend beyond an RFC1918 block; and route-source matching used
subnet containment instead of the exact source address. These are fixed in
agent commit `444a95201b75ec99de8c6f946a98e096eec91b80`, with regression tests;
the current hosted checks and review are pending.
Remaining limitations are the service-wide reusable API token (no per-device
enrollment or revocation), stale registration cleanup (no `leave` API), and
public STUN mappings captured only at join time (no same-socket remapping while
WireGuard owns the port). The join-time STUN probe runs before WireGuard starts,
so it does not verify the mapping for the WireGuard socket; the advertised
mapping may be unusable. Same-socket validation in plan section 8 remains open.
A VPN default route may cause safe discovery failure.
Native LAN/NAT traversal and network-change recovery are still unverified, so
the overall Phase 3/4 milestone remains in progress and is not marked complete.

Local final gates passed: service `go mod tidy -diff`,
`CGO_ENABLED=0 go test ./...`, `go vet ./...`, `go build ./...`; the service
hosted `go test -race ./...` caught a nondeterministic map iteration assumption
in a test, fixed in the follow-up commit below and passed locally without race
instrumentation; agent
`CGO_ENABLED=0 make lint`, `make test`, `make build`, Windows amd64 and macOS
arm64 cross-builds after the five review fixes; and `git diff --check` in all three repositories. The
Termux host has Go 1.27.3; CI is pinned to 1.27.4. The service discovery PR
checks pass; agent hosted checks and CodeRabbit review for the corrected head
are running. The docs PR review requested the Phase 4 status and discovery
limitation clarifications; those were fixed and merged in docs PR #20.

Implementation commits: service
[`79593f9`](https://github.com/peerimeter/peerimeter-service/commit/79593f91c0739a339beea8215eb96b57d25defac),
agent code/review fixes
[`444a952`](https://github.com/peerimeter/peerimeter-agent/commit/444a95201b75ec99de8c6f946a98e096eec91b80),
agent documentation
[`290f56a`](https://github.com/peerimeter/peerimeter-agent/commit/290f56a3b0388597ce88f1bc021d6ca09a6f46ed),
and the initial tracker changes merged in
[`256faae`](https://github.com/peerimeter/.github/commit/256faae).

Created the independent public [`peerimeter-service`](https://github.com/peerimeter/peerimeter-service)
repository. Initial implementation commit `686f3f35aebce5f0eb3a5ab9b4ba024daeb81d2b`,
updated with the toolchain pin at `662c4c442a4b43b9889fbfe0e6850ebef9df6182`,
review fixes at `440c4b563451e825f91368b5c97eb98ee4b66ca2`, and config-ID
validation at `eb96f89fca362f3e48ee1f700af705593bb1c3b9`, merged in
[PR #1](https://github.com/peerimeter/peerimeter-service/pull/1) at
`0b68ed62453c92f6638e22c1c83855a345891d82`.
It adds a chi v5 API with versioned network/node JSON `GET`/`PUT`, a Base64URL
v1 join bundle, Bearer API-key authentication, serialized in-memory writes to
an atomically replaced mode-0600 JSON file, structured logs and graceful HTTP
shutdown. The static API key is intentionally the join bundle's
`enrollment_token`; it is reusable, global, and has no per-agent scope or expiry.
TLS must terminate at a trusted proxy. This does not satisfy one-time scoped
enrollment from the long-term plan.

PR #1's Dockerfile uses a multi-stage non-root image. Follow-up
[PR #2](https://github.com/peerimeter/peerimeter-service/pull/2) switches its
runtime stage to `scratch`, copies CA certificates and minimal passwd/group
files for `app` UID/GID 1000, and preserves the writable config directory. The
project owner confirmed this service image has not been deployed and no data
volume needs migration. PR #2 also adds main-branch SemVer releases from
Conventional Commits: it calculates against the exact event SHA, publishes
multi-platform `vX.Y.Z` and `X.Y.Z` GHCR tags only for an explicit major/minor/
patch bump, and recovers a missing GitHub Release without republishing a tag.
The first main run seeds `v0.0.0` at the repository's root commit. The workflow
uses `ietf-tools/semver-action@v1.12.0`, recorded with its immutable SHA in the
generated Actions lock. The first main run
[`43844562558`](https://github.com/peerimeter/peerimeter-service/actions/runs/43844562558)
passed all jobs, calculated `v0.1.0`, and pushed both image tags for amd64 and
arm64. Buildx reported manifest digest
`sha256:67890b5898c542cc6bd1cfc6315c5da798c0f339ec0c65a9ce62df4b72b293a2`.
The [GitHub Release](https://github.com/peerimeter/peerimeter-service/releases/tag/v0.1.0)
was created for the same main commit.

The service module declares `go 1.27`, while `.go-version` pins the exact
development/CI/container toolchain to Go 1.27.4. Local validation of PR #1 at
`eb96f89` used Go 1.27.3 on Linux ARM64 with `CGO_ENABLED=0`: tests, vet, build,
action-lock verification and `git diff --check` passed. Local race tests and
Docker builds were unavailable there; hosted PR run
[`43834468132`](https://github.com/peerimeter/peerimeter-service/actions/runs/43834468132)
passed Go 1.27.4 format, vet, race/tests, binary build, Docker build,
action-lock verification and aggregate `check`. CodeRabbit's Buildx and shutdown findings were fixed at
`440c4b5`. A later review found that empty node IDs could overwrite network
config and asked to document exact version tags; both were fixed at `eb96f89`,
along with disabling checkout credential persistence. PR #2 merged at
`b2fba5e0972938bdfa4a27a03c4b00c05240ad0a`; its required checks and Docker build
passed, and CodeRabbit withdrew the UID finding after the owner confirmed there
is no existing deployment or data volume. Service PR #3 records the repository
settings policy in its local AGENTS.md and merged at
`f82c3f2308fd467a4ef950da87c92e8d869c9453`. The service `main` branch matches
the agent's settings: strict
`check` and CodeRabbit requirements, resolved review conversations, linear
history, and no force-push or deletion. Repository auto-merge is enabled. PR #1
merged after its current CodeRabbit review passed. Phase 3 remains in progress;
durable agent enrollment/runtime sync, scoped credentials, PostgreSQL/MQTT,
host-file DNS and independent phase review remain outstanding.

Agent [PR #49](https://github.com/peerimeter/peerimeter-agent/pull/49), current
head `01f7959`, added Base64URL join-token validation and one-shot network/node
JSON config pull/push. At that point the join bundle's `enrollment_token` was
the service's shared static API key and granted access to all stored network/node configs.
Tokens are read from owner-protected files or can be passed directly with a
CLI-history/process-list warning; the agent does not persist them. PR #49 leaves
offline static WireGuard operation intact. It does not yet enroll devices,
apply the downloaded objects, reconcile managed runtime, or isolate credentials
per agent. CodeRabbit found that output temp files inherited broad Windows
directory ACLs. The fix creates them with a protected DACL and verifies the
open file before writing; its regression test runs natively on Windows. At the
current head Linux/Windows build tests, Linux/Windows WireGuard integration,
Linux/Windows packages and action-lock have passed. macOS jobs and CodeRabbit
review are pending. Local full tests, lint, build, and Windows/macOS
cross-builds passed with Go 1.27.3 and `CGO_ENABLED=0`; hosted CI pins 1.27.4.

Shared guidance and tracker changes are in
[`.github` PR #13](https://github.com/peerimeter/.github/pull/13), separate from
the service repository. PR #13 records the `go 1.27` module directive, exact
`.go-version` pin, and required generated Action lock/CI verification workflow.
PR #13 passed its required CodeRabbit status and auto-merged at
`76cde408f29631516ed16a64a9dfe1ea188181a9`; its review's earlier CI-pending
finding was corrected before the merge. Service PR #1 merged at
`0b68ed62453c92f6638e22c1c83855a345891d82`; PR #2 added the scratch runtime
and SemVer GHCR release workflow. Agent PR #49 contains the first service
config API client increment. The previously merged `feat/static-config-api`
service branch was deleted after confirming no open PR used it. Shared repo
settings guidance is in [`.github` PR #17](https://github.com/peerimeter/.github/pull/17)
and the local service AGENTS.md policy merged in PR #3. Auto-merge is enabled
at repo and PR level.

## Planned managed multi-network Join and connection controls — 2026-11-17

The project owner clarified the expected Phase 3 service-mode behavior. When
the installed agent service is running, `join` must register the new network
with that service through authenticated local IPC; the service then persists
the profile and automatically brings up that network's tunnel without starting
a competing agent process or interrupting other networks. A newly joined
network defaults to connected. Persist each network's requested connection
state so an explicitly disconnected network stays down after service restart.

Add network-scoped `connect --network <network-id>` and
`disconnect --network <network-id>` commands. Disconnect stops only the
selected tunnel while retaining membership, credentials, keys and desired
configuration; `connect` resumes it. `leave` remains distinct and removes the
managed profile, with server-side revocation dependent on API support. The
static `run --config` path remains single-profile and offline. These requirements
are captured in `additional-plan.md` A12 and `plan.md`'s managed CLI contract;
they are planned and are not current capabilities.

Agent PR [#64](https://github.com/peerimeter/peerimeter-agent/pull/64) is open
for the narrower UX change that makes foreground tunnel startup opt-in via
`join --run`. That PR does not implement joining through the installed service
or multi-network lifecycle control. Its required CI and CodeRabbit review were
still running at the time of this tracker update; auto-merge is enabled under
branch protection.
