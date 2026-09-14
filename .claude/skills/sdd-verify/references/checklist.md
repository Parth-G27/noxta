# sdd-verify — expanded mechanical checklist

## Build/test gates
- [ ] `npm run verify` passes (lint + typecheck + test + build, every workspace
      touched by the diff)
- [ ] Server migrations apply cleanly from an empty database
- [ ] No `.only` or `.skip` anywhere in the diff (Vitest, any workspace)
- [ ] No `console.*` added under `server/src/` (use `utils/logger.ts`)
- [ ] No new `TODO` / `FIXME` / `XXX` in changed code
- [ ] No secrets, API keys, hardcoded connection strings, or real email
      addresses in the diff
- [ ] Every task in `tasks.md` is checked off with a Progress Log entry and a
      commit sha

## Layer rules (from /CLAUDE.md)
- [ ] No `req` / `res` / `next` parameter appears in any file under
      `server/src/services/` or `server/src/repositories/`
- [ ] No raw SQL string appears outside `server/src/repositories/` and
      `server/src/db/`
- [ ] Every new repository method touching user-owned data takes `userId` as a
      required argument
- [ ] Every new server file follows `<resource>.<role>.ts` naming
- [ ] Every new client feature file lives under `client/src/features/<feature>/`
      and cross-feature imports go through the feature's `index.ts` barrel only
- [ ] Every external input at a controller boundary is parsed by a Zod schema
      from `validators/` (or `shared/`) before touching a service

## Contract fidelity
- [ ] Every endpoint in `plan.md`'s API contract exists, with the stated method,
      path, and auth requirement
- [ ] Every stated status code is actually reachable and returns the stated
      shape
- [ ] The applied database schema matches `plan.md`'s data model delta exactly
      (no undocumented columns/constraints, none missing)
- [ ] `docs/data-model.md` and `docs/api.md` reflect the new state
- [ ] `specs/INDEX.md` row for this spec is accurate (AC/task counts, branch)

## Scope discipline
- [ ] Every changed file appears in `plan.md`'s file inventory, or the diff
      explains why it doesn't
- [ ] Nothing in the diff implements a capability not covered by an AC in
      `spec.md`
