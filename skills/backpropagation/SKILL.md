---
name: backpropagation
description: >
  The technique of tracing bugs and manual fixes back to specs and prompts, then fixing at the source
  so the iteration loop can reproduce the fix autonomously. Covers the 6-step backpropagation process,
  commit classification, spec-level root cause analysis, and regression test generation.
  Trigger phrases: "backpropagate", "back-propagate", "trace bug to spec", "fix the spec not the code",
  "why did this bug happen", "update specs from bug"
---

# Backpropagation: Tracing Bugs Back to Specs

In SDD, backpropagation means tracing a production defect upstream through the specification chain until you find the gap that allowed it. The name borrows loosely from neural network training, where output errors are propagated backward to adjust earlier layers. In practice, when the built software has bugs or gaps, you trace the issue back to the specs and prompts and fix at the source -- not just in code.

**Key insight:** When a fix lives only in code with no corresponding spec update, the next iteration loop may reintroduce the same defect. The goal is that specs plus the iteration loop can reproduce any fix autonomously.

---

## 1. Why Backpropagation Matters

Without backpropagation, every bug fix is a one-off patch. The next time the iteration loop runs, it may reintroduce the bug because nothing in the specs or plans prevents it.

With backpropagation:
- Bug fixes become **spec improvements** that persist across all future iterations
- The iteration loop becomes **self-correcting** -- it learns from every manual intervention
- Specs become **progressively more complete** over time
- The gap between "what specs describe" and "what works" **shrinks monotonically**

```
Without backpropagation:
  Bug found -> Fix code -> Bug may return next iteration

With backpropagation:
  Bug found -> Fix code -> Update spec -> Re-run iteration loop -> Fix emerges from specs alone
```

---

## 2. The 6-Step Backpropagation Process

This is the complete process for tracing a bug back to its spec-level root cause and closing the loop.

### Step 1: Identify and Fix the Defect

Locate the bug -- whether through manual testing, automated failures, user reports, or monitoring alerts -- and resolve it through normal debugging. This produces a working code change, but the job is far from done: until the underlying spec gap is closed, this fix is fragile.

```bash
# The fix produces commits that we will analyze
git log --oneline -5
# a1b2c3d Fix: connection pool exhaustion under concurrent load
# e4f5g6h Fix: missing rate limit headers in API responses
```

### Step 2: Analyze What the Spec Missed

This is the pivotal step. Ask: **"Where in the specification chain did this requirement slip through?"**

Break the analysis into four dimensions:
- **WHAT** changed (files, functions, observable behavior)
- **WHY** it was wrong (which assumption proved false)
- **The RULE** (the invariant that should have been stated)
- **The LAYER** (which spec, plan, or prompt should have contained this)

Example analysis:

```markdown
## Backpropagation Analysis: Database Connection Pooling

**WHAT changed:** Added pool size limits and idle timeout in `src/db/pool.ts`
**WHY:** The data layer spec assumed unlimited connections; under load the database
         rejected new connections once the server-side limit was reached
**RULE:** "The database module MUST configure a bounded connection pool with
          idle timeout and max-connection limits matching the deployment target"
**LAYER:** spec-data.md (no mention of pool configuration), plan-data.md (no task for pool tuning)
**Spec implications:** Add requirement R5 to spec-data.md covering connection pool settings
```

### Step 3: Update the Specification

Add the missing requirement or constraint to the appropriate spec file. Focus on acceptance criteria that are concrete enough for the iteration loop to act on:

```markdown
# In context/specs/spec-data.md, add:

### R5: Database Connection Pool Configuration
**Description:** The database module must use a bounded connection pool
with configurable limits to prevent resource exhaustion under load.
**Acceptance Criteria:**
- [ ] Maximum pool size is configurable and defaults to a sensible value
- [ ] Idle connections are reaped after a configurable timeout
- [ ] Pool exhaustion returns a clear error rather than hanging indefinitely
- [ ] Connection health checks run before returning a connection from the pool
**Dependencies:** R1 (database client setup), R2 (environment configuration)
```

### Step 4: Propagate Changes to Plans and Tracking

Trace the spec update through every downstream context file:

1. **Identify affected plan files:** Which plans govern the changed source paths?
2. **Update plans:** Add or close tasks reflecting the new requirement.
3. **Update impl tracking:** Record the backpropagation event and its root cause.
4. **Annotate:** Mark updated sections with backpropagation metadata so future reviews can trace lineage.

```markdown
# In context/plans/plan-data.md, add:

### T-DATA-005: Configure bounded connection pool
- **Status:** DONE (backpropagated from manual fix a1b2c3d)
- **Spec:** R5 in spec-data.md
- **Files:** src/db/pool.ts
- **Acceptance criteria:**
  - [ ] Max pool size enforced
  - [ ] Idle timeout configured
  - [ ] Exhaustion handled gracefully
```

### Step 5: Apply Systemic Prompt Improvements (If Pattern Detected)

When the defect represents a recurring class of problem rather than a one-off, elevate the fix to the prompt level so it applies across all domains:

**Signs you are looking at a pattern:**
- The same category of bug has surfaced in more than one module
- The gap is structural (e.g., no specs anywhere address resource limits)
- A missing validation gate allowed the issue through

**Example systemic fix:**

```markdown
# In prompt 003, add to the validation section:

## Resource Management Validation
For every external resource integration, verify:
- [ ] Connection or handle limits are bounded and configurable
- [ ] Idle resources are cleaned up on a timeout
- [ ] Exhaustion scenarios return actionable errors
- [ ] Resource lifecycle is covered by tests under load
```

### Step 6: Verify and Lock In

Run the iteration loop against the updated specs to prove the fix emerges from specifications alone, then generate regression tests to prevent future recurrence:

```bash
# Proof step: remove the manual fix and re-run from specs
git stash  # temporarily remove the manual fix
iteration-loop context/prompts/003-generate-impl-from-plans.md -n 5 -t 1h
# Verify the fix appears in the generated implementation

# If it does NOT, the spec update is insufficient -- return to Step 3
```

Once verified, create regression tests:

```bash
# Generate tests targeting the updated spec
{TEST_COMMAND} --spec context/specs/spec-data.md

# Or manually create a regression test
# tests/db/connection-pool-limits.test.ts
```

The regression tests should:
- Map directly to the acceptance criteria from Step 3
- Fail if the fix is reverted
- Run as part of the standard test suite going forward

---

## 3. Back-Propagate Analysis (Automated)

The back-propagate analysis automates Steps 2-4 by examining recent git history.

### 3.1 Classify Commits

Analyze recent commits and classify each as:

| Classification | Meaning | Action |
|---------------|---------|--------|
| **Manual fix** | Human or interactive agent fixed a bug | Trace back to spec -- this is a backpropagation target |
| **Iteration loop** | Automated iteration loop made the change | No action -- this is the system working as intended |
| **Infrastructure** | Build config, CI, tooling changes | No action -- not spec-related |

**How to classify:**
- Commits from iteration loop sessions have predictable patterns (automated commit messages, batch changes)
- Manual fixes are typically single-issue, focused commits with descriptive messages
- Infrastructure changes touch config files, build scripts, CI pipelines

### 3.2 Analyze Each Manual Fix

For each commit classified as a manual fix, determine:

```markdown
## Commit: abc1234 "Fix: auth token not refreshing on 401"

### WHAT changed
- File: src/auth/client.ts
- Function: handleApiResponse()
- Behavior: Added 401 detection and token refresh logic

### WHY it was wrong
- The auth module did not handle 401 responses
- Tokens would expire and never refresh, causing cascading auth failures

### RULE (invariant that should have been specified)
- "Authentication tokens must be refreshed automatically on 401 responses"

### LAYER (which context file should have caught this)
- spec-auth.md: Missing requirement for error-based token refresh
- plan-auth.md: No task for 401 handling

### Spec Implications
- Add R7 to spec-auth.md: Token Refresh on Authentication Failure
- Add T-AUTH-007 to plan-auth.md: Implement token refresh on 401
```

### 3.3 Discover Affected Plan Files

Dynamically discover which plan files govern the changed source paths:

```
Changed file: src/auth/client.ts
  -> Matches pattern: src/auth/*
  -> Governed by: plan-auth.md
  -> Spec: spec-auth.md

Changed file: src/data/api.ts
  -> Matches pattern: src/data/*
  -> Governed by: plan-data.md
  -> Spec: spec-data.md
```

Use file ownership tables (from prompts) or directory conventions to map source files to plan/spec files.

### 3.4 Update Context Files

For each backpropagation target, update:

1. **Spec file:** Add missing requirement with acceptance criteria
2. **Plan file:** Add task referencing the new requirement
3. **Impl tracking:** Record the backpropagation event

```markdown
# In context/impl/impl-auth.md, add:

## Backpropagation Log
| Date | Commit | Issue | Spec Update | Plan Update |
|------|--------|-------|-------------|-------------|
| 2026-03-14 | abc1234 | 401 not handled | R7 added to spec-auth.md | T-AUTH-007 added |
```

### 3.5 Run Tests

After updating context files, run the test suite to verify nothing broke:

```bash
{BUILD_COMMAND}
{TEST_COMMAND}
```

### 3.6 Generate Regression Tests

For each backpropagation target, generate a regression test that:
- Tests the specific acceptance criteria from the new spec requirement
- Would fail if the fix were reverted
- Is included in the standard test suite going forward

---

## 4. Patterns and Anti-Patterns

### Signs the process is working

| Pattern | What You Observe |
|---------|--------------------|
| **Declining manual intervention** | Each iteration cycle requires fewer hand-applied fixes because specs capture more of the ground truth |
| **Broader spec coverage per fix** | A single backpropagation event adds constraints that block an entire family of related defects, not just one |
| **Cross-domain prevention** | Prompt-level adjustments made after a bug in one module prevent analogous bugs from appearing in other modules |
| **Autonomous reproducibility** | After a spec update, the iteration loop independently produces the same correction that a human applied manually |

### Warning signs and remedies

| Anti-Pattern | Symptom | Remedy |
|-------------|---------|-----|
| **Code-only patches** | The same category of defect resurfaces across iterations | Follow the full 6-step process; never stop after the code fix in Step 1 |
| **Overly specific spec additions** | Each backpropagation prevents only the exact bug encountered, while slight variations slip through | Formulate the RULE as a general invariant, not a narrow patch |
| **Skipping verification** | Specs are updated but nobody confirms the iteration loop can reproduce the fix independently | Always execute Step 6; a spec that does not drive correct generation is incomplete |
| **Brittle over-specification** | Specs dictate implementation minutiae, causing breakage on minor refactors | Constrain the WHAT and WHY; leave the HOW to the implementation |
| **Accumulated backprop debt** | A backlog of manual fixes sits un-traced, growing with each sprint | Set a cadence (e.g., end of each iteration) to clear the backlog; debt compounds quickly |

---

## 5. When NOT to Backpropagate

Not every code fix needs backpropagation:

- **One-off environment issues** (wrong config, missing dependency) -- these are infrastructure, not spec gaps
- **Typos and formatting** -- trivial fixes that do not reflect missing requirements
- **Exploratory changes** during prototyping -- specs are still being formed
- **Performance optimizations** that do not change behavior -- unless performance is a spec requirement

**Rule of thumb:** If the iteration loop could plausibly reintroduce the bug, backpropagate. If not, skip it.

---

## 6. Backpropagation and Convergence

Backpropagation directly improves convergence:

```
Iteration 1: 350 lines changed, 8 manual fixes needed
  -> Backpropagate all 8 fixes into specs
Iteration 2: 140 lines changed, 3 manual fixes needed
  -> Backpropagate 3 fixes
Iteration 3: 30 lines changed, 1 manual fix needed
  -> Backpropagate 1 fix
Iteration 4: 10 lines changed, 0 manual fixes needed
  -> Convergence achieved
```

Every backpropagation cycle tightens the specs, so the iteration loop settles into a stable solution in fewer passes. If convergence is not improving, the most likely cause is that manual fixes are being applied without tracing them back to specifications.

**Stalled convergence paired with ongoing manual fixes is a clear sign of backpropagation debt.** The specs have not absorbed the lessons from past corrections, so the loop keeps regenerating flawed output that demands human repair.

---

## 7. Integration with Other SDD Skills

- **Convergence monitoring:** Use `sdd:convergence-monitoring` to detect when manual fixes are decreasing (good) or increasing (backpropagation debt).
- **Prompt pipeline:** Backpropagation may trigger changes to prompts (Step 6), which affects the `sdd:prompt-pipeline` design.
- **Validation-first design:** Stronger validation gates catch issues earlier, reducing the need for backpropagation.
- **Gap analysis:** Systematic gap analysis (`/sdd:gap-analysis`) identifies backpropagation targets proactively, rather than waiting for bugs.

---

## Cross-References

- **Convergence patterns:** See `references/convergence-patterns.md` for how backpropagation drives convergence.
- **Prompt pipeline:** See `sdd:prompt-pipeline` skill for how prompt 006 (rewrite pattern) implements automated backpropagation.
- **Impl tracking:** See `sdd:impl-tracking` skill for the backpropagation log format in implementation tracking documents.
- **Validation gates:** See `sdd:validation-first` skill for validation layers that catch issues before they require backpropagation.
