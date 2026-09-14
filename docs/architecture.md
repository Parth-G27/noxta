# Noxta — Architecture

Read on demand (not preloaded into every session — see `CLAUDE.md` for the always-loaded rules).

## High-level shape

```
React SPA (client/, Vercel)
        |  HTTPS JSON, credentials: include
        v
Express API (server/, Railway) --- web process, stateless, N replicas
        |
        +-- scheduler.worker  (tick ~15s) --\
        |                                    |  hosted Postgres (Neon)
        +-- dispatcher.worker (tick ~5s) ---/       - users, sessions, auth_tokens
        |                                            - reminders (schedule state)
        +-- retention.worker  (nightly)              - reminder_deliveries (outbox + audit)
                                                       - email_events (webhook ingestion)
                v
          Resend (transactional email) --- bounce/complaint webhook --> web process
```

## Why a Postgres-native scheduler, not a job queue

The schedule's source of truth (`reminders.next_fire_at`) must live in Postgres no
matter what — a queue like BullMQ would only be a transport layer on top of state
that still has to be durable in the database. Adding Redis buys nothing at this
scale and creates a second system that can silently disagree with Postgres (an
eviction or flush loses reminders). `SELECT ... FOR UPDATE SKIP LOCKED` gives
multiple worker replicas safe, coordination-free claiming for free. The escape
hatch to a real queue later is one file (`dispatcher.worker.ts`) because the
outbox table (`reminder_deliveries`) is already the interface — see the plan in
`/Users/bidarip/.claude/plans/lets-being-in-plan-effervescent-gizmo.md` §2.2 for
the full reasoning and crash/restart semantics.

## Why the scheduler advances independently of delivery success

`next_fire_at` is advanced in the same transaction as the outbox insert, before
any email is attempted. Delivery success/failure never feeds back into the
schedule. This decoupling means a Resend outage stalls delivery, never the
schedule itself — no thundering herd on recovery. The only feedback path is a
safety valve: N consecutive failed deliveries auto-pauses that reminder.

## Auth model

Argon2id-hashed passwords. Short-lived (15 min) access JWT + a rotating opaque
refresh token, both in httpOnly cookies (access: `SameSite=Lax`, refresh:
`SameSite=Strict`, scoped to the refresh endpoint's path only). Refresh token
reuse (a token used after it was already rotated) revokes its entire session
family — the standard stolen-token response. See the plan §3.3 for the full
cookie/CSRF/domain scheme, including why cookies were chosen over `localStorage`.

## Error model

Services throw a typed `AppError` subclass (`NotFoundError`, `ValidationError`,
`ConflictError`, ...). Controllers never catch and map status codes themselves —
a single terminal error-handling middleware does that mapping in one place.
Zod validation happens at the controller boundary only; services trust their inputs.

## Timezone handling

Instants are always stored as `timestamptz` (UTC). User intent (what the user
actually typed) is stored separately alongside an IANA timezone string.
Recurring schedules are recomputed in-zone every cycle — never `+24h` arithmetic,
which silently drifts across DST transitions twice a year. See the plan §2.3 for
the full data model and the catch-up/no-thundering-herd policy after downtime.

## This file is intentionally thin

It explains *why*, not *what currently exists*. For the current schema and API
surface, see `docs/data-model.md` and `docs/api.md` — those are updated as an
explicit task at the end of every spec and are the source of truth for current
state. This file only needs updating when an architectural *decision* changes.
