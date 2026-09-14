---
name: sdd-plan
description: >
  Turn an approved Noxta spec into a technical plan and an ordered task list. Use
  when the user says "plan it", "plan this spec", "how should we build NNN",
  "what's the approach", immediately after sdd-spec gets a spec approved, or when
  the user wants to re-plan or change the technical approach of an existing spec.
  Reads specs/<NNN-slug>/spec.md together with the architecture rules in
  /CLAUDE.md, then writes plan.md and tasks.md. Requires explicit user approval
  of the plan before any code is written.
---

# sdd-plan

## Steps

1. **Resolve the spec** — from an explicit argument, else the current branch
   (`spec/NNN-slug`), else ask. **Refuse if `status: draft`** — the ACs aren't
   agreed yet; send the user back to `sdd-spec` first.

2. **Load context:** the spec's `spec.md`, `/CLAUDE.md`, `docs/architecture.md`,
   `docs/data-model.md`, `docs/api.md`, and any spec listed in `depends_on`.

3. **Research the codebase** for anything needing more than ~5 file reads —
   dispatch a read-only Explore/general-purpose subagent and ask it to return
   *file paths and existing patterns*, not code dumps. Summarize findings into
   plan.md's "Decisions" section. Only create a separate `research.md` if the
   findings genuinely exceed about a page (e.g. comparing libraries) — most
   specs don't need one.

4. **Write `plan.md`** from `templates/plan.md`:
   - *Approach* — 5–10 lines, the shape of the solution.
   - *Data model delta* — exact `CREATE TABLE`/`ALTER TABLE` DDL, indexes,
     constraints, and the Drizzle model file(s) it corresponds to. State the
     migration name drizzle-kit will generate it under.
   - *API contract* — per endpoint: method, path, auth requirement, request
     schema, response schema, and every status code with its trigger.
   - *File inventory* — every file to add or modify, exact path, one-line
     purpose, tagged NEW/MOD, grouped by layer, conforming to the naming rules
     in `/CLAUDE.md`.
   - *Client changes* — components, hooks, api module, routes, under the
     feature-folder convention.
   - *Decisions* — each with the alternatives rejected and why. This is what
     stops re-litigating the same choice three specs later.
   - *Risks & mitigations* — call out anything touching email delivery,
     idempotency, timezones, or duplicate sends explicitly, even if the answer
     is "not applicable to this spec."
   - *Test strategy* — a table mapping **every AC → test type → test file**. An
     AC with no row here is a planning bug — fix the plan, not the table.

5. **Conformance-check the plan against `/CLAUDE.md`'s layer rules.** If the
   design genuinely requires breaking one, add a *Deviations* section stating
   the rule, the reason, and the blast radius, and ask the user explicitly.
   Never deviate silently.

6. **Write `tasks.md`** from `templates/tasks.md`. Order bottom-up so the tree
   always compiles: migration → repository → validators → service → controller
   → route → app wiring → client api → client UI → tests → docs. Each task
   carries: id, layer tag, AC refs, imperative description, target path(s).
   Tasks with no AC ref are tagged `[infra]` and should be rare. The **final
   three tasks are always**: update `docs/data-model.md` and `docs/api.md`; run
   `npm run verify` and fix any fallout; update `specs/INDEX.md` and set
   `status: done`.

7. **Size gate:** 8–25 tasks. Over 25 means the spec is too big — stop and
   propose a split before writing the rest.

8. **Present for approval:** a short summary, the API contract, and the task
   list. **Write no code.**

9. **On approval:** commit `docs(NNN): add plan and tasks` with trailer
   `Spec: NNN-slug`; set `status: in-progress`; point the user at `sdd-implement`.
