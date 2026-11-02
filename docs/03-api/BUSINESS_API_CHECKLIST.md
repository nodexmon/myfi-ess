# MyFi ESS — Business-Level API Requirements Checklist

**Status:** APPROVED by project owner  
**Approval date:** 2026-11-01  
**Version:** 1.0  
**Scope:** Phase 1 Employee Self-Service leave-management MVP (approximately 1–50 employees)

## Purpose and approval boundary

This is the plain-English, approved statement of **what the system must let people do** and **what must happen afterward**. It is the business-behavior baseline for API design and testing—not approval of REST endpoint paths, JSON schemas, technical error codes, exact dependency versions, or deployment using real employee records. `API_SPECS.md` remains a proposed technical contract and must conform to this checklist and other owner-approved decisions.

## 1. Employee

- [x] **View own information:** See eligible leave types, earned/used/reserved/spendable balances, request history, status, personal notifications, and approved department absences.
- [x] **File leave:** Choose leave type, dates and reason; view calculated working days, eligibility, credits and non-blocking overlap warnings; attach required files before submission.
- [x] **Track requests:** See stages such as supervisor review, HR review, returned for correction, approved, rejected, withdrawn, or expired.
- [x] **Correct returned requests:** Re-submit within HR's correction deadline; type/date/duration changes require renewed supervisor review, while document/remarks-only corrections return to HR.
- [x] **Withdraw own pending requests:** No review required in pending-supervisor, pending-HR or returned-for-correction states; optional withdrawal reason; release reserved credits once and preserve history.
- [x] **Request cancellation of approved leave:** HR reviews; any valid credit restoration uses auditable reversal transactions, not deletion.

## 2. Assigned supervisor / higher approver

- [x] **Review assigned applications:** View only the request information necessary for assigned staffing decisions; see overlap warnings.
- [x] **Approve or reject:** Visible Remarks input on every decision; optional for approval and required, nonblank for rejection.
- [x] **No self-approval:** A supervisor's own leave is routed to a separate higher/backup approver; no reviewer can act on their own leave.
- [x] **Respect file privacy:** See **whether** supporting documents are attached, but no filename, file metadata, preview, contents or download link. Supervisor permission alone never grants document access.

## 3. HR administrator / authorized HR reviewer

- [x] **Manage staff and routing:** HR-managed invitations, employee records and Excel imports; assign departments, employment status, supervisor and backups. Disable accounts where authorized.
- [x] **Manage policy through forms:** Basic Settings and collapsed Advanced Settings, descriptions, plain-English preview and explicit publish. Published versions do not silently change previous requests. Policy figures are provisional demo defaults until confirmed by company HR.
- [x] **Handle the final stage:** Independently verify eligibility, credit reservations, and supporting documents; finalize approval (optional remarks), reject finally (mandatory remarks), or return for correction (mandatory explanation and deadline).
- [x] **Review cancellations and adjustments:** Authorized HR decision with appropriate remark rule, audit evidence and credit integrity; no self-finalization.
- [x] **Access only authorized files:** Only explicitly privileged, record-scoped HR can open/download leave attachments, whether medical or non-medical. Being an HR user in name alone is not a blanket grant.

## 4. System behavior and privacy

- [x] **Authentication:** Account invitation and password sign-in, mandatory TOTP MFA, controlled recovery, no public signup; backend protects access, not just the interface.
- [x] **Credit accounting:** On paid-leave submission reserve once; HR final approval converts the reservation to one debit. Rejection, expiry or withdrawal releases once. LWOP never consumes paid credit. Approved cancellation may create an authorized compensating credit entry.
- [x] **Final rejection:** Both supervisor and HR rejection permanently end **that** request. A required remark is stored; the employee may file a **new** request instead of editing the rejected one.
- [x] **Corrections:** A returned request remains nonterminal with credits reserved until resolved or deadline expires; HR may extend an unexpired correction deadline.
- [x] **Documents:** Upload securely before filing; verify server-side; keep unsubmitted temporary files private and automatically remove abandoned uploads after a configured expiry. All supporting files have the same strict access rule.
- [x] **Notifications and calendar:** Inform appropriate people; approved absence projections show employee name and dates only—not reason, leave type, remarks, documents or medical data.
- [x] **Audit and history:** Preserve decisions, timestamps, changes, reviewer remarks, policy versions and credit movements; repeated clicks or concurrent operations must not produce duplicate changes.

## 5. Approved explanatory examples (not company policy)

- Hire-date-anchored **monthly** accrual starts after **one completed month** of service (e.g., hired January 15 → first grant February 15). Month-end and year-end edge cases still need exact implementation rules.
- Sample VL/SL annual allowance is **10 days each**, provisional only. Probationary employees may accrue paid VL/SL without spending until regularization under the **sample** rule; LWOP may be requested under its own policy.
- Demo workweek defaults to Monday–Friday and excludes applicable holidays; half-days are out of Phase 1.
- Leave-credit amounts use fixed-precision units and auditable source lots in the approved data-model design.

## 6. Not approved by this checklist

This approval **does not** settle statutory/company leave entitlements, all HR policy values, data-retention periods, attachment cleanup duration, legal compliance sign-off, file-scanning operations, exact API route/DTO shapes, vendor/platform compatibility, package pins, production readiness, or permission to use real employee records. Any conflicts or new business rules require fresh owner approval.

## Next documents

- **Technical proposal:** `API_SPECS.md` (still **NOT APPROVED** as a complete technical contract; reconcile schema-level changes deliberately before migrations).
- **Testing proposal:** `../05-testing/TEST_PLAN.md` (draft for review; test executions and results do not yet exist).
- **Source requirements:** `../01-business/BRD.md`, `../01-business/LEAVE_WORKFLOW_ONE_PAGE.md`, `ROLE_PERMISSIONS_OWNER_APPROVED.md`, `../01-business/LEAVE_POLICY_SETUP_REVIEW.md`, and `../04-security/SECURITY.md`.
