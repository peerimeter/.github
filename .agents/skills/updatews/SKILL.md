---
name: updatews
description: Update the Peerimeter workspace repositories to their latest main branches and clean up merged local branches.
---

Update all four repositories in the workspace: `.github`, `peerimeter-agent`,
`peerimeter-service`, and `peerimeter-dashboard`.

1. Locate the workspace root containing `.github`, `peerimeter-agent`,
   `peerimeter-service`, and `peerimeter-dashboard`.
2. Record the current branch and inspect the working tree in each repository.
   Leave repositories with uncommitted changes untouched and report them.
3. In each clean repository, fetch from `origin` and prune deleted remote
   references. Fast-forward local `main` when possible. If `main` has commits
   missing from `origin/main`, preserve them on a new branch before updating
   `main`, and report that branch.
4. Return to the branch recorded before the update, unless it was `main`.
5. Delete only local branches already merged into `origin/main`, excluding
   `main` and the branch being resumed. Preserve every branch with unmerged
   commits.
6. Inspect linked worktrees only in repositories that passed step 2. Exclude
   every workspace checkout and every worktree used by the current update,
   including one whose branch is merged. Remove a different worktree only when
   there is evidence it is unused, its checked-out branch is merged into
   `origin/main`, and it has no changes to tracked or untracked files. Check
   ignored files explicitly before removal; preserve them unless their contents
   are confirmed disposable. Remove approved targets only with
   `git worktree remove <path>`. Do not run `git worktree prune` or delete
   worktree directories manually. Preserve missing worktree entries, worktrees
   with uncommitted changes or unmerged commits, detached HEAD, and any
   ambiguous or potentially active worktree; report why each was kept.
7. Report each repository's result, including skipped repositories, preserved
   commits, and branches that could not be updated safely.

Use the tools and Git client available on the current operating system. Do not
assume a shell, terminal, or command syntax specific to one OS.
