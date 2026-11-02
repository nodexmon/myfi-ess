# MyFi ESS — Deployment Guide: Simple Owner Review

**Version:** 0.1  
**Status:** OWNER-APPROVED v0.1 deployment approach (2026-11-01). This does **not** authorize or perform deployment.  
**Test Plan v0.2:** APPROVED (2026-11-01); no tests run yet.

## What does “deployment” mean?

Deployment means **putting the finished MyFi ESS application online** so someone can open it in a browser. First, we will use **fictional employees and documents** to test that it works. We are **not** going live with real staff at this stage.

## Where would the demo run?

| Part of MyFi ESS | Service | Simple explanation |
| --- | --- | --- |
| Website | Vercel | Shows employee, supervisor and HR screens |
| System logic | Render | Checks logins, approvals, credits and permissions |
| Records | Neon PostgreSQL | Stores fictional leave requests, accounts and credit history |
| Files | Cloudflare R2 | Privately stores sample supporting documents |
| Notifications | In-app + Resend test email | Demonstrates updates without emailing real staff |

**Hosting caution:** Vercel's free Hobby plan allows personal/noncommercial use only, and Render's free service is for preview/testing—not production. Before putting a company-related demo online, we'll check that our chosen account plans permit it. Render Free may take time to wake after inactivity.

## What must be true before a demo?

- [ ] Software has been implemented (not done yet).
- [ ] All critical tests from the approved Test Plan pass with actual evidence (not done yet).
- [ ] All demo accounts and attachments are fictional.
- [ ] Employee → Supervisor → HR approvals work without self-approval.
- [ ] MFA is required; leave credits cannot be duplicated.
- [ ] Only employee and authorized HR can open supporting files.
- [ ] No real staff email is sent; all demo notifications use safe recipients.
- [ ] Owner explicitly approves any code push, merge and deployment.

## What is different for real company use?

Before real employee information is used, management/HR/IT must approve actual company policies, hosting contracts, privacy protections, backups, document malware scanning, email setup and incident procedures. That will be a **separate decision**.

## What is *not* decided yet?

The exact service plans/costs, application package versions, email domain, scheduled-job provider, upload limits, and various HR policy details. The detailed API contract remains under review.

**Approval question:** Does this two-stage approach—safe fictional-data demo first, separately approved company pilot later—match your deployment expectations?
