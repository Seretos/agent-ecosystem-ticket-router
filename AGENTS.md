# agent-ecosystem-ticket-router

Pure skill plugin — no binary, no MCP server. Ships two skills under `skills/`, both driving the `agent-project-issues` MCP (declared as a runtime dependency in `.claude-plugin/plugin.json`).

## Contracts an agent won't infer from the tree

- **Release is orphan-branch + marketplace dispatch.** `release.yml` (manual: Actions → release → `version=X.Y.Z`) stamps the version, then force-pushes an orphan `release` branch holding only install-ready files and POSTs a dispatch (`category: skill`) to `Seretos/agent-marketplace`. `main` and `release` share no history. Clients install at the tag `agent-ecosystem-ticket-router--vX.Y.Z`.
- **Required secret:** `MARKETPLACE_DISPATCH_TOKEN` — fine-grained PAT, `Contents: RW` + `Pull requests: RW` on `Seretos/agent-marketplace` only.
- **`assets/icon.png` is a release artifact, not just a repo file.** The dispatch payload sends a `raw.githubusercontent.com/${repo}/${TAG}/assets/icon.png` URL to the marketplace, so the file must live on the orphan `release` branch at the tagged commit — `release.yml`'s stage step copies `assets/` into the staging tree for exactly that reason.
- **Skill design rule: project-agnostic.** The two skills must not bake in ecosystem-specific knowledge (which projects exist, plugin↔lib pairings, per-project conventions). Triage and routing decisions are driven by the MCP's project list + tool descriptions at call time, not by hardcoded names. A consuming host repo carries the ecosystem-specific context (e.g. its own `AGENTS.md`); this plugin stays generic.
