# MyFi ESS — Approved Business Amendments to Data Model v0.1

**Status:** Approved **business behavior**, awaiting physical Prisma-schema reconciliation and technical API signoff.  
**Prepared:** 2026-11-01  
**Authority:** Owner decisions made after logical Data Model v0.1 approval; this is a cross-reference, **not** a new database implementation or independent schema approval.

The original `DATA_MODEL.md` design was approved for fixed-precision credits, source lots, append-only ledger events, request history, and policy versions. Subsequent owner-approved business behavior must be reflected in the eventual physical model; the initial draft did not spell out every new field/state.

| Approved behavior | Required modeling consequence (technical design pending) |
| --- | --- |
| Employee can withdraw pending/returned request, reason optional | Distinct `WITHDRAWN` terminal outcome; actor/time/optional reason captured; reserved credits released once; immutable history retained |
| Supervisor or HR rejection is final; remarks mandatory | Record rejection as terminal and immutable for that request ID; preserve rejection actor and remark; one reservation release |
| Approvals have optional remarks; correction returns and rejections require remarks | Nullable approval remark and validated nonblank mandatory remark on other decisions; employee/authorized reviewer visibility only |
| Supporting attachments upload before leave submission | Short-lived employee-bound upload-intent record; attach only verified/owned files at submission; unused temporary uploads cleaned up; no draft-leave feature |
| Supporting documents private for both medical and nonmedical leave | Record-scoped authorization for employee and designated HR; supervisors see only an attached-present indicator, with no metadata or signed URL |
| Service-based accrual by completed qualifying month | Ledger event provenance includes qualifying service period, policy version, and deduplication/effect key; no daily attendance dependency |
| Major corrections reroute supervisor approval, minor corrections to HR | Revision history and decision history remain separate and traceable across transitions |

**Planning reconciliation completed:** field/table/state candidates, transactions and constraints are now in `DATA_MODEL_TECHNICAL_RECONCILIATION.md`; API coverage is in `../03-api/API_TECHNICAL_RECONCILIATION.md`. Generated Prisma models, exact library compatibility, configurable limits and code/test evidence **remain future-work gates**. No migrations or coding are authorized. **No migrations or coding are authorized by this document.**
