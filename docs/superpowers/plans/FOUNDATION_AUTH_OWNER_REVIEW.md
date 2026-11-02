# MyFi ESS — First Implementation Plan: Simple Owner Review

**Status:** DRAFT — for owner approval, 2026-11-01  
**Detailed plan:** `2026-11-01-foundation-auth-compatibility.md`

## What would we build first?

Before building leave forms or employee accounts, we need to confirm that our chosen technologies work securely together on a developer's computer.

1. **Start the project:** Create the basic website and backend in one project, with a local test database.
2. **Test login components:** Check that Better Auth, NestJS and PostgreSQL work together using fictional accounts.
3. **Protect the system:** Verify that a password alone is not enough—an authenticator code must be completed before normal ESS access.
4. **Check the connection:** Make sure the Next.js website can safely pass login sessions to the NestJS backend, including logout and direct-backend access checks.
5. **Lock working versions:** Record the exact package versions that pass automated tests, then configure CI to repeat those checks.

## What is deliberately NOT included?

No employee leave filing, no credits, no HR settings, no real staff records, no real email delivery, no Cloudflare R2 uploads, and **no online deployment**. We will plan those as separate, reviewable stages.

## What does success look like?

- The developer can start both applications locally.
- Public sign-up is blocked.
- A test account without completed MFA cannot view protected information, even by calling the backend directly.
- A completed test MFA sign-in works through the website; logout immediately revokes the session.
- Automated tests prove the key checks, and we can see their actual results.
- Working dependency versions and integration limitations are documented.

**Future approval request (not granted):** In a separate development session, the owner may authorize *only this local foundation and authentication compatibility plan* in an approved project repository. This planning session cannot execute it. Deployment, remote pushes, production credentials and future modules still need separate approval.

**Current reality:** This document is a plan, not an implemented application. No tests have run.
