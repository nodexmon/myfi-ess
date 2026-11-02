# MyFi ESS — Agentic Development Governance

**Status:** Approved governance model: Balanced; initial operational contract (2026-10-31)  
**Owner:** MyFi ESS project owner  
**Applies to:** Codex, Claude Code, future coding agents, and any humans following the same delivery process.

## 1. Purpose and authority

MyFi ESS handles sensitive HR data, privileged approvals, and leave balances. Agents may accelerate delivery but are not authorized to independently change what was agreed, claim unobserved test results, or send information to external systems. This is a **proposal/demo** until the company explicitly authorizes production use.

The **owner** approves tasks, material changes, external side effects, merges and deployments. An **implementing agent** builds only the approved task. A **reviewer** (ideally a separate agent or independent context) examines behavior, security, tests and diff. A reviewer recommends; the owner decides. No agent's approval substitutes for the owner's approval or for HR/legal confirmation of employer policy.

## 2. Decision hierarchy and ownership

| Artifact | Owns | Change condition |
| --- | --- | --- |
| `docs/superpowers/specs/2026-10-31-myfi-ess-design.md` | Approved Phase 1 product behavior and boundaries | Owner approval for material change |
| `docs/01-business/BRD.md` (future) | Business objectives, stakeholders, acceptance criteria | Owner review; HR validates production policies |
| `docs/02-architecture/`, `docs/adr/` (future) | Technical boundaries and recorded design choices | Owner review for material decisions |
| `docs/04-security/SECURITY.md` (future) | Security design and threat controls | Owner review for changed controls |
| `docs/03-api/` (future) | API contract | Review for breaking changes |
| `docs/superpowers/plans/` | Exact approved implementation slices | Approval before executing each slice |
| `AGENTS.md` | Shared agent conduct and scope | Owner approval |
| `CLAUDE.md`, `.claude/`, `.codex/` | Tool-specific instructions and defaults | Owner approval for permission expansion |

A plan cannot silently alter its parent design spec. If documents disagree, stop and ask rather than choose the newest file blindly. Refer to ADRs for durable architecture decisions. Only record decisions that materially affect what is built.

## 3. Allowed autonomy and approval gates

| Agent action | Allowed without a new prompt? | Condition |
| --- | --- | --- |
| Read non-sensitive repo files, inspect diffs, research docs | Yes | Within task scope and sandbox limits |
| Edit code/docs/tests, run local checks | Yes | Must be covered by a specifically owner-approved task |
| Refactor, add tests, make small local commits | Yes | Behavior preserved or changes are in the approved task; never rewrite others' commits |
| Introduce a new production dependency | No | Owner approves purpose, license, maintenance and security impact |
| Change business rules, security policy, architecture, or scope | No | Propose change, impact, and tests; receive explicit approval |
| Change leave-credit ledger or auth/MFA code within an approved plan | Yes, with extra review | Must include negative cases and independent review before merge |
| Access real employee data, real medical files, or live credentials | No | Separate company authorization, privacy/legal review, and approved secure access required |
| Run destructive database operations, production migrations, or history rewrites | No | Explicit approval, backup/rollback plan, and environment verification |
| Remote push, PR merge, publish package/release, deploy, change cloud infrastructure | No | Explicit approval for that exact action and target |
| Disable safeguards, skip tests, bypass MFA/RBAC | Never as a workaround | Stop and report blocker; require redesign/review instead |

An "approved task" means a bounded user instruction or a Superpowers plan/phase that the owner has expressly approved. General approval of the design is **not** blanket authorization to implement every module.

## 4. Required delivery cycle

1. **Scope:** Read approved spec and relevant docs; establish intended changes, assumptions, acceptance tests, and affected data/privacy surfaces. If no execution plan is approved, plan first.
2. **Isolate:** Create/use a feature branch or worktree when supported. Do not modify unrelated work or uncommitted user changes.
3. **Test:** Add failing behavior tests before implementation, except docs-only tasks or justified exploratory spikes. Cover error and authorization cases, not just happy paths.
4. **Implement:** Make the smallest understandable change. Avoid speculative services, dependencies or abstraction layers; preserve API/data compatibility unless change is approved.
5. **Verify:** Run relevant tests, lint, formatting, typecheck, migration/contract validation, and diff/secret checks that exist. Report exact commands and **observed** outcomes, including failures and skipped checks.
6. **Review:** For high-risk modifications, seek independent review from the other agent (Codex/Claude Code) or another fresh context, if available. Review must have approved spec and complete diff.
7. **Commit:** Local focused commits are permitted. Stop at task boundary, summarize the change, and await owner's approval before pushing or advancing unapproved work.
8. **Integrate:** Human approves external push, merge and deployment separately after reviewing the evidence. CI/branch protections gate merge.

## 5. Risk tiers

**Elevated (independent review required before merging):** authentication, password/TOTP/recovery, role assignments and authorization, medical document access, audit trails, leave-credit ledger/reservations, policy versioning and eligibility, data migrations, public API breaks, infrastructure, secrets, and production integrations. Tests must include unauthorized access, duplicate requests, concurrency/race conditions, and rollback where applicable.

**Routine (focused agent review and CI normally sufficient):** scoped UI layout, copy, non-sensitive documentation, small pure utilities, and bug fixes that don't change security or credit behavior. Reclassify if the diff reaches an elevated area.

**Stop-and-escalate conditions:** ambiguous policy, undocumented change to leave rules, destructive or irreversible operation, unexpected changes to other work, external data transfer, potential secret exposure, vulnerability finding, or test failures that cannot be explained.

## 6. Definition of done (each implementation slice)

- [ ] Approved task and acceptance criteria can be traced to the design/plan.
- [ ] Changes are limited to the approved scope; no unapproved dependency, API or migration changes.
- [ ] Tests demonstrate new behavior and relevant failure cases, including permissions and idempotency where relevant.
- [ ] Available formatting, lint, typecheck, tests and contract/migration checks pass, or blockers are explicitly reported.
- [ ] No actual employee data, medical file, or secret in code, logs, snapshots, PRs, or sample fixtures.
- [ ] Privacy boundaries and business invariants checked; audit/events updated where required.
- [ ] Any required docs and ADRs updated; new env vars described without values.
- [ ] Local diff reviewed; local commit created if appropriate; no unapproved remote operations.
- [ ] Completion report includes scope, file list, test commands/results, known limitations, and owner approvals still needed.

## 7. Required high-risk acceptance tests

When the owning slice is built, ensure tests for: no self-approval even with multiple roles; employee/supervisor access denied to unrelated medical files; pending paid leave reserves once; concurrent filing cannot oversubscribe spendable credit; expiration releases at most once; HR finalization converts reservation to debit at most once; LWOP never touches paid credits; minor versus major corrections route correctly; TOTP required before issuing a normal session; MFA recovery codes single-use; historical requests retain prior policy version. These scenarios originate in the approved design.

## 8. Enforceable controls, not just prompt text

- Restrict agent execution to a sandbox with least privilege. Treat `.claude/settings.json` and `.codex/config.toml` as **defaults**, not immutable company policy; inspect actual effective settings at runtime.
- Keep secrets outside the repository; maintain `.gitignore` and secret scanning. Use development-only credentials and least-privilege access. Never provide production credentials to a demo coding agent.
- Protect the primary Git branch and require PR review and passing CI checks. On GitHub, a solo owner can act as the final reviewer; independent agent review is supplemental.
- Require explicit manual approvals for pushes, merges, external writes, schema changes with data risk, and deployments. Tool-specific command pattern rules do **not** intercept every spelling, wrapper, MCP tool or CI action.
- CI should later check typechecking, lint, tests, API contracts, dependency security, secrets and migrations. Do **not** invent CI script names before package manifests exist.
- Check for changes to policy/approval code during review; restrict production service accounts and infrastructure independently of repository prompts.

## 9. Work request and handoff template

For each agent execution, provide:

- **Goal + authorized slice:** Link to exact plan/task and acceptance criteria.
- **Allowed paths:** Files/modules expected to change and exclusions.
- **Boundaries:** Explicit prohibited scope, external actions, data and dependencies.
- **Verification:** Tests to create/run; expected behavior and important negative cases.
- **Stop conditions:** Unknown business policy, architecture conflict, failed validation, credential requirement, or action needing owner approval.
- **Handoff:** Actual commands/results, diff summary, commit IDs if local, risks/unknowns, next owner decision.

## 10. Rollout checklist before agents begin coding

1. Install this bundle in the **root of the actual Git repository**; reconcile with existing instructions and settings rather than overwriting them blindly.
2. Verify Codex project trust, sandbox/approval settings and rules; verify Claude shared settings and that `CLAUDE.md` imports `AGENTS.md`.
3. Set up `.gitignore` for `.env*` (allow only intentional examples), real data exports, and local tool files. Create CI only after package scripts are established.
4. Set up branch protection and review requirements that your GitHub plan supports.
5. Approve the first Superpowers implementation plan; do not treat this document as automatic authority to scaffold the product.
