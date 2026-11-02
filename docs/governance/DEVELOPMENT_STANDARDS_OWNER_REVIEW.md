# MyFi ESS — Clean Code Governance (Owner Review)

**Status:** APPROVED v0.3 — owner-approved 2026-11-01.  
**Important:** This is **planning only**. No agent will implement anything in this conversation.

## The standard in plain English

We want a codebase that a new developer can understand and maintain—not a collection of clever shortcuts, duplicated rules, or unnecessary engineering patterns.

| Rule | What it means in MyFi ESS |
| --- | --- |
| **Framework-first** | Use native Next.js and NestJS patterns, project conventions, and environment configuration. Avoid recreating features the frameworks already provide. |
| **Keep it simple** | If a straightforward solution is safe, correct, and readable, choose it over multiple classes or frameworks. |
| **Use TypeScript properly** | Strict checking; clear types; no using `any` or type-cast tricks to hide errors. |
| **Keep code organized** | Each file has a clear responsibility. Avoid massive mixed-concern files, and also avoid splitting a simple feature into dozens of tiny files. |
| **Reuse with judgment** | Share real common logic, but don't create a complex generic framework for one small task. |
| **Extend without rewrites** | Leave-accrual methods can be extended later without modifying the ledger and approvals. Do not build unused extension machinery now. |
| **Prove it works** | Code changes require relevant automated tests, type checks, formatting/linting and review evidence. |

## File organization example

**Avoid:** A single `leave.service.ts` managing applications, approvals, credit accrual, private uploads and notifications.

**Prefer:** Focused leave-request, approval and credit services when those are genuinely separate responsibilities. But do not split a simple, cohesive function into multiple files just to reduce line counts.

**Rule:** No hard maximum lines per file; review files for readability and distinct reasons to change.

## What "their environments" means

For **Next.js**, follow its App Router, built-in configuration and `.env.local` conventions; public `NEXT_PUBLIC_` settings must never contain secrets. For **NestJS**, follow its modules, controllers, services, dependency injection and `@nestjs/config`. Keep app-specific settings separate, validate them, and use hosting-provider secrets for deployments. This does **not** mean creating two different systems for the same leave policy: NestJS remains the authority for business logic.

## Three examples

**Good:** A straightforward function calculates working days and a clear test checks holidays.  
**Unnecessary:** A new holiday-calculation plugin framework, factory, service registry and extra package for the same task.

**Good:** One focused service handles leave-credit reservations inside a database transaction.  
**Dangerous:** Repeating reservation logic in a controller, React component, and background job.

**Good:** A small, named leave-accrual boundary supports the approved service-based method.  
**Too much now:** Building biometric attendance integration, generic rules engines, and multiple unused accrual plugins for Phase 1.

## How Codex and Claude Code will be held to it

Their shared `AGENTS.md` will point to `DEVELOPMENT_STANDARDS.md`. Future implementation tasks must include evidence that typing, tests, readability and security checks passed. New packages, architecture changes, deployment and real-data use still require separate owner approval.

**What's not decided:** Exact lint settings and dependency versions will be configured/verified during a separately approved development task—not in this planning conversation.

**Decision:** Approved. These governance rules are mandatory for future MyFi ESS implementation tasks. No code execution is authorized in this planning-only conversation.
