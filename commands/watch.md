---
name: ck-watch
description: "Tail the live Cavekit dashboard — phase, current task, tokens used, task completion count."
argument-hint: "[--interval SECONDS] [--once]"
allowed-tools: ["Bash(node ${CLAUDE_PLUGIN_ROOT}/scripts/cavekit-tools.cjs:*)", "Bash(cat .cavekit/*)", "Bash(ls .cavekit/*)", "Bash(sleep *)"]
---

# Cavekit Watch — Live Dashboard

Print the current status block. With no flags, refreshes every 3 seconds until
the user interrupts. With `--once`, prints a single frame and exits.

## Execute

1. Parse arguments:
   - `--interval N` (default 3)
   - `--once` (print one frame and stop)

2. Loop (or run once):

   ```bash
   node "${CLAUDE_PLUGIN_ROOT}/scripts/cavekit-tools.cjs" status
   ```

   Then also print the progress snapshot if it exists:

   ```bash
   cat .cavekit/.progress.json 2>/dev/null || echo "(no progress snapshot yet)"
   ```

3. Between frames:
   ```bash
   sleep ${INTERVAL}
   ```

4. If `.cavekit/.loop.json` is missing, print:
   ```
   [ck:watch] No active loop. Run /ck:make to start one.
   ```
   and exit.

5. On Ctrl-C or user interrupt, exit cleanly.

## Critical rules

- Do not mutate any file under `.cavekit/`. This command is read-only.
- Do not call `route` or `heartbeat` from here — those are the stop-hook's job.
- Keep each frame short (≤ 12 lines) so the scrollback stays usable.
