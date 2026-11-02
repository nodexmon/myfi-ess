# MyFi ESS — Governance Bundle Setup

This is a **starter file bundle**, not an initialized Git repository or completed CI configuration. Copy its contents into the root of your future or existing MyFi ESS repo after checking for conflicting files. The approved Phase 1 design specification is included, with its status updated to approved.

## Files

- `AGENTS.md` — shared agent contract (Codex and Claude Code)
- `CLAUDE.md` — imports `AGENTS.md`
- `.claude/settings.json` — shared Claude Code permission defaults
- `.codex/config.toml` — Codex sandbox and approval defaults
- `.codex/rules/governance.rules` — additional Codex prompts for Git/remote commands
- `docs/governance/AGENT_GOVERNANCE.md` — autonomy, gates, risk tiers, definition of done
- `.github/pull_request_template.md` — evidence template for change review
- `docs/adr/0001-balanced-agent-governance.md` — recorded decision
- `docs/superpowers/specs/2026-10-31-myfi-ess-design.md` — approved product design

## Before using coding agents

1. Verify your installed Codex and Claude Code versions, project trust, effective sandbox/permissions, and availability of command rules; start agents from the repository root.
2. In Claude Code, confirm `CLAUDE.md` imports `AGENTS.md` (e.g. use `/context`). In Codex, confirm repository instructions were loaded. These files are not magic security barriers.
3. Create a `.gitignore` suited to the actual codebase, ignoring real `.env` files, dumps, credentials, `node_modules`, and local agent state. Store only sanitized `.env.example` values.
4. Configure GitHub branch protection and status checks before allowing remote pushes. Configure CI with real package scripts after workspace scaffolding; this bundle deliberately avoids guessing them.
5. Keep demo data fictional. Do not supply personal employee accounts/medical records to agents or hosted demo services without company authorization.
6. Write and approve separate Superpowers implementation plans before implementing product modules.

## Installation example

Copy the archive's directory tree into the Git repository root without flattening hidden `.claude` and `.codex` directories. Review and reconcile any paths already present. Do not overwrite existing policy files automatically.

## Documentation references checked when drafting

- Codex AGENTS.md and config: https://developers.openai.com/codex/config-basic/
- Codex command rules: https://developers.openai.com/codex/rules/
- Claude project instruction loading: https://code.claude.com/docs/en/memory
- Claude permission modes/rules: https://code.claude.com/docs/en/permissions

These are project-level defaults. They can be overridden by user-level/runtime controls; real security gates belong to the execution environment and version-control host.

## Technology stack freeze (2026-10-31)

The approved library catalog is in `docs/02-architecture/TECH_STACK.md`. Review `docs/02-architecture/ARCHITECTURE.md` and accepted ADR 0002–0006. These are **documentation decisions**, not a request to install dependencies or deploy. Before scaffolding, authorize the compatibility spike and approve a Superpowers implementation plan. No exact package versions have been verified or frozen yet.
