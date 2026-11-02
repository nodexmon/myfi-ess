# MyFi ESS — Planning Status and Executive Overview

**Updated:** November 1, 2026  
**Status:** **Core planning baselines approved; some technical verification and company-policy decisions remain open. Planning-only session.**

## What MyFi ESS is

A responsive Employee Self-Service web application for an initial Philippine private-company workforce of roughly 1–50 people. The MVP replaces paper-based leave forms, manual signatures and separate team notifications with one auditable leave-management process. Future ESS modules may be added later, but are **not** part of MVP development.

## Approved employee process

1. **Employee:** view own leave balances, file leave and supporting documents, receive a clear status; paid credits are reserved while pending. Withdraw a pending/returned request without approval; reason optional.
2. **Supervisor:** review only assigned leave, approve with optional remarks or reject with mandatory remarks. Cannot self-approve; supervisor leave escalates.
3. **HR:** independently verify credits, policy eligibility and document evidence; finalize, reject (final, mandatory remarks), or return for correction (mandatory remarks).
4. **Final approval:** reserved paid credits become used once; employee and authorized users are notified; the team calendar shows only employee names and approved dates.
5. **Exceptions:** minor correction routes to HR, significant date/duration/type edits require supervisor reapproval, correction deadlines release reservations when expired, approved leave cancellations require HR review and audited credit restoration.

**Privacy:** All leave attachments (including medical certificates) are readable by the submitting employee and specifically authorized record-scoped HR reviewers only. Supervisors get only a document-present indicator.

## Leave credits and policies

- Provisional demo examples are 10 Vacation Leave and 10 Sick Leave days annually, plus LWOP. These are **not verified employer entitlements**.
- With a monthly, hire-date-anchored demo policy, the first credit posts **after the first completed qualifying month of service**. Daily time clocks or attendance systems are not needed.
- Credit accounting uses fixed-precision units, lots and an immutable transaction history. Pending requests reserve credits; terminal rejection/withdrawal releases them; HR approval uses them exactly once.
- HR policy management uses **Basic Settings → Advanced Settings → human-readable preview → publish**, not editable JSON or a rules engine.
- A small service-accrual calculation boundary leaves room for adding an attendance-based approach later **only if explicitly approved**; no speculative architecture now.

## Selected technology

- **Frontend:** Next.js, TypeScript, Tailwind CSS, shadcn/ui, React Hook Form, Zod, TanStack Query/Table, FullCalendar.
- **Backend:** NestJS on Express, TypeScript, Prisma, PostgreSQL, Better Auth + mandatory TOTP, CASL, generated OpenAPI types.
- **Integration:** same-origin Next.js API proxy, direct-to-private-Cloudflare-R2 presigned uploads, Resend email, PostgreSQL-backed scheduled jobs.
- **Quality:** Vitest, Playwright, Supertest, Testcontainers, ESLint/Prettier, GitHub Actions, Sentry/Pino.
- **Demo hosting plan:** Vercel + Render + Neon + R2, subject to provider terms; separate approval for real-data company use. Docker portability remains a design goal.

## Development governance

Codex and Claude Code must honor approved requirements, maintain strict TypeScript, use native Next.js/NestJS conventions, practice KISS/YAGNI/DRY/pragmatic SOLID, avoid unnecessary layers and **god files**, keep cohesive feature ownership, run relevant tests and checks, and request approval for changes affecting business rules, dependencies, security, production data, remote pushes or deployment. No arbitrary line-count caps.

## Are we finished?

**Yes, with the main product-planning and governance decisions. Not 100% implementation-ready.** Approved: design, BRD, architecture, stack selection, logical data model, security design, business workflow/roles/API behavior, testing criteria, demo/pilot deployment approach, and developer-agent standards. **Not approved:** the full technical REST API contract and first implementation plan. Package versions and company HR/production policies are not final.

No application code exists in this bundle; no database, user account, deployed site or automated test evidence has been created. This package is a **planning handoff**, not a runnable application.

For detailed status and the next unresolved gates, read `DOCUMENT_INDEX.md`.

## November 1 — technical planning reconciliation

The API action inventory, physical-schema mapping, dependency feasibility research and future implementation sequencing have been reconciled in documentation. See `TECHNICAL_READINESS.md`. The approved business scope and clean-coding governance do not change. These are **candidate developer contracts**, not proof that libraries interoperate or that any tests have passed; actual versions, migration, browser cookies, TOTP enforcement and concurrency must be verified in an independently authorized development session.
