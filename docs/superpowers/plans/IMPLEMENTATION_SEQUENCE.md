# MyFi ESS — Future Implementation Plan Sequence

**Status:** Planning-only, proposed sequencing — **not execution approval**. Each milestone will have a separately reviewed and approved Superpowers task plan. Follow `AGENTS.md` and the approved clean-coding governance.

| Order | Narrow, independently reviewable slice | Proof before next slice |
| --- | --- | --- |
| 1 | **Foundation & auth compatibility** — local pnpm workspace, NestJS/Next.js, Prisma/PostgreSQL, Better Auth, same-origin proxy, mandatory TOTP | Exact pinned version report; local auth/proxy/MFA negative tests; no public signup |
| 2 | Employee / department / supervisor assignments & HR invitation workflow | Scoped read/write tests, single-use invitation, no self-approval routing, import preview/commit |
| 3 | Leave-type policies and work calendars | Basic/Advanced HR form and preview; immutable published versions; date-only holiday calculations |
| 4 | Leave credit ledger and service-based accrual | Fixed units and lots; 12-month reconciliation, deduplication, concurrency and expiry tests |
| 5 | Leave request submission + supervisor / HR decisions | All approved transitions including rejection, corrections, withdrawal, finalization, remarks, reserved credits; idempotent transactions |
| 6 | Private upload intents and document access | Direct R2 test upload; immutable validated attachments; expiry cleanup; HR/owner-only access |
| 7 | Calendar, notifications and reliable jobs | Privacy-minimal views; outbox retry; duplicate scheduler runs safe |
| 8 | Final acceptance, hardening and demo handoff preparation | Approved 36-scenario Test Plan and new reconciliation tests; backups/recovery, demo fictional data only |

## Engineering requirements for every slice

- Start from approved business requirements and their tests; define exact file responsibilities, type-safe interfaces and negative/edge cases before writing code.
- Prefer native NestJS/Next.js tools and small modules; no God Files, no needless abstraction, no speculative attendance integration.
- Implement failing test → minimal implementation → passing tests → review → focused commit, only within a separately owner-approved task.
- Stop for unapproved company HR policies, new business behaviors, infrastructure changes, new dependencies or inability to securely enforce MFA.
- Never interpret an approved documentation packet as permission to run these tasks in this planning-only session.

## Next planning artifact

The first slice's detailed draft is `2026-11-01-foundation-auth-compatibility.md`. It is an engineering-ready **plan for review**, not authorization to build. For every later slice, prepare an independent `docs/superpowers/plans/YYYY-MM-DD-<feature>.md` after validating the preceding slice's observed interface/test results.
