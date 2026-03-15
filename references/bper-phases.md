# BPER Phases Reference

Complete reference for the four-phase SDD lifecycle: **B**rainstorm, **P**lan, **E**xecute, **R**eview.

---

## 1. Overview

BPER is the four-phase lifecycle of Spec-Driven Development. Each phase has dedicated prompts that drive it, explicit inputs and outputs, and defined roles for both the AI agent and the human engineer.

The core principle that governs all phases:

> **Specify before building — never jump from raw requirements directly to implementation.**

Specifications sit between intent and implementation. Every project — whether greenfield or rewrite — must pass through a specification stage before any code is written.

| Project Type | Starting Point | Specifications | Deliverable |
|---|---|---|---|
| **Greenfield** | PRDs, design docs, domain knowledge | Technical specs decomposed from source materials | Working application with tests |
| **Rewrite** | Existing application source code | Implementation-agnostic specs extracted from current behavior | New application in target stack |

---

## 2. Phase Table

| Phase | Input | Output | AI Role | Human Role |
|-------|-------|--------|---------|------------|
| **Brainstorm** | Old code, reference docs, research | Implementation-agnostic specs | Extract, structure, decompose | Verify specs capture all requirements |
| **Plan** | Specs + framework research | Framework-specific implementation plans | Architect, sequence, define dependencies | Approve technical direction |
| **Execute** | Plans + specs | Working code + tests + tracking docs | Implement, verify, generate coverage, track progress | Observe execution and flag deviations |
| **Review** | Failed validations, gaps, manual fixes | Updated specs/plans, regression tests, progress reports | Root-cause failures, propagate fixes upstream, surface metrics | Evaluate outcomes, set priorities, initiate backpropagation |

---

## 3. Phase Details

### 3.1 Brainstorm Phase

**Purpose:** Transform source material into implementation-agnostic specifications that define WHAT needs to be built.

**Inputs:**
- Reference materials (PRDs, language specs, old code docs, design documents)
- Feature scope documents (what is in/out of scope)
- Existing codebase (for brownfield/rewrite projects)

**Outputs:**
- Domain-specific spec files (`specs/spec-{domain}.md`)
- Spec overview/index file (`specs/spec-overview.md`)
- Cross-references between related specs

**AI Role:**
- Read and analyze all reference materials
- Decompose into domain-specific specifications
- Write specs with testable acceptance criteria
- Cross-reference specs where domains interact

**Human Role:**
- Review specs for completeness and accuracy
- Verify acceptance criteria are testable
- Ensure scope is correct (not too broad, not too narrow)
- Validate domain decomposition makes sense

**Key Principles:**
- Specs are implementation-agnostic -- they describe WHAT, not HOW
- Every requirement must include testable acceptance criteria
- Specs must be hierarchical -- one index file linking to domain-specific sub-specs
- Specs must be cross-referenced -- related specs link to each other

**Spec Format Template:**
```markdown
# Spec: {Domain Name}

## Scope
{What this spec covers}

## Requirements

### R1: {Requirement Name}
**Description:** {What must be true}
**Acceptance Criteria:**
- [ ] {Testable criterion 1}
- [ ] {Testable criterion 2}
**Dependencies:** {Other specs/requirements this depends on}

### R2: ...

## Out of Scope
{Explicit exclusions}

## Cross-References
- See also: spec-{related-domain}.md
```

**Greenfield Pattern:**
- Reference material -> specs (single prompt, e.g., `001-generate-specs-from-refs.md`)
- Agent reads `context/refs/` and produces `context/specs/`

**Rewrite Pattern:**
- Old code -> reference docs -> specs (multiple prompts)
- `001`: Generate reference materials from old code
- `002`: Generate specs from reference + feature scope
- `003`: Validate specs against codebase

---

### 3.2 Plan Phase

**Purpose:** Transform implementation-agnostic specs into framework-specific implementation plans that define HOW to build.

**Inputs:**
- Specs from the Brainstorm phase
- Framework documentation and research
- Existing implementation tracking (if any)

**Outputs:**
- Domain-specific plan files (`plans/plan-{domain}.md`)
- Feature frontier document (`plans/plan-feature-frontier.md`)
- Known issues backlog (`plans/plan-known-issues.md`)

**AI Role:**
- Read specs and research framework patterns
- Architect the implementation approach
- Decompose into tasks with dependencies
- Sequence implementation order
- Define test strategies per feature

**Human Role:**
- Validate architecture decisions
- Review framework choices
- Approve dependency ordering
- Verify test strategy coverage

**Key Principles:**
- Plans are framework-specific -- they describe HOW to implement
- Plans reference specs for the WHAT
- Plans include feature dependencies (what must be built first)
- Plans include test strategies (how each feature will be validated)
- Plans include acceptance criteria (runnable checks)

**Plan Format Template:**
```markdown
# Plan: {Domain Name}

## Framework
{FRAMEWORK} — {version and key dependencies}

## Implementation Sequence

### Task T-1: {Task Name}
**Spec Reference:** spec-{domain}.md R1
**Dependencies:** None
**Files:** {files to create/modify}
**Approach:** {How to implement}
**Tests:** {Test strategy}
**Acceptance:** {BUILD_COMMAND} passes, tests pass

### Task T-2: {Task Name}
**Spec Reference:** spec-{domain}.md R2
**Dependencies:** T-1
**Files:** {files to create/modify}
...

## Feature Frontier
| Tier | Features | Dependencies |
|------|----------|-------------|
| 1 (Foundation) | Core data models, basic routing | None |
| 2 (Core) | Business logic, API integration | Tier 1 |
| 3 (Advanced) | Performance, polish, edge cases | Tier 2 |

## Known Issues
| Priority | Issue | Workaround |
|----------|-------|------------|
| P0 | {Critical blocker} | {Temporary fix} |
| P1 | {High priority} | {Approach} |
```

**Bidirectional Flow:**
Plans and implementation tracking files update each other. The Plan phase reads `impl/` for feedback from prior execution passes, and the Execute phase updates plans when it discovers new information. This bidirectional flow is expected and healthy -- it is how the system self-corrects.

---

### 3.3 Execute Phase

**Purpose:** Build working code from plans and specs, with full validation.

**Inputs:**
- Plans from the Plan phase
- Specs from the Brainstorm phase
- Implementation tracking from prior iterations

**Outputs:**
- Source code (`src/`)
- Tests (`tests/`)
- Implementation tracking documents (`impl/impl-{domain}.md`)

**AI Role:**
- Read plans and identify highest-priority unblocked task
- Implement the task
- Generate and run tests on changed files
- Update implementation tracking
- Commit progress frequently

**Human Role:**
- Monitor progress
- Review implementation tracking for anomalies
- Intervene if agent is stuck or going in wrong direction

**Key Principles:**
- Always implement the highest-priority unblocked task
- Run validation gates after each significant change (build -> test -> verify)
- Generate tests for all changed source files
- Update implementation tracking with: files created/modified, issues found, dead ends
- Commit frequently, never push (git as working memory)
- Use sub-agents for discrete subtasks to preserve context window

---

### 3.4 Review Phase

**Purpose:** Identify gaps, trace failures back to specs/plans, and observe the running system to steer direction.

**Inputs:**
- Failed validations from the Execute phase
- Gaps identified during monitoring
- Manual fixes made by humans
- Running application, git history, worktree activity

**Outputs:**
- Updated specs with missing requirements/validation
- Updated plans with corrected approaches
- Regression tests
- Systemic prompt improvements
- Issues, anomalies, progress reports
- Convergence metrics

**AI Role:**
- Diagnose the root cause of failures
- Trace issues back to spec or plan gaps
- Update specs (not just code) with missing requirements
- Generate regression tests
- Re-run validation to verify the fix emerges from updated specs alone
- Periodically scan worktrees, git history, and context changes
- Report on convergence metrics (test pass rate, change velocity)

**Human Role:**
- Serve as **reviewer and decision-maker**, not hands-on coder
- Review proposed spec changes
- Make systemic improvements to prompts when issues represent patterns
- Steer direction of backpropagation
- Make go/no-go decisions on phase transitions

**The Backpropagation Process:**
1. **Surface and resolve the defect** — identify the issue in the running application and fix it through a standard agent debugging session
2. **Trace the gap in the specification chain** — determine where in the spec/plan/prompt hierarchy the requirement slipped through
3. **Patch the spec with the missing requirement** — update the specification to capture the requirement or validation rule that was absent
4. **Propagate changes to plans and tracking documents** — map the fix back into the relevant context files so downstream artifacts stay consistent
5. **Apply systemic prompt corrections if the issue represents a pattern** — when a defect class recurs, update prompts to prevent the category of error
6. **Re-execute the loop and add regression coverage** — confirm the fix emerges from updated specs without manual intervention, and expand tests to guard against recurrence

**Key Metrics:**
- Test pass rate (approaching 100%)
- Change velocity (decreasing = converging)
- Forward progress (% of spec requirements with passing tests)
- Dead end accumulation (increasing = possible spec problem)

**Key Insight:** A code-only fix that can't be traced to a spec gap means the specifications need strengthening. The goal is that specs plus the execution loop can reproduce any fix autonomously.

---

## 4. The Human is an Auditor, Not an Implementer

This is a critical principle throughout BPER:

> The human monitors the process, requests changes as needed, and makes systemic improvements to specs and prompts. The human does NOT write code.

**What the human does:**
- Reviews specs for completeness and accuracy
- Validates architecture decisions in plans
- Monitors execution progress
- Audits review results
- Steers direction when agents are off track
- Makes systemic improvements to prompts
- Triggers backpropagation for discovered issues
- Makes go/no-go decisions at phase gates

**What the human does NOT do:**
- Write code directly
- Fix bugs by editing source files
- Implement features
- Write tests manually

When the human discovers a bug, the correct action is to trace it back to a spec gap and fix the spec, not to fix the code. The execution loop should then reproduce the fix autonomously from the updated specs.

---

## 5. Phase Gates (Transition Criteria)

Phase gates are mandatory verification checkpoints between phases. No phase transition occurs without passing its gate.

### 5.1 Brainstorm -> Plan Gate

| Criterion | Verification |
|-----------|-------------|
| All domains identified and spec files created | Spec overview lists all domains |
| Every requirement has testable acceptance criteria | Review each `R-` requirement for `[ ]` criteria |
| Cross-references are complete | Each spec links to related specs |
| Scope is defined (in-scope and out-of-scope) | Each spec has explicit exclusions |
| Human has reviewed and approved specs | Human sign-off |

### 5.2 Plan -> Execute Gate

| Criterion | Verification |
|-----------|-------------|
| All spec requirements mapped to plan tasks | Cross-reference check |
| Task dependencies are defined and acyclic | Dependency graph review |
| Test strategies defined for each feature | Each task has test approach |
| Feature frontier established | Tier system documented |
| Framework research complete | Plan references framework docs |
| Human has reviewed architecture decisions | Human sign-off |

### 5.3 Execute -> Review Gate

| Criterion | Verification |
|-----------|-------------|
| Build passes | `{BUILD_COMMAND}` exits cleanly |
| Unit tests pass | `{TEST_COMMAND}` exits cleanly |
| Implementation tracking is current | `impl/` files reflect actual state |
| All completed tasks verified | Each DONE task has passing tests |
| No P0 issues outstanding | `plan-known-issues.md` check |

### 5.4 Review -> Brainstorm Gate (Cycle Back)

| Criterion | Verification |
|-----------|-------------|
| All backpropagation targets addressed | Spec changes committed |
| Regression tests generated and passing | Test suite expanded |
| Execution loop re-run confirms fixes | Clean iteration pass |
| Implementation tracking updated | Dead ends documented |
| Convergence detected or ceiling diagnosed | Change velocity analysis |
| Gap analysis complete | Built vs intended comparison |
| New requirements or scope changes identified | Spec updates needed |
| Human decision to cycle back | Explicit go/no-go |

---

## 6. The CI Pipeline Analogy

The SDD lifecycle mirrors a build pipeline — each stage transforms inputs into verified outputs:

**Traditional CI/CD:**
```
Code -> Build -> Test -> Deploy
```

**SDD AI Pipeline:**
```
Spec Change (Brainstorm)
  -> Generate Plans (Plan)
    -> Generate Implementation (Execute)
      -> Validate (Tests + Review)
        -> Human Audit (Review & Steer)
          -> [Gap Found]
            -> Backpropagate (Review)
              -> Spec Change (cycle back to Brainstorm)
```

Each stage feeds the next. Failures at any stage propagate back to the appropriate source (spec, plan, or prompt) rather than being patched at the code level.

---

## 7. When to Use Full BPER vs. Lightweight SDD

### Full BPER

Use when:
- The project spans multiple modules or has significant architectural surface area
- Requirements are expected to shift, requiring specs and code to evolve together
- The workflow involves coordinating multiple agents or chained prompt stages
- You are working on production or brownfield systems where change traceability is essential
- Multiple teams collaborate and need a shared specification layer to stay aligned
- The codebase handles sensitive operations where validation gates reduce risk
- Agents will run extended autonomous sessions without continuous human oversight

### Lightweight SDD

Use when:
- The task is focused but non-trivial
- You want spec benefits without full pipeline overhead

**Lightweight approach:**
1. Write a focused `context/specs/spec-task.md` capturing requirements
2. Add a `context/plans/plan-task.md` sequencing the implementation
3. Skip full BPER; just run the execution loop against the plan

This minimal approach captures the key advantages — clear intent, reproducible outcomes, traceable decisions — without a full multi-phase setup.

### Skip SDD

When:
- The task is small and self-contained (~5 files, clear requirements, single session)
- One-off standalone tools with well-defined scope
- Exploratory prototyping where requirements are completely unknown
- Simple bug fixes or small feature additions

**Heuristic:** If the entire task fits comfortably in a single agent session, full SDD adds unnecessary ceremony.

---

## 8. Prompt Pipeline Patterns

### Greenfield Pattern (3-Prompt)

| Prompt File | Lifecycle Stage | Reads From | Produces |
|--------|-------------|-------|--------|
| `001-generate-specs-from-refs.md` | **Brainstorm** | `context/refs/` | `context/specs/` |
| `002-generate-plans-from-specs.md` | **Plan** | `context/specs/` | `context/plans/` |
| `003-generate-impl-from-plans.md` | **Execute** | `context/plans/` + `context/specs/` | `src/`, `tests/`, `context/impl/` |

### Rewrite Pattern (6-9 Prompts)

| Prompt File | Lifecycle Stage | Reads From | Produces |
|--------|-------------|-------|--------|
| `001-generate-refs-from-code.md` | **Brainstorm** (prep) | Old app source | `shared-context/reference/` |
| `002-generate-specs.md` | **Brainstorm** | Feature scope + reference | `shared-context/specs/` |
| `003-validate-specs.md` | **Brainstorm** (verify) | Reference + specs | Validation report |
| `004-create-plans.md` | **Plan** | Specs + framework research | `context/plans/` |
| `005-implement.md` | **Execute** | Plans + specs | `src/` + `tests/` |
| `006-update-specs.md` | **Review** | Working prototype | Updated specs |

The final prompt (006) feeds corrections back into prompt 002, closing the loop.

### Shared Principles Across All Pipelines

| Principle | Description |
|-----------|-------------|
| Single prompt per lifecycle stage | Each phase boundary is a clean handoff point |
| Declared read/write paths | Every prompt has an explicit contract for its inputs and outputs |
| Git as session memory | Agents reconstruct state from commit history across iterations |
| Deterministic exit conditions | A structured signal (`<all-tasks-complete>`) marks completion |
| Two-way spec/plan synchronization | Implementation feedback flows back into plans and vice versa |
| Automatic coverage on touched files | Any source file change triggers corresponding test generation |

---

## 9. Context Directory Structure

Every SDD project follows this standard structure:

```
context/
+-- refs/           # Source of truth (language specs, PRDs, old code docs)
+-- specs/          # Implementation-agnostic specifications
|   +-- CLAUDE.md   # "Specs define WHAT needs implementing"
+-- plans/          # Framework-specific implementation plans
|   +-- CLAUDE.md   # "Plans define HOW to implement something"
+-- impl/           # Living implementation tracking
|   +-- CLAUDE.md   # "Impls record implementation progress"
+-- prompts/        # BPER pipeline prompts (001, 002, 003...)
```

Each subdirectory gets a `CLAUDE.md` that describes its conventions. Agents automatically load these when working in that directory. CLAUDE.md is hierarchical -- it loads from the directory AND all parent directories.

---

## 10. Why Specifications Matter

> Robust specifications with comprehensive validation make the entire application rebuildable from documentation alone.

This principle underpins the SDD approach to durability. Specs are:
- **Structured** — organized as a navigable tree, enabling agents to load only what they need
- **Human-legible** — engineers can audit at a higher level than code
- **Stack-independent** — decoupled from any single framework or language
- **Independently evolvable** — specs can be refined without touching implementation
- **Verifiable** — every requirement includes acceptance criteria agents can check

The same specs can drive implementations across different frameworks, enabling apples-to-apples comparison of technology choices.
