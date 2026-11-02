# MyFi ESS — Relational / Prisma Schema Reconciliation Blueprint

**Version:** 0.2 planning candidate (2026-11-01)  
**Status:** Technical reconciliation against **approved business behavior**, not an executed Prisma schema, migration or new HR policy. Maintains original `DATA_MODEL.md` v0.1 as the owner-approved logical baseline.

## 1. Cross-cutting rules

- Use PostgreSQL UUID primary keys and foreign keys with explicit deletion restrictions on historical tables. Date-only leave and hire-day values map to PostgreSQL `date`; action instants use UTC `timestamptz` and evaluate company policy in `Asia/Manila`.
- Keep **signed BIGINT 1/10,000-day units** on ledger and lots; no JS floating point for credit accounting. Implement fixed-precision JSON day strings for API responses.
- Business history is immutable append-only (`leave_revision`, `leave_decision`, `leave_request_event`, `leave_ledger_event`, `audit_event`, published policy versions). Corrections/reversals create new rows, not destructive edits.
- Auth tables are **generated from the pinned Better Auth version/plugins**, not handwritten in this blueprint. `employee.auth_user_id` is nullable until invite accepted, then unique, with verified FK/Prisma relation shape. No public signup.
- All mutations are NestJS-owned, within DB transactions when touching credits/history/outbox; UI and proxy never decide business state.

## 2. Proposed physical tables and required fields

| Table | Proposed columns / important constraints |
| --- | --- |
| `employee` | `id`, unique `employee_code`, nullable unique `auth_user_id`, `hired_on`, department, active flag, created/updated; employment status is dated in child table |
| `employment_status_period` | `employee_id`, `status`, `effective_from`, nullable `effective_until`; reject overlaps with transaction-level checks/PG constraint |
| `reporting_assignment` | employee, primary/backup approver, valid-from/until; prevent cycles and self-approval; reviewer snapshot retained on application |
| `leave_type`/`leave_policy_draft` | structured settings only, editable draft, version column; basic and advanced UI is presentation only |
| `leave_policy_version`/`policy_eligibility` | immutable publish snapshot, effective start, annual units, accrual anchor/allocation/eligibility; unique type+version and effective date |
| `work_calendar`/`calendar_holiday` | versioned weekday/holiday exceptions, explicit timezone; historic filed revisions retain calendar identity/version |
| `leave_request` | `id`, owner, `status`, `current_revision_id`, assigned supervisor and HR verifier, `correction_due_at`, `version`, `submitted_at`, updated/created timestamps. Status includes **`WITHDRAWN`** terminal state alongside `APPROVED`, `REJECTED`, `EXPIRED`, and pending/returned states. No `DRAFT` feature. |
| `leave_revision` | immutable submitted/resubmitted snapshot: revision number, dates, duration units, type, reason, policy/calendar versions, created-by/at; unique `(request_id,revision_no)` |
| `leave_decision` | immutable `stage`, `decision`, `revision_id`, `actor_employee_id`, **nullable `remark`**, `acted_at`, unique operation key. For rejection/return the remark must be nonblank, enforced in service validation and tested; approval/finalization can store null. Never expose remarks to department viewers. |
| `leave_request_event` | immutable status transition, actor, previous/next states, `revision_id`, timestamp, **nullable `withdrawal_reason`** for `WITHDRAWN`, safe metadata; uniqueness per state-changing operation. Do not create a mutable withdrawal column on the main request. |
| `leave_cancellation` | owner, approved original request FK, reason, `PENDING_HR`/`APPROVED`/`REJECTED`, `reviewed_by`, timestamps, `version`; one active cancellation per approved request. **HR decision `remark` nullable only when approved**, mandatory nonblank when rejected. Prefer dedicated immutable `leave_cancellation_decision` table or request event with cancellaton ID; do not rely on mutable last-remark field. |
| `leave_credit_account` | unique `(employee_id,leave_type_id)`, posted/reserved/used units, version; total and lot invariants enforced within transaction |
| `leave_credit_lot` | account FK, policy version, service earning-period identifier, source `effect_key`, grant/expiry dates, posted/reserved/used units; unique source key. Expired/consumed lots cannot be silently revived. |
| `leave_ledger_event` | immutable signed posting, reserved and used deltas, lot/account/request FKs, unique `effect_key`, actor/job origin; balance projections reconcile to sum. |
| `leave_reservation_allocation` | current request/lot reservation projection, unique `(request_id,lot_id)`; releases and re-allocations journaled. |
| `service_accrual_period` (conceptual, optional separate table) | `(employee_id,leave_type_id,period_start,period_end,policy_version_id)`, qualifying status, calculation metadata; unique period key. **Prefer representing period provenance in `leave_credit_lot` and ledger** unless separate service intervals are genuinely necessary. Do not add a speculative table by default. |
| `document_upload_intent` | `id`, `owner_employee_id`, claimed media/size, random quarantine key, `expires_at`, `status` (`ISSUED`,`UPLOADED`,`VERIFYING`,`READY`,`ATTACHED`,`EXPIRED`,`REJECTED`), current immutable verified object reference, timestamps; **no request required to initiate**. Unique signed grant and operation reference. Server-verification-only status transitions. |
| `document_object` | unique immutable object key, owner employee, optional `upload_intent_id`, nullable `request_id` until submission, safe metadata, content digest, `scan_status`, `verified_at`, created/attachment times. Never expose raw key or temporary signed URLs in record DTO. |
| `scheduled_job`/`outbox_event` | job kind, due/state/lease/retry, idempotency key, relevant resource IDs; notification delivery may retry without duplicate credit effects. |
| `notification`/`audit_event` | scoped employee notifications and privileged immutable audit of reads/grants/denials, approvals, upload intents, recoveries and policy updates. Privacy-safe stored payloads. |
| `api_idempotency_record` | `actor_id`, action/route/target scope, opaque key, request hash, canonical result/receipt, TTL/created time; unique `(actor,action,target,key)`; duplicate identical returns prior result, differing body conflict. |

## 3. State transition mapping — only these user-visible transitions

| Starting status | Actor/action | Result | Credit effect |
| --- | --- | --- | --- |
| New submission | Employee submit with verified attached intents | `PENDING_SUPERVISOR` | Reserve once or no effect for LWOP |
| `PENDING_SUPERVISOR` | Assigned supervisor approve | `PENDING_HR` | None |
| `PENDING_SUPERVISOR` | Assigned supervisor reject | `REJECTED` | Release once; mandatory remark |
| `PENDING_HR` | Independent authorized HR finalize | `APPROVED` | Convert reservation to used once |
| `PENDING_HR` | Independent HR reject | `REJECTED` | Release once; mandatory remark |
| `PENDING_HR` | HR return for correction | `RETURNED_FOR_CORRECTION` | Hold reserved credits; mandatory remark |
| `RETURNED_FOR_CORRECTION` | Owner resubmit minor/major correction | `PENDING_HR` / `PENDING_SUPERVISOR` | Keep or atomic release/re-reserve as appropriate |
| Pending supervisor/HR or returned | Owner withdraw with optional reason | `WITHDRAWN` | Release once |
| `RETURNED_FOR_CORRECTION` | Deadline expires | `EXPIRED` | Release once |
| `APPROVED` | Owner requests cancellation, HR later decides | Original approval history stays; separate cancellation becomes `APPROVED`/`REJECTED` | Restore eligible credit only if approved; no duplicate restoration |

All rejected/expired/withdrawn requests are terminal and not edited or resubmitted. For each action record actor, old/new version, revision, timestamp and permitted remark. No actor may approve their own request; HR finalization must be independent of supervisor decision.

## 4. Atomicity and constraints

1. Before reserve/finalize/release/accrue/expire/reverse, lock affected account rows in a stable UUID order and validate lot sums. Commit projections, immutable ledger rows, request status, decision and outbox **together**.
2. Uniqueness on `effect_key` and API operation prevents double grants, deductions, releases, reversals or credit resurrection. Withdraw-v-finalize and expiry-v-extension races must yield one legal winner under the DB transaction.
3. Employee-bound temporary uploads must be `READY`, unexpired, owned by the submitting employee and **unattached** at submission. Attach in the same transaction as creation of the leave request; repeat submission with same key returns same request, not a second record.
4. A presigned PUT can be reused until expiry. Verify size, content signature, content digest and trusted scan status; **promote/copy to immutable object key** before assigning `READY` and binding to the application. Never assume one-time storage URL semantics. Expired/orphaned uploads are deleted by scheduled cleanup; do not delete attached files.
5. Supervisors get only `hasSupportingDocument: boolean` in their DTO; never filename, storage key, media type, signed URL or original content. Employee and specifically authorized HR get narrowly scoped document metadata and time-limited read grants.
6. Existing approved policy versions remain immutable. Monthly accrual starts only after a completed **qualifying** month from hire-date anchor; future attendance-based method is a separate adapter at the calculation boundary, with identical ledger output.
7. 1–50 employee MVP: use focused Prisma models/modules and ordinary indexes, not triggers/event sourcing frameworks or complex generic rule engines. PostgreSQL-specific constraints may require reviewed SQL inside migrations when necessary; never pretend Prisma alone enforces all interval exclusion rules.

## 5. Named technical decisions NOT silently locked

- Which absences/interruptions qualify as service, proration, leave year and year-end expiry/carryover/cash conversion must come from HR and applicable law. A hire on 29–31 needs an explicit and tested month-end policy; no default should be represented as employer-approved.
- Cutoff for retroactive filing, attachment threshold inclusive/exclusive, return deadline extensions and canceled leave on already-past dates need policy approval where unspecified.
- Sample-only demo scanner simulation must **never** label unscanned real files `CLEAN`. Production requires real malware-scanning solution, retention policy and HR/DPO approval.
- Choose upload intent TTL, max bytes, session/recovery time limits and idempotency TTL in security configuration at actual technical signoff; document them in the future test evidence.
- Better Auth user/session/2FA models must be schema-generated from the tested pinned version before Prisma migrations; this blueprint deliberately does not invent their fields.

## 6. Tests required from this schema mapping

- Concurrent paid reservations cannot overspend; duplicate operation cannot double-debit.
- Two retries of the same monthly service period create exactly one grant and all 12 **qualifying** grants reconcile to configured annual total according to a reviewed period/year rule.
- Withdraw-v-approval and correction expiry-v-extension produce one terminal effect; rejected request is immutable.
- R2 upload intent replay or object replacement cannot change finalized approved file bytes.
- HR-only file access and same-origin/proxy direct NestJS denial produce identical authorization outcomes.
- Calendar and notification projections never contain reason, attachment details, reviewer remarks or leave type.

**This is a data-model handoff contract, not a generated `schema.prisma`. Do not migrate or code without subsequent explicit authorization.**
