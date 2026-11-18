---
name: updatews
description: Update the Peerimeter workspace repositories to their latest main branches and clean up merged local branches.
---

Update the workspace according to the **Keep the workspace up to date** section
in `.github/AGENTS.md`.

1. Locate the workspace root containing `.github`, `peerimeter-agent`,
   `peerimeter-service`, and `peerimeter-dashboard`.
2. Inspect the working tree and current branch in each repository. Do not switch
   branches in a repository with uncommitted changes. Leave it untouched and
   report it so the user can decide how to preserve those changes.
3. In each clean repository, fetch from `origin` and prune deleted remote
   references, switch to `main`, then fast-forward from `origin/main`.
4. Delete only local branches that are already merged into `origin/main`.
   Preserve every branch with unmerged commits.
5. Report each repository's result, including any skipped repository or
   branch that could not be updated safely.

Use the tools and Git client available on the current operating system. Do not
assume a shell, terminal, or command syntax specific to one OS.
