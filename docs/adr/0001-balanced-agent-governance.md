# ADR 0001 — Balanced Agentic Development Governance

- **Status:** Accepted by project owner
- **Date:** 2026-10-31
- **Decision makers:** MyFi ESS project owner

## Context

MyFi ESS is a solo-built proposal, with Codex and Claude Code intended to write and review code. Leave-credit accounting, MFA, roles, and confidential HR files make unrestricted autonomous implementation or deployment too risky. Excessively strict approvals for every local edit would also undermine the development workflow.

## Decision

Adopt **Balanced Agent Governance**: agents can autonomously work within an explicitly approved implementation slice, run local checks, and commit locally. The owner approves material changes to scope/business/security/architecture, production dependencies, remote operations, destructive changes, and deployment. Use `AGENTS.md` as the shared contract with platform-specific permission defaults, independent high-risk review, test evidence, and protected Git branches.

## Considered alternatives

- Strict: approval for almost every edit/command; more control but high interruption overhead.
- Balanced (selected): bounded execution with risk-based approval gates; appropriate for a solo MVP developer.
- High autonomy: agents can push or deploy by default; fast but harder to audit and constrain for HR-sensitive data.

## Consequences

Governance takes some setup and review time. File-based instructions alone are insufficient: tool permissions, sandbox restrictions, repository protections and CI are required. Changes to governance itself need owner approval.
