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
6. Inspect linked worktrees in each repository. Prune only worktrees that are
   clearly unused: they are not the primary worktree, are clean, and their
   checked-out branch is already merged into `origin/main`. Remove them with
   Git's worktree removal/prune commands; do not delete worktree directories
   manually. Preserve worktrees with uncommitted changes, unmerged commits,
   detached HEAD, or any ambiguous state, and report why they were kept.
7. Report each repository's result, including skipped repositories, preserved
   commits, and branches that could not be updated safely.

Use the tools and Git client available on the current operating system. Do not
assume a shell, terminal, or command syntax specific to one OS.
