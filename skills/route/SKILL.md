---
name: route
description: Triage a new ticket and route it to the right project in a multi-project ecosystem using MCP tool descriptions.
---

# route

Stub — real content lands in `Seretos/agent-ecosystem-ticket-router#1`.

Typical structure once implemented:

1. **When to reach for this skill** — the user dropped a new request / bug / idea and the right project is ambiguous; pick one out of the configured ecosystem.
2. **Mental model** — projects are discoverable via the `agent-project-issues` MCP (`list_projects` / `find_projects`); each project's description carries the routing signal.
3. **Tool inventory** — the `agent-project-issues` tools you'll lean on; nothing project-specific gets baked in here.
4. **Patterns** — match-by-keyword, fan-out for cross-project work, dedup against existing tickets.
5. **Pitfalls** — over-triggering on every user message, ignoring closed tickets that re-occur, skipping the dedup search.
