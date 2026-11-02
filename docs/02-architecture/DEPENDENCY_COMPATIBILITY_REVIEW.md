# MyFi ESS — Dependency and Integration Compatibility Review

**Prepared:** 2026-11-01  
**Status:** Documentation verification; proposed version families, **not** an installed or tested dependency graph. This document does not approve any new package or software execution.

## Working candidate matrix (to pin and verify later)

| Package/system | Conservative candidate / documented requirement | Verification still needed |
| --- | --- | --- |
| Node.js | Choose a **supported LTS** version; Node 24 LTS is a practical starting candidate | Pin exact patch and check every host/CI environment |
| pnpm | Approved package manager: pnpm Workspaces | Pin compatible pnpm major/patch using Corepack after Node selection |
| Next.js | **16.x** (active LTS listed in official support policy) | Match React, Sentry, shadcn/Tailwind and app proxy APIs; no unsupported plugin assumptions |
| NestJS | **11.x** as conservative baseline, or **12.x** only after ESM/TS compatibility audit | Wrapper's peer deps; `@nestjs/swagger`, config, throttler and pino; choose one Nest major, not a mixed graph |
| TypeScript | Strict TS; Nest 12 CLI requires **TypeScript 6**; Prisma 7 requires TS **5.4+** | If choosing Nest 11, pin supported TS 5.x after peer audit; if Nest 12, verify TS 6 ecosystem |
| Prisma | **7.x** candidate with matching Prisma CLI/client and `@prisma/adapter-pg` | Driver, ESM and generated client output path; migration and Neon connection pool behavior |
| Better Auth | **>=1.5** required by selected Nest wrapper; latest stable at scaffold time is candidate | Pin matching `better-auth`, `@better-auth/prisma-adapter`, plugin/config API; verify adapter package vs import path |
| `@thallesp/nestjs-better-auth` | Documented current line **2.x**, requires Better Auth >=1.5 | Community package maintenance, peer dependencies, body-parser handling, guard and version pair |
| PostgreSQL | Neon demo; Testcontainers real PostgreSQL for local tests | Native DB indexes/transactions, pool limits, runtime connection details |
| R2 presign | AWS S3 SDK packages already approved | TTL, private bucket CORS, PUT overwrite behavior, immutable promotion, scanner gate |

**Important correction:** A package being documented does not establish compatibility with every other selected major. No `package.json`, lockfile, `prisma generate` or browser test exists for this planning package. No versions are represented as *passed*.

## Critical integration findings from published documentation

1. **NestJS and Better Auth:** The community wrapper documents `NestFactory.create(..., { bodyParser: false })` and re-adds body parsers for non-auth routes. If a future version changes this, use the verified installed-version instructions, not outdated recipes. Configure CSRF and trusted proxy boundaries separately. [Source](https://better-auth.com/docs/integrations/nestjs) · [Wrapper](https://github.com/ThallesP/nestjs-better-auth).
2. **Better Auth and Prisma 7:** Prisma 7 requires an adapter for PostgreSQL (`@prisma/adapter-pg`) and explicit generated-client output; Better Auth's official Prisma docs demonstrate the `@better-auth/prisma-adapter` package and `prismaAdapter` import from `better-auth/adapters/prisma`. Better Auth CLI can *generate schema*, but Prisma applies migrations. Do not confuse package installation with import paths or expect Better Auth to migrate Prisma tables automatically. [Source](https://better-auth.com/docs/adapters/prisma) · [Prisma](https://www.prisma.io/docs/orm/v7/prisma-client/setup-and-configuration/introduction).
3. **MFA:** Better Auth 2FA defaults do not mean every employee is **required** to enroll. Explicit API admission rules must reject unenrolled users and prevent enrollment-phase sessions and TOTP-incomplete sessions from accessing ESS; disable unapproved alternatives and trusted-device bypass. [Source](https://better-auth.com/docs/plugins/2fa).
4. **NestJS 12:** Official migration guide requires newer Node for CLI codegen and TypeScript 6-compatible tooling, and uses ESM-centric packages. Choosing Nest 11 until wrapper/peer compatibility is proven is a conservative planning proposal, not a frozen selection. [Source](https://docs.nestjs.com/migration-guide).
5. **Next.js / Vercel external rewrites:** A rewrite is a documented same-origin proxy mechanism; **cookie domain, SameSite, CSRF origin, forward headers and Set-Cookie must still be tested end-to-end**. No second auth server in Next.js. [Source](https://vercel.com/docs/routing/rewrites).
6. **Version support:** Check supported Node LTS and Prisma's actual floor when implementing. [Node.js release schedule](https://nodejs.org/en/about/previous-releases) · [Prisma system requirements](https://www.prisma.io/docs/orm/v7/reference/system-requirements) · [Next.js support policy](https://nextjs.org/support-policy).

## Exact pinning procedure (future authorized development session only)

- Inspect official current release notes, package peers, transitive/security advisories and hosting runtime; choose a single stable major per package, not automatically `latest`.
- Create an isolated lockfile; capture `node --version`, `pnpm --version`, the entire dependency tree and exact versions of `@nestjs/*`, Better Auth, Prisma and adapters.
- Run local compile, auth handler integration, 2FA/authorization tests, PostgreSQL migration tests, proxy browser tests, test suite and audit; record observed outcomes in `AUTH_COMPATIBILITY_REPORT.md`.
- Freeze exact versions only **after** successful checks. If wrapper can't securely enforce mandatory MFA with supported versions, stop and request owner-approved alternative rather than inventing unreviewed custom authentication.

**Recommendation:** Keep the approved technology *choices*. Treat the specific version family rows as engineering candidates, not a claim of a tested stack.
