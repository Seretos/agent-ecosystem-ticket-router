---
name: route
description: Triage a new ticket and route it to the right project in a multi-project ecosystem using MCP tool descriptions.
---

# route

## When to reach for this skill

Activate this skill when:

- The user signals intent to file a bug, request, or idea and the **target project is ambiguous or cross-cutting** (i.e. the ticket could affect more than one project, or the user hasn't named a specific destination).
- The user explicitly asks to "triage" or "route" an existing intake ticket.

Do **not** trigger on every user message. The signal is an explicit filing/triage intent, not mere mention of a project name.

## Step 1 — Idempotency check

Before creating anything, call `get_ticket` on the intake ticket.

- Inspect its labels: if a `triaged` label is present, the intake has already been routed.
- Inspect its comments via `list_comments`: if any comment body contains the prefix `<!-- ticket-router:children`, routing has already run.

If either condition is true, **skip all creation steps** and delegate immediately to the `close` skill to sync child status against the intake's checklist. Do not re-route.

## Step 2 — Discover routable projects

Call `list_projects` to retrieve the full project list.

From the result, build the **candidate pool** by applying these two filters:

1. Drop any project whose `description` field is empty or absent — it carries no routing signal.
2. Drop the intake's own project — the intake project is never a routing target.

Never hardcode project ids, names, or pairings. The candidate pool is assembled entirely from live data at call time.

## Step 3 — Classify against descriptions

For each project in the candidate pool, reason **semantically** over the project's `description` versus the intake's title and body to decide whether that project is affected by the reported bug/request/idea.

Rules:

- Use natural language reasoning — no keyword matching or hard-coded routing tables.
- When a wrapper project and a library project are **both** in the candidate pool and their descriptions reference each other (by id or clear alias), include **both** in the shortlist. Never silently drop one half of a coupled pair.
- A project is included if the intake plausibly affects its stated responsibilities; excluded if there is no meaningful overlap.

## Step 4 — Create children sequentially

For each project in the shortlist, call `create_ticket` **one at a time**. Await each call before starting the next — parallel calls to `create_ticket` may fail silently.

Each child ticket must:

- Carry a back-link in the body: a line reading `Intake: <web_url of intake>` (use the `web_url` returned by `get_ticket`).
- Inherit the intake's **type label** (`bug`, `documentation`, or `enhancement`) if one is present — use `update_ticket` on the child after creation to add it. If the label does not yet exist in the target project, call `create_label` to create it there first, then apply it via `update_ticket`.
- Inherit the intake's **severity label** (`severity:high`, `severity:med`, or `severity:low`) if one is present — add it the same way. If the label does not yet exist in the target project, call `create_label` to create it there first, then apply it via `update_ticket`.

Do not write `#ai-generated` anywhere in ticket bodies or comments.

## Step 5 — Post the routing comment

After all children are created, call `add_comment` on the intake. The comment body must follow this exact shape:

```
## Routed children

<!-- ticket-router:children
project_a_id#42
project_b_id#17
-->

- [ ] project_a_id#42
- [ ] project_b_id#17
```

Conventions:

- The machine-readable block between `<!-- ticket-router:children` and `-->` uses `project_id#number` references, where `project_id` is the `id` field returned by `list_projects` (not the display name).
- The `- [ ]` list below the block is the human-visible progress checklist; it mirrors the machine block line-for-line.
- One line per child, no trailing content on the ref line.

## Step 6 — Add `triaged` label

Call `update_ticket` on the intake to add the `triaged` label. If the label does not exist in the project, call `create_label` to create it first, then retry. This step is best-effort: if it fails for any other reason, log the error and continue — do not abort the overall workflow.

## Project-agnostic rule

This skill must never hardcode project ids, project names, or routing pairings. All routing signal comes from the `description` fields read at call time via `list_projects`. Ecosystem-specific context belongs in the consuming host repo's own `AGENTS.md`, not here.
