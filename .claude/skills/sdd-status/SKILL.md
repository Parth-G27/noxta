---
name: sdd-status
description: >
  Report where the Noxta project stands in its spec-driven workflow and what the
  single next action is. Use when the user asks "where are we", "what's next",
  "status", "what was I working on", "what's left on NNN", "how far along are
  we", at the start of a session after a break, or whenever it is unclear which
  spec is active or which stage it is in.
---

# sdd-status

## Steps

1. Read `git branch --show-current`, `git status --short`, and `specs/INDEX.md`.

2. Derive the active spec from the branch name (`spec/NNN-slug`). Read its
   `tasks.md` for the checked/total count and the last Progress Log entry.

3. Report:
   - Active spec + its stage (draft / approved / in-progress / done / blocked).
   - The current `specs/INDEX.md` table.
   - Any uncommitted changes.
   - **One** recommended next action, naming the specific skill to use
     (`sdd-spec` / `sdd-plan` / `sdd-implement` / `sdd-verify`).

4. **Drift detection** — report any of these loudly if present:
   - On a `spec/*` branch with no matching `specs/` directory.
   - A spec marked `in-progress` with no `plan.md`, or no branch.
   - All tasks in a spec checked but its `status` is still `in-progress`
     (needs `sdd-verify`).
   - A spec marked `status: done` but its branch is unmerged.
   - Two specs marked `in-progress` at once — flag it and ask which to
     continue.
   - Uncommitted application changes sitting directly on `main`.
