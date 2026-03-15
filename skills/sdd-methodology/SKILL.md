---
name: sdd-methodology
description: |
  Core Spec-Driven Development (SDD) methodology — the master skill that teaches the BPER lifecycle
  and routes to all sub-skills. Covers the Specify Before Building principle, the scientific method analogy,
  the four-phase BPER lifecycle, decision matrix for when to use SDD, and build pipeline analogy.
  Trigger phrases: "use SDD", "spec-driven", "start SDD project", "sdd methodology",
  "how should I structure this project for AI agents"
---

# Spec-Driven Development (SDD) Methodology

## Core Principle: Specify Before Building

**Always define what you want before telling agents how to build it. Go through a specification stage — never jump straight from raw requirements to implementation.**

SDD is a methodology for building software with AI coding agents that **puts specifications at the center of the development process — code is derived from them, not the other way around**. Whether starting from scratch or modernizing an existing system, the principle is the same:

- **Greenfield projects:** reference material → specs → code
- **Rewrites:** old code → specs → new code

In both cases, the specs become a living contract that agents consume to continuously build, validate, and refine the application.

### Why Specs Are the First-Class Citizen

| Property | Benefit |
|----------|---------|
| **Structured** | Organized as a navigable tree, enabling agents to load only what they need |
| **Human-legible** | Engineers can audit requirements at a higher level than code |
| **Stack-independent** | Decoupled from any single framework or language |
| **Independently evolvable** | Specs can be refined without touching implementation |
| **Verifiable** | Every requirement includes acceptance criteria agents can check |

> **Key Insight:** Well-written specs with strong validation make your application reproducible — any agent can rebuild it from the specifications alone. Think of it as continuous regeneration.

---

## The Scientific Method Analogy

LLMs are inherently non-deterministic — like running an experiment, each individual call may yield different results. But through the right methodology — clear hypotheses, controlled conditions, and repeated trials — we extract reliable, reproducible outcomes from a stochastic process.

**SDD applies the scientific method to software construction — hypothesize, test, observe, refine.**

| Layer | Analogy | What It Does |
|-------|---------|-------------|
| **LLM calls** | Individual experiments | Each run may produce different results; no single output is authoritative |
| **Specifications** | Hypotheses | Define what you expect to observe — the predicted behavior |
| **Validation gates** | Controlled conditions | Ensure reproducibility by constraining what counts as a valid outcome |
| **Convergence loops** | Repeated trials | Build statistical confidence through successive passes |
| **Implementation tracking** | Lab notebook | Record what was tried, what worked, and what failed |
| **Backpropagation** | Revising the hypothesis | When results contradict expectations, update the theory upstream |

The outcome: a disciplined, repeatable engineering process layered on top of probabilistic generation.

---

## The 5 BPER Phases

BPER stands for **Brainstorm, Plan, Execute, Review**. Each phase has dedicated prompts that drive it.

| Phase | Input | Output | AI Role | Human Role |
|-------|-------|--------|---------|------------|
| **Spec** | Source materials, domain knowledge, existing systems | Implementation-agnostic specs | Extract requirements, structure knowledge | Verify specs capture intent accurately |
| **Plan** | Specs + framework research | Framework-specific implementation plans | Design architecture, break down work, order steps | Approve architectural choices |
| **Implement** | Plans + specs | Working code + tests + tracking docs | Write code, run tests, check against specs | Watch for drift and blockers |
| **Iterate** | Failed validations, gaps, manual fixes | Updated specs/plans via backpropagation | Identify root causes, propagate fixes upstream | Evaluate outcomes, set priorities |
| **Monitor** | Running application, git history | Issues, anomalies, progress reports | Scan for regressions, surface metrics | Interpret reports, guide next steps |

### Phase Transitions

Each phase has **gate conditions** that must be met before moving to the next:

1. **Spec → Plan:** All domains have specs with testable acceptance criteria. Human has reviewed for completeness.
2. **Plan → Implement:** Plans reference specs, define implementation sequence, and include test strategies. Architecture decisions validated.
3. **Implement → Iterate:** Code builds, tests pass at current coverage level, implementation tracking is up to date.
4. **Iterate → Monitor:** Convergence detected (changes decreasing iteration-over-iteration). Remaining changes are trivial.
5. **Monitor → Spec (cycle):** Gap found or new requirement identified. Backpropagate to specs and restart the cycle.

The **Iterate** phase is where the human serves as **reviewer and decision-maker**, not hands-on coder. You monitor the process, request changes as needed, and make systemic improvements to specs and prompts.

> For the full BPER phase reference, see `references/bper-phases.md`.

---

## Decision Matrix: When to Use SDD

### Full SDD

Use when the project has significant scope, evolving requirements, or needs autonomous agent execution.

| Indicator | Threshold |
|-----------|-----------|
| Codebase size | 50+ source files |
| Requirements | Evolving, multi-domain |
| Agent coordination | Multi-agent or multi-prompt pipelines |
| Environment | Production, security-sensitive, brownfield |
| Team structure | Multi-team or cross-team |
| Execution mode | Long-running autonomous work (overnight, unattended) |

**What you get:** Full BPER lifecycle, context directory with specs/plans/impl tracking, prompt pipeline, convergence loops, backpropagation, validation gates.

### Lightweight SDD

Use when scope is moderate — too complex for ad-hoc but not worth a full pipeline.

| Indicator | Threshold |
|-----------|-----------|
| Codebase size | 5-50 files |
| Requirements | Mostly clear, focused |
| Agent coordination | Single agent, possibly with sub-agents |
| Execution mode | Interactive with occasional iteration loops |

**What you do:**
1. Write a focused `context/specs/spec-task.md` capturing requirements
2. Add a `context/plans/plan-task.md` sequencing the implementation
3. Skip full BPER — just run an iteration loop against the plan

This is the "SDD floor" — most of the benefit without the overhead of a full multi-phase pipeline.

### Skip SDD

Use when the task is trivially small.

| Indicator | Threshold |
|-----------|-----------|
| Codebase size | Less than 5 files |
| Task type | One-off tools, simple bug fixes, exploratory prototypes |
| Implementation | Fits comfortably in one agent session without needing external references |

**Heuristic:** If the whole task fits in one context window with room to spare, full SDD adds more overhead than value.

### Growth Path

Start with lightweight SDD even if the project is small. If the scope expands, you already have the structure in place to scale up. It is much harder to retrofit specs onto a large codebase than to grow a spec directory from the beginning.

---

## The CI Pipeline Analogy

SDD mirrors a **build pipeline** — each stage transforms input into validated output, with feedback loops that propagate corrections upstream:

```
Traditional CI/CD:
  Code → Build → Test → Deploy

SDD AI Pipeline:
  Spec Change
    → Generate Plans (iteration loop)
    → Generate Implementation (iteration loop)
    → Validate (Tests + Review)
    → Human Audit (Monitor & Steer)
    → [Gap Found]
    → Backpropagate
    → Spec Change (cycle repeats)
```

Every stage can run as an iteration loop — the same prompt executed repeatedly until output stabilizes. The iteration loop is what transforms nondeterministic LLM output into predictable, validated software.

### The Iteration Loop

The iteration loop is the fundamental execution unit in SDD. Execute the same prompt against the same codebase multiple times until the delta between runs approaches zero.

**Mechanics:**
1. Execute a prompt against the current codebase
2. The agent inspects git history and tracking documents to understand what has already been done
3. The agent applies changes and commits its progress
4. Return to step 1

**Convergence signal:** A shrinking volume of modifications across successive passes — the diff gets smaller each time until only cosmetic changes remain. You are looking for diminishing returns, not absolute zero.

**When the loop isn't stabilizing, the problem is upstream — fix the inputs (specs, validation, coordination), not the iteration count.**

If the diff is not shrinking between runs:
- Specifications are ambiguous (agents interpret them differently each time)
- Validation criteria are too loose (the agent has no way to confirm it got things right)
- Multiple agents are overwriting each other's work (ownership boundaries are unclear)

---

## Cross-References to Sub-Skills

SDD is composed of techniques that work together. This methodology skill is the index — each sub-skill below is self-contained but cross-references others.

### Foundation Skills

| Skill | Purpose | When to Use |
|-------|---------|-------------|
| `sdd:spec-writing` | Write implementation-agnostic specs with testable acceptance criteria | Spec phase — always the first step |
| `sdd:context-architecture` | Organize context for progressive disclosure | Project setup and ongoing maintenance |
| `sdd:impl-tracking` | Track implementation progress, dead ends, test health | Implement and Iterate phases |
| `sdd:validation-first` | Design validation gates agents can execute | All phases — validation is continuous |

### Pipeline Skills

| Skill | Purpose | When to Use |
|-------|---------|-------------|
| `sdd:prompt-pipeline` | Design numbered prompt pipelines for BPER | Setting up automation |
| `sdd:backpropagation` | Trace bugs back to specs and fix at the source | Iterate phase — after finding gaps |
| `sdd:brownfield-adoption` | Adopt SDD on existing codebases | Starting SDD on legacy projects |

### Advanced Skills

| Skill | Purpose | When to Use |
|-------|---------|-------------|
| `sdd:adversarial-review` | Use a second agent to challenge the first | Quality gates, architecture review |
| `sdd:speculative-pipeline` | Stagger pipeline stages for parallelism | Optimizing long pipelines |
| `sdd:convergence-monitoring` | Detect convergence vs ceiling | Monitoring iteration loops |
| `sdd:documentation-inversion` | Turn documentation into agent-consumable skills | Library/module documentation |

### Integration with Existing Skills

SDD works **with** existing skills, not as a replacement:

| Existing Skill | SDD Integration |
|----------------|-----------------|
| `superpowers:brainstorming` | Use during spec generation to explore requirements |
| `superpowers:writing-plans` | Use during plan generation for structured planning |
| `superpowers:test-driven-development` | TDD-within-SDD: spec acceptance criteria become failing tests |
| `superpowers:verification-before-completion` | Use for gate validation in every phase |
| `superpowers:executing-plans` | Use during implementation phase |
| `superpowers:dispatching-parallel-agents` | Use for agent team coordination |

---

## Quick Start

### For a New Project (Greenfield)

1. **Set up context directory:**
   ```
   context/
   ├── refs/           # Source materials (PRDs, language specs, research)
   ├── specs/          # Implementation-agnostic specifications
   ├── plans/          # Framework-specific implementation plans
   ├── impl/           # Living implementation tracking
   └── prompts/        # BPER pipeline prompts
   ```

2. **Write specs** from your reference materials (see `sdd:spec-writing`)
3. **Generate plans** from specs (see `sdd:prompt-pipeline`)
4. **Implement** with validation gates (see `sdd:validation-first`)
5. **Track progress** in implementation documents (see `sdd:impl-tracking`)
6. **Iterate** — when gaps are found, backpropagate to specs (see `sdd:backpropagation`)

### For an Existing Project (Brownfield)

1. **Set up context directory** (same structure as above)
2. **Designate existing codebase as reference material**
3. **Generate specs from code** (see `sdd:brownfield-adoption`)
4. **Validate specs match behavior** — run tests against generated specs
5. **Proceed with normal BPER** — future changes flow through specs first

---

## Summary

SDD is not a tool — it is a methodology. The core loop is simple:

1. **Describe what you want** (specs with testable criteria)
2. **Let agents build it** (plans → implementation → validation)
3. **Fix the specs, not the code** (backpropagation)
4. **Repeat until converged** (iteration loops)

Agents become more capable the more precisely you constrain them — clear specifications, automated validation, and structured iteration loops let them operate with increasing autonomy. None of this eliminates the need for software engineers. Your judgment on architecture, your ability to write precise specs, and your instinct for what "done" looks like are the inputs that make the whole system function. SDD is a force multiplier: one engineer's clarity of thought, scaled across an entire implementation pipeline.
