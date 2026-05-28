---
name: close
description: Close a ticket once the work it tracks has been verified done, recording the closing evidence as a comment.
---

# close

Stub — real content lands in `Seretos/agent-ecosystem-ticket-router#1`.

Typical structure once implemented:

1. **When to reach for this skill** — a piece of work referenced by a ticket has been verified done (commit landed, behaviour confirmed, regression test passes).
2. **Mental model** — closing is a two-step act: post a comment that names the closing evidence (commit / PR / observation), then transition the ticket to `closed:completed`.
3. **Tool inventory** — `agent-project-issues` `add_comment` + `update_ticket`; status-discovery via `list_ticket_statuses` when the provider's state-space is unknown.
4. **Patterns** — commit-message-driven close, manual verification close, batch close of an `#ai-generated` ticket run.
5. **Pitfalls** — closing without evidence, picking the wrong `closed:*` state for the provider, closing tickets that are merely paused.
