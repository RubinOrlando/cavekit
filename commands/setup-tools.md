---
name: ck-setup-tools
description: "Detect MCP servers, Claude Code plugins, and CLI tools available in the current environment. Writes .cavekit/capabilities.json. Safe to re-run any time."
argument-hint: "[--summary-only]"
allowed-tools: ["Bash(node ${CLAUDE_PLUGIN_ROOT}/scripts/cavekit-tools.cjs:*)", "Bash(cat .cavekit/capabilities.json)"]
---

# Cavekit Setup Tools

Discover what is actually available in this environment so the rest of the
pipeline can bind to real capabilities. Run this at project init, after
installing a new tool, or whenever a task fails with "command not found".

## Run discovery

```bash
node "${CLAUDE_PLUGIN_ROOT}/scripts/cavekit-tools.cjs" discover
```

This writes `.cavekit/capabilities.json`. Read it back and summarize:

```bash
cat .cavekit/capabilities.json
```

## Summarize for the user

Print a short, human-readable table:

```
═══ Cavekit Capabilities ═══

CLI tools:
  ✓ git, gh, node, python3, docker
  ✗ codex, graphify, supabase

MCP servers:
  (none detected)

Codex peer review: UNAVAILABLE (install `codex` on $PATH to enable /ck:judge)
Knowledge graph:   UNAVAILABLE (run `graphify build .` to enable graph-backed routing)
```

With `--summary-only`, skip writing the JSON file and only print the summary.

## Recommendations

After the summary, offer a short list of optional upgrades based on what is
missing:

- If `codex` is missing: mention the peer-review workflows that would unlock.
- If `graphify` is missing: mention the `graphify-integration` skill.
- If `gh` is missing: mention that `/ck:check` can post gap reports as
  GitHub issues when `gh` is present.

Do not actually install anything. Just recommend.

## Critical rules

- Capabilities are advisory, never prescriptive. A missing tool is not an
  error — it is information for `/ck:sketch` to consider.
- Do not store credentials. Availability ≠ reachability; credential checks
  belong in separate kits.
- Re-run on every new project. The default `/ck:init` already calls this.
