@AGENTS.md

# Claude Code-specific workflow

- Treat the imported `AGENTS.md` as the canonical shared contract. Do not duplicate or weaken its rules here.
- Use plan mode when a task is not yet approved, contains architectural decisions, or affects authorization, leave balances, MFA, employee privacy, migrations, or external infrastructure.
- Check tool permission prompts and ask the owner when a task reaches an approval gate; do not work around a blocked command with another tool or wrapper.
- Keep outputs scoped to the approved task. Provide actual test evidence before saying work is complete.
