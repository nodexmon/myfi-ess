# MyFi ESS — Development Standards & Clean Code Governance

**Status:** APPROVED v0.3 — owner-approved 2026-11-01  
**Scope:** Future implementation by Codex, Claude Code, and human contributors  
**Authority:** Supplements `AGENTS.md`, `docs/governance/AGENT_GOVERNANCE.md`, and the approved technical stack. If they conflict, stop for owner review.  
**Planning boundary:** This document defines future development rules; it does **not** authorize coding, package installation, pushing, merging, or deployment in the current planning session.

## 1. North star

**Build the simplest clear, correct, secure solution that fulfills an approved requirement.** Optimize for future maintainers understanding the code, not for minimum line count, fashionable patterns, or hypothetical scale. A slightly longer readable solution beats a clever short one. A justified abstraction beats copied complex rules; an unnecessary abstraction is worse than a little straightforward duplication.

We follow **KISS**, **YAGNI**, **DRY with judgment**, **single responsibility**, **explicit boundaries**, **composition where useful**, and the **Open/Closed Principle only where there is a real extension requirement**. Avoid treating SOLID as a demand to make interfaces and classes for everything.

## 2. Framework-native first (owner-requested amendment)

**Adopt the conventions and runtime configuration of Next.js and NestJS before creating custom alternatives.** This is not permission to introduce more frameworks or complexity. Each framework owns its established responsibilities; avoid fighting the framework.

- **Next.js (`apps/web`):** use its App Router, layouts, Server Components where appropriate, Client Components only for interactivity, and native routing/configuration. Keep presentation and the same-origin API proxy here. Do **not** duplicate NestJS authorization, leave rules, persistence, or backend orchestration inside Next.js Route Handlers or Server Actions.
- **NestJS (`apps/api`):** use feature modules, controllers, injectable providers and dependency injection. Apply built-in guards, pipes, interceptors and exception handling **where useful**, rather than writing a parallel middleware/framework layer. Use the approved validation, authentication and authorization integrations. Keep controllers thin and domain rules in their owning services.
- **Environment configuration:** use Next.js's native environment-file loading for the frontend and the already-selected NestJS `@nestjs/config` with validated configuration for the backend. Do not add a custom dotenv loader, configuration framework, or cross-app environment abstraction without a demonstrated need.
- **Separated application settings:** document required keys in `apps/web/.env.example` and `apps/api/.env.example` (never real secret values). Local developers use untracked `apps/web/.env.local` and an app-local API `.env` as appropriate; test environments use isolated test configuration/data. Hosting secrets come from approved deployment-provider configuration, not committed environment files.
- **No frontend secrets:** `NEXT_PUBLIC_` variables are exposed to the browser and are typically embedded at build time. Never put API credentials, database URLs, private keys, or R2 secrets in public environment variables. Only deliberately public, non-sensitive values may use the prefix. Server-side secrets stay server-side.
- **Predictable operations:** validate required backend settings at startup and fail clearly if they are absent or invalid. Keep local/test/demo/production distinctions explicit; never point tests at production data. Avoid depending on undocumented monorepo working-directory assumptions when loading `.env` files.
- **Use official ecosystem tooling:** prefer native framework CLIs, framework-supported testing techniques, and approved utilities before homegrown solutions. Generated OpenAPI contracts remain the frontend/backend type boundary. Add another layer only if it solves an actual approved problem.

Official references: [Next.js environment variables](https://nextjs.org/docs/app/guides/environment-variables) and [NestJS configuration](https://docs.nestjs.com/application/configuration).

## 3. Code-quality rules (future agent MUST)

1. **One approved task at a time.** Follow its acceptance criteria; do not add speculative capabilities or perform unrelated refactors.
2. **Prefer existing approved tools.** Use built-in language/framework features before adding dependencies or writing a generic helper. A new production dependency needs owner approval.
3. **Name things by business intent.** Use `reserveLeaveCredits` rather than `processData`, `handleThings`, or abbreviations whose meaning is unclear. Keep domain terms consistent across UI, API, database, and tests.
4. **Keep functions focused and control flow readable.** Prefer guard clauses, simple loops, and named helpers where they clarify intent. Avoid deep nesting, giant controllers, nested ternaries, and chains that obscure errors.
5. **Do not create abstractions merely in anticipation.** No generic CRUD repositories, unnecessary factories, base classes, service locators, custom state frameworks, unnecessary event buses, or shared packages without a demonstrated need. Keep an abstraction if it protects a real boundary (e.g., future accrual method, storage adapter).
6. **Respect module ownership.** Next.js owns presentation and the same-origin proxy; NestJS owns business rules, authorization, and database writes. Put rules in the owning feature; do not scatter the same eligibility or credit logic across layers.
7. **Do not duplicate authoritative facts.** Share/generated API types from OpenAPI rather than hand-maintaining two contracts. Do not duplicate leave-balance logic in the frontend or mutate historical policy/ledger entries.
8. **Make failure behavior explicit.** Validate inputs, return meaningful errors, record actionable but private-safe logs, and never hide a failed operation behind an empty `catch`, silent fallback, or success response.
9. **Prioritize correctness for high-risk functions.** Leave-credit changes are transactional and idempotent; permissions use server-side CASL plus no-self-approval checks; document access remains private. Simple must never mean skipping safeguards.
10. **Leave code easier to understand than it was.** Remove dead code and unnecessary complexity within the approved change; avoid unrelated cleanup and speculative performance tuning. Comments explain *why*, not what obvious code already says.

## 4. File size, focus, and code organization (owner-approved amendment)

**Do not create “god files” or dump unrelated concerns into one module.** Keep related code together and split a file when it acquires distinct responsibilities, is difficult to navigate or test, or changes for several unrelated reasons. Judge by cohesion and complexity—not line count.

- **One understandable responsibility per file or component.** A feature may have several files when they represent meaningful responsibilities, but a file may contain several small, tightly related helpers.
- **NestJS:** keep controllers thin; use cohesive services for distinct domain behavior. Separate leave requests, approvals, and credit postings when the logic warrants it. Do not create one class per trivial operation or a generic service hierarchy.
- **Next.js:** keep page/layout files understandable; move substantial interactive UI or complex forms into named feature components when useful. Do not scatter a simple page across many tiny components just to shorten files.
- **Functions:** favor a small, clear purpose, descriptive names, guard clauses, and shallow control flow. Extract helpers for understandable concepts, not arbitrary line counts.
- **Group by feature and ownership:** leave application logic belongs with leave requests; policy rules, credit-ledger operations, authentication, documents, and notifications retain their established boundaries. Avoid catch-all `utils.ts`, `helpers.ts`, or `service.ts` becoming dumping grounds.
- **No arbitrary hard file-length cap:** a cohesive 350-line file can be clearer than ten 35-line files. A long file triggers a responsibility/readability review, not automatic splitting.
- **No needless fragmentation:** avoid single-use wrappers, a file for every tiny function, or new folders/layers that make the code harder to navigate.
- **Refactor safely:** split only when it improves clarity without changing behavior; preserve external contracts, security rules, side effects, and existing tests. Keep refactoring within approved task scope.

**Future agent review questions:** Can a new developer locate the relevant logic quickly? Does this file have more than one unrelated reason to change? Does splitting or combining it demonstrably simplify understanding and tests? If not, keep the straightforward arrangement.

## 5. TypeScript standards

- Use **TypeScript strict mode** across web, API, tests, and shared packages. Typecheck must pass without suppressing failures.
- Prefer inferred types for obvious local values; explicitly type exported/public boundaries and complex domain contracts when clarity benefits. Use meaningful `type`/`interface` declarations, not unnecessary wrappers.
- **Do not use `any` to evade typechecking.** Use `unknown` at untrusted boundaries and narrow/validate it before use. Any unavoidable third-party interop exception requires a local comment, bounded scope, and a follow-up issue/review.
- Do not use unchecked `as` casts, `as unknown as ...`, non-null assertions (`!`), `@ts-ignore`, or `@ts-nocheck` as shortcuts. A justified exception requires evidence and review; prefer runtime validation or better type design.
- Prefer **discriminated unions** for finite workflow states where that makes invalid states harder to represent. Do not use strings or magic numbers scattered through the code when a named domain constant/type suffices.
- Keep nullability and optional fields honest. Do not use optional chaining/defaults to conceal missing required data.
- Validate external input at runtime: client forms with Zod; NestJS requests using approved DTO validation; verify authorization and business eligibility independently. **TypeScript is not runtime validation.**
- Use **generated OpenAPI types** at the frontend boundary and Prisma-generated types only within appropriate backend layers; do not manually duplicate generated shapes or leak persistence types into UI logic unnecessarily.
- Represent leave-credit arithmetic in the **approved fixed-precision ledger units** (1/10,000 day), not binary floating-point calculations; format for display separately.
- Prefer `async`/`await`, explicit return behavior, and clear error propagation over complicated promise chains. Type-only imports where appropriate.
- ESLint + Prettier are the shared style authorities. Avoid personal formatting debates and blanket lint-rule disabling; changes to global rules require review.

## 6. Pragmatic architecture for this project

| Need | Prefer now | Avoid unless separately approved |
| --- | --- | --- |
| Backend | Feature-focused NestJS modules; thin controllers; cohesive services | Microservices, generic orchestration framework |
| Database | Prisma queries/transactions within feature-owned code | Abstract repository framework for every table |
| UI state | React state/Context for local state; TanStack Query for server state | Redux/Zustand or hand-built global stores without a demonstrated need |
| HR policies | Structured Basic/Advanced forms and versioned policy records | Scripting language or universal rule-engine designer |
| Leave accrual | One small service-based calculation boundary feeding the ledger | Plugin platform / biometric integration in MVP |
| Async jobs | Approved PostgreSQL job/outbox approach with idempotency | Redis or distributed workflow engine in MVP |
| Reuse | Extract logic when real duplication or domain ownership warrants it | Premature shared `packages/` or helpers for one trivial call |

**Open/Closed in practice:** adding a future attendance-based calculation should require a new accrual strategy behind a *small established interface*, not modifying the leave ledger, approval workflow, or historical grants. Build only the service-based strategy in Phase 1. Do not introduce extension frameworks before they are needed.

## 7. Testing, review, and change discipline

- For an **approved implementation task**, write relevant failing behavior tests first where practical, then the smallest implementation, then pass the checks. Document justified deviations (e.g., compatibility spikes or docs-only tasks).
- Test real business behavior, not implementation details: positive, negative, unauthorized, boundary, concurrency, duplicate/retry, and privacy cases according to the approved Test Plan.
- A change is not complete until applicable **typecheck, lint, format, tests, and contract/database checks** pass. Report commands and actual results; never claim a test ran when it did not.
- Review for: readability, clear ownership, redundant abstractions, unintended dependencies, incorrect types/casts, security/leave-ledger invariants, and accidental change outside scope.
- For auth/MFA, CASL permissions, document privacy, leave-credit arithmetic, migrations, and other elevated-risk areas, request independent review before merge under `AGENT_GOVERNANCE.md`.
- Agents may suggest simplification, but **must preserve observable behavior**, existing validations, error behavior, and audited history. Never silently rewrite a large area while completing a small task.
- Owner approval is mandatory before architectural deviation, new production dependency, changed business policy, external push/merge/deploy, real data use, or destructive action. A proposed refactor is not automatically authorized.

## 8. Decision test for extra complexity

Before introducing an abstraction, tool, dependency, or extra layer, the agent must be able to explain in the task/PR summary:

1. **What actual current requirement or risk justifies it?**
2. **Why wouldn't a simpler implementation achieve the same result?**
3. **What concrete change becomes safer, clearer, or easier because of it?**
4. **What tests show it works and existing behavior remains intact?**

If there are no convincing answers, use the simpler design. There are **no arbitrary limits** for file length, number of classes, or test coverage percentages: measure understandable responsibilities, business correctness, and maintainability.

## 9. Agent change-review checklist

Use this checklist when reviewing *future* code changes (no code is authorized in this planning conversation):

- [ ] Matches the approved requirement and smallest useful scope; no speculative feature.
- [ ] TypeScript strict checks pass; no unchecked `any`, unsafe assertion, or suppressed error.
- [ ] Code uses clear names and focused responsibilities; no god files, unrelated concerns bundled together, or unnecessary fragmentation.
- [ ] Uses existing approved stack and generated contracts; boundaries remain intact.
- [ ] Input validation, authorization, and private data handling are correct.
- [ ] Leave ledger, policy history, and retries remain correct when touched.
- [ ] Relevant tests/checks passed; actual evidence recorded.
- [ ] No unrelated edits, secrets, real employee data, or unapproved external actions.
- [ ] Complexity is justified in plain English, or simplified before handoff.

**Approval:** Owner-approved 2026-11-01 as binding coding governance for future Codex, Claude Code, and human implementation tasks. This **does not authorize implementation** in the planning session; task-specific approval is still required.
