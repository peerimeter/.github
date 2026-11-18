# Repository guidance

This is the independent `peerimeter/.github` repository. It contains the Peerimeter organization profile, project website, assets, and shared planning documents. The sibling `../peerimeter-agent`, `../peerimeter-service`, and `../peerimeter-dashboard` directories are separate Git repositories; the workspace parent is not a repository.

## Where to look

- `README.md`: public project overview and current status.
- `docs/index.html` and `docs/CNAME`: project site and domain.
- `assets/`: site and profile artwork.
- `docs/planning/plan.md`: product scope, architecture, phases, and acceptance criteria.
- `docs/planning/additional-plan.md`: later clarifications and requirements.
- `docs/planning/progress.md`: overall progress and native verification evidence.
- `docs/planning/codex-prompt.md`: historical implementation handoff; check current code and status before using it.

## Workspace setup and maintenance

### Workspace setup

Start AI coding agents from the workspace root, the directory containing
`.github`. Ensure the three code repositories are checked out there as sibling
directories: [`peerimeter-agent`](https://github.com/peerimeter/peerimeter-agent),
[`peerimeter-service`](https://github.com/peerimeter/peerimeter-service), and
[`peerimeter-dashboard`](https://github.com/peerimeter/peerimeter-dashboard).
If any are missing, check them out into the workspace root.

Also make the workspace instructions available as `AGENTS.md` in the workspace
root, pointing to `.github/workspace/AGENTS.md`. Prefer a symlink; if the OS or
permissions do not allow one, copy the file there instead. If a root-level
`AGENTS.md` already exists, inspect it and preserve its instructions while
adding the shared workspace pointer; do not overwrite it. The pointer directs
agents to `.github/AGENTS.md` for the shared workspace guidance.

Make the `.agents` directory available from the workspace root as a symlink to
`.github/.agents`, so Codex can discover the project skills when started from
the workspace root. If `.github/.claude` exists, also symlink it into the
workspace root as `.claude` so Claude Code can discover its project settings or
skills. If root `.agents` is already a directory, preserve it, create its
`skills/` directory if needed, and make each project skill available there
with a symlink (or a non-overwriting copy if symlinks are unavailable). Leave same-name paths
untouched and report conflicts. Preserve any existing root `.claude` path too;
inspect it before deciding whether it can be reused, and do not overwrite it.

### Keep the workspace up to date

Before starting work, update all four repositories in the workspace: `.github`,
`peerimeter-agent`, `peerimeter-service`, and `peerimeter-dashboard`. Record the
current branch and inspect each working tree first. Leave any repository with
uncommitted changes untouched and report it. In each clean repository, fetch
and prune `origin`, then update local `main` from `origin/main`. If local `main`
contains commits that are not in `origin/main`, preserve them on a new branch
before moving `main` to `origin/main`; report the saved branch. When the update
finishes, return to the branch recorded at the start (except when that branch
was `main`). Remove only other local branches already merged into
`origin/main`, and keep every branch with unmerged commits.

## Working agreements

- Keep planning claims aligned with the agent, service and dashboard READMEs, their current implementations, `docs/planning/progress.md`, and actual CI evidence. The shared tracker is in this repository; do not expect duplicate per-repository progress files. Distinguish implemented features, planned features, and unverified network scenarios.
- Preserve the three-code-repository separation (`peerimeter-agent`, `peerimeter-service`, `peerimeter-dashboard`). Make changes and commits in the affected repository only; do not treat sibling changes as one atomic commit. This `.github` repository remains the shared planning/site repository.
- Peerimeter uses direct WireGuard connections without a relay. Do not claim STUN alone guarantees NAT traversal.
- Follow the shared [repository conventions](docs/repository-conventions.md) for secrets, commits, branches, pull requests, checks and completion claims.
- Use the authenticated GitHub CLI (`gh`) for GitHub-hosted operations, including pull requests, issues, reviews, checks, and repository settings. Use local `git` for working-tree and commit operations, and for remote transport with `git fetch` and `git push`. Prefer `gh` over GitHub connectors or browser-based workflows, and check `gh auth status` when the authenticated identity matters.
- In `peerimeter-service`, keep the Go module directive at `go 1.27` and pin the exact development/CI patch release in `.go-version`. Service CI and container builds should use that pinned release.
- In `peerimeter-service`, use `direnv` whenever running commands in the repository so `.envrc` loads the local `.env` (run `make env` if `.env` is missing). Keep `.env` local and untracked; run commands with the shell's direnv integration so those variables are loaded.
- In GitHub Actions workflows, use explicit action release tags and maintain `.github/workflows/actions.lock` with `gh actions-lock`. CI must run `gh actions-lock --verify`; update the generated lock whenever workflow action references change, and never edit the lock by hand.
- Assign every PR, including documentation-only PRs, to the GitHub user who requested the work. Use `@me` with `gh pr edit` only when the authenticated `gh` account is that requester; otherwise specify the requester's GitHub username explicitly.
- Auto-merge is the default for every PR. After opening a PR, verify repository settings require status checks, all review conversations to be resolved, and **Require branches to be up to date before merging**; then enable auto-merge immediately and let branch protection gate the merge. If any of those protections are absent, leave auto-merge off and report the configuration gap.
- Keep general settings for the Peerimeter repositories aligned: enable auto-merge, **Always suggest updating pull request branches**, delete head branches after merge, allow rebase merges only, and disable merge commits and squash merges. Protect code `main` branches with strict required checks (`check` and `CodeRabbit`), resolved conversations, linear history, and no force-push or branch deletion; approval count may remain zero unless the project owner changes it. The `peerimeter/.github` docs/site repository requires `CodeRabbit` only; do not require a `check` status there unless a workflow that reports it is added.
- Inspect automated review feedback on every current PR head, including CodeRabbit findings. Verify each finding against the current change, fix valid issues, and resolve every review conversation before the PR can merge. Recheck feedback after each push; required checks, conversation resolution, and strict branch freshness must pass before auto-merge completes.
- Track each pull request through its lifecycle until it is merged or explicitly closed. Recheck CI, automated reviews, merge conflicts, and auto-merge status after updates; resolve blockers when possible and report the current state. An open PR is ongoing work, not a completed task.
- Before feature work, update local `main` from `origin/main` and branch from it. After a PR merges, fast-forward local `main` and leave the repository checked out there.
- For site or documentation changes, inspect the rendered content and check that links, status claims, and references to the other repository are accurate. Record the actual checks performed.
