---
name: setupws
description: Set up the Peerimeter workspace with its four sibling repositories and shared AGENTS.md instructions.
---

Set up the workspace according to the **Workspace setup** section in
`.github/AGENTS.md`.

1. Locate or establish the workspace root, which contains `.github`.
2. Ensure `peerimeter-agent`, `peerimeter-service`, and
   `peerimeter-dashboard` are checked out as sibling repositories in that root.
   If any are missing, check them out there using the available Git client.
3. Make the shared workspace instructions available as root `AGENTS.md`,
   pointing to `.github/workspace/AGENTS.md`. Prefer a symlink; if the OS or
   permissions do not support one, use a copy.
4. If a root `AGENTS.md` already exists, inspect it and preserve its existing
   instructions while adding the shared workspace pointer. Never overwrite it.
5. Make root `.agents` a symlink to `.github/.agents` when possible. If
   symlinks are unavailable, create root `.agents/skills/` and copy project
   skills there without overwriting existing paths. If root `.agents` is
   already a directory, preserve it, create `skills/` if needed, and link each
   project skill there; copy only when symlinks are unavailable. Leave
   same-name paths untouched and report conflicts. Preserve and inspect any
   other existing root `.agents` path; never overwrite it.
6. If `.github/.claude` exists, make root `.claude` a symlink to it. Preserve
   any existing root `.claude` path and inspect it before deciding whether it
   can be reused.
7. To refresh copied skills later, compare them with `.github/.agents` and
   merge changes manually. Keep local edits and report conflicts; never
   overwrite changed copies automatically.
8. Report which repositories and instruction paths were found, created, or
   left unchanged, including any skill conflicts.

Use tools and Git operations that work on the current operating system; do not
assume shell-specific syntax.
