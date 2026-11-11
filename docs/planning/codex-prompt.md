# Peerimeter — Codex CLI starting prompt

Copy the following into Codex CLI from the **workspace root containing both sibling repositories** (or create them there):

```text
Read the following files completely before starting:
- plan.md
- additional-plan.md
- progress.md (create it if missing)

Implement Peerimeter according to these documents. Create exactly TWO independent Git repositories: peerimeter-agent and peerimeter-service. Each repository needs its own go.mod, Git history, CI and releases; the workspace itself is not a monorepo. Start in peerimeter-agent and do not create service code until Phase 2. Where a detailed requirement in additional-plan.md supersedes older scheduling in plan.md, follow additional-plan.md.

HIGHEST PRIORITY: Build a working cross-platform WireGuard agent for Linux, macOS and Windows that operates entirely offline using a local, versioned static JSON configuration. This must work without an API server, MQTT broker, STUN server, dashboard, or GitHub SSO.

Start with Phase 0 and Phase 1:
1. Implement a headless Go agent and CLI on all three platforms.
2. Support `peerimeter-agent run --config peerimeter.static.json`.
3. Manage real WireGuard interfaces and peers, and run continuously as an OS service.
4. On Linux, default the interface name to `pm-$networkName` subject to Linux naming limits and collision handling, with an agent initialization/config override. Follow additional-plan.md for full rules.
5. Validate configs strictly and add unit and integration tests. From Phase 0 create GitHub Actions workflows in peerimeter-agent: ci.yml (native build/unit/static JSON on ubuntu-24.04, macos-15 and windows-2022), wireguard-integration.yml (real privileged interface/handshake/bidirectional traffic), and connectivity-e2e.yml (Linux namespaces, IP/NAT changes and later mixed-OS tests).
6. Verify actual native runtime connectivity on every platform. GitHub-hosted sudo/admin may not suffice for macOS utun or Windows drivers; probe this early and use secured self-hosted runners if necessary. A native integration pass requires real WG handshake and bidirectional tunnel packets, restart and cleanup; cross-compilation alone is not proof. Explicitly record failed/blocked/not-run checks, never silently skip them.
7. Before closing Phase 1 or any later phase, run an independent review agent (a different model is optional), resolve blocking findings, rerun affected checks and record reviewer/model, reviewed SHA and final outcome in progress.md. Keep each repository independently buildable after each coherent task, run tests, fix regressions, and update progress.md with completed work, decisions, test commands/results, blockers, and next steps. Populate the per-OS CI verification matrix with commit SHA, runner, logs and status; do not claim OS support is verified without native tunnel evidence.

CROSS-REPO CONTRACT: The service owns versioned API/JSON/MQTT schemas; the agent consumes pinned compatible schemas. Do not couple the agent to the service source tree or use relative Go module replacements. Record per-repo commits and test outcomes in progress.md.

FOR LATER PHASES: Implement managed enrollment with `peerimeter-agent join --token <BUNDLE>`, where BUNDLE is unpadded Base64URL-encoded versioned JSON containing both `api_url` and `enrollment_token`. The agent must obtain the server address from the bundle without a separate --server argument. Treat the enrollment token as a secret; Base64 is not encryption.

DO NOT implement relay/DERP/TURN or desktop applications. The UI is a browser-based dashboard only, in a later phase.

Do not begin the API server, database, MQTT, STUN, or dashboard implementation before the cross-platform offline-static milestone has been implemented as far as available native test environments allow. Document any missing native verification in progress.md rather than claiming completion.

Start implementing Phase 0 immediately, then proceed into Phase 1. Do not stop at scaffolding.
```

## Subsequent session prompt

```text
Read plan.md, additional-plan.md, and progress.md from the workspace root. Inspect both independent Git repositories, continue the next incomplete task in priority order, execute relevant tests and CI verification on Linux/macOS/Windows, and update the per-OS matrix in progress.md with verified results and remaining blockers. Do not implement relay or desktop UI.
```
