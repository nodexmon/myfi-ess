# MyFi ESS — Business to Technical API Reconciliation

**Version:** 1.0 planning handoff (2026-11-01)  
**Status:** Business behaviors are **owner-approved**; route names, DTO shapes, HTTP error details are a **reconciled engineering candidate** requiring generated-OpenAPI checks and separate implementation authorization.

## 1. Overview

Technical proposal: one NestJS REST API under `/api/v1`, Better Auth routes mounted at `/api/auth/*` according to the tested integration, Next.js as same-origin proxy only. No duplicate business validation in Next.js; use NestJS Swagger as the later executable API schema and generate frontend client typings from it.

**Never reinterpret approval of `BUSINESS_API_CHECKLIST.md` as approval of every path listed in `API_SPECS.md`.** This companion document checks coverage and identifies remaining implementation-verification gates.

## 2. Approved behavior → proposed action and outcome

| Approved user action | Proposed route(s) (technical candidate) | Guard and business effect |
| --- | --- | --- |
| Employee submits | `POST /api/v1/leave-requests` | Authenticated employee with complete MFA; only owned, verified pre-submit documents; reserve paid credit once; `PENDING_SUPERVISOR` |
| Supervisor approves/rejects | `POST /api/v1/leave-requests/:id/supervisor-approve` / `supervisor-reject` | Assigned reviewer cannot be request owner; approval remark optional, rejection remark mandatory; reject releases once |
| HR finalizes/rejects/returns | `POST /api/v1/leave-requests/:id/hr-finalize` / `hr-reject` / `hr-return` | Independent HR; approve converts reservation to used; reject releases; return retains reservation and starts deadline; remarks per approved policy |
| Employee corrects | `POST /api/v1/leave-requests/:id/resubmit` | Returned, owned, unexpired; new immutable revision; major edit repeats supervisor, minor to HR |
| Employee withdraws pending | `POST /api/v1/leave-requests/:id/withdraw` | Owner only from pending/returned states; optional reason; terminal `WITHDRAWN`; release once |
| Employee cancels finalized leave | `POST /api/v1/leave-requests/:id/cancellations` | Employee creates HR-reviewed cancellation; no immediate restoration |
| HR decides cancellation | `POST /api/v1/leave-cancellations/:id/approve` / `reject` | Independent HR; optional positive remark, mandatory rejection remark; restore eligible credit once on approval |
| Employee uploads supporting file before submission | `POST /api/v1/document-upload-intents`; `POST /api/v1/document-upload-intents/:id/finalize` | Temporary employee-owned private intent, verified immutable R2 object; abandoned cleanup; no draft leave feature |
| Employee/HR accesses document | `POST /api/v1/leave-requests/:requestId/documents/:documentId/access-grants` | Signed download grant only for submitting employee or specifically authorized HR; supervisors can see only `hasSupportingDocument` |
| HR configures policy | Draft/read/publish actions in `API_SPECS.md` section 7 | Structured Basic/Advanced form fields; preview; immutable effective-dated publication; existing history unchanged |
| HR/admin manages staff | Employee/department/invitation/import endpoints in section 6 | HR-scoped, invitation-only; no public signup |
| Employee checks credits/team calendar | `GET /api/v1/leave-balances/me`; `GET /api/v1/team-calendar` | Exact-day strings; no private leave reasons or document metadata in team data |

## 3. Contract conventions chosen for engineering proposal

- **Time:** Local work dates are `YYYY-MM-DD`; timestamps UTC ISO 8601; policy date logic Asia/Manila. Monetary-like day quantities are strings with four decimals; DB stores signed BIGINT units (1 day = 10,000 units).
- **Pagination:** `page` 1-based, `pageSize` default 20, maximum 100. Reject arbitrary sort fields and unauthorized employee/department filters.
- **Success:** `{ "data": ... }`, optional `{ "meta": ... }` for pagination. Explicit permission-specific DTOs only.
- **Failure:** RFC 9457-style Problem Details for NestJS business routes; keep Better Auth's own response format. Suggested: `400` invalid structure, `401` missing/incomplete MFA, `403` denied role or self-approval, `404` not found/masked, `409` business/credit/idempotency conflict, `412` stale version, `422` policy violation, `428` missing version precondition, `429` throttled. Exact mapping to generated Swagger remains checkable.
- **Mutations:** State transitions include an opaque `Idempotency-Key` and version precondition `If-Match`; server validates actor scope, expected current state, policy, and credit effect in a DB transaction. No arbitrary `PATCH /leave-requests/:id/status` or direct `PATCH` balance operations.
- **Remarks:** Always represented as `remark: string | null` on decision DTOs; positive approval/finalization/cancellation approval may be null; supervisor/HR rejection, correction return and cancellation rejection require trimmed nonblank content. Remarks visible only to owner and assigned authorized reviewers.
- **Upload safety:** Pre-submit intent doesn't require a leave ID; `documentIds[]` must be owned, verified, unexpired and bound only once. Presigned URL replay is possible until expiry; finalization verifies and freezes bytes. No direct public objects.
- **Auth ownership:** Do not hand-code the provider's MFA/registration endpoint routes; discover Better Auth's real routes after pinning and disable any signup/passwordless routes not allowed by policy. Auth/proxy cookie and MFA admission tests mandatory.

## 4. New missing endpoint resolved in this reconciliation

`POST /api/v1/leave-requests/{requestId}/withdraw` was approved as business behavior but not explicitly included in the previous working endpoint inventory. It is now added to the draft. Proposed input `{ "reason": null }` or omitted (trimmed empty treated as null); requires owner+completed MFA, `If-Match` and `Idempotency-Key`. A successful response shows `status: "WITHDRAWN"` and updated version; credit release exactly once. Reject attempts from `APPROVED`, `REJECTED`, `EXPIRED`, `WITHDRAWN`. Approval racing with withdrawal yields a single successful transition, not both. No reviewer remarks required for self-withdrawal.

## 5. Mismatches resolved; outstanding verification

- The older logical data model mentioned `CANCELLED` for pending requests; owner approved **`WITHDRAWN`**, distinct from cancellation after final approval. See `DATA_MODEL_TECHNICAL_RECONCILIATION.md`.
- The older `leave_decision.reason` column was too ambiguous. Use immutable decision `remark`, with nullability rules from approved behavior. Cancellation reviewers also have immutable decision remarks.
- Previous supporting-document model assumed a request ID before upload. A temporary owner-bound upload intent and optional association until submit resolves it.
- **Unresolved runtime-only:** exact Better Auth paths/session MFA guarantees, Next/Vercel rewritten cookie behavior, Prisma generated auth table schema, plugin peer versions, deployed R2/scan limitations. These can only be validated through future integration work.
- **Unresolved HR/security policy:** qualifying service interruption treatment, proration, leave year and expiry, month-end anniversary, retroactive filing anchor, exact upload TTL/max size, session lifetime, sensitive retention. Cannot be silently invented by development agents.

## 6. Engineering review checklist before accepting executable contract

- [ ] Test actual Next-to-Nest proxy vs direct backend under missing MFA and revoked sessions.
- [ ] Test every approved state transition and negative case with real PostgreSQL and a transactionally consistent ledger.
- [ ] Test uploads, file access, temporary intent expiry/overwrite and no supervisor-only document visibility.
- [ ] Generate NestJS OpenAPI for domain controllers and Better Auth route contract from the installed provider.
- [ ] Diff generated contract against this proposal; document any changes and seek owner approval for altered business behavior/new dependencies.

**Owner has approved the business outcomes. This proposal clarifies developer implementation expectations, but does not attest to tested software or authorize code.**
