---
name: updatews
description: Update the Peerimeter workspace repositories to their latest main branches and clean up merged local branches.
---

Update the workspace according to the **Keep the workspace up to date** section
in `.github/AGENTS.md`.

1. Locate the workspace root containing `.github`, `peerimeter-agent`,
   `peerimeter-service`, and `peerimeter-dashboard`.
2. Record the current branch and inspect the working tree in each repository.
   Leave repositories with uncommitted changes untouched and report them.
3. In each clean repository, fetch from `origin` and prune deleted remote
   references. Fast-forward local `main` when possible. If `main` has commits
   missing from `origin/main`, preserve them on a new branch before updating
   `main`, and report that branch.
4. Return to the branch recorded before the update, unless it was `main`.
5. Delete only other local branches already merged into `origin/main`. Preserve
   every branch with unmerged commits and the branch being resumed.
6. Report each repository's result, including skipped repositories, preserved
   commits, and branches that could not be updated safely.

Use the tools and Git client available on the current operating system. Do not
assume a shell, terminal, or command syntax specific to one OS.
