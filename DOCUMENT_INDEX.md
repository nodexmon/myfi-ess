# MyFi ESS — Documentation Index and Approval Register

**As of:** November 1, 2026 — technical planning reconciliation complete  
**Project state:** Planning and documentation only. No application code, local project setup, tests, hosting configuration or deployment has been performed as part of this planning work.

**Read first:** `PROJECT_OVERVIEW.md` for the plain-English project summary. Then choose the relevant document below.

## Approved baselines

| Document | Approval and purpose |
| --- | --- |
| `docs/superpowers/specs/2026-10-31-myfi-ess-design.md` | Approved — Phase 1 product design |
| `docs/01-business/BRD.md` | Approved v0.1 — business requirements (company policy values remain provisional) |
| `docs/01-business/LEAVE_POLICY_SETUP_REVIEW.md` | Approved v0.2 — simple HR Basic/Advanced form design and first completed month accrual |
| `docs/01-business/LEAVE_WORKFLOW_ONE_PAGE.md` | Approved — employee → supervisor → HR workflow |
| `docs/01-business/MyFi_ESS_Leave_Workflow_Approved.pdf` | Approved — printable one-page workflow snapshot |
| `docs/02-architecture/TECH_STACK.md` | Approved v1.0 — library choices; exact versions not pinned |
| `docs/02-architecture/ARCHITECTURE.md` | Approved v1.1 — modular monolith and integration boundaries |
| `docs/02-architecture/DATA_MODEL.md` | Approved v0.1 — logical entities, ledger, lots and credit precision |
| `docs/02-architecture/DATA_MODEL_APPROVED_AMENDMENTS.md` | Approved business changes since v0.1; **technical schema blueprint documented**, actual schema/generation/migration pending |
| `docs/03-api/ROLE_PERMISSIONS_OWNER_APPROVED.md` | Approved — role permissions and document privacy |
| `docs/03-api/BUSINESS_API_CHECKLIST.md` | Approved v1.0 — user-visible API outcomes, not route schemas |
| `docs/04-security/SECURITY.md` | Approved v0.1 — security design only, not implemented controls |
| `docs/05-testing/TEST_PLAN.md` | Approved v0.2 — 36 planned scenarios, none executed |
| `docs/05-testing/TEST_PLAN_OWNER_REVIEW.md` | Approved — plain-language test criteria |
| `docs/06-deployment/DEPLOYMENT_GUIDE.md` | Approved v0.1 — fictional-data demo / separate company-pilot approach, **no deployment authorized** |
| `docs/06-deployment/DEPLOYMENT_OWNER_REVIEW.md` | Approved — plain-language deployment guide |
| `docs/governance/AGENT_GOVERNANCE.md` | Approved — balanced Codex/Claude Code permissions and approval gates |
| `docs/governance/DEVELOPMENT_STANDARDS.md` | Approved v0.3 — strict TypeScript, framework-first clean coding, no god files, KISS/YAGNI/DRY/pragmatic SOLID |
| `docs/governance/DEVELOPMENT_STANDARDS_OWNER_REVIEW.md` | Approved — plain-language developer rules |
| `docs/adr/0001-balanced-agent-governance.md` | Accepted — agent operating model |
| `docs/adr/0002-service-based-leave-accrual-extensibility.md` | Accepted — qualifying-service accrual with modest future extension boundary |

## Approved agent handoff documents

- `AGENTS.md`: canonical instructions for Codex, Claude Code and future developers, including planning-only boundary for this conversation.
- `CLAUDE.md`: references shared agent instructions.
- `GOVERNANCE_SETUP.md`: guidance for future repository adoption; not evidence of a configured repo.

## Planning-completed technical design candidates — not runtime verified or approved for execution

- `docs/03-api/API_SPECS.md` — planning-reconciled technical REST/OpenAPI design v0.11; routes/DTOs are candidate contracts, not generated or independently executed/approved. The *business-level API checklist* is approved.
- `docs/03-api/API_OWNER_REVIEW.md` — explanatory guide, not an approval of routes.
- `docs/superpowers/plans/2026-11-01-foundation-auth-compatibility.md` and `FOUNDATION_AUTH_OWNER_REVIEW.md` — draft future implementation plan; **no approval to execute**.

## New planning-level reconciliation documents

- `TECHNICAL_READINESS.md` — plain-English progress, readiness gates and remaining owner/company decisions.
- `docs/03-api/API_TECHNICAL_RECONCILIATION.md` — approved business actions matched to proposed REST actions; fixes missing withdraw route.
- `docs/02-architecture/DATA_MODEL_TECHNICAL_RECONCILIATION.md` — proposed physical-table mapping for withdrawal, remarks, upload intents, service accrual and transaction constraints.
- `docs/02-architecture/DEPENDENCY_COMPATIBILITY_REVIEW.md` — evidence from current vendor docs and exact version-pin/run-tests requirements; no version combination claimed tested.
- `docs/05-testing/TECHNICAL_RECONCILIATION_TESTS.md` — extra proposed technical cases beyond the 36 owner-approved v0.2 scenarios.
- `docs/superpowers/plans/IMPLEMENTATION_SEQUENCE.md` — order of future independently authorized feature slices.

## Outstanding **runtime** verification and company approvals

1. Verify mutually compatible versions for Better Auth, NestJS, Prisma, Next.js and the proxy; validate cookie/MFA/route handling before pinning versions.
2. Reconciliation designs are now documented; generate/verify actual Prisma models and executable OpenAPI in a future separately approved development session.
3. Confirm company HR/security policy values before use: qualifying-service interruptions, year/end-of-month treatment, upload cleanup TTL, file limits, session/MFA recovery timing; technical error conventions are now proposed for future generated OpenAPI checks.
4. Obtain company HR/legal/IT/management authorization for real employee use; verify statutory leave entitlements, schedules, holidays, personal/medical-data retention and security controls.
5. Recheck hosting plan terms and commercial eligibility, scanner, backup restore, email domain, and operational readiness before a company pilot.

Approval of the document set **does not grant permission to code, install dependencies, initialize a repo, push, merge, migrate, deploy or connect real staff data in this session.**
