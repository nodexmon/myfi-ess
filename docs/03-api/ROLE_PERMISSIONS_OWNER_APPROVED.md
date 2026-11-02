# MyFi ESS — Role Permissions (Plain-English Owner Approval)

**Status:** APPROVED business-level permissions (2026-11-01)  
**Scope:** Phase 1 leave-management MVP. The detailed REST API contract remains proposed.

## What each person can do

| Employee | Assigned Supervisor | Authorized HR |
| --- | --- | --- |
| View own profile, leave balance, request history, and notifications | View leave requests assigned to them for review | Manage authorized employee records and supervisor assignments |
| File requests with supporting documents | View relevant dates and staffing overlaps | Configure leave types and policies using structured forms |
| Correct requests returned by HR | Approve with optional remarks or reject with mandatory remarks | Verify leave eligibility, credits, and documents |
| Withdraw pending requests without approval (optional reason) | View authorized approval history | Final-approve (optional remarks), reject (mandatory remarks), or return for correction (mandatory remarks) |
| Request HR-reviewed cancellation of approved leave | View team absence summaries as authorized | Decide cancellation requests and view scoped audit/credit history |
| View approved department absence calendar | Cannot finalize the HR stage or self-approve | View company-wide approved absence calendar as authorized |

## Shared rules

- Supervisor approval comes before independent HR finalization. Assigned approvers must never approve their own leave or perform both independent review stages on the same request.
- A supervisor or HR rejection is final for that request, with mandatory remarks and release of reserved credits. Employees may file a new request instead.
- HR can return fixable requests for correction. Material date/duration/type changes require renewed supervisor approval; minor fixes return directly to HR.
- Employees can withdraw pending or correction-returned requests; this releases reserved credits and preserves an audit record.
- After final approval, employees must request HR-reviewed cancellation rather than withdraw.
- Department calendars and team notices show only approved absence names/dates, never leave reasons, remarks, or supporting files.
- All access requires an authenticated, MFA-complete session and server-side record-scoped authorization. HR policies and actual medical-document access rights remain subject to the approved security design and further owner review.

## Approval boundary

The owner approved this Employee–Supervisor–HR responsibility division in plain English. It is **not** approval of all route paths, request schemas, or technical implementation choices in `API_SPECS.md`. Sensitive supporting-document access for supervisors has **not** yet been explicitly decided.
