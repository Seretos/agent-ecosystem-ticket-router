---
name: close
description: Close a ticket once the work it tracks has been verified done, recording the closing evidence as a comment.
---

# close

## When to reach for this skill

This skill is reached two ways:

- **Directly** — the user asks to close a ticket whose tracked work is done.
- **Indirectly** — the `route` skill delegates here when it detects that an intake has already been triaged, in order to sync child status against the routing checklist.

## Step 1 — Parse the children block

Call `list_comments` on the intake ticket. Scan every comment body for a block matching:

```
<!-- ticket-router:children
...
-->
```

If a matching comment exists, extract each `project_id#number` line from inside the block. These are the child ticket references to check.

If no such block exists anywhere in the comments, treat this as a **standalone ticket** and jump to the standalone path in Step 5.

## Step 2 — Check each child status

For each `project_id#number` extracted in Step 1, call `get_ticket`. Record whether the ticket is in a closed/completed state or still open. Do this for every child before proceeding.

## Step 3 — Update the checklist

Using the live status recorded in Step 2, rewrite the checklist in the routing comment (the one containing the `<!-- ticket-router:children` block) to reflect the current state of every child: set `- [x] project_id#N` for each child that is currently closed, and `- [ ] project_id#N` for each child that is currently open.

Apply all updates in a single `update_comment` call by rewriting the full updated comment body. This ensures that a child which was previously closed but has since been reopened is correctly shown as unchecked.

## Step 4 — Gate on open children

After updating the checklist, count the children that are still open.

If **any child is still open**, call `add_comment` on the intake listing the open refs:

```
The following child tickets are still open — intake cannot be closed yet:

- project_a_id#N
- project_b_id#M
```

Then **stop**. Do not proceed to close the intake. Return the list of open refs to the user.

## Step 5 — Close the intake

### Routed path (all children closed)

When every child is confirmed closed:

1. Call `list_ticket_statuses` to discover the available statuses if the correct closed-state name is not already known.
2. Call `add_comment` on the intake with a summary naming all child refs that were closed, e.g.:

   ```
   All child tickets closed. Closing intake.

   Closed: project_a_id#42, project_b_id#17
   ```

3. Call `update_ticket` to transition the intake to the appropriate closed/completed status.

### Standalone path (no children block)

When no `<!-- ticket-router:children` block was found in Step 1:

1. Ask the user for closing evidence: a commit SHA, PR link, or concrete observation confirming the work is done. Do not close without it.
2. Call `add_comment` on the intake naming the evidence provided.
3. Call `list_ticket_statuses` if the closed-state name is unknown, then call `update_ticket` to transition to the closed/completed status.

## Contracts an agent won't infer

- **Status discovery at call time.** Never assume the closed-state label. Always call `list_ticket_statuses` when in doubt; pick the status whose name most closely matches "closed" or "completed" for the provider.
- **Checklist is the source of truth.** The `<!-- ticket-router:children` block is the authoritative list of child refs. Do not infer children from any other source.
- **No silent closes.** A standalone ticket may not be closed without user-supplied evidence; a routed intake may not be closed while any child remains open.
