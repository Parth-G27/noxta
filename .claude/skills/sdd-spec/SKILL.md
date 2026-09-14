---
name: sdd-spec
description: >
  Create or revise a Noxta feature specification under specs/. Use this BEFORE
  writing any application code whenever the user proposes new functionality, a
  behavior change, or a bug fix that changes behavior — for example "let's add
  X", "I want users to be able to Y", "build the Z screen", "we need an endpoint
  for W", "change how V works", "make it so that...". Also use when the user says
  "new spec", "spec this out", "write a spec", or asks to edit an existing spec's
  requirements or acceptance criteria. Produces specs/<NNN-slug>/spec.md with user
  stories, numbered acceptance criteria and open questions, and asks clarifying
  questions before the user approves it.
---

# sdd-spec

Read `/CLAUDE.md` first if you have not already this session — it has the layer
rules and naming conventions this spec's plan will eventually need to respect,
though this skill itself writes no technical detail.

## Steps

1. **Decide create vs revise.** Revise if the user names an existing spec id, or
   the current branch is `spec/NNN-*` and they are changing requirements for it.

2. **Allocate the ID.** Read `specs/INDEX.md`'s "Next spec number" line. Zero-pad
   to 3 digits.

3. **Choose a slug** — 2–4 kebab-case words naming the *user-facing capability*,
   never the implementation (`create-reminder`, not `reminders-crud-endpoint`).

4. **Orient before drafting.** Read `specs/INDEX.md` and skim existing spec
   titles/statuses. If this idea belongs inside an in-progress spec, or
   duplicates one, say so and stop — do not create a duplicate spec.

5. **Draft from `templates/spec.md`.** Hard filter: WHAT and WHY only. If you
   write a file path, a table name, an HTTP verb, or a library name, delete it —
   that belongs in `plan.md`, produced later by `sdd-plan`.

6. **Write acceptance criteria**, numbered `AC-1`, `AC-2`, ... Each must be
   independently testable and phrased Given/When/Then. Required coverage: the
   happy path, at least one validation/error case, and — for anything touching
   user data — at least one authorization criterion (e.g. "user B cannot see
   user A's reminders"). If an AC cannot be checked by an automated test, mark it
   `[manual]` and state exactly what the human does to check it.

7. **List open questions**, then ask the user the top 3–5 via `AskUserQuestion`.
   Do not ask what `CLAUDE.md` or an existing spec already answers. Do not ask
   implementation questions — those are `sdd-plan`'s job. If there are zero real
   ambiguities, ask nothing.

8. **Fold in the answers**, delete resolved questions, set `status: draft`, write
   the file to `specs/<NNN-slug>/spec.md`.

9. **Present the AC list back to the user** and ask plainly: approve, or revise?
   **Do not create `plan.md` or `tasks.md`. Do not write any application code.**

10. **On approval:**
    - Set `status: approved`.
    - `git checkout -b spec/NNN-slug` from an up-to-date `main` (or the current
      branch if `main` doesn't exist yet, e.g. for `000-foundation`).
    - Commit: `docs(NNN): add spec for <title>` with trailer
      `Spec: NNN-slug`.
    - Add the row to `specs/INDEX.md` and bump "Next spec number".

11. **Size check** before approval: if the spec has more than ~8 ACs, or clearly
    spans auth + UI + a background worker as one unit, propose splitting it into
    two or three specs with `depends_on` links instead.

12. Tell the user the next step is `sdd-plan`. Never estimate effort in days —
    that's not what this process tracks.

## Template

See `templates/spec.md` for the exact file shape to fill in.
