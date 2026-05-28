# agent-ecosystem-ticket-router

A Claude Code **skill** plugin. Triage, route, and close tickets across a project ecosystem using MCP tool descriptions.

This plugin ships **only the skill content** — no binaries, no MCP server. It depends on the `agent-project-issues` MCP at runtime (declared in `.claude-plugin/plugin.json`); Claude Code installs/loads it automatically.

## Install

```
/plugin marketplace add Seretos/agent-marketplace
/plugin install agent-ecosystem-ticket-router@agent-marketplace
```

## What the skills teach

- `skills/route/SKILL.md` — triage a new ticket and route it to the right project.
- `skills/close/SKILL.md` — close a ticket once the work it tracks has been verified done.
