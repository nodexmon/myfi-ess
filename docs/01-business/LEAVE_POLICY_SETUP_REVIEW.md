# MyFi ESS — Leave Policy Setup (Plain-Language Review)

**Version:** 0.2 — approved simplified HR experience  
**Status:** APPROVED UX approach and first-month accrual rule; sample allowance and other HR policy values are provisional  
**Purpose:** Help the project owner understand exactly what an HR administrator would configure before considering the underlying API.

## What HR actually does

HR opens **Settings → Leave Policies**, selects a leave type, fills out a short form, previews the result, and publishes it with an effective date. HR does **not** write code, rules, or JSON.

### Example: Vacation Leave (illustrative, not confirmed company policy)

| What HR sees | Sample selection | Explanation |
| --- | --- | --- |
| Leave name | Vacation Leave (VL) | The name employees see |
| Paid or unpaid? | Paid | Paid leave uses credits; unpaid leave does not |
| Annual allowance | 10 days | *Provisional demo assumption only* |
| How are credits given? | Monthly accrual | Alternative: grant at the start of an eligible year/period |
| Who can use it? | Regular employees | In the demo, probationary staff may earn but not use paid credits |
| Supporting document | Not required | Alternatives: always, or only if leave lasts more than X working days |
| When can employees file? | Use the company-approved filing window | HR sets advance and/or retroactive deadlines only if required |
| Unused credits at year-end | Expire | Optional carryover, subject to a confirmed HR policy |

**Sample Sick Leave:** Separate SL policy with its own document requirement and filing window. Do not assume a medical certificate threshold without HR confirmation.  
**Sample Leave Without Pay:** Unpaid; no paid credit allowance, reservation or deduction. The existing demo assumption allows probationary employees to file LWOP.

### What the employee experiences

1. Choose **Vacation Leave**, start date and end date.
2. The system automatically shows eligible working days, available credits and any document requirement.
3. Submit the request; available credits are temporarily reserved.
4. Supervisor approves or rejects (remarks visible; optional on approval, required on rejection).
5. HR verifies and finalizes or returns/rejects (required remarks for return/rejection).

The employee does not see how accrual, policy versioning or ledger entries work internally.

## Approved HR screen structure

Use **Basic Settings** first, with optional **Advanced Settings** underneath. Users see regular language and a plain-English summary, never JSON or developer fields.

- **Basic Settings:** leave name, paid/unpaid, annual allowance, monthly-versus-annual allocation, eligible employee groups, filing window, document requirement, unused-credit carryover/expiry. Group related fields visually so the screen remains short.
- **Advanced Settings (collapsed):** hire-date anchor details, proration when supported, detailed deadline exceptions, cash conversion when permitted, and other infrequently changed rules. Do not add an unrestricted rule builder.
- **Preview before Publish:** a readable policy summary with sample employee scenarios, followed by explicit effective-date confirmation. Show consequences of policy changes; published history stays immutable.

Actions: **Save Draft → Preview Examples → Publish (effective date)**. Publishing warns HR that future changes are recorded as new versions rather than silently editing past requests.

No custom rule builder, scripting language, or manual ledger edits. A consistent company-wide work calendar is edited separately under **Work Schedule & Holidays**.

## What the technical API document means (plain English)

- **Policy draft endpoint:** save the form before it takes effect.
- **Publish endpoint:** activate the reviewed settings from an effective date.
- **Policy version:** snapshot of the rules used by each filed request, even if HR later changes settings.
- **Leave calculation endpoint:** preview how many working days and credits a request would use; does not reserve anything until submission.
- **Document upload intent:** let an employee safely attach a required PDF/JPG/PNG before submitting a brand-new leave request.

These are implementation mechanisms, **not extra buttons or technical fields HR must understand**.

## Scope proposal for Phase 1

**Show by default:** basic allowance, annual-vs-monthly grant, probationary eligibility, filing window, document requirement and expiry/carryover.  
**Keep behind advanced settings or postpone until company policy confirms:** cash conversion, manual policy exceptions, multiple complicated proration modes, flexible custom rule logic. These aren't deleted from the broader roadmap; they are not necessary to demonstrate an ordinary VL/SL/LWOP flow.

## Approved monthly accrual behavior (project-owner decision, 2026-10-31)

For **monthly accrual anchored to hire date**, do not grant credits immediately on hire. A credit is posted **after the employee completes their first full month of service**, and after each subsequent completed monthly service period where eligible. Example: hire **15 January** → first grant **15 February**. With the illustrative 10-day annual allowance, a monthly service grant represents 1/12 of the yearly amount (display approximately 0.8333 day), with exact integer-unit accounting and rounding as specified in the approved data model. This is the **approved demonstration behavior**, not confirmation of the employer's actual entitlement policy.

The following remain company-policy or technical clarifications, not implied approvals: anniversary dates for employees hired on the 29th–31st; how different policy effective dates affect a cycle; prorating incomplete months; handling the interplay of hire-date monthly accrual with year-end expiry for midyear hires. The example 10 VL/10 SL entitlement, carryover values, medical-document thresholds and employment-status eligibility are still provisional until HR confirmation.

**Scope boundary:** The owner approved the simple UI and first-completed-month rule, **not** the whole `API_SPECS.md` or the proposed pre-submit upload-intent data-model amendment. No implementation authorization is implied.
