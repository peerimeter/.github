# Peerimeter — Additional Implementation Requirements

> Read **both `plan.md` and this file** before implementing any phase. This file supplements `plan.md`; when this file provides a more specific requirement, follow it. Preserve all original non-goals: **no relay, no desktop UI**. Track actual implementation status in `docs/planning/progress.md` in the `.github` repository.

## A1. Configurable Linux WireGuard interface name

### Objective

For a Peerimeter network, the default **Linux** WireGuard interface name must derive from the network name as `pm-$networkName`. An operator must be able to override the local interface name during agent enrollment/initialization. This is a **device-local setting**, not a setting that renames the network or changes interfaces on other peers.

Examples (subject to Linux interface naming limits):

- Peerimeter network `production` -> Linux interface `pm-production`.
- Peerimeter network `dev` -> Linux interface `pm-dev`.
- An agent enrolled with `--interface wg-prod` -> that machine uses `wg-prod` for the network, regardless of other agents' interface names.

### Linux naming constraints — mandatory

Linux network interface names must be **at most 15 bytes** (not characters) and must not be empty or contain forbidden characters (notably `/`, `:`, whitespace); use a conservative ASCII-safe set for generated names. Never blindly concatenate an arbitrary user-supplied network name to `pm-` and pass it to the OS.

Keep the `pm-` build-time default in a single named constant (`ifname.DefaultPrefix`); calculate available slug bytes from its length. Existing persisted names remain authoritative.

Implement a single, deterministic function to derive the default name:

1. Use `pm-` + an ASCII-safe normalized version of the Peerimeter network display name.
2. When the full result would exceed 15 bytes, truncate the human-readable part and append a short **stable hash of the immutable network ID**, maintaining the 15-byte limit. This avoids collisions among long network names sharing a prefix.
3. If normalization yields no useful name, use a stable ID-derived fallback (e.g. `pm-<short-id>`).
4. Detect conflicts with an existing OS interface or with another locally managed Peerimeter network. Do not take over an unrelated interface; return a clear actionable error.
5. Validate explicitly supplied interface names against Linux's length and naming rules; reject invalid names with an actionable error rather than silently truncating or changing them.
6. Test names with hyphens, spaces, Unicode, punctuation, long common prefixes, duplicates, and empty/invalid input.

**Note:** `pm-$networkName` is the conceptual default. For names that do not fit Linux constraints, use the deterministic safe form above. Document the exact algorithm so names remain stable across restarts.

### Agent CLI and persistence

Support a Linux-specific override at enrollment:

```bash
# Default: derived from the server's network name
peerimeter-agent join --token <BASE64URL_JSON_BUNDLE>

# Explicit local interface name
peerimeter-agent join --token <BASE64URL_JSON_BUNDLE> --interface wg-prod
```

Implementation requirements:

- In **static mode**, resolve the interface name from the validated local JSON `network.id` and `network.name`. In **managed mode**, resolve it from the authoritative server-provided network ID and name; do not trust an arbitrary client-provided network label in online enrollment.
- Persist the **resolved local interface name** alongside the enrolled network's identity in the agent's protected local state. A restart or server config sync must not silently rename a functioning interface.
- Precedence during initial enrollment: explicit `--interface` > generated default. After enrollment, the persisted local selection wins over subsequent network display-name changes.
- The interface name must **not** be included in globally shared peer WireGuard configuration as a requirement for other peers; each Linux agent may choose its own local name.
- All WireGuard configuration, addresses, routes, health checks, status output, service startup, and cleanup must use the locally resolved interface name instead of hard-coded values.
- Support joining multiple Peerimeter networks on one machine without local interface naming collisions. If the CLI's enrollment token maps to more than one network, require an explicit network selection or otherwise avoid ambiguous interface creation.
- Avoid implicit renaming of an existing interface. A post-enrollment rename (if supported) must be a separate deliberate, safe operation with rollback; it is **not required** for the MVP.
- The flag is Linux-specific. On macOS/Windows, either reject it with a clear message or document platform-specific behavior; do not claim the requested Linux interface name maps directly to an `utun` device or Windows adapter.
- Do not print or persist enrollment secrets in plaintext logs.

Recommended agent configuration representation (illustrative):

```yaml
networks:
  <network-id>:
    interface_name: wg-prod
    interface_name_source: explicit # or generated
```

Use whichever protected local configuration format is already selected by `plan.md`; avoid adding YAML solely for this requirement.

### CLI UX

`peerimeter-agent status` and `peerimeter-agent wg show` should clearly identify the network and the actual local interface, for example:

```text
Network: production (network-id)
Interface: wg-prod
Status: connected
```

### Acceptance criteria

- [ ] On Linux, `production` defaults to `pm-production` when the name is valid and fits.
- [ ] `--interface wg-prod` creates/manages `wg-prod` instead of `pm-production`.
- [ ] Names exceeding 15 bytes are deterministically shortened with an ID-based suffix, remain unique for tested inputs, and stay stable across restarts.
- [ ] Invalid explicit names are rejected before any networking changes.
- [ ] Locally persisted interface choices survive agent/service restart and network display-name changes.
- [ ] Two locally enrolled networks cannot accidentally manage the same interface.
- [ ] All interface lifecycle and route operations honor the resolved local name.
- [ ] Unit tests pass (`go test ./...`); privileged Linux integration tests verify actual interface creation/cleanup with default and custom names.
- [ ] README documents defaults, overrides, OS-specific behavior, and limitations.

## A2. Codex progress tracking (`progress.md`)

Codex must maintain the shared tracker at `docs/planning/progress.md` in the `.github` repository. If it already exists, preserve and update it; **do not overwrite previous work logs**. Do not create a duplicate tracker in `peerimeter-agent`.

Minimum structure:

1. Project status and current active phase.
2. Checklist for every phase from `plan.md`, initially marked not started unless supported by actual repository evidence.
3. Completed tasks with short implementation notes and relevant file paths.
4. Tests executed with commands and actual results; do not claim success unless executed.
5. Known bugs, limitations, blockers, and next actions.
6. Important architectural decisions, including the interface naming algorithm and its persistence format.

Update `docs/planning/progress.md` after each coherent task and at the end of every Codex session. Keep the phase checklist in `plan.md` consistent if that file also contains progress markers, but treat the shared tracker as the primary running log. Do not erase historical test failures; record when they were resolved.

## A3. Development ordering

Implement interface-name resolution and persistence early, during the first Linux agent/WireGuard integration phase. It must be used by the real Linux networking adapter, not added afterward as a CLI-only flag with no effect.

Do not start relay, DERP, TURN, desktop apps, or mobile apps.


## A4. TOP PRIORITY — Offline, static-JSON cross-platform agent BEFORE the API

This section **overrides any Linux-first or server-first scheduling in `plan.md`**. The first functional milestone must support **Linux, macOS and Windows** using only locally supplied static JSON, with no central API server, MQTT broker, GitHub SSO or STUN server required. This is a fixed requirement, not an optional later optimization. No relay.

### Offline operating mode and CLI

```bash
peerimeter-agent validate-config --config ./peerimeter.static.json
peerimeter-agent run --config ./peerimeter.static.json
peerimeter-agent service install --config /absolute/path/peerimeter.static.json
peerimeter-agent service start
peerimeter-agent status
peerimeter-agent wg show
# Linux only: local override at initialization (also persist for future service use)
peerimeter-agent run --config ./peerimeter.static.json --interface wg-prod
```

- The agent must run continuously and manage a **real WireGuard tunnel** without communicating with any Peerimeter API.
- Static configs supply the exact remote peers and reachable endpoint addresses; offline mode cannot discover new public endpoints or distribute peer changes magically. WireGuard-native authenticated endpoint roaming may still work. Automatic cross-NAT rediscovery comes later, with the API/STUN phase.
- Do not make `join` a prerequisite to `run --config`; static and managed modes share the same WireGuard reconciliation engine but have different config providers.
- Support safe restarts and service installation for systemd (Linux), launchd (macOS), and Windows Service. Native OS adapter integration is mandatory, not simulated by Linux-specific commands.
- Explicitly document required privileges, WireGuard backend/install prerequisites and route/adapter behavior for each OS. Windows/macOS adapters must be implemented and tested on their actual OS; cross-compilation is only an early check.
- The CLI should show usable offline `status` and `wg show` views, using local daemon IPC or local WireGuard state, with no server dependencies.

### Static JSON schema (illustrative, versioned)

```json
{
  "version": 1,
  "mode": "static",
  "network": {"id": "network-home", "name": "home"},
  "interface": {
    "private_key_file": "/secure/path/to/wg.key",
    "address": ["10.80.0.10/32"],
    "listen_port": 51820,
    "name": "wg-prod"
  },
  "peers": [
    {
      "public_key": "<base64-WireGuard-public-key>",
      "allowed_ips": ["10.80.0.11/32"],
      "endpoint": "203.0.113.20:51820",
      "persistent_keepalive_seconds": 25
    }
  ]
}
```

- `interface.name` is **optional** and, if provided, is a Linux-only override; otherwise Linux generates the default `pm-$networkName` with the safe 15-byte algorithm from A1. `--interface` overrides `interface.name` at initial setup; a persisted choice wins on later restarts. Windows/macOS must not pretend an arbitrary Linux interface name is an OS adapter identifier; reject unsupported explicit names clearly.
- Treat the example address and public key placeholder as illustrative, **not a runnable credential**. Provide two valid generated/configurable fixtures and document creating keys on each platform.
- Use `private_key_file` rather than embedding a private key in general config; protect key files, never print secrets, and reject insecure or missing key-file permissions where the OS provides that check.
- Validate duplicates, malformed keys, overlapping/conflicting local addresses, endpoints, ports, network IDs, invalid CIDRs and routes **before** changing OS state. On failure, preserve the previously functional tunnel where possible.
- No outbound API, MQTT or STUN calls in static mode; outbound WireGuard UDP to explicit peer endpoints is expected.
- Static config may be intentionally updated by an operator and reapplied; it is not overwritten by server config.

### Cross-platform acceptance gates

- [x] `go test ./...` and build succeed on Linux, macOS and Windows CI runners.
- [x] Real WireGuard adapter can create tunnel, set IP, add peer, configure route and clean up on each OS.
- [x] At least two agents connect using only static JSON, confirmed with actual WireGuard handshake and traffic, no API server. Verified with two isolated Linux agents; macOS/Windows native agent links use an independent userspace WireGuard reference peer. Mixed-OS agent pairs remain explicitly unverified.
- [x] Continuous daemon/service startup, shutdown, restart and status work on all three OSes.
- [x] Linux generated/custom interface names and collisions are covered by tests.
- [x] Cross-OS scenarios (Linux <-> Windows, Linux <-> macOS, Windows <-> macOS, where available) are tested or explicitly left unverified rather than marked complete.
- [x] No use of server enrollment, MQTT or STUN in the static agent execution path.

## A5. Later managed enrollment: one Base64URL(JSON) join token, including API URL

When Phase 3 introduces server enrollment, **the user must not have to specify `--server` separately**. The server (and later browser dashboard) generates a portable join token that packages an absolute HTTPS API URL and an opaque, short-lived/one-time enrollment secret.

**Token wire format:** `base64url_no_padding(utf8(JSON))`, with strict schema validation and a version discriminator.

```json
{
  "version": 1,
  "api_url": "https://peerimeter.example.com",
  "enrollment_token": "<opaque-random-one-time-secret>"
}
```

```bash
peerimeter-agent join --token <BASE64URL_JSON_BUNDLE>
peerimeter-agent join --token-file /secure/path/join-token.txt
# Linux optional local name override:
peerimeter-agent join --token <BASE64URL_JSON_BUNDLE> --interface wg-prod
```

Implementation rules:

1. Decode Base64URL safely, enforce size limits, reject malformed JSON/unknown version, missing fields and extraneous/unsafe URL parts. Validate URL with `net/url`: HTTPS required by default; no embedded username/password, fragment, query, or unexpected schemes. Development-only HTTP must require a deliberate explicit opt-in; never silently downgrade.
2. **Base64 is encoding, not encryption or authentication.** The inner enrollment token is a secret. It must be cryptographically random, time-limited and redeemed at most once (atomic server-side consumption). Apply rate limiting, expiry and network scoping.
3. Never log, persist in progress files, echo in diagnostics, or expose the full bundle in API errors. Prefer `--token-file` or stdin support for automation because CLI arguments can appear in process listings and shell history. Explicitly document this risk for `--token`.
4. The server is authoritative for device/network membership and issues device credentials independently of WireGuard key material. Prefer locally generated keys and upload only public keys.
5. After successful redemption, persist the validated API URL, device credentials and network ID securely; discard the one-time bundle/secret. Keep startup/update behavior independent of the original enrollment token.
6. `join --token` automatically determines the control-plane address from the decoded bundle. Avoid requiring or silently accepting a separate `--server` in normal enrollment. Add test vectors for Base64URL decode, URL validation, expired/reused tokens and negative cases.
7. Clearly separate two meanings of token: the **encoded join bundle** (transport container) vs the **opaque enrollment credential** inside it. Neither is a WireGuard key.

Acceptance criteria:
- [ ] Server API/dashboard can issue a one-time enrollment credential and export the join bundle.
- [ ] `peerimeter-agent join --token <bundle>` needs no server argument and enrolls successfully.
- [ ] Wrong/expired/reused token is rejected and does not install a partially authorized network.
- [ ] Static `run --config` still works without connectivity to the server.
- [ ] Secret redaction, URL validation and CLI credential handling have tests.

## A6. Revised execution order and Codex instructions

1. **Phase 0:** cross-platform project baseline, versioned static JSON model, functional offline config validation, CLI command/argument contracts, and testable lifecycle/adapter interfaces. Native build/unit/CLI validation and early backend feasibility checks belong here. Runtime-dependent CLI commands must explicitly report unavailable until Phase 1; operational tunnels and local status/`wg show` are not Phase 0 completion gates.
2. **Phase 1:** actual Linux, macOS and Windows static WireGuard agents and services (API-free), including operational `run --config`, local `status` and `wg show`, authenticated local IPC or real local WireGuard state, with native verification. This implements the runtime behind the Phase 0 CLI contracts.
3. **Phase 2:** inventory-aware agent self-update, including confirmation, SHA256SUMS verification, rollback/recovery and native platform tests.
4. **Phase 3:** central API, PostgreSQL, join bundles (`api_url` + `enrollment_token`), MQTT and online config provider, reusing static mode's reconciler.
5. **Phase 4:** STUN, direct NAT traversal, dynamic IP recovery/self-healing.
6. **Phase 5:** browser dashboard and GitHub SSO.
7. **Phase 6:** external WireGuard gateway/client support.
8. **Phase 7:** resilience, additional native integration, packages.
9. **Phase 8:** hardening and release.

**Codex must read `plan.md`, `additional-plan.md` and `docs/planning/progress.md` before coding; preserve and update the shared tracker after each coherent implementation task.** Reconcile any conflicting old wording (e.g. “Linux first”, Phase 6 cross-platform) in favor of A4–A6. Do not start server/dashboard work until Phase 1 cross-platform static-mode requirements have been implemented to the extent test infrastructure permits, with remaining native verification clearly recorded as blockers.


## A7. Mandatory repository separation (new requirement)

- Use **three distinct code repositories**, named `peerimeter-agent`, `peerimeter-service`, and `peerimeter-dashboard`; no shared monorepo or shared Go module.
- `peerimeter-agent` contains the Go CLI, always-on service, static JSON mode, real WireGuard lifecycle for Linux/macOS/Windows, networking/routing, STUN and reconnect. The agent must compile and run without the service repository or any API.
- `peerimeter-service` contains the central Go REST API, configuration storage, PostgreSQL, MQTT coordination, future GitHub SSO API support and management of external-client gateways. No browser frontend, desktop application, or relay.
- `peerimeter-dashboard` contains the browser-only React/TypeScript UI. Its initial sign-in uses the existing service API key; it can be served separately with service CORS restricted to exact trusted origins or behind a same-origin reverse proxy. No desktop app and no relay.
- **Build and verify the offline agent in `peerimeter-agent` before starting the implementation in `peerimeter-service`.**
- Implement the API-key dashboard after the service exposes the required network, peer and join-bundle endpoints.
- Use versioned, documented HTTP/JSON and MQTT contracts at the repository boundary; the service is the schema owner, while the agent pins compatible contracts/releases. Avoid long-term relative Go module replacements, importing another repository's `internal` packages, or duplicating business logic across repositories.
- Keep builds, CI, testing, commits, tags and releases independent; record progress for **each code repo** in `docs/planning/progress.md`.
- A local workspace may contain the code repositories and shared planning repository; the workspace parent is not a Git repository.
- `pm-` is the Linux default interface prefix everywhere. Use this prefix consistently in documentation, defaults, examples, and tests.

## A8. Mandatory native cross-platform GitHub Actions tests (P0/P1)

GitHub Actions is required **from Phase 0**, within `peerimeter-agent` (not in a monorepo). The current agent implementation consolidates routine validation, native integration and release packaging in `pipeline.yml`; keep mixed-topology E2E separate:

1. `.github/workflows/pipeline.yml`: on pull requests, pushes to `main` and manual dispatch, run native `test` and privileged `integration` jobs on `ubuntu-24.04`, `macos-15`, and `windows-2022` with `fail-fast: false`; also build packages and run native installer tests. The aggregate `check` job must fail if any PR gate is unsuccessful. Successful cross-compilation alone does not pass the native gate.
2. `.github/workflows/connectivity-e2e.yml`: manual E2E entry point for Linux network-namespace NAT/IP-change simulation and reachable cross-OS WireGuard connectivity. Until the topology exists, report a visible blocked result; do not claim this placeholder as a connectivity pass. Jobs on separate OS runner VMs are not automatically in one LAN; arrange a reachable test peer or test network. No application relay.

**Acceptance evidence per OS:** runner/image, backend prerequisites, tested commit SHA, native interface creation, successful authenticated handshake, bidirectional tunnel traffic, agent restart/teardown, test log, and status. Test-status vocabulary: `passed`, `failed`, `blocked`, or `not run`. Never silently skip required checks, and never report `passed` for an unverified OS. If GitHub-hosted macOS or Windows cannot run a native WG tunnel, record the limitation and use a self-hosted runner; don't loosen the acceptance criteria.

Secure CI: use temporary WireGuard keys, least-privilege GitHub token permissions, bounded timeouts and teardown; keep secrets out of logs/artifacts. Do not expose a privileged private test network to untrusted pull requests. `peerimeter-service` later receives its own independent API/web GitHub Actions workflows.

**Update `docs/planning/progress.md` after each CI run and milestone with the per-platform matrix and links to results; keep Phase 0 focused on the offline agent rather than building an API to make CI pass.**

## A9. Mandatory phase-end independent review

Before closing the ongoing Phase 1 or any later phase, run a separate review
agent against the phase contract, final implementation and actual test evidence.
The reviewer must not be the author of the reviewed changes. It may use a different
model if available; report the actual reviewer/model and reviewed code SHA.
Resolve blocking findings, rerun affected tests and obtain follow-up verification.
Record findings, fixes, evidence, deferred non-blocking items and the final review
outcome in `docs/planning/progress.md`. Independent review supplements native tests and never
substitutes for real tunnel/traffic verification. A phase with open correctness,
security or acceptance blockers remains in progress.

## A10. Phase 1: standalone STUN command

Add a Cobra `stun` command based on the requested [netguard-client example](https://github.com/zer0go/netguard-client/blob/develop/cmd/stun.go), refactored to Peerimeter conventions. Accept an optional UDP port: use that exact available port when supplied, otherwise bind port 0 and report the OS-assigned port. Return the observed mapped address/port and the local port in structured output; reject invalid ports and close sockets on every exit path. Verify both modes natively on all three OSes. STUN mapping is diagnostic groundwork and must not be presented as successful peer connectivity or automatic NAT recovery.

## A11. Phase 1: refactored wgshow-style peer listing

Use [the requested wgshow example](https://github.com/zer0go/wgctrl-go/blob/47ad6fc7d7a95296fe893b49c1d4584b4eb6ac40/cmd/wgshow/main.go)
as the reference for human-readable WireGuard device/peer presentation. The
reviewed file revision is `47ad6fc7d7a95296fe893b49c1d4584b4eb6ac40` (MIT-licensed).
Refactor the relevant formatting into testable agent code; do not embed the
example main/flag/log.Fatal structure or introduce a second CLI parser.

- CLI implementation update (reviewed 2026-11-14): `wg show` provides the
  traditional wgshow-style human-readable peer listing, and bare `wg` defaults
  to the same view. Both accept `--json` and return the stable public peer
  array. `status` renders the agent/interface state and peer summary; there is
  no separate `peers` CLI command. Keep the Cobra command in its own source file
  and separate state acquisition, formatting, color policy and I/O.
- Read actual authorized local agent/backend snapshots on Linux, macOS and
  Windows. Do not bypass daemon ownership by opening a competing WireGuard
  manager or assume direct wgctrl access works identically on every platform.
- Display the selected network/interface, public peer identity, endpoint,
  AllowedIPs, latest handshake age, received/sent bytes and persistent keepalive.
  Private/preshared keys must never enter the renderer or diagnostic output.
- Refactor unsafe assumptions: absent endpoints, never-handshaken/future-clock
  timestamps, zero/disabled keepalive, empty peer lists, large byte counters and
  singular/plural durations must render clearly without panic or fabricated
  connectivity. Honor NO_COLOR, redirected output and explicit color controls.
- Preserve applicable copyright/license attribution when adapting code. Add
  focused renderer tests, plain/color/JSON CLI tests and native integration
  assertions against real backend state. Automation must opt into `wg --json`
  or `wg show --json` when parsing peer data. Completion requires implementation,
  native verification and independent review; see `docs/planning/progress.md` for evidence.


## A12. Phase 3 — Multiple networks in one agent service

A1 already requires joining multiple networks without interface-name collisions.
Expand that into a complete managed-mode requirement: **one installed agent service
must manage multiple networks concurrently**, on Linux, macOS and Windows. This
is planned work; today's static CLI remains scoped to a selected runtime/profile.

The agent's current sync increment (PRs #68–#70) works against the one
currently running network. It selects the only saved joined network by default;
when multiple joined-network credentials are saved, `--network-id` is
required. It then verifies that the selected ID matches the running agent,
refreshes that node's registration, fetches the peer list and applies it to
the active configuration. This does not provide concurrent multi-network
service operation.

- When the installed service is running, `join` must enroll and register the new
  network with that service over authenticated local IPC. The service must
  persist the network as a separate managed profile and automatically bring up
  its tunnel, without starting a second agent process or interrupting existing
  networks. A successful CLI `join` in service mode therefore means the
  service has accepted the new profile and is managing its connection; it must
  not run the tunnel in the CLI process.
- Provide `connect --network <network-id>` and
  `disconnect --network <network-id>` to bring only the selected network's
  tunnel up or down. `disconnect` is a temporary connection-state change: it
  preserves enrollment, keys, credentials and desired configuration, and the
  network remains listed as managed. `connect` resumes that network. Neither
  command may stop or restart other networks or the whole service. `leave`
  remains the separate operation for removing local managed state and, when
  supported by the API, revoking server-side membership.
- Persist the requested connected/disconnected state across service restarts.
  Newly joined networks default to connected; a previously disconnected
  network must remain disconnected after reboot until explicitly connected.

- Persist separate protected network identities, WireGuard keys, enrollment/device
  credentials, desired configurations and revisions. Define device versus network
  credential scope explicitly; never accidentally reuse one network's credentials.
- Give each network independent interface ownership, listen-port allocation,
  reconciliation, API/MQTT subscriptions, cached offline operation and failure state.
- Extend `sync --network-id <network-id>` to the concurrent service runtime:
  refresh and publish only the selected node's registration and endpoint
  candidates, then fetch and apply that network's authoritative peer state.
  The current one-runtime implementation already performs this sequence for
  its active network, selecting the only saved network automatically and
  requiring `--network-id` when more than one is saved. Sync must not upload
  peer configuration. The agent's `config` CLI command was removed;
  service-side configuration API endpoints remain a separate API surface.
- Expose `peerimeter-agent networks` as an overview of every managed network. In
  managed mode, `status`, `wg show` and `diagnose` accept
  `--network <network-id>` and require it when the target is ambiguous. `sync`
  uses its implemented `--network-id <network-id>` flag.
  `connect --network <network-id>`, `disconnect --network <network-id>`,
  `reconnect <peer-id> --network <network-id>` and
  `leave --network <network-id>` always require an explicit selection.
  `disconnect` stops only the selected tunnel while retaining its managed
  profile; `leave` removes that profile. `service stop` remains service-wide.
  Ambiguous mutations must fail clearly.
- Keep `wg` as an alias for `wg show`; `wg [show] --json` returns the stable
  public peer data. Static
  `run --config` remains profile-scoped and independent of managed network
  selection.
- Joining, connecting, disconnecting, updating, leaving or recovering one network must not restart,
  delete or overwrite another network's interface, routes, credentials or DNS entries.
- Acceptance: at least two networks managed simultaneously by one service on every
  supported OS; verify join to a running service automatically connects the new
  network, independent traffic, connect/disconnect persistence, leave, update,
  restart, offline cache, one-network failure isolation, conflict rejection and
  complete owned-resource cleanup.
- Validate interface names, listen ports, overlapping routes/AllowedIPs and generated
  hostnames across networks before applying changes; reject unsupported conflicts.
- Distinguish network-scoped disconnect/leave from service-wide stop and full
  uninstall. Uninstall must enumerate all owned networks and preserve unrelated
  resources.
- Acceptance: at least two networks managed simultaneously by one service on every
  supported OS; verify independent traffic, join/leave, update, restart, offline cache,
  one-network failure isolation, conflict rejection and complete owned-resource cleanup.

## A13. Designated breaking minor-release compatibility cleanup

Prepare a dedicated breaking minor release that removes migration and
backward-compatibility code retained only for older Peerimeter releases. Peerimeter
is still in `0.y.z`, so a breaking change increments the minor version (for example,
`v0.14.0` to `v0.15.0`), rather than moving to `v1.0.0`. The GitHub release
description must prominently state **Breaking changes included** and explain the
removed compatibility paths and required operator action. Keep the cleanup in its
designated breaking minor release. The Windows `pma.exe`
inventory and self-update compatibility marked for the next breaking release in
`peerimeter-agent` are current examples.

The `v0.39.0` agent release already removed older installation inventories and
markers, self-update transaction filenames, the `pma.exe` alias, and local
`network.json` state. This follow-up cleanup targets `peerimeter-agent v0.40.0`
from `v0.39.0` and `peerimeter-service v0.15.0` from `v0.14.1`. The agent also
retires the combined Unix service layout migration. Upgrade by stopping the
agent, backing up operator configuration and managed credentials/state,
uninstalling with `v0.39.0`, installing `v0.40.0`, then restoring only
current-format state with restrictive permissions. The service retires
file-store imports into PostgreSQL and old inline node/registration sidecar
readers. Keep the file backend for existing file data, or export and import it
manually before changing backends. PostgreSQL's forward schema migrations remain
supported. No obsolete compatibility fallback was found in the dashboard during
this inventory.

- Before implementation, inventory obsolete compatibility readers, one-time data
  migrations, legacy filenames/formats, fallback behavior and release-cleanup markers
  across `peerimeter-agent`, `peerimeter-service` and `peerimeter-dashboard`. Record
  each item and the release/version range it supports.
- Confirm the target minor version and the end of support for the older formats or
  releases. Remove only code whose compatibility window has ended; retain current
  formats, current APIs and required fresh-install behavior.
- Keep canonical forward database/schema migrations needed to create or upgrade a
  supported installation. Remove only obsolete compatibility/import paths after
  their input formats are explicitly retired. Preserve existing user data.
- Make separate reviewable PRs in each owning repository, with CI covering clean
  installs, current upgrades, and the removal of obsolete fallback paths. Update
  user and operator documentation to state the new minimum supported version and
  any required upgrade path.
- Acceptance: every inventoried item has a resolution, no in-scope obsolete
  compatibility shim remains, supported upgrade paths pass CI, and independent
  review confirms no active data or API migration was removed accidentally.
