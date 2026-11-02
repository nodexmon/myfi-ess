# MyFi ESS Foundation and Authentication Compatibility Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Status:** DRAFT v0.2 — planning-reconciled for future owner review; execution NOT authorized (2026-11-01)

**Goal:** Establish a reproducible local pnpm/Next.js/NestJS/PostgreSQL foundation and prove the chosen Better Auth + NestJS + Prisma + same-origin proxy path can enforce MFA-backed sessions securely, before building leave features.

**Architecture:** One pnpm-workspace repository with a Next.js frontend proxying same-origin API requests to a NestJS Express backend; PostgreSQL holds Better Auth state through Prisma. This is a narrow, local-only compatibility slice. It must not build leave policy, documents, approvals, real employee onboarding, scheduling or hosted infrastructure.

**Tech Stack:** Strict TypeScript, pnpm Workspaces, Next.js, NestJS Express, PostgreSQL, Prisma, Better Auth, `@thallesp/nestjs-better-auth`, Better Auth Prisma adapter and two-factor plugin; Vitest, Supertest, Playwright, Testcontainers, GitHub Actions (after verified scripts).

**Spec:** `docs/superpowers/specs/2026-10-31-myfi-ess-design.md`. **Prerequisites to read:** `TECHNICAL_READINESS.md`, `docs/02-architecture/DEPENDENCY_COMPATIBILITY_REVIEW.md`, `docs/02-architecture/DATA_MODEL_TECHNICAL_RECONCILIATION.md`, `docs/03-api/API_TECHNICAL_RECONCILIATION.md`. Additional authoritative baselines: `docs/02-architecture/{TECH_STACK,ARCHITECTURE}.md`, `docs/04-security/SECURITY.md`, `docs/05-testing/TEST_PLAN.md`, `docs/03-api/BUSINESS_API_CHECKLIST.md`, and `AGENTS.md`.

## Global Constraints

- Only fictional seed identities and isolated local/test database records. Never use real employee data, genuine medical documents, or live credentials.
- Do not add an alternate authentication system, Redis, state-management framework, dynamic policy engine, microservices, or a second Better Auth server in Next.js.
- Better Auth and all business API authorization must be enforced by NestJS even for direct requests, not only by page guards.
- Password-only sessions, unverified MFA enrollment, and MFA-incomplete sessions must not access ordinary ESS data; password hash must use approved Argon2id when supported and verified.
- No public signup, trusted-device MFA bypass, email OTP substitution, broad CORS, or session data in `localStorage`.
- Provider-supported packages are approved by the stack catalog; do not introduce other production dependencies without an owner-approved ADR. Install and pin exact compatible versions only after validating them; record the chosen Node LTS, pnpm and Prisma version matrix.
- No deployment, remote pushes, merges, paid services, infrastructure provisioning or real emails without separate owner authorization.
- Technical `API_SPECS.md` remains proposed; this slice may use local-only test probe routes but must not treat route schemas as finalized or expose probes in a production build.

## Review Focus

1. **Malformed and direct requests:** non-auth JSON endpoints remain correctly parsed/validated despite disabling Nest's default parser for Better Auth; unauthorized direct NestJS requests are denied (Task 2/3 tests).
2. **MFA bypass:** user who knows password but has not completed TOTP cannot access protected probe; neither enrollment nor recovery states confer a normal session (Task 3 tests).
3. **Identity abuse:** anonymous signup is blocked (Task 3 tests); expiring, single-use HR invitation acceptance remains explicitly deferred to the subsequent invitation/onboarding plan.
4. **Session origin and caching:** browser receives and sends first-party cookies correctly via Next.js; logout revokes access; authenticated responses are not shared-cacheable (Task 4 tests).
5. **Version/DB mismatch:** Prisma schema, migration, Better Auth plugin tables and the community NestJS integration work together under pinned versions in a real local PostgreSQL instance (Task 2/5 tests).

---

## File layout to establish

```text
pnpm-workspace.yaml
package.json                        root scripts and package pinning
pnpm-lock.yaml                      generated only after compatibility verified
tsconfig.base.json
.gitignore
apps/api/package.json
apps/api/prisma/schema.prisma
apps/api/prisma.config.ts           only if required by pinned Prisma version
apps/api/src/main.ts
apps/api/src/app.module.ts
apps/api/src/auth/auth.ts
apps/api/src/auth/auth.module.ts
apps/api/src/prisma/prisma.service.ts
apps/api/test/compatibility.integration.test.ts
apps/api/test/security.integration.test.ts
apps/web/package.json
apps/web/next.config.ts
apps/web/src/app/page.tsx           fictional-data login/test shell only
apps/web/e2e/auth-proxy.spec.ts
docs/02-architecture/AUTH_COMPATIBILITY_REPORT.md
.github/workflows/ci.yml            created only after verified commands exist
```

Test fixtures are local-only; if a test-only protected probe/controller is necessary, mount it from the test harness rather than registering a public production endpoint. Follow current verified framework file conventions if generated scaffolds differ; document any deviations.

### Task 1: Reproducible workspace and local processes

**Files:** Create root workspace/TypeScript/config files and minimal `apps/api` and `apps/web` manifests; create `.gitignore` and example env keys only. Test: `apps/api/test/compatibility.integration.test.ts` (workspace bootstrap probe) or equivalent smoke command.

**Interfaces:** Produces `pnpm --filter api` and `pnpm --filter web` package selectors, shared local origins (web `http://localhost:3000`, API `http://localhost:3001`), and a documented isolated PostgreSQL test service. All later tasks consume the same origins and package selectors.

- [ ] **Step 1 — Failing verification:** Add workspace smoke command `pnpm run smoke:workspace` that checks both packages and strict TS compilation; run it before scripts/packages exist and record failure.
- [ ] **Step 2 — Minimal implementation:** Scaffold only a Next.js shell and NestJS Express JSON `GET /health` server using approved packages; add strict TS, root scripts, environment validation for test configuration, and `.gitignore` preventing `.env`/secret commits.
- [ ] **Step 3 — Passing verification:** `pnpm run smoke:workspace` builds/types both packages; `GET /health` responds without DB credentials or private user data. Record the actually used commands and package versions; do not claim later auth behavior is verified.
- [ ] **Step 4 — Local commit:** Make one focused commit only in an owner-authorized repo, without push.

### Task 2: Better Auth, Prisma and NestJS Express integration

**Files:** `apps/api/prisma/schema.prisma`, optional `prisma.config.ts`, `apps/api/src/prisma/prisma.service.ts`, `apps/api/src/auth/auth.ts`, `apps/api/src/auth/auth.module.ts`, `apps/api/src/main.ts`, `apps/api/src/app.module.ts`, `apps/api/test/compatibility.integration.test.ts`.

**Interfaces:** Produces a single Better Auth instance, a single Prisma client lifecycle, `AuthModule.forRoot({auth})` or the pinned community integration equivalent, and a backend session-reading boundary consumed by Task 3. Any explicit JSON parser used for **non-auth** API routes must not modify the raw body handed to the auth handler.

- [ ] **Step 1 — Failing integration test:** Start PostgreSQL with Testcontainers, apply test migrations, then assert Better Auth user/session/2FA tables exist and the auth handler handles a correctly formed request; assert a non-auth JSON route still validates input after `bodyParser:false` is applied. The initial test must fail before integration exists.
- [ ] **Step 2 — Minimal implementation:** If adopting Prisma 7, use its mandatory PostgreSQL driver adapter, generated-client output path and tested ESM settings; generate and review the plugin schema, apply only to isolated test DB, configure Better Auth/Nest auth handler, and add narrowly scoped non-auth request parsing + DTO validation.
- [ ] **Step 3 — Passing integration test:** `pnpm --filter api test:integration` passes `compatibility.integration.test.ts` against real PostgreSQL. Confirm auth handler did not consume or corrupt other request bodies; document generated migration and adapter versions.
- [ ] **Step 4 — Local commit:** Commit schema/migration and integration test together; do not apply migrations to shared/live data.

### Task 3: Mandatory MFA and protected API gate

**Files:** `apps/api/src/auth/auth.ts`, `apps/api/src/auth/auth.module.ts`, `apps/api/test/security.integration.test.ts` and test-only probe controller/module.

**Interfaces:** Produces `assertEssAccess(session, employeeStatus): void` or an equivalently named server-side guard that denies access unless active, authenticated and current sign-in has completed required TOTP. It must work on direct API calls, independent of the Next.js frontend. Precise signatures must be verified against the pinned Better Auth session API and recorded in the compatibility report.

- [ ] **Step 1 — Failing test:** Against local fictional identities, assert anonymous request is denied, password-only or MFA-enrollment-incomplete sessions are denied, TOTP-completed session is accepted, and public signup is denied. Test both direct API requests and the test-only probe route.
- [ ] **Step 2 — Minimal implementation:** Configure Better Auth TOTP plugin without trusted-device bypass, guard normal ESS routes in NestJS, disable public self-registration and leave only explicitly allowed enrollment/recovery auth endpoints. Use approved password hashing and session revocation configuration. Do not implement HR invitation issuance or reset workflows in this slice.
- [ ] **Step 3 — Passing test:** `pnpm --filter api test:integration` passes `security.integration.test.ts`; audit any default guard exemptions. Record whether MFA completion can be proven securely with the selected plugin; **stop** if the library cannot expose an enforceable per-session factor state.
- [ ] **Step 4 — Local commit:** Commit auth gate and negative tests together, without bypassing a failing case.

### Task 4: Browser same-origin proxy and cookie lifecycle

**Files:** `apps/web/next.config.ts`, `apps/web/src/app/page.tsx`, `apps/web/e2e/auth-proxy.spec.ts`, local-only test fixtures and root Playwright configuration as needed.

**Interfaces:** The browser calls `http://localhost:3000/api/auth/*` and protected `/api/*`, which proxy to the NestJS backend; it never stores auth tokens in localStorage or hosts a duplicate Better Auth auth handler. Only fictional local accounts and a test TOTP seed are used.

- [ ] **Step 1 — Failing browser test:** Playwright asserts password-only cannot access the protected probe, successful TOTP flow yields a usable first-party cookie, session survives refresh, logout invalidates it, and a cross-origin state-changing request is denied. Assert private session responses include appropriate no-store behavior.
- [ ] **Step 2 — Minimal implementation:** Configure a same-origin rewrite/proxy that preserves needed `Cookie`/`Set-Cookie` semantics and origin protection without blindly trusting forwarded hosts; add the smallest test-only login page required by the browser tests.
- [ ] **Step 3 — Passing browser test:** Run `pnpm --filter web test:e2e` with local NestJS/PostgreSQL; repeat tests with direct `http://localhost:3001` REST requests and confirm they cannot bypass MFA. Capture browser evidence and cookie properties without exposing secret values.
- [ ] **Step 4 — Local commit:** Commit proxy, harness and tests; do not deploy to Vercel/Render as part of this plan.

### Task 5: Freeze compatibility matrix and CI checks

**Files:** `docs/02-architecture/AUTH_COMPATIBILITY_REPORT.md`, `docs/02-architecture/TECH_STACK.md` (version matrix only after observed success), root package scripts and `pnpm-lock.yaml`, `.github/workflows/ci.yml`.

**Interfaces:** Produces documented exact versions + reproducible script names (`smoke:workspace`, `test:integration`, `test:e2e`), evidence for auth/proxy invariants, and stop/go recommendations for the subsequent invitation/onboarding implementation plan.

- [ ] **Step 1 — Failing gate test:** Define a CI job requiring install with frozen lockfile, formatting/lint/types, Task 2/3 integration tests with Docker PostgreSQL, and Task 4 local browser tests. It must fail while scripts/integration are missing.
- [ ] **Step 2 — Minimal implementation:** Pin only versions proven compatible, commit `pnpm-lock.yaml`, add CI using the actual working commands; document the Better Auth package, Prisma adapter/driver, NestJS body parser and proxy-cookie decisions. Make no unapproved dependency substitutions.
- [ ] **Step 3 — Passing verification:** Run the exact local commands captured in CI and report evidence, or mark a failed external CI condition as blocked. Verify no secrets/real employee data and no unapproved dependencies. Never claim CI or E2E passed without observed results.
- [ ] **Step 4 — Local commit:** Commit report, lockfile and CI changes. Wait for owner authorization before any remote push, merge or deployment.

## Exclusions and handoff

- **Outside this plan:** working HR invitations, Excel imports, leave credits, approvals, files/R2, notifications, policy forms, scheduler, external cloud deployment, and company-pilot controls. They require separate approved slices.
- **Stop conditions:** missing library support for trustworthy per-session MFA completion; inability to enforce direct NestJS API authorization; cookie/proxy integration failure; unapproved production package needed; any real data/credentials; unsafe schema migration. Report blockers and request an ADR or revised plan, rather than improvising replacements.
- **Evidence required:** exact versions, commands and actual results for Task 1–5, sample sanitized test output, remaining compatibility gaps and reviewed local commit hashes (if made). No plan checkbox is marked complete until the corresponding operation is performed and verified.
- **Next suggested plan after successful review:** fictional HR invitation/employee onboarding (TP-01 through TP-04), then a separate policy/leave-credit vertical slice. Confirm the technical API contract for each affected feature before its implementation; do not silently treat the proposed entire API draft as approved.

## Planning reconciliation notes (2026-11-01)

- **Version candidate, not pin:** consider Node 24 LTS + Next 16.x + compatible Nest 11.x (or Nest 12 with TypeScript 6) + Prisma 7.x + Better Auth stable >=1.5 + @thallesp/nestjs-better-auth 2.x; choose only after dependency/peer and security audit. Use the package manager's tested exact pins, never an inferred combination.
- **MFA requirement:** enabling the Better Auth two-factor plugin does not enforce enrollment for every user; enforce an explicit per-session completed-TOTP admission policy on direct NestJS business routes, and do not allow a pre-enrollment session to become a bypass. Document any provider limitations rather than assume a session flag exists.
- **Prisma schema ownership:** generate auth and two-factor plugin tables from Better Auth, verify Prisma 7 `@prisma/adapter-pg`, and keep ESS employee tables separate.
- **Evidence report template:** record Node/pnpm, package versions/peers, working auth route list, non-auth JSON parser behavior, session and cookie attributes, all negative MFA and direct backend tests, Prisma migration results, Playwright browser proxy outcomes and blockers in `docs/02-architecture/AUTH_COMPATIBILITY_REPORT.md` **in the future authorized session only**.
- **Stop rule:** if the chosen wrapper cannot support secure mandatory TOTP, stop and request approval of an alternative. No deployment or real staff account setup under this plan.
