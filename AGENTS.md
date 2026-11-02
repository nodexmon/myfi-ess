# MyFi ESS — Shared Agent Instructions

These are the canonical repository instructions for Codex, Claude Code, and other coding agents. Read only the relevant supporting documents; do not guess missing requirements. For a current approval overview, consult `DOCUMENT_INDEX.md`; for high-level scope, see `PROJECT_OVERVIEW.md`.

## Product and scope

- MyFi ESS is a proposal-stage, responsive employee self-service application for 1–50 employees of a Philippine private company.
- Initial module: leave management. Use **fictional employee accounts and sample attachments only** in the online demo.
- Stack chosen by the owner: TypeScript, Next.js (web), NestJS (API), PostgreSQL (Neon demo), Cloudflare R2 (private files), Vercel/Render (demo). Future portability to a company server is required.
- Architecture: a pnpm Workspaces modular monolith with NestJS Express as the authorization/business-rule boundary, same-origin Next.js API proxy, and direct-to-R2 presigned file uploads; do not introduce microservices or a generic rules engine without approval.
- Approved dependency catalog: `docs/02-architecture/TECH_STACK.md` (v1.0). Use its approved libraries when the task requires them; do **not** choose alternatives or introduce production packages without an owner-approved ADR. Exact versions are **not** locked until the integration compatibility gate passes.

## Sources of truth (in order within their subjects)

1. Owner-approved scope and user decisions: `docs/superpowers/specs/2026-10-31-myfi-ess-design.md`.
2. Working and approval rules: `docs/governance/AGENT_GOVERNANCE.md`; mandatory clean-code and file-organization standards: `docs/governance/DEVELOPMENT_STANDARDS.md` v0.3.
3. Approved technology catalog: `docs/02-architecture/TECH_STACK.md`; approved architecture v1.1: `docs/02-architecture/ARCHITECTURE.md`; accepted decisions: `docs/adr/`. approved BRD v0.1: `docs/01-business/BRD.md`; approved one-page workflow: `docs/01-business/LEAVE_WORKFLOW_ONE_PAGE.md`; approved security design v0.1: `docs/04-security/SECURITY.md`; approved data model v0.1: `docs/02-architecture/DATA_MODEL.md`; **owner-approved business API checklist v1.0**: `docs/03-api/BUSINESS_API_CHECKLIST.md`.
4. **Technical API still a draft:** `docs/03-api/API_SPECS.md` v0.10 is awaiting technical-contract approval; `docs/02-architecture/DATA_MODEL_APPROVED_AMENDMENTS.md` records later approved business rules needing physical schema reconciliation. **Approved testing/deployment baselines:** `docs/05-testing/TEST_PLAN.md` v0.2 (no tests executed) and `docs/06-deployment/DEPLOYMENT_GUIDE.md` v0.1 (no deployment authorized).
5. A **specific approved** `docs/superpowers/plans/*.md` plan defines the implementation task and interfaces. No task is approved merely because it appears in a backlog.

If two sources conflict, **stop and ask**; never silently override an approved decision. Provisional HR policies (including 10 VL/10 SL and probationary accrual) are demo defaults, not confirmed employer entitlements or legal advice.

## Mandatory clean code and TypeScript quality (owner-approved v0.3)

- **Approved guiding principle:** prefer simple, readable, strictly typed TypeScript and avoid overengineering; retain practical extension points for real approved requirements. Never trade away security, credit integrity, or auditability for short code.
- **Framework first:** use native Next.js App Router, env handling and server/client boundaries, and native NestJS modules/providers/DI/`@nestjs/config`. Do not reinvent framework facilities or move NestJS business rules into Next.js.
- **No god files:** keep file and function responsibilities cohesive; split long or mixed-responsibility files when clarity warrants. Avoid bundled unrelated features, deep nesting and opaque helpers. Do not split code simply to meet a line count; avoid needless tiny-file fragmentation.
- **Mandatory standard:** `docs/governance/DEVELOPMENT_STANDARDS.md` **(APPROVED v0.3, 2026-11-01)** applies to Codex, Claude Code, and human contributors. Audit changed files for complexity and cohesion. No implementation is authorized merely by approving the standards; this conversation is planning-only.

## Balanced autonomy

- You **may** inspect the repo, make changes within a specifically approved task, run local tests, use approved dependencies, and create small **local** commits.
- You **must request explicit owner approval before** expanding scope, changing agreed business rules or architecture, introducing a new production dependency, using real employee data, destructive database or history operations, pushing remotely, merging, releasing, deploying, or changing infrastructure/credentials.
- Never bypass authentication/MFA, permission checks, approval controls, branch protection, tool sandboxing, or failing tests to make a task appear complete.
- Do not alter unrelated files or auto-start the next task. Do not assume hosted services or external environments are authorized for live data.

## Business and privacy invariants

- **Approved baseline (2026-11-01):** `docs/03-api/BUSINESS_API_CHECKLIST.md` records the owner-approved, plain-English business-level API requirements. Implement only its approved behaviors and other approved sources. `docs/03-api/API_SPECS.md` is still a technical **proposal**. `docs/05-testing/TEST_PLAN.md` v0.2 is **owner-approved**, but its tests have not been executed; the test IDs are requirements for evidence, not permission to begin coding. The deployment guide is **approved as an approach**, not deployment authorization.
- No employee, supervisor, or HR user can approve or finalize their own leave request, even when roles overlap.
- The owner approved the Employee–Supervisor–HR permission division on 2026-11-01; see `docs/03-api/ROLE_PERMISSIONS_OWNER_APPROVED.md`. This confirms user-facing capabilities and separation of duties, not the full technical API specification. Owner-approved (2026-11-01) **all-supporting-document privacy**: supervisors see only whether any supporting document is attached. The submitting employee and explicitly authorized, record-scoped HR personnel alone may open/download any medical **or non-medical** leave supporting file. Do not expose filenames, file metadata, previews, signed URLs or contents to supervisor-only access. Audit grants and denials. A supervisor with an HR role must have independently granted HR document access.
- The owner approved the one-page business workflow on 2026-11-01 (`docs/01-business/LEAVE_WORKFLOW_ONE_PAGE.md`). This is not approval of `API_SPECS.md`, which remains proposed.
- Supervisor approval precedes independent HR finalization. Date/duration/type edits require supervisor reapproval; minor corrections return to HR.
- Every supervisor/HR leave or cancellation decision has a visible Remarks field and `remark` API property; remarks are optional for positive approvals, mandatory and nonblank for rejection or return for correction; restrict remarks to the employee and authorized reviewers. This specific rule is owner-approved; the overall API contract remains in review.
- Owner-approved supervisor **and HR** rejection: each is final for that leave request ID; reviewer remark required, immutable private history, reserved credit released once, no edit/resubmit on `REJECTED`, but employee may create a new request. HR return for correction remains a distinct nonterminal action.
- Paid leave reservations, releases, deductions, reversals, and expiration must be transactional and idempotent; do not double-spend credits or mutate historical ledger entries. LWOP must not reserve or debit paid balances.
- Returned applications retain reservations until resubmitted, rejected, expired, **or self-withdrawn**; expiration or withdrawal releases them once. HR can extend the deadline before expiry.
- Owner-approved pending-request self-withdrawal: request owner alone may withdraw from `PENDING_SUPERVISOR`, `PENDING_HR`, or `RETURNED_FOR_CORRECTION`, no reviewer approval; transition to `WITHDRAWN`, release reserved credits exactly once, keep audit/history, remove active review work. Approved leave requires the separate HR-reviewed cancellation flow. A withdrawal reason is optional and may be omitted or blank; any provided reason is restricted to the request owner and authorized reviewers, never team notifications or calendars.
- Leave policy versions and approval history must be auditable; published policies cannot silently rewrite prior applications.
- Owner-approved HR policy experience: form-based Basic Settings, collapsed Advanced Settings, plain-English preview and explicit publish. For hire-date-anchored monthly accrual, the first credit is earned after the first completed **qualifying** month of service (e.g., January 15 hire → February 15 first grant when service qualifies). Owner-approved (2026-11-01) **service-based** monthly accrual does **not** depend on daily attendance/biometrics/timekeeping. Follow the **Open/Closed Principle** using a minimal accrual-calculation boundary and unchanged transactional ledger posting; implement the service-based method now, with no speculative attendance integration or generic plugin framework. A future attendance-based method requires separate owner approval and must preserve historical grants. Do not assume which employment interruptions count or resolve ambiguous month-end/year-end cases without HR-approved policy.
- Owner-approved pre-submit documents: employees can upload before creating a leave application using an expiring employee-bound intent. An accepted document is atomically linked on leave submission; abandoned uploads are automatically purged after a configurable (not yet fixed) expiry, without deleting submitted documents. No draft-leave feature is authorized.
- Supporting files are private. Direct-to-R2 uploads must use authorized, short-lived presigned URLs and quarantine until server verification. Server-side authorization is mandatory on document access, including signed URLs. Never include leave reasons, reviewer remarks or **any supporting document** in team notices; team calendar shows approved absence names/dates only.
- All users must enroll and use TOTP MFA. Better Auth handles identity and sessions; CASL handles domain permissions. Store password hashes, encrypted TOTP secrets, hashed recovery codes; HR-assisted MFA recovery needs independent controls. Verify plugin behavior with tests; do not assume encryption/backup-code protections are automatic.
- Validate business behavior on the API; never rely on UI restrictions as the sole control.

## How to implement an approved task

1. Read the approved design, governance and **only** the relevant plan/module docs; inspect nearby code before changing it.
2. State scope, intended files, assumptions, risks, and tests. For new work, use an approved Superpowers implementation plan; stop if none exists.
3. Add failing tests first for behavior changes; implement the smallest clear solution, refactor, and avoid speculative abstractions. Follow existing conventions; use strict TypeScript and typed interfaces.
4. Verify relevant unit/integration/E2E tests, formatting, linting, type checks, and any contract/migration checks that exist. Do **not** invent green results or assume commands exist.
5. Review the diff for security/privacy, scope creep, secrets, and generated files. Make focused local commits only if task authorization permits.
6. Report changed files, test commands and observed results, unresolved risks, and whether approval is needed for the next action. Stop at the approved task boundary.

For exact approval thresholds, review evidence, and definition of done, follow `docs/governance/AGENT_GOVERNANCE.md`. These instructions supplement tool permissions and CI; text alone is not an enforcement mechanism.

## Deployment approval boundary (owner decision 2026-11-01)

- Test Plan v0.2 is approved **as test criteria**; do not claim any tests ran or passed.
- Deployment Guide v0.1 is an owner-approved documentation baseline but does **not** authorize account creation, secret access, infrastructure provisioning, GitHub push/merge, real-data processing, or deployment. Obtain explicit owner approval for any actual release.
- Vercel Hobby use must be personal/non-commercial; check plan terms before any company-associated demo. Render Free is not production-grade. Only fictional test data and test-email recipients for the technical demo.

## Current implementation-plan stage (2026-11-01)

Deployment Guide v0.1 is approved as a documentation baseline, **not** permission to provision cloud resources, push code, or deploy. `docs/superpowers/plans/2026-11-01-foundation-auth-compatibility.md` is the first **draft** plan; it may not be executed until its scope is specifically owner-approved. The technical API contract remains unapproved.

## Planning-reconciled technical handoff (2026-11-01)

Read `TECHNICAL_READINESS.md` before proposing development. `docs/03-api/API_TECHNICAL_RECONCILIATION.md`, `docs/02-architecture/DATA_MODEL_TECHNICAL_RECONCILIATION.md` and `docs/02-architecture/DEPENDENCY_COMPATIBILITY_REVIEW.md` are **candidate technical mappings**, not approval to generate schema/execute code or evidence of a passing integration. The older `DATA_MODEL.md` remains the owner-approved logical baseline. No invented package versions or silent company-HR policy defaults; avoid changing approved business decisions when implementing proposed routes.
