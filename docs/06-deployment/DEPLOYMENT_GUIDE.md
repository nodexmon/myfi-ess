# MyFi ESS — Deployment Guide

**Version:** 0.1  
**Status:** APPROVED — deployment approach baseline (owner-approved 2026-11-01); **no deployment authorized**  
**Prepared:** 2026-11-01  
**Stage:** Documentation-only project; application, infrastructure and service accounts have **not** been created by this guide.  
**Approved upstream baselines:** Tech Stack v1.0, Architecture v1.1, Security v0.1, BRD v0.1, Data Model v0.1, Business API Checklist v1.0 and Test Plan v0.2.  
**Not approved:** Detailed technical `API_SPECS.md`; exact library versions; infrastructure provisioning, remote deployment, and any production or real-data use.

## 1. What this guide is for

The goal is to deploy a private or limited-access **fictional-data technical demonstration** of the leave-management MVP, then demonstrate how the same code can eventually be deployed to an approved company server or compliant commercial platform. This is **not** approval to process employee records or send company emails.

Two distinct routes:

1. **Development / technical demo:** Developer-owned fictional records, non-sensitive test documents and test-only email, with the selected Vercel/Render/Neon/R2 providers **only when their terms permit the particular use**.
2. **Company pilot / live use:** A separate organizational authorization, HR policy validation, privacy/security assessment, adequate service levels, supported paid or on-premise infrastructure, incident handling and tested backups. Never treat a passing technical demo as a company pilot.

> **Compliance stop:** Vercel Hobby is for personal/non-commercial use. Do not host a company-sponsored or other commercial demonstration under a plan whose terms prohibit it. Render Free is a preview/testing plan and explicitly unsuitable for production. Choose a permitted plan or environment before that use. Provider limits, terms and prices must be re-checked at deployment time; they can change.

## 2. Deployment architecture and trust boundaries

```text
Employee browser ──HTTPS──> Next.js (Vercel, compatible plan)
   │                         │
   │                         └── same-origin /api and /auth proxy ──HTTPS──> NestJS API (Render)
   │                                                                     │
   │                                                       Prisma / Better Auth / CASL
   │                                                                     │
   │                                                            PostgreSQL (Neon)
   │
   └── temporary, scoped R2 presigned PUT ──HTTPS──> private R2 quarantine object
                                                           │
                            NestJS validates + finalizes object and records metadata

External scheduler ── authenticated endpoint ──> NestJS PostgreSQL-backed jobs/outbox
NestJS outbox ──> Resend (test-only in demo) / in-app notifications
```

- **Trust boundary:** NestJS always verifies session/MFA, record ownership, role/CASL permissions and business rules. The Vercel proxy is not an authorization substitute. A direct request to Render must never bypass these checks.
- **Cookies and proxy:** Browser sees the first-party Next.js origin. Verify all `Set-Cookie`/`Cookie` forwarding, secure/HttpOnly/SameSite semantics, logout, MFA onboarding/recovery, origin/CSRF protections and response caching. Do not simply rewrite untrusted host headers or forward all headers blindly.
- **R2:** Only short-lived, narrowly scoped presigned operations; treat URLs as bearer credentials. Restrict R2 CORS to the permitted frontend origin and necessary methods/headers; never make the supporting-document bucket public. An uploaded object must remain unavailable until server verification and, when needed, scanning; never disclose name, metadata, URL, content or previews to supervisor-only roles.
- **Data and logging:** Database and R2 hold state; hosts' local filesystems hold no durable employee data. Log correlation IDs and operational outcomes, not sensitive reasons, documents, sessions, TOTP, recovery codes or credentials.

## 3. Environment model

| Environment | Who / what | Representative hosting | Rules |
| --- | --- | --- | --- |
| Local development | Developer and tests | Docker PostgreSQL, Next.js, NestJS; mocked email, optional isolated R2 test | Fictional data only; no deployed service required |
| Remote technical demo | Owner-approved, limited access | Next.js/Vercel (eligible plan), NestJS/Render (test tier), Neon test DB, R2 private test bucket | Fictional data, demo MFA, hard budget limit, feature flags disabling real email, scheduled-job observability |
| Company pilot | Authorized company users | Terms-compliant commercial or managed/on-prem services | New approval; backups/restore, scanning, privacy, policy, retention, email domain, DR and security checks |
| Production | Real company operations | Company-selected infrastructure | Separate operations runbook, SLA and written go-live approval |

No cross-environment reuse of sessions, MFA secrets, credentials, databases, buckets or email keys. Never copy real employee data into development.

## 4. Prerequisites before deploying a demo

- [ ] Approved implementation slice exists and successfully builds; owner approves any push/merge/deployment step independently.
- [ ] Complete a compatibility spike for Better Auth + NestJS Express adapter + Prisma + Next.js same-origin proxy. Pin exact Node LTS, pnpm and library versions plus `pnpm-lock.yaml` after checks; don't guess based on examples.
- [ ] GitHub repository is private; secrets and `.env*` files are not committed. CI verifies lint, strict types, tests, migrations, OpenAPI contract, and relevant integration/browser smoke cases.
- [ ] Demo uses fictional employees and safe sample files; no real medical or personal information. Configure all email to mock/test recipients only.
- [ ] Accounts exist with explicit permission for the chosen use; provider plans and current pricing reviewed. Confirm Vercel's personal noncommercial restriction and Render free limitations.
- [ ] Create distinct Neon database/project, R2 private bucket and Resend test key. Restrict roles/keys to least privilege; record owner and secret rotation process.
- [ ] Define allowed frontend origin, backend URL, callback origins and email links. Use HTTPS only outside local development.
- [ ] Create a testable recovery plan for database and object data. Free-tier or preview retention is not a replacement for verified backups.

### Configuration categories (names illustrative; actual app config must be defined in the implementation plan)

| Location | Required configuration category | Cautions |
| --- | --- | --- |
| Next.js | API upstream origin, public website URL, server-side proxy config | Upstream credentials must stay server-only; no private key in `NEXT_PUBLIC_*` |
| NestJS | `DATABASE_URL`, auth base/trusted origins and secret, MFA secret protection key, allowed origins | Validate on startup, rotate safely; only trusted proxy headers |
| NestJS / R2 | R2 account endpoint, scoped bucket, access keys, signed URL expiry, content rules | Keys never sent to browser; private bucket; region/endpoint configuration verified |
| NestJS / Resend | Test API key, sender, allowed recipients, outbound email disable flag | No arbitrary recipient sends until a domain is verified and use is approved |
| Jobs | Scheduler secret/auth, job dedup keys, timezone and processing windows | Never expose unauthenticated scheduled endpoints |
| Both | Monitoring DSN and environment name | Scrub PII/health info; disable sensitive session recording |

These are *categories*, not a ready-to-run `.env` file. Choose exact variable names in a separately approved implementation plan; do not fill in secrets or invent live URLs.

## 5. Recommended deployment sequence (only after development is implemented)

### A. Local and CI verification

1. Build and run Next.js/NestJS locally; use PostgreSQL via Docker for integration tests. Initialize a **fictional** employee, supervisor and HR account and require TOTP enrollment for all users.
2. Exercise all P0 tests from approved Test Plan v0.2 (including TP-09A service-based credits and TP-29 concurrency). P1 functionality is required for full Phase 1 acceptance.
3. Verify module boundaries and direct Render API authorization tests. Test session cookies in a real browser through the proxy, including MFA/logout.
4. Record observed results. Do not mark tests green unless executed. Freeze versions after the compatibility spike.

### B. Create test infrastructure

1. Configure a dedicated Neon PostgreSQL demo database. Confirm network/TLS, pooled runtime connectivity vs migration connection details, migration backups and permitted restore strategy. **No destructive schema reset against shared or live data.**
2. Configure a private R2 bucket and a browser CORS rule permitting only the deployed Next.js origin. Configure quarantine objects, upload size/content checks, time-limited signed URLs and a cleanup job for abandoned upload intents.
3. Configure Resend in test mode or mock. The test sender/domain cannot be used to email arbitrary real employees; any company notification system needs a verified, company-approved sender.
4. Configure restricted Sentry/Pino error reporting with sensitive-data scrubbing.
5. Provision a separate external HTTPS scheduler **only after** it has an authenticated idempotent endpoint. Cron provider remains unselected; a managed paid scheduler can be considered with later approval.

### C. Deploy NestJS API on Render

1. Connect the approved repository/branch only after owner-authorized push/merge. Build from pnpm workspace root as required by the actual monorepo; use verified installation/build/start commands from implemented `package.json` scripts, **not guessed commands**.
2. Set the exact environment variables in Render's secret store. Bind the app to the assigned `PORT` on the required interface. Use explicit allowed hosts/origins and HTTPS.
3. Apply reviewed Prisma migrations in a controlled, logged predeployment step with backup or recovery point and explicit permission; never run schema reset or an unreviewed migration automatically.
4. Expose a shallow `/health` endpoint with no sensitive details, a separate readiness/DB probe for operations, and no public admin/debug functions.
5. Confirm initial login, TOTP enrollment and protected API tests; verify access is denied when someone calls the Render URL directly without the right authenticated/authorized context.
6. **Render Free caveat:** service may sleep after ~15 minutes without traffic and may take roughly a minute to wake; local filesystem is ephemeral. Scheduler must tolerate wake delays, retries, outages and duplicate requests; none of these guarantees make the free tier production-ready.

### D. Deploy Next.js on Vercel (only a permitted plan/use)

1. Configure the `apps/web` project and pnpm-workspace build in accordance with the implemented repo's lockfile; deploy from an owner-authorized branch.
2. Set the server-only upstream API origin and public site URL. Route same-origin `/api` and Better Auth requests to NestJS while preserving only necessary cookie/response headers. Disable inadvertent caching of authenticated responses.
3. Use exact deployed origin as a trusted authentication/CSRF/CORS origin. Verify redirects and login emails use the correct origin.
4. Test MFA, session expiration, logout, signed-in API access, direct API bypass attempts, permissions, and browser console/CORS errors.
5. Confirm the direct-to-R2 document flow *does not* proxy binary uploads through Vercel Functions (which impose a 4.5 MB request/response payload limit). Check short-lived URL expiration and upload cleanup.

### E. Enable scheduler and demo notifications

1. Put job state, outbox records, idempotency keys, retries and deduplication in PostgreSQL. Jobs include qualifying-month accrual, returned-request expiration, unused upload cleanup and pending notifications.
2. Call an authenticated job endpoint with retries and bounded processing. Treat every scheduled tick as **at least once**, not exactly once; record last run, duration, failures, next due and alerts.
3. Use `Asia/Manila` for employee calendar-date/business semantics; schedule triggers may run in UTC and must be converted explicitly. Confirm month-end hire dates and year-end policies remain unresolved until HR approval.
4. Demo email remains mock or provider-approved test recipient only. In-app notifications may be exercised with fictional accounts. Do not disclose leave reasons, document metadata or remarks in team notices.

## 6. Demo launch checklist and evidence

**Required evidence before an owner-approved technical demonstration:**

- [ ] Approved Test Plan v0.2 scenarios run and logged; all P0 pass; agreed Phase 1 P1 scenarios pass for full-feature demo.
- [ ] Fictional users complete the Employee → Supervisor → HR workflow; no self-approval, mandatory MFA and correct private document access.
- [ ] Reservation, deduction and release work atomically; repeated approval/cron/withdrawal does not double-post.
- [ ] R2 private, CORS exact origin, presigned upload tokens expire; abandoned temporary files cleaned; supervisors see only attached Yes/No.
- [ ] Secure cookie proxy and direct Render API tested; no sensitive data in Sentry/logs/notifications.
- [ ] Scheduler outages/delays/retries accounted for, and job processing observable.
- [ ] Hosting use and budget authorized; demo access limited and fictional, no HR data or mass email.
- [ ] Basic monitoring, alert routing, recovery contact and teardown/rotation procedure documented.
- [ ] Owner explicitly authorizes deploy. CI passing alone never authorizes push, merge or deploy.

## 7. Failure, rollback and cleanup (demo)

- A failed frontend/backend release: use the platform's recorded previous deploy **only if** it is compatible with the current schema. A code rollback does not roll back database state; stop and escalate if schema incompatibility is possible.
- A failed migration: stop deployment, preserve logs and relevant backups, obtain authorization for a forward fix or restore; never perform an unplanned destructive revert.
- A leaked token or presigned URL: revoke/rotate affected credentials as applicable; expire access and investigate object state. Presigned URLs are bearer tokens and cannot be assumed one-time-use.
- An unreachable Render Free backend: show a clear retry message, confirm cold start versus outage, keep job retries bounded; do not falsely claim user requests or notifications were processed.
- On demo completion: deprovision test users, revoke API credentials, remove test data and buckets subject to provider retention, and record what was deleted. Do not conflate demo teardown with a formal production retention policy.

## 8. Company pilot / real-data readiness: new approval required

Do not onboard real employees or medical certificates until HR, management, IT/security and relevant privacy stakeholders authorize the system and the hosting and data arrangements. The pilot checklist must include:

- Confirm actual employee leave entitlements, qualifying service periods and statutory rights (including rules for expiry/conversion and probationary eligibility) with qualified HR/legal decision-makers; the 10 VL + 10 SL policy is a **demo assumption**.
- Determine notices, lawful processing basis, retention/deletion, employee access, HR privilege scope and audit retention; consider DPA/NPC obligations and provider data-processing arrangements.
- Select permitted commercial/managed or on-prem hosting, data location and support levels; no free-tier production shortcut.
- Verify malware detection/inspection for documents, private storage, backup encryption, tested restore, availability and disaster recovery, incident reporting and breach response.
- Provide controlled HR-assisted MFA recovery, privileged access review, secure secrets rotation, and audit evidence.
- Establish verified company email domain, approved sender identities, rate limits, mail delivery/bounce handling and access to message logs.
- Sign off staging and production data migration plans; owner must approve push/merge/deploy independently of any technical agent.

## 9. Migration later to an on-premise company server

Preserve the same **Next.js → NestJS → PostgreSQL → S3-compatible private storage** logical boundaries. Use Docker-based packaging and environment variables, not Render/Vercel-specific business logic. Before moving, prove reverse-proxy HTTPS/TLS, TOTP and cookie settings, Postgres version compatibility, object storage endpoint compatibility, scheduler reliability, firewall/backups/restore and a DR strategy. On-prem is **not** automatically safer or cheaper; operational ownership must be identified first. Plan/test a data migration with reversible cutover and a maintenance window; do not perform it without approval.

## 10. Open decisions and status

| Item | Who confirms | Status |
| --- | --- | --- |
| Whether the intended remote demo qualifies for Vercel Hobby or needs a permitted plan | Owner / provider terms | OPEN — **block incompatible use** |
| Exact package versions / Render and Neon connection settings | Compatibility spike and implementation plan | OPEN |
| Scheduler provider, cadence and retry/backoff | Approved implementation plan | OPEN |
| Upload file-size limits, temporary upload expiry and scanner for real data | Owner / IT/security | OPEN |
| Month-end hire-date anniversaries and qualifying-service interruptions | HR / owner | OPEN |
| Company-approved email domain and sending permissions | IT / HR | OPEN before real email |
| Backups, retention, incident response RTO/RPO and budget | IT / management | OPEN before pilot |
| Technical API route schemas and contract verification | API review and tests | OPEN |

**Owner approval (2026-11-01):** This guide is the approved *approach* for a fictional-data demo followed by a separately authorized company pilot. Approval does not authorize cloud account creation, code push, deployment, or processing actual employee data.

## 11. Source references (re-check when deploying)

- Vercel Hobby limits/noncommercial use: https://vercel.com/docs/plans/hobby ; terms https://vercel.com/legal/terms
- Vercel Function 4.5 MB payload limit: https://vercel.com/docs/functions/limitations
- Render free-plan sleep, ephemeral storage and non-production warning: https://render.com/docs/free
- Render environment variables and secrets: https://render.com/docs/configure-environment-variables
- Cloudflare R2 signed URLs and privacy: https://developers.cloudflare.com/r2/api/s3/presigned-urls/
- Cloudflare R2 browser CORS: https://developers.cloudflare.com/r2/buckets/cors/
- Resend test email domain: https://resend.dev/
- Local approved baselines: `../05-testing/TEST_PLAN.md`, `../04-security/SECURITY.md`, `../02-architecture/ARCHITECTURE.md`, `../02-architecture/TECH_STACK.md`.
