# Repository conventions

These rules apply across Peerimeter repositories. Repository-specific `AGENTS.md`
files and development guides add technical requirements; follow both when working
in a repository.

## Changes and commits

- Keep each repository buildable and reviewable. Make commits only in the
  repository that owns the files; sibling repositories have separate histories
  and require separate commits and pull requests.
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
- Enable GitHub auto-merge when the repository supports it and required checks
  are configured. Let branch protection and required checks decide when it merges.
- If auto-merge is unavailable, leave the pull request open for manual review.
  Never bypass required checks or branch protection.
- Run the checks required by the repository's `AGENTS.md` and report those that
  could not run. Do not claim platform or integration behavior without its
  corresponding verification evidence.
- Follow any independent-review or milestone-evidence requirements before
  claiming a phase complete.
