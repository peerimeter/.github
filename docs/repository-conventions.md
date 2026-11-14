# Repository conventions

These rules apply across Peerimeter repositories. Repository-specific `AGENTS.md`
files and development guides add technical requirements; follow both when working
in a repository.

## Changes and commits

- Keep each repository buildable and reviewable. Make commits only in the
  repository that owns the files; sibling repositories have separate histories
  and require separate commits and pull requests.
- Before starting feature work, fetch the latest `origin/main`, fast-forward the
  local `main` branch, and create the work branch from that updated base.
- Work on a feature or pull-request branch. Do not push directly to `main` unless
  explicitly requested.
- Use Conventional Commits, with a repository or component scope where useful.
  Examples: `feat(agent): support multiple generated peers`,
  `fix(stun): handle failed probes`, `docs(plan): define multi-network support`.
- Keep commits focused on a coherent change. Do not include unrelated workspace
  edits in a commit.

## Credentials and generated data

- Never commit private keys, tokens, production credentials, or generated
  configurations containing usable secrets.
- Keep public examples separate from usable credentials. Use clearly fake,
  nonfunctional example values and protect any locally generated secrets.
- Before committing, review the staged diff and run the repository's applicable
  secret checks or tests where available.

## Pull requests and checks

- Open a pull request for review. Describe the behavior changed, validation run,
  and any material limitations. Keep implementation and planning claims aligned.
- Auto-merge is the default for every PR. After opening one, verify that
  repository settings require status checks, resolved review conversations, and
  branches to be up to date before merging; then enable auto-merge immediately.
  Let branch protection and any required approvals gate the merge.
- Inspect automated review feedback on every current PR head, fix valid findings,
  and resolve every review conversation before the PR can merge. Recheck feedback
  after pushing; required checks, conversation resolution, and branch freshness
  must pass before auto-merge completes.
- If required checks, conversation-resolution protection, or the up-to-date
  branch requirement are absent, leave auto-merge disabled and report the
  configuration gap. Never bypass branch protection or merge around required
  checks.
- After a pull request merges, fetch the updated `main`, fast-forward the local
  `main` branch, and leave the repository checked out on `main`.
- Run the checks required by the repository's `AGENTS.md` and report those that
  could not run. Do not claim platform or integration behavior without its
  corresponding verification evidence.
- Follow any independent-review or milestone-evidence requirements before
  claiming a phase complete.
