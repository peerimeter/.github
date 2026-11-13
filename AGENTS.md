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
- Follow the shared [repository conventions](docs/repository-conventions.md) for secrets, commits, branches, pull requests, checks and completion claims.
- For every pull request you create or update, inspect all automated review feedback, including CodeRabbit findings. Verify each finding against the current code, fix valid issues, and recheck the PR after pushing.
- Track each pull request through its lifecycle until it is merged or explicitly closed. Recheck CI, automated reviews, merge conflicts, and auto-merge status after updates; resolve blockers when possible and report the current state. An open PR is ongoing work, not a completed task.
- For site or documentation changes, inspect the rendered content and check that links, status claims, and references to the other repository are accurate. Record the actual checks performed.
