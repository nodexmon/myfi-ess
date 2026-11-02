# MyFi ESS — Consolidated Planning and Governance Documentation

MyFi ESS is a proposed, fictional-data Employee Self-Service MVP for a 1–50 employee Philippine private company. The Phase 1 module is digital leave management. **This is a documentation-only planning package; no application code or runtime configuration is included.**

**Planning readiness:** `TECHNICAL_READINESS.md`  
**Current overview:** `PROJECT_OVERVIEW.md`  
**Approval and outstanding items register:** `DOCUMENT_INDEX.md`

## Start here

1. Approved product spec: `docs/superpowers/specs/2026-10-31-myfi-ess-design.md`
2. Approved technology choices: `docs/02-architecture/TECH_STACK.md`
3. Approved architecture v1.1: `docs/02-architecture/ARCHITECTURE.md`
4. Agent governance ADR: `docs/adr/0001-balanced-agent-governance.md`; approved service-based accrual extensibility ADR: `docs/adr/0002-service-based-leave-accrual-extensibility.md`
5. Agent governance: `AGENTS.md` and `docs/governance/AGENT_GOVERNANCE.md`
6. Approved business requirements v0.1: `docs/01-business/BRD.md`
7. Approved security design v0.1: `docs/04-security/SECURITY.md`
8. Approved PostgreSQL data model v0.1: `docs/02-architecture/DATA_MODEL.md`
9. Owner-friendly API guide: `docs/03-api/API_OWNER_REVIEW.md` (read before technical details)
10. Approved business-level role permissions: `docs/03-api/ROLE_PERMISSIONS_OWNER_APPROVED.md`
11. Approved simplified HR policy UX (Basic/Advanced with preview) and first-completed-month accrual rule: `docs/01-business/LEAVE_POLICY_SETUP_REVIEW.md`
12. Approved one-page leave workflow: `docs/01-business/LEAVE_WORKFLOW_ONE_PAGE.md` (approved 2026-11-01; PDF copy is included in the same directory)
13. Approved business-level API checklist v1.0: `docs/03-api/BUSINESS_API_CHECKLIST.md` (owner-approved 2026-11-01)
14. Planning-reconciled technical REST API contract v0.11 (not runtime verified): `docs/03-api/API_SPECS.md` (NOT separately owner-approved; no execution authorization)
15. **Approved** Test Plan v0.2: `docs/05-testing/TEST_PLAN_OWNER_REVIEW.md` (plain-English first), then `docs/05-testing/TEST_PLAN.md` (36 scenarios; owner-approved 2026-11-01; **none executed**)
16. **Approved** Deployment Guide v0.1: `docs/06-deployment/DEPLOYMENT_OWNER_REVIEW.md` (read first), followed by `docs/06-deployment/DEPLOYMENT_GUIDE.md` (owner-approved 2026-11-01; **no infrastructure provisioned and no deployment authorized**)
17. **Proposed (unapproved) first implementation plan**: read `docs/superpowers/plans/FOUNDATION_AUTH_OWNER_REVIEW.md` first; technical plan `docs/superpowers/plans/2026-11-01-foundation-auth-compatibility.md` (DRAFT — needs separate owner approval before coding)
18. Setup instructions: `GOVERNANCE_SETUP.md`

## Current approval boundaries

- **Monthly leave-accrual basis APPROVED (2026-11-01):** credits are earned for completed qualifying months of employment service, *not* actual daily time-in/time-out. Keep calculation extensible behind a small boundary; do not build attendance features in MVP. HR/legal must specify qualifying-service exceptions. See ADR-0002.

- **Business-level API requirements checklist v1.0 APPROVED (2026-11-01)** at `docs/03-api/BUSINESS_API_CHECKLIST.md`. It finalizes user-visible responsibilities, workflow and privacy behaviors—not the detailed route schemas. Test Plan v0.2 is **owner-approved** as a testing baseline (2026-11-01); **none of its 36 scenarios have run**. Deployment Guide v0.1 is **owner-approved as an approach**, but does not authorize provisioning, release, or deployment.

- **Employee–Supervisor–HR division of responsibilities is owner-approved (2026-11-01).** The plain-English permission matrix is accepted, but exact API endpoints remain to be approved; supervisor-only access to all supporting attachments is denied by the approved business rule.
- **All supporting-document privacy is owner-approved (2026-11-01):** employee + specifically authorized, record-scoped HR only can open/download medical **and non-medical** leave attachments. Supervisors see only a document-present indicator, never filename, metadata, preview, content or download link. Company real-data privacy approval remains separate.
- **The one-page employee → supervisor → HR leave workflow is owner-approved (2026-11-01).** This confirms the user-visible workflow, not the route schemas or full API contract. Business API checklist is now approved; reconcile remaining technical details separately.
- Simple HR policy setup and hire-date-anchored first-completed-month credit timing are approved; business behaviors are consolidated in the approved checklist. Pending-request self-withdrawal (no HR/supervisor approval, one reservation release, permanent history) is owner-approved; the employee withdrawal reason is optional (owner-approved). Supervisor **and HR** rejection are owner-approved as final for that request (mandatory remark, credit release, no editing/resubmit; new request allowed). HR return for correction is separate and nonterminal. **Pre-submit supporting-document uploads and their temporary upload-intent data-model amendment are approved**, including automatic cleanup of abandoned private files; the cleanup duration still needs configuration. The technical `API_SPECS.md` remains unapproved.
- Authorize a narrow compatibility spike to verify Better Auth/NestJS/Prisma/Next.js proxy and pin versions.
- Write Superpowers implementation plans, each for a testable vertical slice, and approve one before coding.
- Obtain HR/legal/IT approval and suitable hosting before handling real employee data or external production deployment.

- **Test Plan v0.2 owner-approved (2026-11-01):** This approves acceptance criteria only, not test results or implementation. **Deployment Guide v0.1 is owner-approved as an approach (2026-11-01)**; no deployment is authorized. Hosted demo requires permission for its intended use (Vercel Hobby is noncommercial/personal only; Render Free is for preview, not production). Real employee data needs a distinct pilot approval.

## Next stage: approved documents → implementation plans

The first proposed plan is `docs/superpowers/plans/2026-11-01-foundation-auth-compatibility.md`. It is a **draft awaiting owner approval**. Do not scaffold application code, install dependencies, deploy, or use real employee information on the basis of the documentation approvals alone. Subsequent implementation plans must be separately reviewed and approved.

## Clean code governance — approved v0.3

- `docs/governance/DEVELOPMENT_STANDARDS_OWNER_REVIEW.md` — approved plain-English summary of the rules.
- `docs/governance/DEVELOPMENT_STANDARDS.md` — **APPROVED v0.3 (2026-11-01)**. Mandatory future-agent rules for framework-native Next.js/NestJS conventions, strict TypeScript, readable feature-focused code, explicit avoidance of god files and unnecessary fragmentation, justified abstractions, and evidence-based reviews. No arbitrary file-length limits. **Planning-only session: no code implementation authorized.**

## Planning readiness update (2026-11-01)

The missing planning-level API/physical schema/compatibility/implementation sequencing details have been reconciled as **technical proposals**. See `TECHNICAL_READINESS.md` and the new technical reconciliation docs. Published-source compatibility has been reviewed, **not proven by installed software**. Implementation and real data remain prohibited in this session.
