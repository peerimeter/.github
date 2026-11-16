# Peerimeter — Implementation Progress

> Living log for Codex CLI. Read `plan.md` and `additional-plan.md` first. Update this file after each coherent implementation task and each session. Entries below record verified work and explicitly separate native backend feasibility from agent tunnel integration.

## Current status

- **Active work:** ✅ Phase 0 complete; ✅ Phase 1 complete; ✅ Phase 2 agent self-update complete; 🔄 Phase 3 service/control plane in progress.
- **Last verified milestone:** Agent self-update passed native build/test, installed-binary E2E, WireGuard integration, packaging and action-lock checks on Linux, macOS and Windows. Phase 3 service PR #1's Go 1.27.4 test/build/Docker/lock CI passed at the updated head; automated re-review of fixes is pending.
- **Repositories:** `peerimeter-agent` (Phases 0–2, complete); `peerimeter-service` (Phase 3 onward, in progress). Two independent Git repositories.
- **Next action:** Complete review and hosted checks for service PR #1, then integrate the agent's managed join and config sync in small increments.

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
| 3 — Server + join bundles + sync | 🔄 In progress | Initial API/config storage in service PR #1; join token currently carries the shared static key. Agent integration, scoped enrollment, PostgreSQL/MQTT and host-file DNS remain outstanding |
| 4 — NAT traversal + self-healing | ⬜ Not started | Direct connections only; no relay |
| 5 — Browser dashboard + GitHub SSO | ⬜ Not started | |
| 6 — External WireGuard client gateway | ⬜ Not started | |
| 7 — Cross-platform resilience + packaging | ⬜ Not started | |
| 8 — Release and hardening | ⬜ Not started | |

## Two-repository checklist

- [x] `peerimeter-agent` initialized with independent `.git`, `go.mod`, and CI.
- [x] `peerimeter-service` initialized separately at Phase 3 with independent `.git`, `go.mod`, and CI workflow (PR #1 hosted results pending).
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
| `peerimeter-agent` | `8a6fb0d` (progress UX merged); `1f7a1fc` (confirmation and native updater CI); `091bd40` (accepted Phase 1) | ✅ Phase 2 complete | Native build/tests, installed-binary E2E, WireGuard integration, all package targets and CodeRabbit review passed |
| `peerimeter-service` | `eb96f89fca362f3e48ee1f700af705593bb1c3b9` (PR #1) | 🔄 Phase 3 initial API/storage | Local Go 1.27.3 ARM64 `CGO_ENABLED=0` tests/vet/build passed; hosted Go 1.27.4 race/tests/vet/build/Docker/action-lock passed at run `43834468132`; CodeRabbit re-review pending |

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
- Separate `peerimeter-agent` and `peerimeter-service` Git repositories; no monorepo.
- Cross-platform static JSON WireGuard agent is P0; API is optional until Phase 3.
- Managed join bundle includes both HTTPS API URL and opaque enrollment token; Base64URL is not encryption.
- Linux interface naming is local to each enrolled agent/network; see `additional-plan.md`.

## Next steps

1. Verify the published v0.17.0 installation path when requested.
2. Begin Phase 3 control-plane/API/enrollment work only when requested; retain the offline agent contract.
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

Created the independent public [`peerimeter-service`](https://github.com/peerimeter/peerimeter-service)
repository. Initial implementation commit `686f3f35aebce5f0eb3a5ab9b4ba024daeb81d2b`,
updated with the toolchain pin at `662c4c442a4b43b9889fbfe0e6850ebef9df6182`,
review fixes at `440c4b563451e825f91368b5c97eb98ee4b66ca2`, and config-ID
validation at `eb96f89fca362f3e48ee1f700af705593bb1c3b9`, is under
[PR #1](https://github.com/peerimeter/peerimeter-service/pull/1).
It adds a chi v5 API with versioned network/node JSON `GET`/`PUT`, a Base64URL
v1 join bundle, Bearer API-key authentication, serialized in-memory writes to
an atomically replaced mode-0600 JSON file, structured logs and graceful HTTP
shutdown. The static API key is intentionally the join bundle's
`enrollment_token`; it is reusable, global, and has no per-agent scope or expiry.
TLS must terminate at a trusted proxy. This does not satisfy one-time scoped
enrollment from the long-term plan.

The service Dockerfile is multi-stage and non-root. Its version-tag workflow
tests/builds the service and image, sets up a Buildx builder for multi-platform
pushes, verifies `.github/workflows/actions.lock`,
and publishes `linux/amd64` and `linux/arm64` images to GHCR after valid
`vX.Y.Z` tags. Current Actions use explicit release tags whose immutable SHAs
are recorded in the generated lock. The service module declares `go 1.27`,
while `.go-version` pins the exact development/CI/container toolchain to Go
1.27.4.

Local validation at the current commit used Go 1.27.3 on Linux ARM64 with
`CGO_ENABLED=0`: `go test -count=1 ./...`, `go vet ./...`, `go build`,
`gh actions-lock --verify`, and `git diff --check` passed. The local environment
has no C compiler or Docker engine, so race tests and a container build have not
run locally. Invoking the local Go 1.27.4 toolchain segfaults before reporting
its version; hosted CI uses `.go-version` to install and test with 1.27.4.
Hosted PR run [`43834468132`](https://github.com/peerimeter/peerimeter-service/actions/runs/43834468132)
passed: its Ubuntu runner installed Go 1.27.4 from `.go-version`, then passed
format, vet, race/tests, binary build, Docker build, action-lock verification
and aggregate `check`. CodeRabbit's Buildx and shutdown findings were fixed at
`440c4b5`. A later review found that empty node IDs could overwrite network
config and asked to document exact version tags; both were fixed at `eb96f89`,
along with disabling checkout credential persistence. Review of this head is
pending. The service `main` branch now mirrors the agent's protections: strict
`check` and CodeRabbit requirements, resolved review conversations, linear
history, and no force-push or deletion. Repository auto-merge is enabled and
PR #1 is waiting on its current CodeRabbit status. Phase 3 remains in progress;
agent enrollment/sync, scoped credentials, PostgreSQL/MQTT, host-file DNS and
independent phase review remain outstanding.

Shared guidance and this tracker update are in
[`.github` PR #13](https://github.com/peerimeter/.github/pull/13), separate from
service PR #1. That PR records the `go 1.27` module directive, exact
`.go-version` pin, and required generated Action lock/CI verification workflow.
PR #13 passed its required CodeRabbit status and auto-merged at
`76cde408f29631516ed16a64a9dfe1ea188181a9`; its review's earlier CI-pending
finding was corrected before the merge. Service PR #1 remains open with all
hosted CI gates passing. The service `main` protections were applied through the
GitHub API and match the agent branch rules. Auto-merge is enabled at repo and
PR level; PR #1 is waiting for its current CodeRabbit check.
