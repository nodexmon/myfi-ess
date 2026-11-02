# MyFi ESS — Test Plan: Simple Review

**Version:** 0.2  
**Status:** OWNER-APPROVED v0.2 (2026-11-01); no application tests executed.  
**Business-level API checklist:** APPROVED.

## What we need to see working

| Area | What counts as a pass |
| --- | --- |
| **Leave credits** | After an employee completes a qualifying month of service, the correct credit is earned. No attendance scanner or timekeeping records are needed. |
| **Credit accuracy** | The same monthly job cannot grant credits twice. Twelve monthly grants for a ten-day annual allowance total ten days before expiry or other adjustments. |
| **Employee requests** | Filing reserves leave credits; withdrawal or rejection releases them; HR final approval uses them exactly once. |
| **Approvals** | Only the assigned supervisor and authorized HR can act; no self-approval. Rejections require remarks; approvals allow optional remarks. |
| **Corrections & cancellation** | HR can return a request for correction; changing dates needs supervisor review again. Approved leave requires HR-reviewed cancellation. |
| **Documents** | Employees may attach files before filing; only the employee and authorized HR can open them. Supervisors see **Attached: Yes/No** only. |
| **Privacy & security** | Mandatory authenticator MFA; department calendar shows approved names/dates, not reasons or medical documents. |
| **Reliable system** | Simultaneous requests and repeated clicks never duplicate credits or approvals; notifications don't reveal private details. |
| **HR settings** | Basic and Advanced policy settings display understandable descriptions and a preview before publishing. |

## Example: monthly leave credit

- **January 15:** Maria joins the company. She does **not** earn a credit immediately.
- **February 15:** If that first month counts as qualifying service, Maria earns the first monthly portion of the annual allowance.
- **March 15:** Her second qualifying completed month earns the next portion.

The example annual allowance of ten days is **provisional**, not confirmed company policy. Exactly which absences or employment interruptions count as service still requires HR confirmation. Employees hired on the 29th–31st also need an agreed anniversary-date convention.

## How it will be checked

Agents will write automated tests of the calculations, roles, database transactions and browser workflows, then demonstrate the system with **fictional employee data**. `TEST_PLAN.md` lists **36 proposed scenarios**, including one added for service-based accrual. **No tests have been run yet**, and passing demo tests alone will not authorize use of real employee information.

**Review question:** Is this what you want MyFi ESS to prove before we call the Phase 1 demo ready?
