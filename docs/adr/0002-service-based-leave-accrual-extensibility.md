# ADR-0002 — Service-based leave accrual with an extensible calculation boundary

**Status:** Accepted (owner decision 2026-11-01)  
**Applies to:** MyFi ESS Phase 1 monthly leave accrual  
**Related:** `docs/01-business/LEAVE_POLICY_SETUP_REVIEW.md`, `docs/02-architecture/ARCHITECTURE.md`, `docs/02-architecture/DATA_MODEL.md`

## Decision

For **monthly** paid-leave policies in Phase 1, calculate earned credits from **completed qualifying months of employment service** anchored to hire date, rather than requiring actual daily attendance or biometric/time-in-time-out logs. The first grant occurs after the first qualifying completed month. The existing option of annual upfront allocation remains available.

Follow the **Open/Closed Principle** at the *calculation* boundary only: a small calculator contract produces an exact earning amount for a specific employee, policy version, and earning period. The **existing leave-credit ledger** alone posts grants transactionally with a unique idempotency key. The MVP implements one service-based monthly calculator. If requirements change, another independently approved calculator can be added without rewriting balances, reservations, approval transitions, or historical entries.

## Why

- The MVP is leave management, not a time-and-attendance system.
- Service tenure and qualifying-service periods can be computed from HR-maintained employee/policy data.
- Avoiding an attendance dependency makes the demo smaller, understandable, and portable.
- Keeping calculations separate from ledger posting reduces the impact of a future integration.

## Constraints and explicit non-decisions

- **No Phase 1 biometric, time-clock, timesheet, attendance APIs, or generic rule/plugin engine.** Don't implement unused alternative calculators.
- Service interruptions, unpaid periods, excluded service and special absences must follow **HR-approved policy and applicable law**; do not equate every elapsed month with a qualifying month by default.
- Crediting of hire dates on the 29th–31st, midyear policy changes, and partial-month proration remain explicit follow-up decisions.
- The sample 10 VL + 10 SL allowance is demonstration data, **not confirmed company policy**.
- All calculation versions must preserve published policy identity, auditable grants and history; unique earning-period/effect keys prevent duplicate grants.

## Acceptance evidence

- An employee with a qualifying start date earns **no** monthly grant on hire day and **one** grant after the first qualifying complete month.
- Accrual works without any attendance table, clock-in record or external attendance service.
- Scheduler retry/concurrency does not duplicate the grant.
- Past grants remain unchanged on policy updates.
- Additional calculation implementations would use the existing ledger posting contract; no speculative implementation is required for MVP.
