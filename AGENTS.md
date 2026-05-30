# agent-ecosystem-ticket-router

Pure skill plugin — no binary, no MCP server. Ships two skills under `skills/`, both driving the `agent-project-issues` MCP (declared as a runtime dependency in `.claude-plugin/plugin.json`).

## Contracts an agent won't infer from the tree

- **Release is orphan-branch + marketplace dispatch.** `release.yml` (manual: Actions → release → `version=X.Y.Z`) stamps the version, then force-pushes an orphan `release` branch holding only install-ready files and POSTs a dispatch (`category: skill`) to `Seretos/agent-marketplace`. `main` and `release` share no history. Clients install at the tag `agent-ecosystem-ticket-router--vX.Y.Z`.
- **Required secret:** `MARKETPLACE_DISPATCH_TOKEN` — fine-grained PAT, `Contents: RW` + `Pull requests: RW` on `Seretos/agent-marketplace` only.
- **`assets/icon.png` is a release artifact, not just a repo file.** The dispatch payload sends a `raw.githubusercontent.com/${repo}/${TAG}/assets/icon.png` URL to the marketplace, so the file must live on the orphan `release` branch at the tagged commit — `release.yml`'s stage step copies `assets/` into the staging tree for exactly that reason.
- **Skill design rule: project-agnostic.** The two skills must not bake in ecosystem-specific knowledge (which projects exist, plugin↔lib pairings, per-project conventions). Triage and routing decisions are driven by the MCP's project list + tool descriptions at call time, not by hardcoded names. A consuming host repo carries the ecosystem-specific context (e.g. its own `AGENTS.md`); this plugin stays generic.

## Contracts for consuming host repos

These conventions must be satisfied in the host ecosystem for the skills to work correctly. They are not enforced by this plugin — they are the caller's responsibility.

- **Project description convention.** Any project that should be reachable by the `route` skill must have a non-empty `description` in the MCP project list (as returned by `list_projects`). Projects with an empty or absent description are silently excluded from the candidate pool and will never receive routed tickets.

- **Wrapper/lib cross-reference rule.** When two projects form a coupled pair (e.g. a wrapper and its underlying library), each project's `description` must name the other — by `id` or an unambiguous alias. This is the only signal the router uses to detect a pair and include both. Without mutual description references, the router may include only one half of the pair.

- **Inbox convention.** The caller is responsible for naming the intake project (the project where the original ticket lives). The plugin never assumes which project is the inbox. The intake project is always excluded from the routing candidate pool; tickets are never re-routed back to their origin project.

- **Label conventions.** The following labels are propagated automatically from the intake to each child ticket by the `route` skill:
  - Type label: `bug`, `documentation`, or `enhancement` — exactly one of these should be present on the intake.
  - Severity label: `severity:high`, `severity:med`, or `severity:low` — optional; propagated when present.
  
  Labels must already exist in each target project, or the `route` skill will create them on demand via `create_label`. The `triaged` label is added to the intake itself after routing completes.
