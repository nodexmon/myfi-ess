# MyFi ESS — Technical Reconciliation Tests (Proposed Supplement)

**Status:** Engineering test-design supplement, **not executed** and not a change to the owner-approved 36-test v0.2 baseline. Add to the relevant feature plans after authorization.

| ID | Future test | Expected result |
| --- | --- | --- |
| RT-01 | Withdraw-v-supervisor/HR decision run concurrently on same request | Only one legal state transition, one reservation release or finalization, never both |
| RT-02 | Two identical `Idempotency-Key` calls for submit/withdraw/finalize; then same key with altered payload | Identical call returns same result, altered request yields conflict, one ledger effect |
| RT-03 | Supervisor role retrieves attached metadata/read grant by API | Only boolean attachment indicator visible, no filename, URL, bytes, metadata or signed download grant |
| RT-04 | Owner uploads pre-submit, deletes/replaces, abandons; cleanup runs twice | Only verified final object can bind to request; attached file safe; orphan cleaned once |
| RT-05 | Monthly grant retried, year-crossing hiring date and 29–31 hire-date anchors | One grant per qualifying period; no accidental January double grant; HR-approved month-end policy applied |
| RT-06 | Password-only, unenrolled, MFA-pending, revoked and disabled sessions through proxy **and** direct backend | Access denied consistently, without trusting client assertions |
| RT-07 | Body parser disabled for auth wrapper, normal NestJS DTO JSON validation active | Both auth and business API route parsing correct with actual pinned wrapper |
| RT-08 | Prisma 7 generated auth/2FA models, migration and DB adapter | Real PostgreSQL test DB works with pinned packages; no unreviewed changes to auth tables |
| RT-09 | Finalize cancellation with expired source lot / multiple actors | No expired lot silently restored, one authorized compensation and audit record |
| RT-10 | Team calendar and notification serialization | Only employee display name and approved absence dates; no leave type, reason, reviewer remark or file details |

**Note:** These are planned tests. Not a claim of passing test results or a request to execute them now.
