# MyFi ESS — Phase 1 Test Plan

**Version:** 0.2  
**Status:** OWNER-APPROVED v0.2 — testing baseline; 36 scenarios are planned, none have been executed.  
**Prepared:** 2026-11-01  
**Product stage:** Documentation only; **no application code or tests executed**

## 1. What success looks like

A fictional employee can securely sign in, file leave, pass independent supervisor and HR review, see correct credits and private notifications, and safely handle corrections, rejections, withdrawal and approved-leave cancellation. HR can manage staff and leave rules through understandable forms. No one can see information beyond their role, and repeated or concurrent actions cannot corrupt balances.

**Plain-English acceptance rule:** Every **Must** requirement in the approved BRD and every essential approved workflow/privacy behavior must pass in automated tests or documented manual acceptance checks before the MVP is called demo-ready. An approved test plan does not itself imply working software.

## 2. Review scope

**In scope:** employee invitation and TOTP, HR-managed users, leave policy setup, leave calculations, requests and revisions, supervisor/HR decisions, document uploads and access, ledger transactions, cancellation, calendar/notifications, background jobs, audit logs, security controls, and responsive web UI. Include Excel import and approved supporting features before a complete Phase 1 sign-off.

**Out of scope:** payroll processing, native mobile apps, half-day leave, custom policy scripting, real-company HR records, production launch and company-server migration. Production operations have a separate readiness gate (Section 7).

**Test data:** Entirely fictional employees, departments, email addresses and PDF/JPG/PNG documents. Resend test recipient(s) only; no arbitrary staff email delivery. All third-party secrets are test-only and supplied through protected environment variables.

## 3. Test levels and evidence

| Level | Tools already chosen | What we need evidence of |
| --- | --- | --- |
| Domain and date calculations | Vitest | Credit calculations, date windows, holiday handling, state transition guards, validations |
| Frontend components | Vitest + React Testing Library + MSW | Form wording, helpful validation, visible remarks, role-specific displays, policy preview |
| API integration with a real DB | NestJS Testing + Supertest + Testcontainers PostgreSQL + Prisma migrations | Real authorization, row locks/transactions, idempotency, ledger entries and request state |
| Browser end-to-end | Playwright | Invitations/MFA, cookie proxy, leave filing/approval, corrections, accessibility and mobile layouts |
| Storage and scheduled-job integration | Authorized R2 test bucket/emulator where fidelity permits; isolated HTTP scheduler tests | Private object flow, quarantine/finalization, expiry cleanup, retry-safe jobs and outbox messages |
| Owner acceptance | Scripted manual check with fictional people | End-user language and screens match approved one-page workflow and policy setup |

**Evidence per test:** ID, BRD/approval reference, test data, expected result, execution command/environment, actual result, and any issue reference. Automated CI collects machine-readable outcomes; manual checks use a signed review checklist. Do not claim any test passed until actually run.

## 4. Business acceptance checklist

The table is deliberately written so the owner can judge the behavior without reading API internals. **P0** = cannot claim demo-ready if it fails. **P1** = required before the full Phase 1 feature set is accepted, but can be sequenced after the first secure vertical slice.

| Test | Priority | Scenario: what a person does | Pass criteria / proof |
| --- | --- | --- | --- |
| TP-01 | P0 | HR invites a fictional employee; a stranger attempts public signup | Invited user can activate once; public self-registration and reused/expired invite are denied (BR-001, BR-004) |
| TP-02 | P0 | Employee signs in with password only, then enters correct TOTP | Ordinary screens and API denied until TOTP completes; success grants scoped access (BR-005) |
| TP-03 | P0 | Employee uses an already-used recovery code; HR initiates MFA reset | Recovery code cannot be reused; privileged reset requires separately authorized identity verification (BR-005) |
| TP-04 | P0 | Deactivated employee attempts to use an old session | Access revoked, including direct API access, and sensitive actions denied (BR-006) |
| TP-05 | P1 | HR adds staff and supervisor assignments | Correct employee/department/backup routing; duplicate employee identity and approver cycles rejected (BR-002, BR-007) |
| TP-06 | P1 | HR previews and imports a spreadsheet containing duplicates/invalid rows | Problems shown before confirmation; existing records not silently overwritten (BR-003) |
| TP-07 | P0 | HR configures a leave policy in Basic Settings | Understandable labels, Advanced collapsed, plain-English preview, explicit publish (BR-008–013; approved policy UX) |
| TP-08 | P0 | HR publishes a new policy after an employee already filed | Older request and credit history retain their original applicable policy (BR-013) |
| TP-09 | P0 | Hire-date monthly policy grants a probationary employee credits | First grant occurs after first **qualifying** completed month, not on hire date; probationary spending restricted under demo policy (BR-009, BR-014; RULE-13) |
| TP-09A | P0 | Service-based monthly accrual runs without attendance records or timekeeping integration | Eligible month posts exactly once from hire date, approved qualifying-service status and policy; no grant on hire date; retry cannot double-credit. For an annual allowance of 10 days, 12 qualifying monthly grants total exactly 10 days before expiry/adjustments, with consistent fixed-precision rounding. The calculation strategy remains separate from the unchanged ledger posting interface (RULE-13). |
| TP-10 | P0 | Employee requests Friday through Monday under Mon–Fri demo schedule | Two working days charged, except excluded holiday dates; date only, no half-days (BR-016) |
| TP-11 | P0 | Employee files paid leave exceeding available **spendable** credit | Clear error; no partial request/credit reservation (BR-017, BR-027) |
| TP-12 | P0 | Employee files eligible paid leave | Correct request stored and corresponding credits **reserved**, not yet permanently used (BR-015, BR-027) |
| TP-13 | P0 | Supervisor approves, then independent HR approves | Required stages enforced; optional approval remarks accepted; one final debit and approved status (BR-018, BR-019, BR-027) |
| TP-14 | P0 | Supervisor or HR rejects a request | Mandatory nonblank remark; terminal history; reservation released exactly once; same ID cannot be edited; new application allowed (BR-018/019/028) |
| TP-15 | P0 | HR returns a request with an explanation | Mandatory remark and deadline; reservation remains; employee notified (BR-019, BR-021) |
| TP-16 | P0 | Employee corrects a missing file versus changing dates/type/duration | Minor corrections return to HR; material revisions repeat supervisor approval (BR-020) |
| TP-17 | P0 | Correction deadline expires, is extended by HR, or races with resubmission | Exactly one authorized outcome and one release if expired; no duplicate effects (BR-021/022) |
| TP-18 | P0 | Employee withdraws a pending or correction-returned request with reason left blank | Withdrawal succeeds without reviewer; balance released once; immutable history kept; no withdrawal after final approval |
| TP-19 | P0 | Employee asks to cancel approved leave; HR reviews | No automatic cancellation; valid approval reverses eligible credits and updates calendar; rejection retains leave (BR-023) |
| TP-20 | P0 | Supervisor applies for own leave or acts with an HR role | Self-approval and single-actor two-stage approvals denied; higher/independent review required (BR-024) |
| TP-21 | P0 | Employee files LWOP | No paid credit reservation/debit; approved unpaid days recorded (BR-029) |
| TP-22 | P1 | Employee requests dates overlapping colleagues' pending/approved absences | Warning appears without automatically rejecting or leaking private details (BR-025) |
| TP-23 | P0 | Employee views approved team calendar and an unrelated department record | Only authorized approved employee names/dates appear; private details and unauthorized departments hidden (BR-030) |
| TP-24 | P0 | HR approves/cancels leave; team receives an announcement | Notifications/outbox created once; team message includes name/dates only, not leave type, reasons, remarks or documents (BR-031) |
| TP-25 | P1 | User opens, reads and revisits notifications | Persistent unread/read status and appropriate authorized action links (BR-032) |
| TP-26 | P0 | Employee uploads required PDF/JPG/PNG before submission | Authorized upload bound to employee and request on submission; invalid/unverified file cannot be used; no general public download (BR-034) |
| TP-27 | P0 | Supervisor tries to list/view/download any supporting document | Only an attachment-present indicator allowed; no name/metadata/preview/URL/content; owner and record-authorized HR access only (approved privacy rule) |
| TP-28 | P0 | Employee abandons an uploaded file or retries finalization | Temporary file removed only after configured expiration; submitted verified file retained; finalization cannot be duplicated (BR-034) |
| TP-29 | P0 | Two concurrent requests compete for the last five available days | Atomic transactions prevent overspending, duplicate reservations, lost updates (BR-027/028) |
| TP-30 | P0 | Double-click approval, repeat cron trigger, or retry an interrupted job | One credit debit/grant and one logical notification event; delivery retries do not create duplicate event records; safe idempotent retries |
| TP-31 | P0 | HR changes policy, adjusts credits, or an approver acts | Immutable ledger/revision/audit trail identifies actor, time, decision/remarks without rewriting prior history (BR-033) |
| TP-32 | P0 | User bypasses Next.js and calls Render/NestJS routes directly | MFA-complete session, CSRF/origin, role, ownership and request-state checks still enforced (security baseline) |
| TP-33 | P0 | Browser logs in/out through same-origin proxy, including MFA | Correct secure session-cookie propagation/revocation; no authenticated response cached publicly |
| TP-34 | P0 | Application logs an error or sends telemetry | Password, TOTP, session secrets, leave reasons, attached files and sensitive health data absent from logs, errors and team notice payloads |
| TP-35 | P1 | Employee uses form on a narrow mobile viewport | Core leave filing, validation, approval actions, keyboard navigation and accessible labels remain usable |

The test catalog is a starting set. Each approved implementation plan will add edge-case tests and exact fixtures for the feature it implements. This document does not authorize new requirements beyond the approved business baseline.

## 5. High-risk edge cases to add before implementation

- **Month-end accrual:** Hires on the 29th–31st; February, leap days, different monthly lengths. Exact anniversary normalization requires explicit design approval, not a guessed policy.
- **Precision and balances:** Fractional monthly grants in fixed 1/10,000-day units, rounding conventions, year-end expiration, carryover lots, policy changes, cancellation after a lot expires.
- **Date behavior:** Philippine work dates/holidays, weekends, same-day or retroactive filing windows, invalid reversed dates, scheduled non-working Saturdays.
- **Document lifecycle:** Replayed presigned URLs, overlarge or wrong-signature files, malware-scanning gate for real use, stale upload intent, concurrent delete/submit, supervisor access through metadata/listing side channels.
- **Concurrent decisions:** Withdrawal versus approval, HR expiry extension versus job execution, rejection versus approval, overlapping ledger changes, duplicate scheduler/outbox execution.
- **MFA security:** Enrollment incomplete, lost device/recovery, session theft/revocation, HR self-reset prevention, roles that overlap employee/approver/HR.

## 6. Definition of done and test sequence

1. **Before coding:** approved feature design, approved implementation slice, Testcontainers/Docker environment, fictional data, compatibility checks for Better Auth/Prisma/NestJS/Next.js proxy, and unresolved policy decisions explicitly recorded.
2. **During each slice:** a failing test that describes the acceptance behavior → minimal implementation → unit/integration tests → browser tests where the user journey needs them → review of audit/privacy impact.
3. **CI:** dependency installation from a locked pnpm lockfile, formatting/lint, strict TypeScript checks, unit/integration tests, generated OpenAPI contract checks, build and smoke E2E where supported. Do not bypass failing checks.
4. **Demo acceptance:** All P0 tests green, the full Phase 1 committed P1 functionality verified, critical security/privacy findings resolved, scripted owner walkthrough completed with fictional staff. Evidence and remaining defects documented.
5. **No automatic release:** Local/CI success does not authorize Git push, merge, production deployment, data migration or use of real employee records; follow `AGENTS.md` approval gates.

## 7. Company pilot / real-data readiness is a separate gate

Before real employees or medical documents: confirm actual company policies/statutory entitlements with HR/legal; determine privacy lawful basis, notices, retention and privileged access; deploy to contractually permitted hosting; validate malware scanning, domain email sending, backups/restore, incident response, least-privilege reviews, monitoring and recovery procedures. Capture written organizational authorization. A passing demo test suite alone is not permission to process real data.

## 8. Approval scope and remaining test prerequisites

**Already approved:** Business-level API checklist (`../03-api/BUSINESS_API_CHECKLIST.md`), employee–supervisor–HR responsibilities, supporting-document privacy, leave workflow, and simplified policy configuration.

**Still to settle for implementation:** which absences and employment interruptions count as qualifying service (HR confirmation), month-end accrual semantics, unused-upload cleanup duration, demo file size/validation limits, sample policy settings and HR-confirmed real leave entitlements. These are not assumed approved by this test plan.

**Owner disposition:** Approved as Test Plan v0.2. Future implementation slices must reference its test IDs and report real outcomes. Approval does not mean the tests were executed, and it does not authorize application development within this planning-only session.

## References

- Approved business checklist: `../03-api/BUSINESS_API_CHECKLIST.md`
- Approved business requirements: `../01-business/BRD.md`
- Approved one-page workflow: `../01-business/LEAVE_WORKFLOW_ONE_PAGE.md`
- Approved roles and privacy: `../03-api/ROLE_PERMISSIONS_OWNER_APPROVED.md`
- Approved policy setup: `../01-business/LEAVE_POLICY_SETUP_REVIEW.md`
- Approved architecture: `../02-architecture/ARCHITECTURE.md`
- Approved data model: `../02-architecture/DATA_MODEL.md`
- Approved security design: `../04-security/SECURITY.md`
- Proposed technical API draft: `../03-api/API_SPECS.md`
