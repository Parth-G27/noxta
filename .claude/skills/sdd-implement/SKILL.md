---
name: sdd-implement
description: >
  Implement the next unchecked tasks from a Noxta spec's tasks.md. Use when the
  user says "implement it", "build it", "start coding", "continue", "next task",
  "keep going", "carry on", or immediately after a plan is approved by sdd-plan.
  Works one task at a time from specs/<NNN-slug>/tasks.md, follows the layer,
  naming and testing rules in /CLAUDE.md, writes tests alongside code, checks
  tasks off, and commits per task. Refuses to implement anything not covered by
  the current spec's tasks.
---

# sdd-implement

## Steps

1. **Establish the active spec** from the current branch (`spec/NNN-slug`). If on
   `main` or a non-spec branch, stop and resolve that first — never write
   feature code on `main`.

2. **Re-read `spec.md`, `plan.md`, `tasks.md` every invocation.** Do not rely on
   earlier conversation turns. Sessions resume from files, not memory — this is
   what makes the process survive a fresh session or `/clear`.

3. **Select the next unchecked task.** One at a time, unless the user says "all"
   or names several explicitly. Never work ahead of the checklist order.

4. **Path-check before writing.** The file path must appear in `plan.md`'s file
   inventory and satisfy the `<resource>.<role>.ts` naming convention from
   `/CLAUDE.md`. If the plan is wrong, amend `plan.md` first with a one-line note
   under *Decisions*, then proceed — never diverge from the plan silently.

5. **Implement, then test.** Every test covering an acceptance criterion is named
   with the AC id first: `it('AC-3: rejects a reminder scheduled in the past', ...)`.

6. **Run scoped checks** — the touched workspace's tests plus `npm run lint`.
   Green before checking the task off, always.

7. **Check the task off** in `tasks.md` and append to its Progress Log:
   `T004 ✓ YYYY-MM-DD — <one-line note> — <commit sha>`.

8. **Commit** — `feat(NNN):` / `test(NNN):` / `chore(NNN):` / `fix(NNN):` with
   `Spec: NNN-slug` and `Tasks: T00N` trailers. One commit per task; tests land
   in the same commit as the code they cover.

9. **Scope guard — the most important step.** If the user asks for something not
   in `tasks.md`, stop and classify out loud:
   - *(a) Covered by an existing AC, task just missing* → add the task to
     `tasks.md`, note it, implement it.
   - *(b) Not covered by any AC* → say so plainly. Offer either: amend `spec.md`
     with a new AC (requires re-approval and a `plan.md` touch-up before
     proceeding), or open a new spec. **Never just do it.**
   - *(c) Cosmetic / zero behavior change* → allowed inline per the `/CLAUDE.md`
     exemption list; mention it.

10. **Never** check off a task with failing or skipped tests, a stubbed
    function, or a new `TODO` in the changed code. A partially done task stays
    unchecked, with a Progress Log note explaining why.

11. **When every task is checked, do not declare the spec done.** Tell the user
    the next step is `sdd-verify` — verification must be independent of the
    context that wrote the code.
