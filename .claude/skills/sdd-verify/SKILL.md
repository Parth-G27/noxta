---
name: sdd-verify
description: >
  Independently verify that a Noxta spec is actually satisfied, before it is
  merged. Use when the user says "verify", "check it", "is NNN done", "did we
  actually build the spec", "ready to merge?", when every task in a tasks.md has
  been checked off, or before opening a pull request for a spec branch. Runs
  lint, tests and build, then maps every acceptance criterion in spec.md to
  concrete evidence in the diff and the test suite, and reports gaps as a
  pass/fail table. Read-only: it reports problems, it does not fix them.
---

# sdd-verify

**This skill must run as a fresh subagent** (dispatch via the Agent tool) that
did not participate in writing the implementation. The invoking context should
give it a self-contained prompt: the spec id, the branch name, and this
instruction verbatim — *"You did not write this code. Assume it does not
satisfy the spec until the diff and the tests prove otherwise. Report only;
change nothing."* A context that just wrote the code is structurally biased
toward believing it works — do not verify inline.

## Steps (run by the subagent)

1. **Gather evidence:** `git diff main...HEAD` (full diff and `--stat`),
   `specs/<NNN-slug>/spec.md`, `plan.md`, `tasks.md`.

2. **Mechanical gates** (each pass/fail), see `references/checklist.md` for the
   full expanded list. Minimum:
   - `npm run verify` (lint + test + build, all workspaces) passes
   - migrations apply cleanly from empty on the test database
   - no `.only` / `.skip` anywhere in the diff
   - no `console.*` added under `server/`
   - no `TODO` / `FIXME` added in changed code
   - no secrets, hardcoded URLs, or credentials in the diff
   - every task in `tasks.md` is checked off

3. **AC trace — the core of this skill.** For each `AC-n` in `spec.md`: locate
   the test(s) named with that AC id, **read them and confirm they actually
   assert the criterion** (a correctly named test that asserts nothing is a
   FAIL), then locate the implementing code. Emit a table:

   | AC | Verdict | Test | Implementation |
   |---|---|---|---|
   | AC-1 | pass | `path/to/test.ts:41` | `path/to/impl.ts:22` |
   | AC-4 | **missing** | — | `path/to/impl.ts:58` |

   A missing test for an AC is a FAIL, not a warning.

4. **Contract check:** actual routes/status codes/response shapes match
   `plan.md`'s API contract; the applied schema matches the data model delta;
   `docs/api.md` and `docs/data-model.md` were actually updated.

5. **Rule check against `/CLAUDE.md`:** SQL only in `repositories/`+`db/`, no
   `req`/`res` below controllers, validation at the boundary, errors thrown not
   status-mapped inline, naming convention on every new file.

6. **Scope check:** every changed file appears in `plan.md`'s file inventory or
   has a stated justification. Flag unexplained drift.

7. **Output a verdict:** `PASS` / `PASS WITH NOTES` / `FAIL`, the AC table, and
   — for any FAIL — a numbered gap list naming the exact file and the exact
   missing thing. **Fix nothing.**

## Back in the main thread (after the subagent reports)

- On **FAIL**: append the gaps to `tasks.md` as new tasks
  (`T0NN [gap] [AC-n] ...`) and hand off to `sdd-implement`.
- On **PASS**: proceed to the merge ritual — flip `status: done` and set
  `merged:` on the branch, update `specs/INDEX.md`, open the PR (title
  `[NNN] <spec title>`, body = Problem + AC list + the AC evidence table),
  merge `--no-ff`, delete the branch.
