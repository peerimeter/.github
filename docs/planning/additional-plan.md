# Peerimeter — Additional Implementation Requirements

> Read **both `plan.md` and this file** before implementing any phase. This file supplements `plan.md`; when this file provides a more specific requirement, follow it. Preserve all original non-goals: **no relay, no desktop UI**. Track actual implementation status in `progress.md`.

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

`peerimeter-agent status` and `peerimeter-agent peers` should clearly identify the network and the actual local interface, for example:

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

Codex must create `progress.md` in the repository root at the start of the first implementation session if it does not exist. If it already exists, preserve and update it; **do not overwrite previous work logs**.

Minimum structure:

1. Project status and current active phase.
2. Checklist for every phase from `plan.md`, initially marked not started unless supported by actual repository evidence.
3. Completed tasks with short implementation notes and relevant file paths.
4. Tests executed with commands and actual results; do not claim success unless executed.
5. Known bugs, limitations, blockers, and next actions.
6. Important architectural decisions, including the interface naming algorithm and its persistence format.

Update `progress.md` after each coherent task and at the end of every Codex session. Keep the phase checklist in `plan.md` consistent if that file also contains progress markers, but treat `progress.md` as the primary running log. Do not erase historical test failures; record when they were resolved.

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
peerimeter-agent peers
# Linux only: local override at initialization (also persist for future service use)
peerimeter-agent run --config ./peerimeter.static.json --interface wg-prod
```

- The agent must run continuously and manage a **real WireGuard tunnel** without communicating with any Peerimeter API.
- Static configs supply the exact remote peers and reachable endpoint addresses; offline mode cannot discover new public endpoints or distribute peer changes magically. WireGuard-native authenticated endpoint roaming may still work. Automatic cross-NAT rediscovery comes later, with the API/STUN phase.
- Do not make `join` a prerequisite to `run --config`; static and managed modes share the same WireGuard reconciliation engine but have different config providers.
- Support safe restarts and service installation for systemd (Linux), launchd (macOS), and Windows Service. Native OS adapter integration is mandatory, not simulated by Linux-specific commands.
- Explicitly document required privileges, WireGuard backend/install prerequisites and route/adapter behavior for each OS. Windows/macOS adapters must be implemented and tested on their actual OS; cross-compilation is only an early check.
- The CLI should show usable offline `status`/`peers`, using local daemon IPC or local WireGuard state, with no server dependencies.

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

When Phase 2 introduces server enrollment, **the user must not have to specify `--server` separately**. The server (and later browser dashboard) generates a portable join token that packages an absolute HTTPS API URL and an opaque, short-lived/one-time enrollment secret.

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

1. **Phase 0:** cross-platform project baseline, versioned static JSON model, functional offline config validation, CLI command/argument contracts, and testable lifecycle/adapter interfaces. Native build/unit/CLI validation and early backend feasibility checks belong here. Runtime-dependent CLI commands must explicitly report unavailable until Phase 1; operational tunnels and local status/peers are not Phase 0 completion gates.
2. **Phase 1:** actual Linux, macOS and Windows static WireGuard agents and services (API-free), including operational `run --config`, local `status` and `peers`, authenticated local IPC or real local WireGuard state, with native verification. This implements the runtime behind the Phase 0 CLI contracts.
3. **Phase 2:** central API, PostgreSQL, join bundles (`api_url` + `enrollment_token`), MQTT and online config provider, reusing static mode's reconciler.
4. **Phase 3:** STUN, direct NAT traversal, dynamic IP recovery/self-healing.
5. **Phase 4:** browser dashboard and GitHub SSO.
6. **Phase 5:** external WireGuard gateway/client support.
7. **Phase 6:** resilience, additional native integration, packages.
8. **Phase 7:** hardening and release.

**Codex must read `plan.md`, `additional-plan.md` and `progress.md` before coding; create `progress.md` if missing and update it after each coherent implementation task.** Reconcile any conflicting old wording (e.g. “Linux first”, Phase 6 cross-platform) in favor of A4–A6. Do not start server/dashboard work until Phase 1 cross-platform static-mode requirements have been implemented to the extent test infrastructure permits, with remaining native verification clearly recorded as blockers.


## A7. Mandatory repository separation (new requirement)

- Use **two distinct Git repositories**, named exactly `peerimeter-agent` and `peerimeter-service`; no shared monorepo or shared Go module.
- `peerimeter-agent` contains the Go CLI, always-on service, static JSON mode, real WireGuard lifecycle for Linux/macOS/Windows, networking/routing, STUN and reconnect. The agent must compile and run without the service repository or any API.
- `peerimeter-service` contains the central REST API, PostgreSQL, MQTT coordination, browser dashboard, GitHub SSO and management of external-client gateways. No desktop application, no relay.
- **Build and verify the offline agent in `peerimeter-agent` before starting the implementation in `peerimeter-service`.**
- Use versioned, documented HTTP/JSON and MQTT contracts at the repository boundary; the service is the schema owner, while the agent pins compatible contracts/releases. Avoid long-term relative Go module replacements, importing another repository's `internal` packages, or duplicating business logic across repositories.
- Keep builds, CI, testing, commits, tags and releases independent; record progress for **each repo** in `progress.md`.
- A local workspace may contain both sibling directories and the planning Markdown files; it is *not* a third Git repository.
- `pm-` is the Linux default interface prefix everywhere. Use this prefix consistently in documentation, defaults, examples, and tests.

## A8. Mandatory native cross-platform GitHub Actions tests (P0/P1)

GitHub Actions is required **from Phase 0**, within `peerimeter-agent` (not in a monorepo). Implement three independent workflows:

1. `.github/workflows/ci.yml`: on push and pull request, native jobs on `ubuntu-24.04`, `macos-15`, `windows-2022` with `fail-fast: false`; `go test ./...`, `go build`, static JSON validation and CLI smoke tests. Successful cross-compilation alone does not pass the gate.
2. `.github/workflows/wireguard-integration.yml`: on pull request and `workflow_dispatch` (optionally scheduled); privileged platform-specific tests that create an actual WireGuard interface, configure two peers, verify a real handshake and bidirectional traffic, and prove cleanup and restart. Prefer GitHub-hosted runners **only where the WireGuard backend really works**; probe early. Fall back to secured self-hosted macOS/Windows/Linux runners when required. Keep integration tests separately tagged (e.g. `-tags=integration`).
3. `.github/workflows/connectivity-e2e.yml`: manual/scheduled tests for Linux network namespace NAT/IP-change simulation, and reachable cross-OS WireGuard test connectivity. Jobs on separate OS runner VMs are not automatically in one LAN; arrange a reachable test peer or test network. No application relay.

**Acceptance evidence per OS:** runner/image, backend prerequisites, tested commit SHA, native interface creation, successful authenticated handshake, bidirectional tunnel traffic, agent restart/teardown, test log, and status. Test-status vocabulary: `passed`, `failed`, `blocked`, or `not run`. Never silently skip required checks, and never report `passed` for an unverified OS. If GitHub-hosted macOS or Windows cannot run a native WG tunnel, record the limitation and use a self-hosted runner; don't loosen the acceptance criteria.

Secure CI: use temporary WireGuard keys, least-privilege GitHub token permissions, bounded timeouts and teardown; keep secrets out of logs/artifacts. Do not expose a privileged private test network to untrusted pull requests. `peerimeter-service` later receives its own independent API/web GitHub Actions workflows.

**Update `progress.md` after each CI run and milestone with the per-platform matrix and links to results; keep Phase 0 focused on the offline agent rather than building an API to make CI pass.**

## A9. Mandatory phase-end independent review

Before closing the ongoing Phase 1 or any later phase, run a separate review
agent against the phase contract, final implementation and actual test evidence.
The reviewer must not be the author of the reviewed changes. It may use a different
model if available; report the actual reviewer/model and reviewed code SHA.
Resolve blocking findings, rerun affected tests and obtain follow-up verification.
Record findings, fixes, evidence, deferred non-blocking items and the final review
outcome in progress.md. Independent review supplements native tests and never
substitutes for real tunnel/traffic verification. A phase with open correctness,
security or acceptance blockers remains in progress.

## A10. Phase 1: refactored wgshow-style peer listing

Use [the requested wgshow example](https://github.com/zer0go/wgctrl-go/blob/47ad6fc7d7a95296fe893b49c1d4584b4eb6ac40/cmd/wgshow/main.go)
as the reference for human-readable WireGuard device/peer presentation. The
reviewed file revision is `47ad6fc7d7a95296fe893b49c1d4584b4eb6ac40` (MIT-licensed).
Refactor the relevant formatting into testable agent code; do not embed the
example main/flag/log.Fatal structure or introduce a second CLI parser.

- `peers` should provide wgshow-style human-readable output, with `--json` for
  stable machine-readable data. Keep the Cobra command in its own source file
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
  assertions against real backend state. Existing automation must opt into
  `peers --json` when parsing peer data. This task is pending until implemented
  and reviewed; adding the requirement is not evidence of completion.
