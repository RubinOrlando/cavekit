---
name: backpropagation
description: |
  Trace bugs backwards through the Hunt lifecycle. When a test fails, a review
  flags a gap, or production surfaces a defect, identify the kit requirement
  that should have prevented it, update the kit, add a regression test, and log
  the trace in .cavekit/history/backprop-log.md. Runs automatically via the
  auto-backprop hook when a test command fails during /ck:make. Also invokable
  manually via /ck:backprop. Trigger phrases: "trace this bug", "backprop",
  "why wasn't this caught", "fix the kit", "regression test".
---

# Backpropagation

Bugs are spec bugs until proven otherwise. Before patching code, verify the
kit actually covered the failing behavior. If it did not, the fix is a kit
amendment plus a regression test — not just a patch.

## Six-step procedure

### 1. TRACE — match failure to a requirement

Read the failure output. Find the single acceptance criterion (or gap of one)
that, if asserted, would have caught this. Identify the kit and R-ID:

```
Kit: cavekit-auth.md
Requirement: R004 (rate-limit middleware)
Closest AC: AC2 ("returns 429 when limit exceeded")
Missing coverage: 429 header case sensitivity not asserted
```

If no requirement fits, the bug is in a dimension the kit never addressed —
this is a **missing requirement** (case D below).

### 2. ANALYZE — classify the gap

One of:

- **A. missing_criterion** — the requirement exists but the acceptance criteria
  do not cover this case. Add an AC.
- **B. incomplete_criterion** — an AC partially covers it but is too vague to
  catch the failure. Tighten the AC.
- **C. wrong_criterion** — an AC asserts the wrong thing. Correct it.
- **D. missing_requirement** — no requirement addresses this dimension at all.
  Add a new R.

### 3. PROPOSE — draft the spec change

Write the exact amendment. Required fields:

```
Classification: missing_criterion
Kit: cavekit-auth.md
Change: R004 append AC5
Proposed AC: "AC5 — Rate-limit headers (X-RateLimit-*, Retry-After) are
  returned with canonical lowercase casing per RFC 7230 §3.2."
Trace: auth.test.ts → 429 path → header casing mismatch
```

Show the proposed change to the user. **Wait for explicit approval** before
writing to the kit. Do not silently amend specs.

### 4. GENERATE — regression test

Before fixing the bug, write a test that asserts the new AC and currently
**fails**. This locks in the requirement in executable form.

Commit the failing test separately from the fix:
```
git commit -m "test: add regression for rate-limit header casing (cavekit-auth.md R004 AC5)"
```

### 5. VERIFY — fix until the test passes

Patch the code. Run the full test suite. Confirm the new regression test
passes and nothing else regresses. Commit the fix referencing the AC.

### 6. LOG — write a trace entry

Append to `.cavekit/history/backprop-log.md`:

```markdown
## Entry {n}

- id: 12
- date: 2026-04-17T14:22Z
- classification: missing_criterion
- kit: cavekit-auth.md
- requirement: R004
- ac_added: AC5
- failing_test_before_fix: auth.test.ts::rate-limit headers casing
- fix_commit: abc1234
- pattern_category: input_validation
```

## Pattern detection

The log is a corpus. If the same `pattern_category` appears 3+ times in one
project, the issue is systemic and belongs one layer up — at the
brainstorming / kit-writing layer, not per-requirement. Categories:

- `input_validation` — unvalidated / mis-cased / range-unchecked input.
- `concurrency` — races, missed locks, stale reads.
- `error_handling` — swallowed exceptions, missing retries, partial rollbacks.
- `integration` — contract drift between services, missing end-to-end tests.
- `observability` — silent failures, missing logs/metrics.

When a threshold hits, propose a cross-kit amendment (e.g., a validation rule
added to every input-accepting kit). Announce this to the user.

## Auto-backprop integration

The `auto-backprop.js` hook writes `.cavekit/.auto-backprop-pending.json` when
a test command fails during `/ck:make`. The stop hook reads this flag and
prepends a backprop directive to the next iteration's prompt, which tells the
agent to run the six steps above before resuming normal task execution. Once
the directive fires, the flag is atomically deleted.

Disable via `auto_backprop = false` in `.cavekit/config.json` if you need to
run `/ck:make` without this safety net (e.g., during exploratory spikes).

## Anti-patterns

- **Patch first, spec later** — fixes that never make it back to the kit.
  Every rerun of the loop may re-introduce the bug.
- **Silent amendment** — updating a kit without user approval. The spec is
  the contract; amendments are a negotiation, not a refactor.
- **Per-bug regression test with no spec change** — the test passes, but the
  next reader of the kit still will not know why. The AC must be written down.
- **Bulk backprop** — rolling up N failures into one spec change. Each failure
  gets its own entry. Patterns emerge only if each is logged separately.
