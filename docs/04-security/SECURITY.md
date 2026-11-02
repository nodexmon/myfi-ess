# MyFi ESS — Security Design and Privacy Controls

**Version:** 0.1 — approved security design baseline  
**Date:** 2026-10-31  
**Status:** OWNER-APPROVED — security design baseline; controls are not yet implemented, tested or assessed for a real-data company pilot.  
**Applies to:** Phase 1 fictional-data demonstration; requirements for future company deployment are identified separately  
**Authoritative inputs:** `docs/superpowers/specs/2026-10-31-myfi-ess-design.md` (approved); `docs/01-business/BRD.md` (approved v0.1); `docs/02-architecture/ARCHITECTURE.md` (approved v1.1); `docs/02-architecture/TECH_STACK.md` (approved v1.0); `docs/governance/AGENT_GOVERNANCE.md`.

> **Security status:** This is a design proposal, **not evidence that controls are implemented, tested, or compliant**. No real company or employee records (including medical files) are authorized for the publicly accessible MVP. Actual policies, lawful grounds, data retention, access authority, incident procedures and vendor arrangements require company HR/legal/IT/privacy review before processing real data.

## 1. Objectives, scope, and trust boundaries

**Protect:** employee identity and personal email, credentials/TOTP secrets/recovery codes, department and supervisor assignments, sensitive leave reasons and medical attachments, approval decisions, balances and ledger records, notification recipients, audit logs, system credentials, and availability of the application.

**Architecture:** a Next.js browser frontend on Vercel makes same-origin `/api/*` requests via a proxy to NestJS on Render. NestJS owns authorization, workflows, validation, and audit. Better Auth + PostgreSQL/Prisma own user authentication and sessions. CASL controls domain actions with supplemental no-self-approval and hierarchy checks. The browser uploads permitted sample attachments directly to private Cloudflare R2 using temporary upload grants; NestJS authorizes and verifies finalization. Neon hosts demo PostgreSQL; Resend uses a test sender; an external scheduler triggers bounded PostgreSQL-backed jobs.

**Trust boundaries and checks:**

1. **Browser → Next.js proxy:** treat client input/cookies as untrusted, prohibit arbitrary proxy destinations and prevent header spoofing.
2. **Next.js proxy → NestJS:** validate session, origin, CSRF protection, ability, resource ownership and business invariants **at the NestJS API**, even if someone contacts Render directly.
3. **NestJS → database:** use parameterized ORM queries, transaction boundaries, least-privileged database role, constraints and audit trail.
4. **Browser → R2:** presigned PUT URL is a temporary bearer credential and may be reused until it expires. A successful PUT does **not** mean the file is verified or safe.
5. **NestJS → external providers:** redact private data; use least-privilege credentials and safe retries. Do not trust email or monitoring vendors as private document repositories.
6. **Agents/CI → repository and deployments:** development agents receive demo-only credentials; human approval is required for pushes, merges, deployments, destructive migrations and access to real data.

## 2. Threat model (initial)

| ID | Threat / failure | Main defenses | Required negative test |
| --- | --- | --- | --- |
| TH-01 | Public registration or invitation reuse activates unauthorized accounts | Closed registration, HR-provisioned invitations, single-use expiring tokens, verified email | Anonymous signup and replayed invitation denied |
| TH-02 | Password-only or recovery-state user accesses ESS | Server-side MFA state gate for every business route; limited enrollment/recovery routes | Direct Render call before MFA denied |
| TH-03 | Supervisor or HR user accesses another employee's private request/document | CASL + ownership/department scope + role separation | Unrelated-user requests return deny, without revealing record details |
| TH-04 | Multi-role user approves own leave or own MFA reset | Actor/subject inequality, independent approver checks, dual-control on privileged resets | Self-approval and privileged self-reset denied |
| TH-05 | Concurrent requests overspend or release credits twice | SQL atomicity, invariant checks, unique idempotency keys, append-only ledger | Parallel requests cannot oversubscribe; retries are exactly-once for state effects |
| TH-06 | Stolen/reused R2 signed URL overwrites verified file | Quarantine-only upload keys, expiry, finalize once, immutable final object | Upload to quarantine after finalization cannot change READY bytes |
| TH-07 | Malicious file or medical data disclosure | Type/size/content verification, malware scanning before real use, private storage, scoped reads | Invalid file rejected; unscanned/unauthorized download denied |
| TH-08 | Proxy/cookie/CSRF misconfiguration enables session abuse | Same-origin proxy, secure cookie attributes, trusted origins, CSRF tests, strict CORS | Cross-site state-changing attempt denied |
| TH-09 | Logs, errors, or email disclose sensitive leave details | Structured redaction, minimal team notices, disabled replay, safe error responses | Tests inspect payloads and sample logs for forbidden fields |
| TH-10 | Repeated cron calls duplicate accruals/notices | Authenticated bounded scheduler, outbox, leases, retries, DB uniqueness | Repeat/concurrent runs are safe |
| TH-11 | Leaked agent/dev secrets or compromised CI runner | No real credentials, scoped secrets, locked branches, secret scan, manual release | CI secret scan; unprivileged agent cannot deploy |
| TH-12 | Lost data or compromised account cannot be recovered | Revocable sessions, backups/restore drills, access review, incident playbook | Restore drill and account revoke test before production |

## 3. Data classification and privacy-by-design

| Class | Examples | Access/display rule |
| --- | --- | --- |
| Highly restricted | Password hashes; encrypted TOTP secrets; recovery code verifiers; session tokens; API/DB/R2 keys | Authentication subsystem only, never expose in application responses, logs, exports or diagnostics |
| Sensitive HR | Health-related reasons, medical certificates, other protected special-leave supporting evidence | Employee concerned and **specifically authorized** HR verifier; a supervisor sees only what policy and approved authority require, not blanket health-document access |
| Confidential HR | Employee personal email, hire date, employment status, private application details, leave ledger, exception history | Employee for own records; designated approvers for assigned review; HR on least privilege |
| Limited departmental | Approved absence **employee name and dates only** | Same department (or authorized supervisor/HR) through team calendar and notices |
| Operational metadata | Request ID, approved policy version, event category, masked user reference | Authorized operators and audited system services; no secrets or sensitive free-text fields |

- **Data minimization:** collect only necessary information, restrict free-text collection, and never include leave reason, medical diagnosis, leave type or documents in team notices. Team-visible summary is separate from full request DTO.
- **Legal basis:** processing employee and especially health-related information requires organization-specific legal basis/purpose analysis, notices and appropriate safeguards. Do **not** assume an employee's consent is universally sufficient or required; HR/legal/DPO must determine lawful basis.
- **Privacy rights:** design controlled processes for access/correction, retention, deletion and legal holds without rewriting append-only financial/audit history. Production procedures to be approved by the company.
- **Demo rule:** seed entirely fictional employee identities and non-real sample attachments; no personal mail addresses except an explicitly authorized developer's test inbox.
- **Regulatory review:** Philippine Data Privacy Act of 2012 and IRR; NPC Circular 2023-06 (private-sector personal-data security), and applicable later NPC guidance. Confirm organization controller/processor roles, vendor data transfers, DPO/PIA/privacy program duties before live processing.

## 4. Identity, invitations, and authentication

### 4.1 Identity lifecycle

- Better Auth manages password-based sessions; NestJS owns employee lifecycle and HR authority. Disable public self-registration; any account creation/provisioning API must require authenticated, MFA-complete HR administrator privilege. Audit all invitations and role assignment changes.
- HR creates employee profile or validated Excel import, assigns one active identity, department, employment status, supervisor/higher approver and backup approver. Prevent duplicate account activation by employee ID/email (case normalization policy to be set in DATA_MODEL).
- Invitation tokens are cryptographically random, one-time, time-limited, stored as one-way verifiers (not plaintext), tied to a single intended employee and verified email. Reissue invalidates outstanding tokens. Avoid sending plaintext passwords.
- Account activation establishes employee-owned credentials and mandatory TOTP enrollment before any ordinary ESS access. Password reset/email change must not bypass verified identity/MFA and should revoke or revalidate active sessions according to the approved policy.
- Disable/deactivate employee access promptly: revoke sessions, pending invites, sensitive grants and privileges; retain legally required records according to retention policy.

### 4.2 Passwords, MFA and recovery

- Configure Argon2id through Better Auth's supported password hashing hooks after compatibility test; passwords never stored or logged in plaintext. Strength and breach-screening policy to be finalized with HR/IT, not assumed from example values.
- Require **TOTP for every user**, including HR and supervisors, on each new sign-in. Do not offer email OTP as a substitute for required TOTP. Disable trusted-device bypass. MFA enrollment must verify a code; do not set `skipVerificationOnEnable: true`.
- Explicit server-side session **state** distinguishes invited, password-authenticated but MFA-incomplete, MFA-enrolling, authenticated and recovery-only users. Only minimal enrollment/recovery endpoints are usable in intermediate states. Reject normal API calls against Render even when a browser view hides them.
- Single-use recovery codes are displayed only during enrollment/regeneration, with secure storage using supported one-way verifiers. Verify Better Auth's configured storage behavior rather than assuming it hashes by default. Disallow replay and rate-limit attempts.
- HR-assisted MFA reset requires recorded identity verification and **independent authorization**, especially for privileged users; cannot be self-approved. Reset revokes current sessions/recovery material and requires fresh enrollment. Email possession alone is not identity proof.
- Step-up verification for sensitive operations such as HR-admin role changes, MFA reset approval, and credential changes is recommended; define exact mechanism and expiration at implementation design. No permanent override/bypass accounts.

### 4.3 Session and proxy security

- Prefer database-backed revocable sessions; use `Secure`, `HttpOnly` and appropriate `SameSite` cookie settings with host-only domain where possible. Do not store session credentials in `localStorage`.
- For Vercel→Render proxy: test cookie creation, refresh, rotation and revocation, request origin and host handling, forwarding of `Set-Cookie`, and absence of session leakage through cache/CDN. Proxy must not forward attacker-controlled `Host`/`X-Forwarded-*` values blindly.
- Preserve Better Auth origin/CSRF checks; strict allowlists for trusted web origins, and no permissive `Access-Control-Allow-Origin: *` with credentials. State-changing operations require CSRF/origin protection independent of UI.
- Add Helmet/CSP based on actual Next.js/shadcn/Sentry requirements; avoid unsafe-inline allowances without documented reason. Use secure no-store caching for authenticated/sensitive API responses.
- Default session duration/idle timeout, device list and privileged reauthentication window remain **proposals to settle before production**, not silent implementation assumptions.

## 5. Authorization, roles, and separation of duties

**CASL is not the only check.** Every protected NestJS route must enforce authenticated MFA-complete session **and** CASL ability/record predicates; domain services additionally enforce workflow state, assigned approver, no-self-approval, and independent finalization. Do not rely solely on display visibility, role strings in JWT/cookies, or client-provided employee IDs.

Approved role-scoping principles; implementation detail must also follow the later approved all-document privacy decision (A = allowed when explicitly scoped and eligibility verified; P = allowed only with separate HR privilege; — = not permitted):

| Action | Employee | Assigned supervisor / higher approver | HR verifier | HR admin |
| --- | :---: | :---: | :---: | :---: |
| View own request and balance | A | A | A | A |
| File/correct/withdraw **own pending** request; request HR-reviewed approved-leave cancellation | A | A | A | A |
| View assigned request details needed for approval | — | A | A | A |
| Approve/reject assigned request | — | A | — | — |
| HR finalize/reject/return assigned request | — | — | A | A |
| Open/download private supporting document (record scoped) | Own | — (presence indicator only) | A | A |
| View approved department absence names/dates | A | A | A | A |
| Manage leave types/policy versions | — | — | — | P |
| Register/import employee and modify roles | — | — | — | P |
| Initiate MFA recovery for another user | — | — | P | P |
| Independently authorize privileged user's MFA reset | — | — | Separate authorized operator | Separate authorized operator |
| Read audit events (scoped) | Own events | Team decisions | A | P |

**Matrix interpretation:** Users with multiple roles receive the **union of applicable permissions minus explicit prohibitions**; self-approval and self-finalization remain forbidden. HR verifier/admin cannot act as both the recommending supervisor and final verifier for the same request. A technical deployment administrator has **no implied medical-document viewing privileges**. For HR staff and supervisors filing their own leave, route to an assigned higher-level approver and separate HR verifier. Supervisor-only document access is **prohibited for all leave attachments** by the later approved business privacy rule; production HR/privacy review must verify enforcement and the precise authorized HR scopes.

**Resource checks:** use actor ID from verified session, request owner ID, current department and assigned approver relation, approval-stage state, document owner/request link, policy version, and operation-specific intent. Prevent IDOR by indistinguishable unauthorized/missing responses where practical. Revalidate on every access; never trust stale React Query cache as an authorization source.

## 6. Leave integrity as a security boundary

- Submitting paid leave creates a reservation in a transaction with a balance/eligibility check; two parallel submissions cannot oversubscribe. Use database-enforced unique idempotency keys per state transition and exact decimal/integer credit units.
- Supervisor approval changes state, **not** the definitive credit deduction. HR finalization atomically converts one reservation to one debit; LWOP never alters paid credit ledger.
- HR return holds reservation until re-submission, rejection or expiry. Expiry/HR extension races resolve atomically and exactly once; expired/rejected cases release once. Approved cancellations use auditable reversal and eligibility check, not deletion.
- Material edits change type/dates/duration → revalidate credits/overlap/eligibility and require supervisor reapproval. Minor edits may return directly to HR. No historical edits to published policy snapshots or ledger entries.
- Adjustment, cash conversion recording and HR exception actions require differentiated privileges, explicit reason, time/actor/event IDs, and duplicate prevention. Bulk API and scheduler endpoints must use same invariant-preserving services.

## 7. Private uploads and document retrieval

1. Only an authenticated, MFA-complete user authorized for the intended request can request an upload grant. Record `actorId`, `requestId`, object key, allowlisted type, maximum size, expiry and one-time **finalization** state in PostgreSQL.
2. Generate a short-lived, PUT-only signed URL pointing to a random private **quarantine** object key. **Presigned URLs are reusable until expiry**; do not claim the URL itself is single-use. Never expose R2 permanent credentials or an unrestricted bucket listing.
3. Browser uploads directly to R2; configure only necessary CORS methods/origins. Do not allow HTML/SVG/executables; explicitly allow PDF/JPG/PNG. Extension and MIME checks are preliminary, not proof of content type.
4. On finalization, use server-side verification of actual bytes (signature and parser safety), recorded size and digest, appropriate malware scan status, and object ownership. Copy/promote to a separate immutable/protected final key or equivalent immutable snapshot so a reused quarantine PUT cannot replace READY content. Idempotent finalization prevents multiple attachment records.
5. **Only READY documents** may be downloaded, following fresh CASL and resource checks; never expose quarantine files. Short-lived download grants remain bearer credentials and must not be embedded in logs, team notifications, or long-lived caches.
6. Track view/download actions as security events with actor and document ID (not the file contents). Define expiry, orphan cleanup, failed scan purge and legally compliant retention/deletion procedures before production.
7. **Fictional demo:** allow only developer-controlled sample documents through an isolated demo bucket. A missing malware scanner must be visible as `NOT_SCANNED`, not silently treated as safe. Real employee medical files are strictly prohibited until the scanning pipeline and policy have been tested and approved.

## 8. API security and input validation

- Enforce DTO validation in NestJS; reject unknown/malformed fields and unsupported enum values. Do not trust Next.js forms, generated OpenAPI clients or an untrusted reverse proxy as validation.
- Use Prisma parameterization and DB constraints; review any raw SQL used for job leases and ledger locks. Explicitly validate sorting/filtering parameters to prevent query abuse.
- Strict CORS allowlist, Better Auth CSRF/origin protection, safe redirect allowlist, Helmet and content-security policy. Avoid browser exposure of service credentials.
- Rate-limit authentication attempts, invitation redemption, recovery-code/TOTP verification, MFA reset, file grant creation, downloads, and scheduler endpoints. Per-user/account and network-based limits where suitable; limits are to be tested, not guessed.
- Return generic authentication errors and suitable access denials without leaking existence of employee records, private documents, or email accounts.
- Guard file export and import workloads against oversized or malformed data, formula injection in generated spreadsheets, and duplicate/privilege escalation through imported employee rows.

## 9. Scheduled tasks, notifications, and abuse controls

- External scheduler invokes a protected backend endpoint with a secret provided in **headers**, not URL query parameters; apply scoped credentials, rate limits, rotation and audit. Do not expose this endpoint in the browser UI.
- PostgreSQL job claims/leases, idempotency keys and transactional outbox protect leave-credit events; batch size and execution time limits must account for sleeping Render free infrastructure. Crash/retry and concurrent-trigger scenarios must be tested.
- Resend test domain cannot send to arbitrary employee emails; no real department announcement emails in demo. Avoid embedding sensitive request details in template props, outbox payloads, and provider metadata.
- Department notifications carry **approved absence name and date range only**. Login/reset emails do not contain plaintext passwords or MFA/recovery secrets; password reset/invite token URLs must be expiring and not logged.
- In-app notification endpoints filter by authenticated recipient; polling does not broaden access. Do not use raw `recipientId` input without an ownership check.

## 10. Logging, monitoring, secrets and audit

- Pino and Sentry events: allowlist safe fields; redact passwords, TOTP, backup codes, personal email where unnecessary, full leave reasons, health documents, session/cookie headers, presigned URLs, authorization headers, IP/user-agent where not needed, and raw request/response bodies by default.
- Disable Sentry Session Replay for Phase 1. Monitor failures without sending sensitive values to third parties; review data residency, contracts and retention before real rollout.
- Audit **who/when/what** for invitations, privilege changes, MFA reset request/approval, leave workflow decisions, policy publishing, credit adjustments, document access, export, scheduler runs and secrets changes. Append-only audit records are not publicly mutable; protect them from application-role tampering and define retention/archival later.
- Store API keys and encryption secrets only in authorized environment secret stores, never in source, PR comments, logs or test fixtures. Separate demo/prod keys, R2 buckets, Neon projects, Resend modes, and Sentry projects as available. Rotate on compromise.
- `AGENTS.md` controls intentions only; enforce least privilege in Vercel/Render/Neon/R2/GitHub, sandbox permissions, secret scanning, protected branches, independent review and deployment approval.

## 11. Incident response, availability, and recovery

**Demo:** capture and review suspicious authentication attempts, failed uploads, unsafe document access, unusual HR operations and job failures. Disable exposed credentials and isolate affected demo assets promptly. Use only fictional test data.

**Before real data:** company must designate incident coordinator and Data Protection Officer/privacy contact; define severity, evidence preservation, containment, communication and vendor escalation, restoration priorities, and periodic access reviews. Establish and test database/object backups and restoration; confirm RTO/RPO with IT. Produce a record-retention schedule and subject-rights process before import.

**Philippine breach notification:** assess incidents under applicable National Privacy Commission rules. Under the Data Privacy Act IRR, **not every incident automatically requires notification**; qualifying personal-data breaches may require notice to NPC and affected data subjects within 72 hours. The company PIC/DPO must determine applicability and lead notifications—not the coding agent. Escalate suspected breaches immediately so legal deadlines can be assessed.

## 12. Verification and acceptance criteria

| Test ID | Requirement links | Minimum acceptance evidence |
| --- | --- | --- |
| SEC-T01 | BR-001, BR-004 | Public signup denied; invite expired/replayed denied; only HR-authorized provisioned users activate |
| SEC-T02 | BR-005, BR-006 | MFA-incomplete/recovery-only/disabled sessions denied on all business endpoints, including direct Render URL |
| SEC-T03 | BR-005 | TOTP challenge works; one-time backup code cannot be replayed; trusted device bypass disabled |
| SEC-T04 | BR-005, BR-007, BR-024 | HR/admin cannot approve own privileged MFA reset; a multi-role user cannot approve/finalize own leave |
| SEC-T05 | BR-006, BR-030, BR-034 | Unauthorized employee, supervisor or technician cannot read private leave reason/medical file |
| SEC-T06 | BR-018, BR-019, BR-024 | Independent supervisor→HR review enforced; actor cannot be both supervisor and HR finalizer on same request |
| SEC-T07 | BR-027, BR-028 | Concurrent paid leave requests and duplicated job calls never overspend/double-release/double-deduct |
| SEC-T08 | BR-021, BR-022, BR-023 | Return expiry/extension and cancellation races yield valid single ledger/state transition |
| SEC-T09 | BR-034 | Upload of wrong magic bytes, unsupported file, oversize file denied; quarantined file non-downloadable |
| SEC-T10 | BR-034 | Reused signed PUT after document finalization cannot alter final READY attachment |
| SEC-T11 | BR-031 | Team calendar/emails/in-app notices do not leak leave type, reason or supporting documents |
| SEC-T12 | NFR-02, NFR-06 | Cross-origin CSRF/cookie tests and Render direct-route authorization tests pass |
| SEC-T13 | NFR-04, NFR-09 | Sensitive fields absent from representative log, Sentry, email and audit outputs |
| SEC-T14 | NFR-10 | Production release blocked until HR/legal/IT/privacy approvals and backup/scanning/incident evidence exist |

Test runners: Vitest + NestJS Testing, Supertest + Testcontainers for API/integrity, Playwright for browser/SSO-proxy/MFA states, and a staging-only R2 integration suite. CI should report executed checks and outcomes, not assume success.

## 13. Demo vs production security gates

| Control | Fictional-data online MVP | Real employee deployment (mandatory before use) |
| --- | --- | --- |
| Data | Synthetic employees and safe sample attachments | HR-authorized collection; appropriate lawful basis/notice and data minimization |
| Hosting | Only contract-permitted demo infrastructure; no availability promise | Production-suitable hosting, security review, domain/DNS and SLA |
| TOTP/MFA | Required and tested | Required, recovery formally controlled and independently reviewed |
| Access permissions | CASL/department/owner controls with negative tests | Approved named approvers, HR access matrix and privileged access reviews |
| R2 files | Isolated demo bucket, quarantined/verified sample files | Approved malware scanning, retention, privacy impact assessment, encryption and DPO signoff |
| Email | Resend test sender / developer testing recipients | Verified company mail delivery, recipient controls and abuse monitoring |
| Backup/restore | Repeatable seed/migrations; no real data | Tested PostgreSQL+object backup/restore and incident response exercises |
| Dependencies | Verified lockfile, CVE/secret scanning | Continuous vulnerability maintenance and change approvals |
| Agents | Demo-only credentials; governed tasks and local commits | No autonomous access to live employee data or unapproved infrastructure |

## 14. Open decisions requiring owner/company confirmation

**Before implementation planning:** choose a safe mechanism for testing Better Auth session/MFA/role interaction and exact treatment of first-time invitation sessions; choose session lifetime and reauthentication defaults; define how demo file verification exposes `NOT_SCANNED`; confirm security ownership of job endpoint credentials. These may be settled in a scoped integration spike and technical follow-on decisions without changing the approved product workflow.

**Before any real company rollout:** verify enforcement of the approved rule denying supervisor-only access to **all** supporting documents; confirm authorized HR record scopes, employee onboarding identity proof and privileged recovery process, lawful basis for sensitive data, PIA/data processing agreements, document retention and deletion periods, scanner and malware response workflow, incident owners, exact policy entitlements, server security and backups, production-domain/hosting terms, data residency/cross-border transfers, and rollout approval authority.

**Explicitly not implemented by this document:** software code, services, integrations, encryption configuration, security incident procedures, legal sign-off or passed security tests.

## 15. References (reviewed 2026-10-31)

- National Privacy Commission — Data Privacy Act of 2012: https://privacy.gov.ph/data-privacy-act/
- NPC — Implementing Rules and Regulations: https://privacy.gov.ph/implementing-rules-regulations-data-privacy-act-2012/
- NPC — Security of Personal Data, Circular 2023-06 overview: https://privacy.gov.ph/npc-issues-circulars-to-strengthen-personal-data-protection-in-ph/
- Better Auth — Two-Factor Authentication: https://better-auth.com/docs/plugins/2fa
- Better Auth — NestJS integration: https://better-auth.com/docs/integrations/nestjs
- Cloudflare — R2 presigned URLs: https://developers.cloudflare.com/r2/api/s3/presigned-urls/
- OWASP — File Upload Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html
- OWASP — Input Validation Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html
