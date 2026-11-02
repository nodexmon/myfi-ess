# MyFi ESS — API Review in Plain English

**Version:** 0.2  
**Status:** REVIEW GUIDE ONLY — simplified HR policy UX and first-full-month accrual rule approved; technical API remains unapproved.

You don't need to approve route names, database field names, JSON or error codes one by one. The technical `API_SPECS.md` is a developer reference. The main business behaviors you are approving are:

| Feature | What should happen |
| --- | --- |
| Employee accounts | HR invites staff; employees use secure login and mandatory authenticator MFA |
| Leave policy | HR uses Basic Settings (Advanced collapsed), reads a plain-English policy preview, then publishes a version with an effective date |
| Monthly accrual | For hire-date-anchored monthly policies, credits begin only after the first completed month of service, not on the hire date |
| Employee filing | The system calculates working days and checks eligibility and credits |
| Approval | Supervisor decides, then HR verifies and finalizes; reviewers never approve their own requests |
| Remarks | Always shown; optional when approving, required for rejection or return |
| Credit balances | Pending paid leave reserves credits; HR approval deducts them; rejection/expiry releases reservations |
| Documents | Required PDF/JPG/PNG can be attached before leave submission, stored privately and checked before use |
| Notifications/calendar | Approved leave appears on the relevant team calendar and triggers appropriate private notices |
| Historical records | Later HR policy changes do not silently rewrite previous leave applications or credit transactions |

## Why the technical document looks complicated

It specifies how NestJS enforces these rules, how frontend code sends data and how repeated clicks or simultaneous requests avoid causing duplicate actions. Those details matter to the implementation team but should not clutter HR's policy setup.

## Review sequence

1. Review the simple HR policy setup: `../01-business/LEAVE_POLICY_SETUP_REVIEW.md`.
2. Confirm that the essential employee/HR actions above match your expectations.
3. Once those are clear, approve the technical contract *as an implementation of those behaviors*, subject to the agreed scope and compatibility checks.

**Not approved yet:** Technical API spec as a whole; the separate data-model amendment for pre-submit document-upload intents. The 10 VL/10 SL annual allowance is still a provisional demo example.
