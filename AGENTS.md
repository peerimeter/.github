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
- Auto-merge is the default for every PR. After opening a PR, verify repository settings require status checks, all review conversations to be resolved, and **Require branches to be up to date before merging**; then enable auto-merge immediately and let branch protection gate the merge. If any of those protections are absent, leave auto-merge off and report the configuration gap.
- Inspect automated review feedback on every current PR head, including CodeRabbit findings. Verify each finding against the current change, fix valid issues, and resolve every review conversation before the PR can merge. Recheck feedback after each push; required checks, conversation resolution, and strict branch freshness must pass before auto-merge completes.
- Track each pull request through its lifecycle until it is merged or explicitly closed. Recheck CI, automated reviews, merge conflicts, and auto-merge status after updates; resolve blockers when possible and report the current state. An open PR is ongoing work, not a completed task.
- Before feature work, update local `main` from `origin/main` and branch from it. After a PR merges, fast-forward local `main` and leave the repository checked out there.
- For site or documentation changes, inspect the rendered content and check that links, status claims, and references to the other repository are accurate. Record the actual checks performed.
