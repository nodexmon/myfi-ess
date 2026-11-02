# MyFi ESS — Phase 1 Design Specification

**Date:** 2026-10-31  
**Status:** OWNER-APPROVED product design baseline (2026-10-31); subsequent approvals are recorded in business, security, architecture, and API-checklist documents; **not** an implementation plan  
**Project:** MyFi ESS (Employee Self-Service)  
**Audience:** Solo developer; prospective HR, IT, and management reviewers  
**Initial scale:** 1–50 employees in a Philippine private company

## 1. Purpose and context

The company currently handles leave using printed forms, physical signatures, and employee notifications in Viber. MyFi ESS is a proposed responsive web application that digitizes leave filing, approval, HR verification, leave balances, and department absence notifications. The owner is building an online demonstration using **fictional** staff and sample documents, intending to propose it for corporate adoption. Production use depends on management approval, HR policy validation, a security/privacy review, and deployment assessment.

### Goals

1. Let employees file and track leave without paper, including Leave Without Pay (LWOP) for probationary staff.
2. Route requests to the correct approving manager and then independent HR verification, with no self-approval.
3. Track accrual, allocation, reservations, deductions, reversals, and historical policy changes accurately.
4. Let HR create leave types through **structured configuration** (not a programming/rule scripting interface).
5. Automatically notify the team about **approved absences** while keeping private leave details confidential.
6. Demonstrate secure, usable desktop and mobile-browser workflows on low-cost portable infrastructure.

### Non-goals of Phase 1

- Payroll calculations or automatic payroll deductions; record LWOP and possible cash conversion for future integration only.
- Attendance, timesheets, overtime, payslips, native apps, Viber integration, or a generic expression-based rule engine.
- Half-day leave, rotating shifts, and complex six-day schedules. Default demo schedule is Monday–Friday; support additional schedules later after HR confirms applicability.
- Replacing all statutory special-leave processes at initial launch. HR must retain an interim process until configured/implemented and legally reviewed.
- Real employees or genuine medical certificates in the publicly hosted demo.

### Anti-goals

No publicly readable medical files, no shared admin passwords, no self-approval, no silently retroactive policy recalculation, no double spending of credits, and no production reliance on a free service without verifying its terms, backup capabilities, and security posture.

## 2. Confirmed requirements versus provisional policy

### Confirmed design choices (agreed by project owner)

- Up to 50 initial users; department-based organization, with a supervisor per department and higher-level/backup approvers.
- A configurable leave type catalog. Ship example Vacation Leave (VL), Sick Leave (SL), and Leave Without Pay (LWOP).
- Form fields: auto-filled employee, leave type, start/end dates, reason, supporting documents, calculated working days and balance.
- Structured leave policies with annual or monthly credit allocation, eligibility by employment status, document rules, advance/retroactive deadlines, carryover, expiration, and cash-conversion eligibility.
- Approval: employee submits → assigned supervisor/higher manager decides → HR independently verifies/finalizes. HR may reject or return for correction. Major changes to leave dates, duration or type require supervisor reapproval; minor changes return to HR.
- Strict no-self-approval; fallback managers; immutable review trail.
- Reservation of credits from submission through pending and returned-for-correction statuses; final deduction only on HR approval. Expiration or rejection releases reservations. Approved leave cancellation requires HR authorization and restores credits when applicable.
- HR-configurable correction deadline; HR may extend before deadline; auto-expire and release reservation if overdue.
- Overlap warnings for department requests that are pending or approved; warn, do not block.
- Department team leave calendar with employee name and approved absence dates only; department email and in-app notification upon final approval/cancellation, not Viber.
- HR invitation-based accounts plus manual registration and .xlsx bulk import; personal emails supported.
- Email/password authentication with **mandatory TOTP MFA for all**; single-use recovery codes and independently authorized HR-assisted recovery.
- Responsive web app; TypeScript, Next.js frontend, NestJS backend, PostgreSQL database; online demo on Vercel + Render + Neon; Cloudflare R2 for private document storage; target future migration to a physical company server.
- Project built by a solo developer as a **proposal**, not as an authorized existing company deployment.

### Provisional HR-policy defaults, NOT confirmed company entitlements

- 10 VL + 10 SL days per year; separate balances.
- During probation: VL/SL *accrue from hire date* but are not spendable until regularization; LWOP can be requested. Both entitlement and usability rules are HR-configurable.
- Example filing windows: VL 3 working days ahead; SL filing within 3 days after absence. These are **examples only**, not decided company rules.
- Working calendar Monday–Friday, with applicable holidays excluded when non-working.
- None of the above overrides statutory minimums or special-leave eligibility. A Philippine private company's policies are **not automatically the same as CSC civil-service policies**. The applicable Philippines labor law, company policy, and employee class must be reviewed by HR/legal before production.

## 3. Approach selection

**Chosen: A — structured policy configuration.** Each leave type uses typed, validated settings shown on HR forms. Versioned policy records determine eligibility, accrual, documentary rules, filing windows, and year-end handling. This is manageable for one developer and easier to test and audit.

**Not chosen: B — generic dynamic IF/THEN rule builder.** Offers unrestricted HR-authored conditions but adds significant complexity in security, testing, migration, conflict resolution, and explaining outcomes. Custom rule scripting is not a Phase 1 requirement; new conditions may require a controlled code change.

**Design principle:** Keep policy evaluation behind a NestJS service so the user interface and database are not tightly coupled to policy logic; *do not* build a generic policy interpreter in advance.

## 4. Actors and permissions

| Role | Primary capabilities | Boundaries |
| --- | --- | --- |
| Employee | Activate account, set up MFA, file/edit/resubmit requests, attach files, view own balances/history, initiate cancellation, view own department calendar | Cannot view others' reasons, balances, medical attachments, or pending applications |
| Supervisor / Approving Manager | View requests routed to them, warnings, relevant staffing calendar; approve/reject with remarks | Cannot finalize HR verification or approve own request |
| HR Operator | Register/import employees, configure departments/assignments, manage leave types and policies, verify/finalize/return/reject/cancel, view authorized records, handle recovery under dual control | Cannot finalize own requests, cannot bypass documented authorization, cannot silently alter ledger history |
| HR Administrator | Authorized HR functions plus account/role management and controlled policy publishing | Highest-risk actions are logged; recovery of HR accounts must use independent authorization |

Role combinations are possible, but **per-request self-approval checks** apply even if a user has multiple roles. Approval is assigned from a reporting/approver relationship; when the employee is a supervisor or HR staff member, route upward or to an independent designated approver. Reject invalid approver cycles and missing approval assignments before submission or show an HR configuration blocker.

## 5. Employee and HR user journeys

### 5.1 HR onboarding

1. HR creates a department and assigns an approver and fallback.
2. HR creates an employee manually, or uploads an `.xlsx` template, previews rows, fixes validation errors/duplicates, and confirms the import.
3. System sends an expiring, single-use invitation to the employee's registered email.
4. Employee verifies link, establishes password, enrolls TOTP by scanning QR, verifies a code, and receives one-time recovery codes.
5. No normal application session is granted until MFA is enrolled. Disabled employees cannot log in.

### 5.2 Leave filing

1. Employee chooses an active, eligible leave type and a date range; UI computes scheduled working days and displays policy, available/reserved credit, and supporting document requirements.
2. Submission validates eligibility, document rule, working calendar, date restrictions, sufficient *spendable* credit (paid leave), assigned approver, and overlapping own requests. LWOP does not reserve or debit paid credits.
3. A single transaction creates the request, stores a policy version/schedule snapshot, and **reserves** paid credits atomically.
4. System notifies the approving manager by email and in-app event.
5. Manager reviews the application and staffing overlap warning. Manager can approve or reject, with reason for rejection.
6. If manager approves, HR verifies remaining eligibility and documents, then finalizes; finalization atomically turns the reservation into a ledger debit. HR can return for correction or reject with a reason.
7. On final approval, update the team's approved-absence calendar and send private employee/manager notifications plus minimal-detail department notice.

### 5.3 Returned request and resubmission

- Return status records a required correction reason and HR-defined due date; credits stay reserved.
- While within deadline, employee edits or uploads documents. A **major** change (dates, duration, type) re-runs all checks and routes to the manager; a **minor** change (remarks/attachment) returns to HR. Major changes reconcile old/new reservations in one transaction; a larger reservation requires sufficient remaining credit.
- HR may extend the deadline before it expires, with reason and audit trail.
- If overdue, the request becomes `EXPIRED`, reservation releases, notification is sent, and a new submission is required. Timer-based job must be **idempotent**; run a periodic sweeper independent of the web process, or use an equivalent reliable hosted scheduler.

### 5.4 Cancellation

- Pending applications: allow authorized withdrawal; release reservation.
- Finalized approvals: employee requests cancellation with reason; HR verifies that it is not already consumed and accepts or rejects. Only acceptance reverses credited debit via a compensating ledger entry. Affected calendar entries and team notifications update.
- A request with partially elapsed dates requires explicit HR handling; no automatic reversal for a consumed day.

### 5.5 Team calendar and conflicts

- Employees see approved absences **in their own department**; do not expose leave type, medical reason, documents, balances or pending requests.
- Supervisors and HR see appropriately scoped pending/approved requests for staffing review. Detect overlap by *scheduled days*, not just raw date ranges; distinguish approved from pending.
- Warnings never block approval. Department notifications occur after final approval or confirmed cancellation, not merely on request submission.

## 6. Leave policy model and evaluation

Each leave type has an identifier/code, title, description, paid/unpaid flag, active state, and one or more **immutable published policy versions**. A structured version includes, as applicable:

- effective dates, eligible employment statuses, paid-credit availability vs accrued balance, annual entitlement, grant/upfront or monthly accrual, proration method and rounding rules;
- carryover and maximum, expiration, optional cash-conversion eligibility (record only; no payroll payout);
- working-day count convention, negative-balance prohibition, advance/retroactive filing thresholds and exception authorization;
- supporting-document rule (`OPTIONAL`, `ALWAYS_REQUIRED`, `REQUIRED_AT_OR_ABOVE_N_DAYS`) and allowed file constraints;
- correction deadline, request restrictions, and HR-facing policy explanations.

Publishing a version validates internal consistency; edits create a **new version** rather than rewriting old versions. New submissions resolve the effective version by leave date and policy rule; a request **stores the version and relevant calculation snapshot** so old decisions remain explainable. When a policy change affects already accrued credit, any adjustment is explicit and audited; no silent rebalance. Builder should choose the simplest deterministic effective-date convention and document it in implementation tests, with HR confirmation before production.

**Labor safeguards:** Policy UI may be configurable, but production configuration must respect applicable statutory leave and wage obligations. A legal/HR review checklist should identify protected leave types, eligibility, documentary restrictions, and retention requirements before real employee use.

## 7. Credit accounting and state machine

Keep a **ledger of credit events** instead of only a mutable `balance` column. Credit events may include `GRANT`, `ACCRUAL`, `ADJUSTMENT`, `RESERVE`, `RELEASE`, `DEBIT`, `REVERSAL`, `EXPIRE`, `CARRYOVER`, and `CONVERSION_RECORD`. Events link to employee, leave type, request (if applicable), policy version, author/system process, and idempotency key. Balances are computed from the ledger or maintained as a reconciled read model.

- `earnedBalance`: net earned usable/locked credits (depending on effective policy).
- `reserved`: pending credit commitments.
- `spendable`: eligible credits minus outstanding reservations, respecting status-based restrictions and expirations.
- `used`: finalized debits.
- LWOP: no paid-credit reservation/deduction, but capture unpaid working days for HR/export.

**Atomic rule:** The submit/resubmit/finalize/reject/expire/cancel transitions and their ledger changes must commit together under database transaction/locking to prevent two simultaneous requests consuming the same credit. Use unique idempotency keys for all operations and jobs, and prevent duplicate ledger events. Historic adjustment may require an HR-authorized, auditable correction operation, never deletion.

Suggested request statuses: `PENDING_SUPERVISOR`, `PENDING_HR`, `RETURNED_FOR_CORRECTION`, `APPROVED`, `REJECTED`, `CANCELLED`, `EXPIRED`. Cancellation of an approved request is a **separate workflow/status** (e.g. `CANCELLATION_PENDING`) so original approval is preserved until HR decides. Store an append-only status/activity history. Define server-side transition rules; never trust client-supplied statuses.

## 8. Core data domains (proposed, builder may refine)

- `users`, `employee_profiles`, `departments`, `employment_history`, `reporting_assignments`, `role_assignments`
- `auth_invites`, `sessions`, `mfa_enrollments`, `mfa_recovery_codes`, `security_events`
- `leave_types`, `leave_policy_versions`, `employee_policy_assignments`, `work_calendars`, `holidays`
- `leave_requests`, `leave_request_revisions`, `approval_actions`, `correction_deadlines`, `cancellation_requests`
- `leave_ledger_entries`, `leave_balance_read_models` (optional cache)
- `documents` (storage key, owner, type/size, scan status), `notification_events`, `email_delivery_attempts`, `audit_events`

Store time stamps in UTC, dates as local **calendar dates** when appropriate, and establish an organizational timezone (demo: Asia/Manila). Avoid treating dates as timestamps in working-day arithmetic. Employee hire and regularization dates are essential for accrual and eligibility.

## 9. User interface, accessibility and privacy

**Employee:** sign in / enroll MFA / recover account, dashboard with balances (earned, reserved, spendable), apply for leave, history/detail/corrections, cancellation request, team approved-absence calendar, notifications, profile/email verification.

**Supervisor:** pending approvals, application detail and staffing overlap warning, decision with remarks, team calendar and notifications.

**HR:** verification queue, returned/rejected/finalized requests, correction deadlines and extensions, employee CRUD/import, departments and approver hierarchy, policy catalog and version publishing, ledger/adjustments, document review, audit history, notifications and account recovery.

Mobile-first forms, accessible labels and errors, keyboard navigation, human-readable policy outcomes, clear status badges, and limited disclosure. Restrict file access via backend authorization. Email notices should contain minimal data and deep links into the secured ESS; **no medical attachment links or sensitive reasons in department notices**.

## 10. Architecture and operations

### Application boundaries

- **Next.js + TypeScript** provides the responsive UI and communicates with NestJS over HTTPS.
- **NestJS + TypeScript** is the single API/authorization boundary, organized into modules: identity/auth, employees/departments, policies, leave requests/approvals, ledger, calendar, documents, notifications, audit. Modular monolith, not microservices.
- **PostgreSQL on Neon** stores business records and transactional ledger.
- **Cloudflare R2** stores private object files; NestJS issues tightly scoped download grants or streams downloads after access checks. Validate content type by content, not extension alone; set file limits and use malware scanning/quarantine strategy before production.
- **Vercel** hosts frontend; **Render** hosts API. Use an email provider with an abstraction; actual service is not yet selected. A reliable background task process must handle email retries, reminders, and expirations despite sleeping free Render services; do not depend on an in-process timer.
- Local development and future company deployment use environment configuration and containerized services. Storage adapter can target an S3-compatible on-premises service; data migration and secrets reconfiguration remain operational tasks.

### Security

- Invitation-only user provisioning. Passwords hashed with Argon2id or equivalent modern password hashing; never stored/retrieved in plaintext. Verify email ownership on activation and on email change.
- Mandatory authenticator TOTP after password. Store TOTP seeds encrypted with managed secret keys, allow small clock drift, prevent reuse where practical, protect rate limits; one-time backup codes stored as hashes. Restrict session issuance until MFA complete.
- HR-assisted reset requires verified identity, independent approver for HR staff, short-lived reset tokens, event logs, and notification to user. Recovery flow cannot silently bypass MFA.
- Secure HttpOnly/Secure cookie sessions with CSRF protection as applicable, hardened CORS/origin checks, server-side RBAC + resource/relationship checks. Credentials and cloud keys in secrets configuration; never expose R2 keys to client.
- Audit privileged actions and protect documents as potentially sensitive personal data. Set retention, backups, deletion, breach procedures and access oversight before production in line with Philippine privacy obligations.

### Hosting caveats

The named free services are for a **fictional-data demo only**. Verify **current** pricing, restrictions, plan limits, and commercial-use terms, especially Vercel Hobby, before showing an externally accessible commercial-facing product. Render sleeping/ephemeral instances can affect scheduled tasks; Neon free limits and storage operations may change. Do not assume the stack is production-grade merely because it runs.

## 11. Example acceptance scenarios

1. HR imports 20 fictional employees, resolves one duplicate, sends invites; each employee activates with TOTP.
2. Probationary employee with accrued but locked VL/SL cannot spend VL, can submit LWOP; manager and HR approve.
3. Regular employee with 5 days available submits 3-day VL; 3 reserved immediately; concurrent 3-day request is blocked for insufficient remaining spendable credits.
4. HR returns for missing attachment, 3-day reservation persists; employee uploads document, returns directly to HR; expiration releases reservation automatically if correction not submitted.
5. Employee changes leave dates after return; reservation recalculated and supervisor must approve again.
6. HR finalizes approved leave; reservation converts once to deduction; calendar and department notices show dates/name only.
7. Two pending applications overlap; manager gets non-blocking warning with pending/approved distinctions.
8. Manager or HR employee files leave; system routes to independent reviewer and refuses self-finalization.
9. HR approves cancellation before leave is consumed; one compensating ledger entry restores credits, calendar clears and team is notified.
10. HR publishes a new SL attachment threshold; previous filed requests retain the policy version and audit history used when submitted.
11. Missing phone: employee uses a one-time recovery code, or HR runs documented dual-control recovery and user enrolls MFA again.
12. A worker with a Monday–Friday schedule files leave Friday–Monday (no holiday): count 2 working days.

## 12. Scope boundaries, uncertainties and builder decisions

**Owner-confirmed policy engine:** Approach A; structured fields, published immutable policy versions and deterministic validation. **No dynamic user-authored rules.**

**Still requires HR/IT confirmation for company rollout:** actual leave entitlements and whether probationary accrual is accepted; holidays/work schedules incl. regular Saturdays; eligible statutory leave types; who legally/operationally can act as HR final verifier; exact approval chains and fallback rules; document retention, privacy notices and consent/lawful basis; whether a department-wide automatic notice fully replaces manual process; company-server OS, network/security and backup process.

**Left to builder:** a conventional database ORM and migration tool for NestJS/PostgreSQL; UI library/design tokens; specific SMTP/email provider; exact file-size limits, retry/backoff and job scheduler; precise policy version selection method and rounding strategy, as long as stated invariants and historical accuracy are upheld. All these require tests and documentation, not additional user decisions at this stage.

**Implementation sequencing (after design approval only):** write a separate implementation plan with vertical slices: identity/onboarding → HR employee structure/policy config → credit ledger/filing → approvals/corrections/cancellations → documents/notifications/calendar → hardening/demo/deployment. Do not scaffold until this design is approved.

## 13. Criteria for design approval

The owner should approve the overall boundaries, the chosen structured policy engine, the distinction between confirmed design and provisional HR policy, the Phase 1 feature set, and the security/infrastructure constraints. Any concern can be resolved in a revised spec *before* preparing implementation tasks.
