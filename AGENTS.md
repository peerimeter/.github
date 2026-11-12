# Repository guidance

This is the independent `peerimeter/.github` repository. It contains the Peerimeter organization profile, project website, assets, and shared planning documents. The sibling `../peerimeter-agent` directory is a separate Git repository; the workspace parent is not a repository. The planned `peerimeter-service` repository is not present yet.

## Where to look

- `README.md`: public project overview and current status.
- `docs/index.html` and `docs/CNAME`: project site and domain.
- `assets/`: site and profile artwork.
- `docs/planning/plan.md`: product scope, architecture, phases, and acceptance criteria.
- `docs/planning/additional-plan.md`: later clarifications and requirements.
- `docs/planning/progress.md`: overall progress and native verification evidence.
- `docs/planning/codex-prompt.md`: historical implementation handoff; check current code and status before using it.

## Working agreements

- Keep planning claims aligned with the agent's `README.md`, `progress.md`, implementation, and actual CI evidence. Distinguish implemented features, planned features, and unverified network scenarios.
- Preserve the two-repository separation. Make changes and commits in the affected repository only; do not treat sibling changes as one atomic commit.
- Peerimeter uses direct WireGuard connections without a relay. Do not claim STUN alone guarantees NAT traversal.
- Never commit private keys, tokens, generated client configurations, or production secrets.
- Commits and pushes to a feature/PR branch are allowed. Use Conventional Commits messages, such as `docs(site): clarify installation` or `fix(plan): correct phase status`. Open a PR for review; do not push work directly to `main` unless explicitly requested.
- For site or documentation changes, inspect the rendered content and check that links, status claims, and references to the other repository are accurate. Record the actual checks performed.
