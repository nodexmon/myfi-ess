# MyFi ESS — Technical Planning Readiness Report

**Version:** 1.0 — documentation reconciliation completed 2026-11-01  
**Status:** **PLANNING HANDOFF PREPARED**; technical specifications are engineering-review candidates, **not runtime-verified or separately owner-approved for execution**.  
**Session:** Planning/documentation only; zero application code, scaffolding, installations, database migrations, tests, provisioning or deployment.

## Plain-English outcome

The original system blueprint, business API checklist, leave workflow, Test Plan and Deployment Guide remain approved. This package adds the missing **technical planning documents**: a reconciled proposed API contract, a relational/Prisma schema blueprint, an evidence-based library compatibility review and a phased implementation plan. The approved business behavior has not changed.

The engineering handoff is now sufficiently specified to *request approval for a narrow compatibility spike*; it is not truthful to claim the stack works before it has been installed and tested. No real company policy or real employee-data rollout is approved by this report.

## Status by gate

| Gate | Outcome / required next step |
| --- | --- |
| User-visible business behavior | **Approved** — `docs/03-api/BUSINESS_API_CHECKLIST.md` and workflow |
| Logical architecture, data principles, security | **Approved** — original baseline documents retained |
| Technical REST endpoints and error conventions | **Reconciled candidate** — `docs/03-api/API_SPECS.md` + `API_TECHNICAL_RECONCILIATION.md`; generated OpenAPI still absent |
| Logical-to-Prisma mapping | **Documented candidate** — `docs/02-architecture/DATA_MODEL_TECHNICAL_RECONCILIATION.md`; no `.prisma` schema, migration or database |
| Dependency feasibility (documentation) | **Reviewed** — `docs/02-architecture/DEPENDENCY_COMPATIBILITY_REVIEW.md`; published guides support the proposed stack in principle |
| Mutually compatible exact pinned versions | **NOT VERIFIED** — future local runtime/peer-dependency and security checks required |
| First local foundation/auth plan | **Reconciled draft** — `docs/superpowers/plans/2026-11-01-foundation-auth-compatibility.md`; not authorized to execute |
| Later implementation sequence | **Planned at milestone level** — `docs/superpowers/plans/IMPLEMENTATION_SEQUENCE.md`; each slice requires a separately approved plan |
| Actual corporate leave policy, statutory treatment, security operations | **HR/legal/IT pending**, before real staff data or company pilot |

## Decisions preserved in technical specifications

- Employee self-withdrawal only while pending supervisor, pending HR, or returned for correction; optional reason; reservation released **once**, terminal `WITHDRAWN` history.
- Supervisor and HR rejection terminal for that request, with mandatory nonblank remarks and single credit release.
- Approval/finalization and approved-cancellation remarks visible but optional. Rejection, return-for-correction and cancellation rejection remarks mandatory.
- Upload before submission via employee-owned short-lived **upload intent**; signed URL is reusable until expiry, so immutable object finalization must prevent overwrite and unauthorized replacement. Abandoned intents are cleaned up automatically (exact TTL not chosen).
- Supporting documents accessible only to owner and independently authorized HR, not supervisor-only access; team calendar and notifications do not reveal private details.
- Monthly accrual after a *completed qualifying service month* anchored to hire date; first grant after month 1. Leave lots and append-only ledger use integer 1/10,000 day units; no daily attendance integration in MVP.
- Pragmatic extension boundaries: add future attendance source later without modifying ledger/approval implementation; avoid framework bloat, giant files, pointless abstractions.

## Mandatory runtime gates for **a later, separately approved development session**

1. Choose/pin exact Node/pnpm/Next/Nest/Prisma/Better Auth/wrapper versions from a compatible peer dependency graph, record `pnpm-lock.yaml`; ensure supported security patches.
2. Verify NestJS Express request parsing and raw auth requests with the community wrapper; test DTO validation on ordinary API routes.
3. Verify Better Auth Prisma schema generation + migration under Prisma 7 driver adapter; session revocation and multi-factor enforcement on **direct NestJS** and same-origin Next.js paths.
4. Verify Vercel rewrite/proxy `Cookie`, `Set-Cookie`, origin/CSRF and cache handling. Rewrites alone do **not** prove cookie correctness.
5. Verify transactions against PostgreSQL under concurrency: duplicate credits, duplicate HR finalization, withdraw-v-approve race, expiry-v-extension race.
6. Validate R2 upload intent expiry, server-side object content checks, immutable promotion, document authorization, quarantine/malware scanning readiness.
7. Confirm real HR service-time/leave entitlements, year-end and proration treatment, qualifying interruptions, statutory rights; don't use provisional demo policy as a real employer entitlement.
8. Verify hosting terms, backup/restore, appropriate email domain, retention, scanner and company authorization before any live pilot.

## Remaining product/policy settings deliberately NOT invented

Qualification of leave during unpaid or interrupted service; calendar-year versus service-year reset; end-of-month hire-date handling (29–31); partial-period proration; retroactive deadline anchor; attachment exactly-on-threshold semantics; expired credited-lot restoration; abandoned-upload TTL; upload size; admin recovery timing; session duration; document retention. These must be confirmed at the appropriate company/policy or security gate. The approved demo example is not legal advice.

## How to use this package

Start with `PROJECT_OVERVIEW.md`, `DOCUMENT_INDEX.md`, the approved product spec, then `docs/03-api/API_TECHNICAL_RECONCILIATION.md`, `docs/02-architecture/DATA_MODEL_TECHNICAL_RECONCILIATION.md`, `docs/02-architecture/DEPENDENCY_COMPATIBILITY_REVIEW.md`, and finally the *draft* foundation/auth implementation plan. All future agents must obey `AGENTS.md` and `docs/governance/DEVELOPMENT_STANDARDS.md`.

**Planning is ready for handoff; technical verification and implementation are not completed or authorized in this session.**
