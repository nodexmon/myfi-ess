# MyFi ESS — Business Requirements Document (BRD)

**Version:** 0.1 — approved business requirements baseline  
**Date:** 2026-10-31  
**Project:** MyFi Employee Self-Service (ESS)  
**Business context:** Proposal-stage product for a Philippine private company; initially 1–50 staff  
**Owner:** Solo developer / project proposer  
**Approval status:** OWNER-APPROVED — business requirements baseline. Company HR policies and production use require separate authorization.  
**Authoritative inputs:** `docs/superpowers/specs/2026-10-31-myfi-ess-design.md`; `docs/02-architecture/ARCHITECTURE.md` (approved v1.1); `docs/02-architecture/TECH_STACK.md` (approved v1.0)

> **Scope and legal caution:** This BRD documents an agreed **demonstration product**, not the company's formally approved policy or a determination of statutory compliance. Entitlements, eligibility, record retention, approval authority, labor-law obligations, and production use require HR/legal/IT confirmation. Use fictional employee records and sample attachments only for the online MVP.

## 1. Executive summary

The current leave process uses printed leave forms, physical signatures, and employee-to-team notices through Viber. Employees have limited visibility into requests, balances, and approval status; supervisors and HR depend on manual routing and records. MyFi ESS will digitize this workflow in a responsive browser-based application with structured leave policy configuration, departmental approval routing, independent HR finalization, reliable credit accounting, and automatic minimal-detail team notifications. This working MVP will be presented for possible company adoption; it is **not** an authorized replacement for the current process yet.

## 2. Business problem and opportunity

### Current state (as reported; validate with HR)

1. Employee prints and completes a leave form.
2. Application is circulated for physical signatures; the detailed current approval chain was not independently verified.
3. Employee also informs the team through Viber.
4. Existing HR recordkeeping, leave-balance calculations, and company policy source documents have not yet been inspected.

### Problems to address

- Paper documents and signatures make leave filing and status checks inconvenient, especially outside the office.
- Separate team notices can be forgotten or inconsistent.
- Manual processing makes eligibility, leave balances, scheduling overlaps, and audit history harder to demonstrate and monitor.
- New or changed leave policies should not require rewriting core application code.

### Why an MVP first

The proponent is building independently without company implementation approval. The initial product must demonstrate the employee, supervisor, and HR experience safely with fictional data, and remain portable to company infrastructure if adopted.

## 3. Business objectives and proposed measures

These are **MVP acceptance targets for a controlled fictional-data demo**, not guaranteed organizational results or baselines.

| ID | Business objective | Proposed proof at demonstration |
| --- | --- | --- |
| OBJ-01 | Replace paper filing for the modeled workflow | Employee submits, supervisor decides, and HR finalizes a sample request fully online |
| OBJ-02 | Give employees visibility | Employee can see own request status/history and earned, reserved, spendable credit totals |
| OBJ-03 | Improve management oversight | Approvers can inspect assigned requests and see non-blocking department overlap warnings |
| OBJ-04 | Support policy changes without coding | HR creates/updates a structured leave policy, publishes a new version, and previously filed requests retain their policy history |
| OBJ-05 | Notify teams consistently | Final approval/cancellation produces an in-app department announcement showing only employee name and absence dates; email event is queued in demo/test mode |
| OBJ-06 | Protect business records | Tests reject self-approval, unauthorized file access, duplicate credit reservations, and incomplete MFA |
| OBJ-07 | Enable proposal review | Demonstrate responsive employee, supervisor, and HR workflows on phones and desktop browsers with fictional data |

**Organizational benefit metrics for a later pilot (not claimed here):** average request processing time, percentage of applications completed without paper, number of overdue requests, error/correction rates, and time spent on HR status inquiries. Collect a real baseline only with company approval.

## 4. Stakeholders and actors

| Stakeholder | Interest and involvement | Decision status |
| --- | --- | --- |
| Employees | Submit and track own applications, see balances, receive updates | Product role approved |
| Department Supervisors / Approving Managers | Assess staffing and approve/reject assigned applications | Product role approved |
| HR Operator / HR Administrator | Maintain employees and policies; independently finalize requests; manage adjustments, recovery, and records | Product role approved; actual named authority unconfirmed |
| Department colleagues | See approved absences and receive privacy-preserving team notices | Product role approved |
| Company Management / Owner | May evaluate benefits and authorize deployment | Actual sponsor not selected |
| Company IT / Security / Privacy personnel | Would evaluate hosting, privacy, access controls, recovery, and operations | Engagement pending |
| Project proposer | Owns MVP build, documentation and proposal | Confirmed |

## 5. Scope

### In scope — Phase 1 demonstration

- Invitation-only employee accounts; manual registration and validated Excel (`.xlsx`) import; departments, supervisors, reporting hierarchy and backup approvers.
- Email/password authentication with mandatory authenticator TOTP MFA, single-use recovery codes, authorized HR-assisted recovery, and account deactivation.
- Configurable leave types; default examples: Vacation Leave (VL), Sick Leave (SL), Leave Without Pay (LWOP).
- Versioned structured leave policy settings for accrual/allocation, employment-status eligibility, attachment requirements, filing dates, carryover, expiration and cash-conversion *recording*.
- Submit, approve/reject, HR verify/finalize, return/correct/resubmit, correct by deadline/expire, and cancel approved leave subject to HR approval.
- Accurate paid leave ledger: accrual/grant, reservation, deduction, release and reversal; LWOP records unpaid working days without consuming paid balance.
- Working-day calculation using a default Monday–Friday demo calendar and non-working holidays; department overlap warnings.
- Restricted supporting-document upload/download using real **sample** files, quarantine and verification workflow.
- Email notifications in test mode, persistent in-app notifications, and approved-absence team calendar.
- Secure audit history, responsive browser UI, and automated tests for critical business/security invariants.

### Out of scope — Phase 1

Payroll calculations, automatic salary deductions, payslip/attendance/overtime modules, native mobile applications, Viber integration, half-day leave, rotating shifts, full six-day scheduling, unrestricted IF/THEN policy scripting, and full digitization of all legally mandated special-leave processes. Future integrations must not be represented as already implemented.

### Not authorized by this BRD

Use of real employee or medical records, live company email blast, migration of existing HR data, deployment to the physical company server, or replacement of company HR processes. These require explicit organizational authorization and review.

## 6. Business requirements (prioritized)

Priority **Must** = required to demonstrate the approved Phase 1 workflow. Priority **Should** = approved Phase 1 supporting capability, sequencing may follow the core workflow. All listed requirements remain within the agreed Phase 1 scope; priority does not remove them.

### 6.1 Employee, organization, and authentication

| ID | Priority | Requirement |
| --- | --- | --- |
| BR-001 | Must | Only HR-authorized invitations create active accounts; public self-registration is disabled. |
| BR-002 | Must | HR can create employee records individually, including email, hire date, employment status, department and approver assignments. |
| BR-003 | Should | HR can preview, validate, detect duplicate entries and confirm an `.xlsx` bulk import without silently overwriting data. |
| BR-004 | Must | Employees activate an expiring one-time invitation, verify email, establish a password, and enroll mandatory TOTP before using ESS. |
| BR-005 | Must | All users complete TOTP verification on sign-in; recovery codes are single-use; HR-assisted MFA reset uses documented identity verification and independent authorization for privileged users. |
| BR-006 | Must | HR can deactivate employees and revoke access; employees cannot see other employees' sensitive requests or documents. |
| BR-007 | Must | Each employee has an appropriate primary and fallback approver; hierarchy is configurable and prevents approver cycles and self-approval. |

### 6.2 Leave types and HR policy management

| ID | Priority | Requirement |
| --- | --- | --- |
| BR-008 | Must | HR can create, edit, activate/deactivate and publish leave types and structured policy versions without code changes. |
| BR-009 | Must | Policies specify allocation/accrual method, annual credit amount (when applicable), eligibility, effective dates, and usable-credit restrictions by employment status. |
| BR-010 | Must | Policies configure attachment requirement as optional, always required, or required after a chosen number of days. |
| BR-011 | Must | Policies configure permitted advance filing and retroactive filing windows, plus HR-authorized exceptions with reasons and audit entries. |
| BR-012 | Should | Policies configure carryover caps, expiration, and eligibility for cash conversion; cash conversion is recorded but not paid by the MVP. |
| BR-013 | Must | Published policy versions are immutable for historical requests; new versions apply according to their effective rules without silent retroactive changes. |
| BR-014 | Must | Paid leave earned during probation may be tracked separately from credit usable after regularization; LWOP remains available when the configured policy allows it. |

### 6.3 Leave request and approval workflow

| ID | Priority | Requirement |
| --- | --- | --- |
| BR-015 | Must | Employees submit a leave type, start/end dates, reason and supporting files as required; name is derived from their account. |
| BR-016 | Must | ESS calculates leave duration using assigned working days and non-working holidays; demo default is Monday–Friday, with regular Saturday schedules pending HR confirmation. |
| BR-017 | Must | API independently validates type eligibility, filing window, documents, available spendable credit, dates and approver relationship. |
| BR-018 | Must | The assigned supervisor/higher manager can approve or reject; approved requests go to a separately authorized HR verifier for finalization. |
| BR-019 | Must | HR can approve/finalize, reject with a reason, or return for correction with instructions. |
| BR-020 | Must | On resubmission, material changes (leave type, dates or duration) require supervisor reapproval; minor changes return to HR. |
| BR-021 | Must | Returned applications retain reservations until resubmission, rejection or expiration; a configurable deadline may be extended by HR before expiration. |
| BR-022 | Must | Overdue uncorrected requests automatically expire and release reservations exactly once; users are notified. |
| BR-023 | Must | Approved leave may be cancelled only through an HR-authorized cancellation review; eligible paid credits are restored via audited reversal, not deletion. |
| BR-024 | Must | No employee, manager or HR user can approve or finalize their own leave request, even with multiple assigned roles. |
| BR-025 | Should | Approvers see non-blocking warnings for other department leave requests that are pending or approved for overlapping dates. |

### 6.4 Credit balances, team visibility, notifications, and audit

| ID | Priority | Requirement |
| --- | --- | --- |
| BR-026 | Must | Employee balance shows earned/credited, reserved, and spendable amounts, with past usage and adjustments. |
| BR-027 | Must | Paid-credit reservations occur atomically on submission, persist through pending/correction, and convert to a single debit at HR finalization. |
| BR-028 | Must | Rejection, timeout, or authorized cancellation release/restore balances exactly once with append-only audit/ledger history. |
| BR-029 | Must | LWOP does not reserve/deduct paid credit and records approved unpaid working days for potential future payroll export. |
| BR-030 | Must | Employees see only approved absence names/dates for their department; supervisors see authorized team calendars, HR sees authorized company views. |
| BR-031 | Must | Final approval/cancellation creates an automatic team announcement through in-app notification and an email delivery intent. Never include leave type, reasons or medical details in team notices. |
| BR-032 | Should | In-app notifications persist with read/unread state; users see their history and action links. |
| BR-033 | Must | Record decisions, reviewers, timestamps, reasons, revisions, exceptions, privileged changes and reconciliation events in auditable history. |
| BR-034 | Should | Document uploads support PDF/JPG/PNG sample files, multiple attachments, secure retrieval, and HR access restricted to authorized request contexts. |

## 7. Business rules and demo configuration

| Rule ID | Business rule | Confirmation level |
| --- | --- | --- |
| RULE-01 | Approval is supervisor/higher manager followed by independent HR verification; no self-approval. | Approved product behavior |
| RULE-02 | Vacation and sick leave are separate balance categories; leave types are configurable. | Approved product behavior |
| RULE-03 | Paid leave credits reserve at submission, including while returned for correction; debit only at HR finalization. | Approved product behavior |
| RULE-04 | A return expires after an HR-configurable deadline unless extended; expiration releases the reservation. | Approved product behavior |
| RULE-05 | Material edits require supervisor reapproval; minor amendments return directly to HR. | Approved product behavior |
| RULE-06 | A department overlap generates a warning and does not block submission or approval. | Approved product behavior |
| RULE-07 | Department-wide notices show name and absence dates only, after final approval and approved cancellation. | Approved product behavior |
| RULE-08 | Sample annual credits: 10 VL and 10 SL days; monthly accrual or annual allocation configurable. | Provisional **demo** setting; HR approval needed |
| RULE-09 | During probation, accrual may begin on hiring while ordinary VL/SL remain locked; LWOP filing allowed. | Provisional **demo** setting; HR approval needed |
| RULE-10 | Default Monday–Friday, exclude applicable non-working holidays; no half-days in Phase 1. | Demo constraint; real schedules require HR validation |
| RULE-11 | Filed requests retain the applied policy version; changes do not silently alter completed history. | Approved product behavior |
| RULE-12 | Private-sector law takes precedence over sample CSC-inspired leave policy settings. | Compliance boundary to be confirmed by HR/legal |

**Default values not fixed by project owner:** leave-credit rounding/precision, actual holiday list, attachment size limits, correction expiry duration, document retention, and user-facing SLA targets. These belong in controlled follow-on decisions; coding agents must not turn guesses into company policy.

## 8. Required process flows

### 8.1 Employee filing and final approval

Employee logs in with MFA → chooses eligible leave → enters dates/reason and required attachments → sees working-day/credit calculations → submits → supervisor/higher manager decides → HR independently verifies → finalizes → ledger transitions reservation to debit (except LWOP) → employee and relevant staff notified → approved absence appears on team calendar.

### 8.2 Returned-for-correction

HR returns with reason/deadline → reservation remains → employee edits/resubmits → material change returns to supervisor, minor change returns to HR → HR finalizes/rejects, **or** correction deadline expires → request marked expired and reserved credit released once.

### 8.3 Cancellation

Employee requests cancellation of approved leave with reason → HR verifies cancellation eligibility and absence consumption → HR decides → on approval reverse eligible credits once, remove approved absence from visible calendar and notify affected team → retain full original history.

### 8.4 HR employee enrollment

HR creates/imports employee → assign department, employment status, approver and fallback → invite is issued → employee verifies email, sets password, enrolls TOTP and records recovery codes → access enabled subject to role and employment state.

## 9. Non-functional business requirements

| ID | Area | Requirement and demonstrable evidence |
| --- | --- | --- |
| NFR-01 | Privacy | Strict separation of employee own data, team absence summaries, and sensitive HR/medical details; no exposure in team emails/logging/monitoring. |
| NFR-02 | Security | Password + mandatory TOTP; server-enforced CASL authorization, secure sessions, MFA recovery controls, and no self-approval. |
| NFR-03 | Integrity | Atomic and idempotent credit ledger, requests, cancellations and scheduled jobs; concurrent requests cannot over-reserve credits. |
| NFR-04 | Traceability | Immutable policy version references, append-only ledger/activity events, documented privileged changes, and controlled schema migrations. |
| NFR-05 | Usability | Responsive desktop/mobile-browser leave flows; accessible forms, clear errors, status indicators and correction instructions. |
| NFR-06 | Availability | Demo handles sleeping free backend and transient network faults without corrupting state; **no production uptime claim** on free infrastructure. |
| NFR-07 | Portability | Database and file storage are migratable to company-managed infrastructure without rewriting leave rules. |
| NFR-08 | Testability | Unit, API/integration, concurrency, security-negative, and browser E2E tests cover critical acceptance flows. |
| NFR-09 | Notifications | Persistent in-app notification history; retry-safe email intents. Resend test domain cannot email arbitrary employee addresses. |
| NFR-10 | Operational readiness | Real deployment is blocked until approved data handling, backup/restore, retention, malware scanning, incident response, hosting terms and HR/legal policy checks are in place. |

## 10. Acceptance scenarios for the demonstration

| ID | Scenario | Expected business result |
| --- | --- | --- |
| UAT-01 | HR creates/imports fictional employees and sends invitations | Duplicate/invalid rows flagged; valid employees can activate with mandatory TOTP |
| UAT-02 | Probationary employee requests VL and LWOP | Provisional policy blocks spendable VL; LWOP can be submitted and approved without paid-credit debit |
| UAT-03 | Employee with 5 spendable days attempts concurrent 3-day paid applications | Only allowed reservation(s) succeed; sum of reservations never exceeds spendable starting credit |
| UAT-04 | Supervisor approves; HR returns for missing sample attachment | Credit remains reserved; minor correction returns to HR |
| UAT-05 | Employee materially changes dates or leave type | New eligibility/capacity evaluated; supervisor reapproval required |
| UAT-06 | Correction deadline expires or is extended near timeout | Exactly one valid transition; release at most once |
| UAT-07 | HR finalizes paid leave | One ledger debit, correct balance, calendar entry and privacy-safe department notice |
| UAT-08 | Another team member has overlapping pending/approved leave | Non-blocking warning with correct status; no sensitive reason shown |
| UAT-09 | Supervisor/HR attempts to authorize own leave | Rejected, regardless of overlapping roles |
| UAT-10 | Employee requests cancellation of approved, unconsumed leave | Requires HR authorization; on approval credits restored once and team calendar updated |
| UAT-11 | HR publishes policy changes | Previously filed requests retain applicable historical version |
| UAT-12 | Unauthorized account or direct backend caller tries to fetch medical attachment | Access denied, including before required MFA completion |
| UAT-13 | Repeated scheduler trigger delivers due tasks | No duplicate accrual, expiration or ledger transitions; email intents are monitored for retry |
| UAT-14 | Employee submits Friday–Monday leave on normal Monday–Friday demo schedule | Counts 2 working days, absent a non-working holiday |

## 11. Dependencies, assumptions, constraints, and risks

| Item | Impact | Required mitigation / owner decision |
| --- | --- | --- |
| Company's actual leave policies are not verified | Provisional VL/SL and probation rules may differ | HR and legal review before real rollout |
| Approval authority/department roster unverified | Routing may not match organization | Confirm approver hierarchy and independent HR finalizers |
| Some employees may normally work Saturdays | Monday–Friday demo calculations could be incorrect in real use | Validate schedules and expand policy support before those employees use it |
| Existing statutory and company leave exceptions | MVP may not cover every legally required leave | Maintain interim HR process; assess compliance before deployment |
| Company server capacity and security unknown | Deployment plan cannot yet be finalized | Company IT assessment and tested backup/restore |
| Demo free tiers and absence of a custom domain | Cold starts, email constraints, potential terms restrictions | Fictional demo only; check commercial demo permissions before publication |
| Sensitive supporting documents | Exposure/security risks | Quarantine, authorization, scanning/retention controls before real use |
| Solo agent-assisted engineering | Scope drift, inconsistent code, unsafe autonomous actions | Shared AGENTS.md, ADR approvals, gated plans, CI and owner review |

## 12. Open decisions before company rollout

1. HR's official annual leave allotments, eligibility dates, accrual method, waiting periods, rounding and statutory leave obligations.
2. Actual holiday calendars and whether individual employees follow six-day or variable schedules.
3. Who may approve supervisors' and HR staff's leave and who may conduct independent HR finalization.
4. Allowed medical document categories, maximum file sizes, retention/deletion rules, lawful handling and breach procedures.
5. Whether a private-company pilot may use any selected free hosting plan, or requires paid/self-hosted services and a verified email domain.
6. Actual business success metrics, sponsor, approval/sign-off authority and pilot acceptance criteria.

These are **not blockers to a fictional demo**, except where platform terms or safe sample-data handling require action. They are blockers to representing the demo as an approved production HR system.

## 13. Traceability, governance and change control

- **BRD owns the business problem, actors, outcomes and externally visible requirements.** The approved product design specifies workflow behavior. Architecture, data-model and API documents define the technical implementation. In conflict, do not silently replace an approved product or architecture decision; request the owner's decision and update the affected documents/ADR.
- Every implementation slice must map its tests/acceptance criteria to relevant `BR-*`, `RULE-*`, `NFR-*` and `UAT-*` IDs, and to an approved Superpowers plan.
- Codex and Claude Code may implement within an approved slice; changes to leave policy semantics, approval separation, sensitive-data handling, scope, dependencies or deployment require explicit project-owner approval according to `AGENTS.md`.
- Proposed policy examples must be marked **demo defaults** everywhere (including fixtures and UI descriptions); do not represent them as the company's actual entitlements.

## 14. Review and approvals

| Role | Review | Status |
| --- | --- | --- |
| Project owner | BRD scope, requirements, priorities and acceptance criteria | **Approved** |
| HR representative | Company policy, business process, required exceptions | Not yet requested |
| IT/security/privacy | Production readiness and data handling | Not yet requested |
| Management | Project sponsorship and adoption | Not yet requested |

**Approval scope:** This BRD is an approved business baseline. Security, Data Model, Test Plan, and Deployment Guide have since been approved as design/plan baselines; the detailed technical API contract and first implementation plan are still under review. This approval does not authorize product coding or use of real employee data. See `../../DOCUMENT_INDEX.md`.
