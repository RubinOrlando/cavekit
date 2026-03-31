# Deep Research for `/bp:draft` — Design Spec

## Overview

A parallel multi-agent research system that runs during the draft phase to ground blueprint design in real evidence — current best practices from the web and deep understanding of the existing codebase. The system follows a DAG-based orchestrator pattern with two-pass synthesis, producing a named research brief that becomes a permanent artifact in the context hierarchy.

The core insight: **blueprints designed without research are blueprints designed on vibes.** An agent building a compiler should know how other compilers solve the same problems. An agent building auth should know the current best practices. Research turns the design conversation from "what do you want?" into "here's what we know — what do you want?"

---

## Architecture

### Research Pipeline

```
User describes project
    |
    v
Lead agent: 1-2 scoping questions
    |
    v
Intelligent trigger: "Deep research recommended? [Y/n]"
    |
    v (yes)
Lead generates research plan:
  - Fixed categories (template)    --+
  - Project-specific sub-questions --+--> Research DAG
  - Dependency ordering            --+
    |
    v
Parallel dispatch:
  +--- Codebase Agent 1 (architecture) ---+
  +--- Codebase Agent 2 (patterns)     ---+
  +--- Web Agent 1 (library landscape) ---+--> Raw findings
  +--- Web Agent 2 (best practices)    ---+    (per agent)
  +--- Web Agent 3 (existing art)      ---+
    |
    v
Pass 1: Synthesizer subagent
  - Cross-validates findings
  - Flags contradictions
  - Ranks confidence
  - Produces compressed brief
    |
    v
Pass 2: Lead agent reads brief
  - Presents summary to user
  - Uses findings to inform design conversation
```

### Output Artifacts

```
context/refs/
+-- research-brief-{topic}.md              # Synthesis (index node)
+-- research-{topic}/
    +-- findings-board.md                  # Shared coordination state
    +-- raw-codebase-architecture.md       # Raw finding per agent
    +-- raw-codebase-patterns.md
    +-- raw-web-library-landscape.md
    +-- raw-web-best-practices.md
    +-- raw-web-existing-art.md
    +-- raw-web-pitfalls.md
```

These artifacts live in `context/refs/` per the context hierarchy spec — they are source material (Tier 1) that feeds into blueprints (Tier 2).

---

## Research Categories

### Fixed Template

Every research session uses these baseline categories. The lead agent adds project-specific sub-questions on top.

**Codebase categories** (skip if greenfield):

| Category | What it investigates | Agent role |
|----------|---------------------|------------|
| Architecture | Directory structure, module boundaries, entry points, build system, framework version | Architecture mapper |
| Patterns | Coding conventions, error handling, naming, state management, abstractions in use | Pattern detective |
| Dependencies | package.json/Cargo.toml, external APIs, integration points, version constraints | Dependency analyst |
| Test infrastructure | Test framework, coverage, fixtures, CI config, what's tested vs. not | Test scout |

**Web categories** (always run):

| Category | What it investigates | Example sub-questions |
|----------|---------------------|----------------------|
| Library landscape | Libraries/tools for the core problem domain | "TypeScript parser generators 2026", "WASM codegen libraries" |
| Best practices | Current architectural patterns, security, performance | "OAuth 2.1 best practices", "WASM memory management patterns" |
| Existing art | How others solved similar problems, reference implementations | "open source Verse compilers", "functional language to WASM" |
| Pitfalls | Known issues, common mistakes, deprecated approaches | "WASM codegen pitfalls", "parser generator anti-patterns" |

### Project-Specific Sub-Questions

The lead agent generates 2-6 additional sub-questions based on the user's project description. These are assigned to the most relevant category agent. Examples:

- "Build a Verse compiler targeting WASM" → "How do existing effect systems handle type inference in compilation?" (assigned to existing-art agent)
- "Add real-time collaboration to our editor" → "What CRDT libraries exist for TypeScript in 2026?" (assigned to library-landscape agent)

### Dependency Ordering

If sub-questions have dependencies ("can't research WASM codegen patterns until we know which parser library we're using"), the lead agent orders them. Independent questions run in parallel; dependent questions wait for their prerequisite to post to the findings board.

In practice, most research sub-questions are independent. Dependencies are the exception, not the rule.

---

## Adaptive Agent Count

The lead agent assesses project size during Step 1 (existing context exploration) and determines agent count:

| Project state | Codebase agents | Web agents |
|---------------|----------------|------------|
| Greenfield (no existing code) | 0 | 2-4 |
| Small existing project (<20 files) | 1 (covers all 4 categories) | 2-3 |
| Medium project (20-200 files) | 2-3 | 2-4 |
| Large project (200+ files) | 4 | 2-4 |

Web agent count scales with the number of distinct research categories that are relevant. A project using well-known technology (React + Postgres) needs fewer web agents than one using novel technology (Verse + WASM).

---

## Shared Findings Board

Web research agents coordinate via a shared markdown file to avoid duplicate searches and build on each other's discoveries.

**Location:** `context/refs/research-{topic}/findings-board.md`

**Format:**
```markdown
# Findings Board

## Agent: library-landscape
- Found: nearley.js is the most popular TypeScript parser generator (npm: 2.1M/week)
- Found: tree-sitter has TypeScript bindings, used by GitHub for syntax highlighting
- Found: chevrotain is fastest pure-JS parser, no codegen step needed
- Confidence: HIGH (multiple sources agree)

## Agent: best-practices
- Found: WASM component model is the 2026 standard for module composition
- Found: binaryen IR preferred over hand-writing WAT
- Conflict: Some sources say tree-sitter, others say pest for parser perf -- needs investigation
- Confidence: MEDIUM (evolving space)
```

**Agent protocol:**
1. Read the findings board before starting searches
2. If another agent already answered a sub-question, skip it and go deeper on unanswered ones
3. Append findings as they go — later agents see earlier agents' work
4. Flag contradictions explicitly with "Conflict:" prefix
5. Include confidence level per section (HIGH/MEDIUM/LOW)

**Staggered dispatch:** Agents that produce foundational findings (library landscape, existing art) dispatch slightly before agents that build on them (best practices, pitfalls). The stagger is small (2-3 seconds) — just enough for first agents to post initial findings.

```
t=0s   library-landscape agent starts
t=0s   existing-art agent starts
t=2s   best-practices agent starts (reads board first)
t=2s   pitfalls agent starts (reads board first)
```

Codebase agents run fully in parallel with no stagger — they explore independent dimensions of the codebase.

---

## Two-Pass Synthesis

### Pass 1: Synthesizer Subagent

A dedicated subagent that receives all raw findings + the findings board. It does not search or fetch — it only reads and synthesizes.

**Responsibilities:**
1. **Cross-validate** — when multiple agents found the same thing independently, mark as high confidence
2. **Resolve contradictions** — the findings board flags conflicts; the synthesizer investigates (reads both raw findings in detail) and picks a side or preserves both with reasoning
3. **Rank by relevance** — filter out tangential findings that don't affect design decisions
4. **Structure the brief** — organize into sections that map to design decisions the lead agent will need to make
5. **Flag unknowns** — areas where research was inconclusive or sources disagreed, surfaced as explicit questions for the user

### Research Brief Format

**File:** `context/refs/research-brief-{topic}.md`

```markdown
# Research Brief: {topic}

**Generated:** {timestamp}
**Agents:** {N} codebase, {N} web
**Duration:** {seconds}s
**Sources consulted:** {N}

## Summary
{2-3 sentence executive summary of key findings}

## Key Findings

### Architecture & Patterns
- {finding} [confidence: HIGH/MEDIUM/LOW] [sources: N]
- {finding} ...

### Library Landscape
- Recommended: {library X} — {why} [confidence: HIGH]
- Alternative: {library Y} — {tradeoff} [confidence: MEDIUM]
- Avoid: {library Z} — {why} [confidence: HIGH]

### Best Practices
- {finding with source attribution}

### Existing Art
- {reference implementation} — {what we can learn from it}
- {reference implementation} — {what we can learn from it}

### Pitfalls to Avoid
- {common mistake} — {why it matters, how to prevent}

## Contradictions & Open Questions
- {topic}: Source A says X, Source B says Y. Assessment: {synthesizer's take or "needs user input"}

## Codebase Context
{only if brownfield}
- Architecture: {summary}
- Key patterns: {conventions, frameworks, abstractions}
- Dependencies: {notable constraints}
- Test coverage: {state of testing}

## Implications for Design
- {design decision this research informs}
- {constraint this research reveals}
- {opportunity this research surfaces}

## Sources
- [Title](URL) — {one-line what it contributed}
```

### Pass 2: Lead Agent

The lead agent loads only `research-brief-{topic}.md` — never the raw findings. It presents a condensed summary to the user:

```
Research complete (45s). Key findings:
- Parser: chevrotain recommended over nearley.js (faster, no codegen, TS-native)
- WASM: binaryen IR preferred over hand-writing WAT
- Existing art: AssemblyScript compiler is closest reference architecture
- Open question: WASM component model vs. classic module -- needs your input

Full brief: context/refs/research-brief-verse-compiler.md
```

The "Contradictions & Open Questions" and "Implications for Design" sections become inputs to the design conversation — open questions become clarifying questions, implications inform approach proposals.

---

## Intelligent Trigger

The research phase is not always warranted. The lead agent decides based on the scoping conversation.

### Trigger Criteria

Research is recommended when any of:
- The project involves technology the agent has low confidence about (novel frameworks, niche domains)
- The project is brownfield and the codebase is medium-to-large (deep codebase understanding needed)
- The user's description implies architectural decisions with multiple viable approaches
- The domain has fast-moving best practices (security, AI/ML, web frameworks)

Research is NOT recommended when:
- The project is a small config change or bug fix
- The technology stack is well-understood and the agent has high confidence
- The user has already provided comprehensive reference materials in `context/refs/`
- The user explicitly asked for a quick draft

### User Prompt

When triggered:
```
This project touches {topics} which would benefit from deep research —
current best practices, library landscape, and {codebase analysis if brownfield}.
This typically takes 30-60 seconds and produces a research brief.

Run deep research? [Y/n]
```

If the user declines, the draft proceeds without research. No penalty, no nag.

---

## `/bp:research` Standalone Command

### Usage

```
/bp:research <description> [--depth quick|standard|deep] [--web-only] [--codebase-only]
```

### Arguments

- `description` — what you're building (required)
- `--depth` — controls agent count and search breadth:
  - `quick` — 1 codebase + 2 web agents, 1 search per sub-question
  - `standard` (default) — adaptive codebase + 3 web agents, 2-3 searches per sub-question
  - `deep` — max codebase (4) + 4 web agents, exhaustive searches, follows cross-references
- `--web-only` — skip codebase research (useful for greenfield)
- `--codebase-only` — skip web research (air-gapped environments)

### Flow

1. Generate research plan (fixed categories + project-specific sub-questions)
2. Assess project size for adaptive agent count
3. Dispatch parallel agents
4. Two-pass synthesis → research brief
5. Save artifacts to `context/refs/`
6. Print summary with path to brief

### Output

Same artifacts as inline research — `research-brief-{topic}.md` + `research-{topic}/` directory with raw findings.

---

## Integration into `/bp:draft`

### Updated Flow

```
Step 1:  Ensure directories exist (existing)
Step 2:  Explore project context -- shallow scan (existing)
Step 3:  Gather input
  3a:    Offer visual companion (existing)
  3b:    Scoping questions -- 1-2 questions (existing)

  -- NEW: Research integration --------------------------
  3c:    Check for existing research brief
         -> if fresh brief exists:
            "Found research brief: research-brief-{topic}.md
             ({age} ago). Use it? [Y/n/rerun]"
            Y: load brief, skip to 3e
            n: ignore brief, skip to 3e
            rerun: run fresh research (3d)
         -> if no brief, assess complexity/novelty
         -> if warrants research:
            "This would benefit from deep research. Run it? [Y/n]"
            Y: proceed to 3d
            n: skip to 3e

  3d:    Research phase (if triggered)
         -> Generate research plan
         -> Dispatch parallel agents (codebase + web)
         -> Two-pass synthesis -> research brief
         -> Present summary to user
  -------------------------------------------------------

  3e:    Clarifying questions -- one at a time (existing)
         -> Open questions from brief become clarifying questions
         -> Findings inform multiple-choice options
  3f:    Propose 2-3 approaches (existing)
         -> Approaches grounded in research evidence
  3g:    Present design incrementally (existing)

Step 4:  Decompose into domains (existing)
Step 5:  Generate blueprints (existing)
...rest unchanged
```

### Brief Staleness

If an existing brief is older than 24 hours (configurable), the prompt mentions the age:
```
Found research brief from 3 days ago. Rerun for fresh results? [Y/n]
```

### What Changes in Existing Steps

- **Clarifying questions (3e):** The lead agent references research findings. Instead of "what auth approach do you want?" it asks "research shows OAuth 2.1 + PKCE is the current standard, and your codebase already has passport.js — build on that or switch?"
- **Propose approaches (3f):** Approaches are backed by evidence. "Approach A uses chevrotain (research confirms 3x faster than nearley for grammars of this size)."
- **Blueprint generation (Step 5):** Blueprints can reference the research brief as a source — "See research-brief-verse-compiler.md for library evaluation."

### What Does NOT Change

- The hard gate ("do NOT generate blueprints until design is approved")
- The design approval flow
- The blueprint-reviewer loop
- The Codex design challenge
- Research feeds into the conversation. It does not bypass any gates.

---

## Subagent Specifications

### Codebase Research Agent

```
Agent(
  subagent_type: "Explore",
  prompt: "ROLE: {category} researcher for {project description}

  RESEARCH QUESTIONS:
  {list of sub-questions assigned to this agent}

  INSTRUCTIONS:
  1. Explore the codebase focused on {category}
  2. Answer each research question with specific evidence (file paths, code examples)
  3. Note patterns, conventions, and constraints that affect design decisions
  4. Flag anything surprising or concerning

  OUTPUT FORMAT:
  For each finding:
  - Finding: {what you found}
  - Evidence: {file:line references}
  - Implication: {how this affects the design}
  - Confidence: HIGH/MEDIUM/LOW"
)
```

Model: Sonnet (focused task, cost-efficient).

### Web Research Agent

```
Agent(
  subagent_type: "general-purpose",
  prompt: "ROLE: {category} web researcher for {project description}

  RESEARCH QUESTIONS:
  {list of sub-questions assigned to this agent}

  SHARED FINDINGS (read before searching):
  {contents of findings-board.md}

  INSTRUCTIONS:
  1. Read the findings board — skip questions already answered
  2. For each unanswered question, search the web (2-3 searches)
  3. Fetch and read the most relevant results
  4. Extract specific, actionable findings
  5. Append your findings to the findings board format
  6. Flag contradictions with other agents' findings

  QUALITY RULES:
  - Prefer official docs, GitHub repos, and engineering blogs over SEO content
  - Include publication dates — findings from 2024+ only
  - Note when a library/practice is deprecated or superseded
  - Always include source URLs

  OUTPUT FORMAT:
  ## Agent: {category}
  - Found: {finding} [source: URL]
  - Found: {finding} [source: URL]
  - Conflict: {if contradicts another agent's finding}
  - Confidence: HIGH/MEDIUM/LOW"
)
```

Model: Sonnet (web search + fetch, cost-efficient).

### Synthesizer Agent

```
Agent(
  subagent_type: "general-purpose",
  prompt: "ROLE: Research synthesizer for {project description}

  You have raw findings from {N} research agents. Your job is to produce
  a single coherent research brief.

  RAW FINDINGS:
  {contents of all raw-*.md files}

  FINDINGS BOARD:
  {contents of findings-board.md}

  INSTRUCTIONS:
  1. Cross-validate: findings confirmed by multiple agents = HIGH confidence
  2. Resolve contradictions: read both sides, pick one or preserve both with reasoning
  3. Filter: remove tangential findings that don't affect design decisions
  4. Structure: organize into the research brief format below
  5. Flag unknowns: areas needing user input become 'Open Questions'
  6. Extract implications: what design decisions does this research inform?

  OUTPUT: Follow the research brief format exactly.
  {research brief template}"
)
```

Model: Opus (complex reasoning task — cross-validation, contradiction resolution, implication extraction).

---

## Summary of Decisions

| Decision | Choice |
|----------|--------|
| Trigger | Intelligent — scoping first, offer if warranted, user confirms |
| Research domains | Codebase + web (refs handled by existing mode) |
| Decomposition | Fixed categories as template + project-specific sub-questions + DAG ordering |
| Codebase agents | Adaptive 1-4 based on project size |
| Web agents | Per-category with shared findings board, staggered start |
| Coordination | Findings board — agents read before searching, append as they go |
| Output | Named brief + raw findings directory in context/refs/ |
| Synthesis | Two-pass — synthesizer subagent compresses, lead agent reads clean brief |
| Commands | Hybrid — inline in /bp:draft + standalone /bp:research |
| Brief reuse | /bp:draft detects existing briefs, offers reuse/rerun with staleness check |
| Existing flow | Research inserted between scoping and design Q&A, no gates bypassed |
| Agent models | Codebase/web: Sonnet (cost-efficient), Synthesizer: Opus (complex reasoning) |

## Sources

Research that informed this design:
- [How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system) — Anthropic's orchestrator-worker pattern, 3-5 parallel subagents, token allocation insights
- [Inside the Architecture of a Deep Research Agent](https://www.egnyte.com/blog/post/inside-the-architecture-of-a-deep-research-agent/) — DAG-based research planning, map/reduce parallel execution, shared state management
- [Multi-Agent Deep Research Architecture](https://trilogyai.substack.com/p/multi-agent-deep-research-architecture) — Topic decomposition, findings board coordination, confidence scoring, contradiction resolution
- [Introducing deep research](https://openai.com/index/introducing-deep-research/) — OpenAI's multi-agent pipeline, clarification + scoping + web grounding pattern
- [Claude Code Sub-Agents: Parallel vs Sequential Patterns](https://claudefa.st/blog/guide/agents/sub-agent-best-practices) — Claude Code parallel dispatch, context window management, model selection for subagents
