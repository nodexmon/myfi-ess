# MyFi ESS — Technology Stack v1.0

**Decision:** APPROVED (technology choices)  
**Date:** 2026-10-31  
**Status of versions:** **Not pinned yet**. Pin exact mutually compatible versions in `package.json` and commit `pnpm-lock.yaml` after an integration spike; this does **not** authorize new libraries.  
**Scope:** Online, fictional-data MVP for 1–50 staff; NestJS/Next.js monorepo, portable to company infrastructure.  
**Authority:** Project owner approval in planning conversation; cross-reference `docs/superpowers/specs/2026-10-31-myfi-ess-design.md` and accepted ADRs.

## 1. Stack freeze policy

- **Approved library** = permitted for its listed purpose. Not every library needs to be installed on day one; install with the feature that first needs it.
- **Version pin** = verified compatible exact direct-dependency versions and `pnpm-lock.yaml`; no guessing versions from stale examples. Record Node.js LTS, pnpm, and framework versions at first scaffold. CI must use the same versions.
- **Change control** = replacing/adding production dependencies requires an owner-approved ADR; patch/security upgrades within compatible ranges follow the approved task and CI/review rules. A major version upgrade needs explicit approval.
- Avoid duplicate tooling solving the same problem (no Redux/Zustand, Axios, BullMQ/Redis, or alternate auth system without ADR).
- All licensing, security, compatibility and hosting restrictions must be reviewed before installation/deployment.

## 2. Core application and developer tooling

| Area | Selected | Application | Notes |
| --- | --- | --- | --- |
| Language | TypeScript (strict mode) | Both | No unjustified `any` or unchecked casts |
| Frontend | Next.js + React | `apps/web` | Responsive App Router approach; version to verify |
| Backend | NestJS + Express | `apps/api` | Modular monolith, REST API |
| Database | PostgreSQL | API persistence | Neon for demo; same SQL engine on premises |
| ORM/migration | Prisma + Prisma Migrate | API | Transactions, versioned migrations; Neon adapter decision at integration test |
| Monorepo | pnpm Workspaces | Root | No Turborepo initially |
| Lint | ESLint | Both | Framework-appropriate rule sets |
| Format | Prettier | Both | Shared formatting policy |
| Local hooks | Husky + lint-staged | Root | Not a substitute for CI |
| CI | GitHub Actions | Repo | Type/lint/test/secret/security/contract gates as built |

## 3. Backend library approvals

| Concern | Package or design | Notes |
| --- | --- | --- |
| Authentication | `better-auth` | Email/password, revocable database-backed sessions, required MFA gate |
| Nest integration | `@thallesp/nestjs-better-auth` | Community maintained; **mandatory integration spike**; body parser interaction |
| Better Auth Prisma | `@better-auth/prisma-adapter` | Adapter package; align with Prisma and Better Auth versions |
| Password hashing | `argon2` | Argon2id via Better Auth supported hash customization; test upgrade behavior |
| TOTP/recovery | Better Auth two-factor plugin | Do **not** add `otplib` by default; no trusted-device bypass |
| Authorization | `@casl/ability` | CASL ability + resource predicates + mandatory no-self-approval invariant |
| Validation | `class-validator`, `class-transformer` | Server DTO validation; keep domain checks server-side |
| Config | `@nestjs/config` | Fail startup on missing/invalid secrets (validation may use existing deps) |
| API specification | `@nestjs/swagger` | Produce OpenAPI; contracts generated from runtime DTO/controller definitions |
| API throttling | `@nestjs/throttler` | Add login, recovery, uploads and job-endpoint protections |
| Security headers | `helmet` | Explicit CORS + Better Auth CSRF/origin protections |
| File multipart | Multer, Nest Express integration | Kept for eligible backend multipart routes; **normal document path is direct-to-R2** |
| Object storage | `@aws-sdk/client-s3`, `@aws-sdk/s3-request-presigner` | R2 presigned PUT/GET and later S3-compatible backend |
| Logging | `nestjs-pino` | Correlation IDs, redaction; no tokens or medical details |
| Error tracking | Sentry NestJS SDK | Scrub employee information; no session replay |
| Spreadsheet imports | `exceljs` | Validate preview, duplicate handling, explicit confirmation |
| Date arithmetic | `date-fns` | Use calendar dates/Asia-Manila semantics for leave |
| Email delivery | `resend` | Test domain only until verified domain; replaceable mail provider |
| Email templates | React Email | Reusable templates; no private leave reasons in team notices |
| Background jobs | PostgreSQL job/outbox tables + scheduled NestJS endpoint | No Redis/BullMQ; idempotent DB transactions and external scheduler |
| Backend tests | `vitest`, `@nestjs/testing`, `supertest`, `testcontainers` | Real PostgreSQL integration tests, Docker required |

**Prisma 7 caution:** The official Better Auth/Prisma documentation uses `@better-auth/prisma-adapter` with Prisma 7 and `@prisma/adapter-pg` for a Node PostgreSQL driver. The exact driver approach must be validated against Neon/Render connectivity and migrations. Do not add one blindly or mix generated client versions.

## 4. Frontend library approvals

| Concern | Selected | Usage |
| --- | --- | --- |
| Styling | Tailwind CSS | Responsive layout, design tokens |
| Components | shadcn/ui | Accessible, locally owned components |
| Icons | `lucide-react` | Shared icon set |
| Forms | `react-hook-form`, `zod`, `@hookform/resolvers` | Form state + client validation only; backend authoritative |
| Server state | TanStack Query (`@tanstack/react-query`) | Requests, invalidation, poll notifications (initial 60-second interval) |
| Admin tables | TanStack Table (`@tanstack/react-table`) | Server-backed paging, filters and sorting |
| Department calendar | FullCalendar (React + month/list plugins as needed) | Approved leave calendar and supervisor overlap views |
| Date inputs | React DayPicker via shadcn/ui | Leave date-range selection |
| API types | `openapi-typescript` | Generated from NestJS OpenAPI; do not manually duplicate DTOs |
| API transport | `openapi-fetch` | Same-origin `/api` fetch; session cookie forwarded |
| Client-only state | React state, reducer, Context | No global state library initially |
| Toast feedback | `sonner` | Ephemeral success/errors (not persistent notifications) |
| Frontend error tracking | Sentry Next.js SDK | Redact private data; replay off |
| Frontend tests | `vitest`, React Testing Library, `playwright`, `msw` | Unit/component/E2E/mock HTTP |

## 5. Infrastructure and service choices

| Purpose | MVP | Production portability |
| --- | --- | --- |
| Web deployment | Vercel Hobby **only where use is within terms** | Self-hosted Next.js/container |
| API deployment | Render free (sleeps; ephemeral) | Self-hosted NestJS/container |
| PostgreSQL | Neon Free | Company PostgreSQL |
| Attachment storage | Cloudflare R2 private bucket | Self-hosted S3-compatible object storage or verified alternative |
| Email sending | Resend testing sender; no arbitrary employee delivery | Company-verified domain/SMTP or provider |
| Scheduled triggers | External cron service such as cron-job.org, protected endpoint | Server cron/dedicated worker or scheduler |
| Telemetry | Sentry free plan, privacy filtering | Approved monitoring system |
| Local dev | Docker-based PostgreSQL/Testcontainers as needed | Same schema/migrations |

**Important:** Vercel Hobby is contractually restricted to personal/non-commercial use. Validate demonstration use before deployment, or use a compliant alternative. Render free is explicitly not production-ready. No real employee/medical data is allowed in the MVP.

## 6. Fixed integration boundaries

1. Browser calls **same-origin** Next.js `/api/*`; an external rewrite/proxy forwards to NestJS on Render. No business decisions in the proxy. Validate cookies, `Set-Cookie`, CSRF, origin trust and authorization end-to-end.
2. **Better Auth in NestJS** handles identity/session; CASL in NestJS handles domain permissions. Enforce mandatory verified TOTP, including during enrollment/recovery, **at the API**.
3. Supporting files: browser obtains a single-intent, short-lived authorized R2 upload grant (the presigned PUT itself is reusable until it expires); uploads directly to quarantine key; backend verifies metadata/content/security status before making file available. Never publish an unverified object.
4. Business state transitions and credit ledger writes are atomic Prisma/PostgreSQL operations. Every job/action has a deterministic idempotency key.
5. PostgreSQL outbox ensures that approval notification intents are committed with business changes; retries are safe. In-app data is persisted and fetched via TanStack Query.
6. Historical policy snapshots/versions remain immutable, independent of future policy edits.

## 7. Compatibility and security gates before first feature

**Required spike (owner may authorize separately):**

- [ ] Choose supported Node.js LTS, pnpm and exact stable Next.js/NestJS/Prisma/Better Auth versions; document tested version matrix.
- [ ] Verify NestJS Express + Better Auth's `bodyParser:false` requirement while safely parsing/validating *non-auth* JSON routes and uploads.
- [ ] Verify `@thallesp/nestjs-better-auth` authentication guard behavior and CASL route protection.
- [ ] Verify Better Auth Prisma adapter, plugin schema generation, migration, PostgreSQL connection and session revocation.
- [ ] Verify TOTP enrollment required before ESS access; subsequent login challenges MFA; backup codes are single-use; trusted device bypass disabled.
- [ ] Verify Vercel-to-Render proxy login/logout/session cookies, CSRF, trusted origins and `Set-Cookie` behavior in browser E2E.
- [ ] Verify presigned R2 upload CORS and upload-finalization security. Define malware scanning/quarantine policy before production.
- [ ] Verify test runner/Testcontainers compatibility, including GitHub Actions Docker.
- [ ] Resolve Vercel terms and free-tier operational limits before any accessible company-facing demo.
- [ ] Commit frozen pnpm lockfile and CI version pins **only after verification**.

## 8. References (reviewed 2026-10-31)

- Better Auth NestJS: https://better-auth.com/docs/integrations/nestjs
- Better Auth Prisma: https://better-auth.com/docs/adapters/prisma
- Better Auth 2FA: https://better-auth.com/docs/plugins/2fa
- Better Auth cookie proxy: https://better-auth.com/docs/concepts/cookies
- Cloudflare R2 presigned URLs: https://developers.cloudflare.com/r2/api/s3/presigned-urls/
- Render free service limits: https://render.com/docs/free
- Vercel Hobby terms: https://vercel.com/legal/terms
