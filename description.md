# agent-ecosystem-ticket-router

A Claude Code skill plugin that triages, routes, and closes tickets across a multi-project ecosystem.
Requires the `agent-project-issues` MCP (installed automatically by Claude Code).

## What you get

- **Route skill** — analyses a new intake ticket and creates linked child tickets in every affected project, determined by live project descriptions (no hardcoded routing tables).
- **Close skill** — checks child ticket status and closes the intake once all children are resolved.
- **Idempotency** — re-running route on an already-triaged ticket is safe; it detects prior routing and skips re-creation.
- **Label propagation** — type (`bug`, `documentation`, `enhancement`) and severity (`severity:high`, `severity:med`, `severity:low`) labels are copied from the intake to each child ticket automatically.
- **Project-agnostic** — routing signal comes entirely from project descriptions at call time; no ecosystem-specific knowledge is baked in.
