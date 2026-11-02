# MyFi ESS — Leave Application Workflow (Approved Baseline)

**Status:** OWNER-APPROVED — plain-English business workflow baseline (2026-11-01). Technical API specification remains under review.  
**Scope:** Demonstration workflow only; leave entitlements and HR policies need company confirmation.

## The normal process

**1. Employee files** → **2. Supervisor reviews** → **3. HR finalizes** → **4. Employee/team notified**

1. **Employee:** Selects leave type and dates, enters a reason, and uploads supporting documents *before* submitting (when required). MyFi ESS calculates working days and verifies eligibility and available credits. On submission, paid-leave credits are **reserved**, not yet used.
2. **Supervisor:** Reviews staffing/overlaps and **approves** (remarks optional) or **rejects** (remarks required). A supervisor's own request goes to a designated higher approver. No self-approval.
3. **HR:** Checks the employee's eligibility, credits, and supporting documents. HR either **approves/finalizes** (remarks optional), **rejects** (remarks required), or **returns for correction** (instructions/remarks required).
4. **After final HR approval:** Reserved paid credits become **used** exactly once. The employee receives a notification. The team calendar and notices show only **employee name and absence dates** — never the leave reason or medical details.

## Other possible outcomes

| Situation | What happens next |
| --- | --- |
| **Supervisor or HR rejects** | Rejection is **final** for that application. Required remark; reserved credits released; read-only history kept. Employee may file a **new** application. |
| **HR returns for correction** | Credits **stay reserved** until the request is resolved. Employee receives instructions and a deadline that HR may extend. Missing attachment/ordinary remark goes back to **HR**; dates, duration, or leave type changes go back to **Supervisor**, then HR. If the deadline expires, the request expires and credits are released. |
| **Employee withdraws before final HR approval** | Employee can withdraw a pending or returned request **without approval**. Withdrawal reason is **optional**; reservations are released; history is kept. |
| **Employee cancels after final HR approval** | Employee requests **cancellation**, with a reason. **HR must decide.** If approved, eligible credits are restored, the team calendar/notifications are updated, and history is kept. If rejected, leave remains approved and HR must provide a remark. |
| **Document uploaded but leave never submitted** | The private temporary upload is automatically deleted after a configured period. |

**Always:** Remarks are visible on every reviewer action; optional for approval and mandatory for rejection or return. Only the employee and authorized reviewers can see private leave details and remarks. Working-day calculations and leave allowances follow the configured HR policy (demo examples are provisional).

**Approval record:** Project owner approved this one-page workflow on 2026-11-01. Approval covers these business behaviors and does not constitute approval of the full technical `API_SPECS.md`.
