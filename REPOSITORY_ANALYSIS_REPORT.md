# Continuous Claude Repository Analysis Report

> A persistent, learning, multi-agent development environment built on Claude Code

---

## Table of Contents

- [1. System Overview](#1-system-overview)
- [2. Core Design Principles](#2-core-design-principles)
- [3. Core Architecture](#3-core-architecture)
- [4. Main Components](#4-main-components)
- [5. Workflows](#5-workflows)
- [6. Quick Start](#6-quick-start)
- [7. Usage Guide](#7-usage-guide)
- [8. Technology Stack](#8-technology-stack)

---

## 1. System Overview

### 1.1 What is Continuous Claude?

**Continuous Claude** is a development environment that transforms Claude Code into a continuously learning system. It solves the core problem in AI-assisted programming: **context loss** — through intelligent code analysis, context management, and specialized agent coordination.

### 1.2 Core Problems and Solutions

| Problem | Continuous Claude's Solution |
|---------|----------------------------|
| Context loss on compaction | YAML handoffs - more token-efficient transfer |
| Starting fresh each session | Memory system + daemon auto-extracts learnings |
| Reading entire files burns tokens | 5-layer code analysis + semantic index (95% token savings) |
| Complex tasks need coordination | Meta-skills orchestrate agent workflows |
| Repeating workflows manually | 109 skills with natural language triggers |

### 1.3 Key Metrics

- **Skills**: 109 modular capabilities
- **Agents**: 32 specialized sub-assistants
- **Hooks**: 30 lifecycle interceptors
- **Token Savings**: 95% reduction (TLDR analysis)

---

## 2. Core Design Principles

### 2.1 Five-Element Architecture

An agent consists of five elements: **Prompt + Tools + Context + Memory + Model**

| Component | Continuous Claude's Optimization |
|-----------|--------------------------------|
| **Prompt** | Skills inject relevant context; hooks add system reminders |
| **Tools** | TLDR reduces tokens; agents parallelize work |
| **Context** | Not just *what* Claude knows, but *how* it's provided |
| **Memory** | Daemon extracts learnings; recall surfaces them |
| **Model** | Becomes swappable when the other four are solid |

### 2.2 Core Principles

#### "Compound, don't compact"

- Automatically extract learnings
- Start fresh with full context
- Each session makes the system smarter

#### Anti-Complexity

- **Time, not money** — No required paid services (Perplexity and NIA are optional)
- **Learn, don't accumulate** — A learning system handles edge cases better than plugin sprawl
- **Shift-left validation** — Hooks run pyright/ruff after edits, catching errors before tests

### 2.3 Skill Activation System

**You don't need to memorize slash commands**. Just describe what you want naturally.

```
User: "Fix the login bug in auth.py"

🎯 SKILL ACTIVATION CHECK
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

⚠️ CRITICAL SKILLS (REQUIRED):
  → create_handoff

📚 RECOMMENDED SKILLS:
  → fix
  → debug

🤖 RECOMMENDED AGENTS (token-efficient):
  → debug-agent
  → scout

ACTION: Use Skill tool BEFORE responding
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### 2.4 Priority Levels

| Level | Meaning |
|-------|---------|
| ⚠️ **CRITICAL** | Must use (e.g., handoffs before ending session) |
| 📚 **RECOMMENDED** | Should use (e.g., workflow skills) |
| 💡 **SUGGESTED** | Consider using (e.g., optimization tools) |
| 📌 **OPTIONAL** | Nice to have (e.g., documentation helpers) |

---

## 3. Core Architecture

### 3.1 System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                        CONTINUOUS CLAUDE                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐             │
│  │   Skills    │    │   Agents    │    │    Hooks    │             │
│  │   (109)     │───▶│    (32)     │◀───│    (30)     │             │
│  └─────────────┘    └─────────────┘    └─────────────┘             │
│         │                  │                  │                     │
│         ▼                  ▼                  ▼                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                     TLDR Code Analysis                       │   │
│  │   L1:AST → L2:CallGraph → L3:CFG → L4:DFG → L5:Slicing      │   │
│  │                    (95% token savings)                       │   │
│  └─────────────────────────────────────────────────────────────┘   │
│         │                  │                  │                     │
│         ▼                  ▼                  ▼                     │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐             │
│  │   Memory    │    │ Continuity  │    │ Coordination│             │
│  │   System    │    │   Ledgers   │    │    Layer    │             │
│  └─────────────┘    └─────────────┘    └─────────────┘             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 3.2 Data Flow: Session Lifecycle

```
SessionStart                    Working                      SessionEnd
    │                              │                             │
    ▼                              ▼                             ▼
┌─────────┐                  ┌─────────┐                   ┌─────────┐
│  Load   │                  │  Track  │                   │  Save   │
│ context │─────────────────▶│ changes │──────────────────▶│  state  │
└─────────┘                  └─────────┘                   └─────────┘
    │                              │                             │
    ├── Continuity ledger          ├── File claims               ├── Handoff
    ├── Memory recall              ├── TLDR indexing             ├── Learnings
    └── Symbol index               └── Blackboard                └── Outcome
                                          │
                                          ▼
                                     ┌─────────┐
                                     │ /clear  │
                                     │ Fresh   │
                                     │ context │
                                     └─────────┘
```

### 3.3 The Continuity Loop (Detailed)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            THE CONTINUITY LOOP                              │
└─────────────────────────────────────────────────────────────────────────────┘

  1. SESSION START                     2. WORKING
  ┌────────────────────┐               ┌────────────────────┐
  │                    │               │                    │
  │  Ledger loaded ────┼──▶ Context    │  PostToolUse ──────┼──▶ Index handoffs
  │  Handoff loaded    │               │  UserPrompt ───────┼──▶ Skill hints
  │  Memory recalled   │               │  Edit tracking ────┼──▶ Dirty flag++
  │  TLDR cache warmed │               │  SubagentStop ─────┼──▶ Agent reports
  │                    │               │                    │
  └────────────────────┘               └────────────────────┘
           │                                    │
           │                                    ▼
           │                           ┌────────────────────┐
           │                           │ 3. PRE-COMPACT     │
           │                           │                    │
           │                           │  Auto-handoff ─────┼──▶ thoughts/shared/
           │                           │  (YAML format)     │    handoffs/*.yaml
           │                           │  Dirty > 20? ──────┼──▶ TLDR re-index
           │                           │                    │
           │                           └────────────────────┘
           │                                    │
           │                                    ▼
           │                           ┌────────────────────┐
           │                           │ 4. SESSION END     │
           │                           │                    │
           │                           │  Stale heartbeat ──┼──▶ Daemon wakes
           │                           │  Daemon spawns ────┼──▶ Headless Claude
           │                           │  Thinking blocks ──┼──▶ archival_memory
           │                           │                    │
           │                           └────────────────────┘
           │                                    │
           │                                    │
           └──────────────◀────── /clear ◀──────┘
                          Fresh context + state preserved
```

### 3.4 Data Layer Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           DATA LAYER ARCHITECTURE                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  TLDR 5-LAYER CODE ANALYSIS              SEMANTIC INDEX                     │
│  ┌────────────────────────┐              ┌────────────────────────┐         │
│  │ L1: AST (~500 tok)     │              │ BGE-large-en-v1.5      │         │
│  │     └── Functions,     │              │ ├── All 5 layers       │         │
│  │         classes, sigs  │              │ ├── 10 lines context   │         │
│  │                        │              │ └── FAISS index        │         │
│  │ L2: Call Graph (+440)  │              │                        │         │
│  │     └── Cross-file     │──────────────│ Query: "auth logic"    │         │
│  │         dependencies   │              │ Returns: ranked funcs  │         │
│  │                        │              └────────────────────────┘         │
│  │ L3: CFG (+110 tok)     │                                                 │
│  │     └── Control flow   │                                                 │
│  │                        │              MEMORY (PostgreSQL+pgvector)       │
│  │ L4: DFG (+130 tok)     │              ┌────────────────────────┐         │
│  │     └── Data flow      │              │ sessions (heartbeat)   │         │
│  │                        │              │ file_claims (locks)    │         │
│  │ L5: PDG (+150 tok)     │              │ archival_memory (BGE)  │         │
│  │     └── Slicing        │              │ handoffs (embeddings)  │         │
│  └────────────────────────┘              └────────────────────────┘         │
│         ~1,200 tokens                                                       │
│         vs 23,000 raw                                                       │
│         = 95% savings                    FILE SYSTEM                        │
│                                          ┌────────────────────────┐         │
│                                          │ thoughts/              │         │
│                                          │ ├── ledgers/           │         │
│                                          │ │   └── CONTINUITY_*.md│         │
│                                          │ └── shared/            │         │
│                                          │     ├── handoffs/*.yaml│         │
│                                          │     └── plans/*.md     │         │
│                                          │                        │         │
│                                          │ .tldr/                 │         │
│                                          │ └── (daemon cache)     │         │
│                                          └────────────────────────┘         │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Main Components

### 4.1 Skills System (109 Skills)

Skills are modular capabilities triggered by natural language. Located in `.claude/skills/`.

#### Meta-Skills (Workflow Orchestrators)

| Meta-Skill | Chain | Use When |
|------------|-------|----------|
| `/workflow` | Router → appropriate workflow | Don't know where to start |
| `/build` | discovery → plan → validate → implement → commit | Building features |
| `/fix` | sleuth → premortem → kraken → test → commit | Fixing bugs |
| `/tdd` | plan → arbiter (tests) → kraken (implement) → arbiter | Test-first development |
| `/refactor` | phoenix → plan → kraken → reviewer → arbiter | Safe code transformation |
| `/review` | parallel specialized reviews → synthesis | Code review |
| `/explore` | scout (quick/deep/architecture) | Understand codebase |
| `/security` | vulnerability scan → verification | Security audits |
| `/release` | audit → E2E → review → changelog | Ship releases |

#### Key Skills (High-Value Tools)

**Planning & Risk**
- **premortem**: TIGERS & ELEPHANTS risk analysis - use before any significant implementation
- **discovery-interview**: Transform vague ideas into detailed specs

**Context Management**
- **create_handoff**: Capture session state for transfer
- **resume_handoff**: Resume from handoff with context
- **continuity_ledger**: Track state within session

**Code Analysis (95% Token Savings)**
- **tldr-code**: Call graph, CFG, DFG, slicing
- **ast-grep-find**: Structural code search
- **morph-search**: Fast text search (20x faster than grep)

**Research**
- **perplexity-search**: AI-powered web search
- **nia-docs**: Library documentation search
- **github-search**: Search GitHub code/issues/PRs

**Quality**
- **qlty-check**: 70+ linters, auto-fix
- **braintrust-analyze**: Session analysis, replay, and debugging failed sessions

**Math & Formal Proofs**
- **math**: Unified computation (SymPy, Z3, Pint) — one entry point for all math
- **prove**: Lean4 theorem proving with 5-phase workflow (Research → Design → Test → Implement → Verify)
- **pint-compute**: Unit-aware arithmetic and conversions
- **shapely-compute**: Computational geometry

#### Decision Tree

```
What do I want to do?
├── Don't know → /workflow (guided router)
├── Building → /build greenfield or brownfield
├── Fixing → /fix bug
├── Understanding → /explore
├── Planning → premortem first, then plan-agent
├── Researching → oracle or perplexity-search
├── Reviewing → /review
├── Proving → /prove (Lean4 formal verification)
├── Computing → /math (SymPy, Z3, Pint)
└── Shipping → /release
```

### 4.2 Agents System (32 Agents)

Agents are specialized AI workers spawned via the Task tool. Located in `.claude/agents/`.

#### Agent Categories

**Orchestrators (2)**
- **maestro**: Multi-agent coordination with patterns (Pipeline, Swarm, Jury)
- **kraken**: TDD implementation agent with checkpoint/resume support

**Planners (4)**
- **architect**: Feature planning + API integration
- **phoenix**: Refactoring + framework migration planning
- **plan-agent**: Lightweight planning with research/MCP tools
- **validate-agent**: Validate plans against best practices

**Explorers (4)**
- **scout**: Codebase exploration (use instead of Explore)
- **oracle**: External research (web, docs, APIs)
- **pathfinder**: External repository analysis
- **research-codebase**: Document codebase as-is

**Implementers (3)**
- **kraken**: TDD implementation with strict test-first workflow
- **spark**: Lightweight fixes and quick tweaks
- **agentica-agent**: Build Python agents using Agentica SDK

**Debuggers (3)**
- **sleuth**: General bug investigation and root cause
- **debug-agent**: Issue investigation via logs/code search
- **profiler**: Performance profiling and race conditions

**Validators (2)** - arbiter, atlas

**Reviewers (6)** - critic, judge, surveyor, liaison, plan-reviewer, review-agent

**Specialized (8)** - aegis, herald, scribe, chronicler, session-analyst, braintrust-analyst, memory-extractor, onboard

#### Common Workflows

| Workflow | Agent Chain |
|----------|-------------|
| Feature | architect → plan-reviewer → kraken → review-agent → arbiter |
| Refactoring | phoenix → plan-reviewer → kraken → judge → arbiter |
| Bug Fix | sleuth → spark/kraken → arbiter → scribe |

### 4.3 Hooks System (30 Hooks)

Hooks intercept Claude Code at lifecycle points. Located in `.claude/hooks/`.

#### Hook Events

| Event | Key Hooks | Purpose |
|-------|-----------|---------|
| **SessionStart** | session-start-continuity, session-register, braintrust-tracing | Load context, register session |
| **PreToolUse** | tldr-read-enforcer, smart-search-router, tldr-context-inject, file-claims | Token savings, search routing |
| **PostToolUse** | post-edit-diagnostics, handoff-index, post-edit-notify | Validation, indexing |
| **PreCompact** | pre-compact-continuity | Auto-save before compaction |
| **UserPromptSubmit** | skill-activation-prompt, memory-awareness | Skill hints, memory recall |
| **SubagentStop** | subagent-stop-continuity | Save agent state |
| **SessionEnd** | session-end-cleanup, session-outcome | Cleanup, extract learnings |

#### Key Hooks

| Hook | Purpose |
|------|---------|
| **tldr-context-inject** | Adds code analysis to agent prompts |
| **smart-search-router** | Routes grep to AST-grep when appropriate |
| **post-edit-diagnostics** | Runs pyright/ruff after edits |
| **memory-awareness** | Surfaces relevant learnings |

### 4.4 TLDR Code Analysis

TLDR provides token-efficient code summaries through 5 analysis layers.

#### The 5-Layer Stack

| Layer | Name | What it provides | Tokens |
|-------|------|------------------|--------|
| **L1** | AST | Functions, classes, signatures | ~500 tokens |
| **L2** | Call Graph | Who calls what (cross-file) | +440 tokens |
| **L3** | CFG | Control flow, complexity | +110 tokens |
| **L4** | DFG | Data flow, variable tracking | +130 tokens |
| **L5** | PDG | Program slicing, impact analysis | +150 tokens |

**Total: ~1,200 tokens vs 23,000 raw = 95% savings**

#### CLI Commands

```bash
# Structure analysis
tldr tree src/                      # File tree
tldr structure src/ --lang python   # Code structure (codemaps)

# Search and extraction
tldr search "process_data" src/     # Find code
tldr context process_data --project src/ --depth 2  # LLM-ready context

# Flow analysis
tldr cfg src/main.py main           # Control flow graph
tldr dfg src/main.py main           # Data flow graph
tldr slice src/main.py main 42      # What affects line 42?

# Codebase analysis
tldr impact process_data src/       # Who calls this function?
tldr dead src/                      # Find unreachable code
tldr arch src/                      # Detect architectural layers

# Semantic search (natural language)
tldr daemon semantic "find authentication logic"
```

#### Semantic Index

Beyond structural analysis, TLDR builds a **semantic index** of your codebase:

- **Natural language queries** — Ask "where is error handling?" instead of grepping
- **Auto-rebuild** — Dirty flag hook tracks file changes; index rebuilds after N edits
- **Selective indexing** — Use `.tldrignore` to control what gets indexed

### 4.5 Memory System

Cross-session learning powered by PostgreSQL + pgvector.

#### How It Works

```
Session ends → Database detects stale heartbeat (>5 min)
            → Daemon spawns headless Claude (Sonnet)
            → Analyzes thinking blocks from session
            → Extracts learnings to archival_memory
            → Next session recalls relevant learnings
```

The key insight: **thinking blocks contain the real reasoning**—not just what Claude did, but why. The daemon extracts this automatically.

#### Conversational Interface

| What You Say | What Happens |
|--------------|--------------|
| "Remember that auth uses JWT" | Stores learning with context |
| "Recall authentication patterns" | Searches memory, surfaces matches |
| "What did we decide about X?" | Implicit recall via memory-awareness hook |

#### Database Schema (4 tables)

| Table | Purpose |
|-------|---------|
| **sessions** | Cross-terminal awareness |
| **file_claims** | Cross-terminal file locking |
| **archival_memory** | Long-term learnings with BGE embeddings |
| **handoffs** | Session handoffs with embeddings |

### 4.6 Continuity System

Preserve state across context clears and sessions.

#### Continuity Ledger

Within-session state tracking. Location: `thoughts/ledgers/CONTINUITY_<topic>.md`

```markdown
# Session: feature-x
Updated: 2026-01-08

## Goal
Implement feature X with proper error handling

## Completed
- [x] Designed API schema
- [x] Implemented core logic

## In Progress
- [ ] Add error handling

## Blockers
- Need clarification on retry policy
```

#### Handoffs

Between-session knowledge transfer. Location: `thoughts/shared/handoffs/<session>/`

```yaml
---
date: 2026-01-08T15:26:01+0000
session_name: feature-x
status: complete
---

# Handoff: Feature X Implementation

## Task(s)
| Task | Status |
|------|--------|
| Design API | Completed |
| Implement core | Completed |
| Error handling | Pending |

## Next Steps
1. Add retry logic to API calls
2. Write integration tests
```

---

## 5. Workflows

### 5.1 Workflow Chains

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           META-SKILL WORKFLOWS                              │
└─────────────────────────────────────────────────────────────────────────────┘

  /fix bug                              /build greenfield
  ─────────                             ─────────────────
  ┌──────────┐  ┌──────────┐            ┌──────────┐  ┌──────────┐
  │  sleuth  │─▶│ premortem│            │discovery │─▶│plan-agent│
  │(diagnose)│  │  (risk)  │            │(clarify) │  │ (design) │
  └──────────┘  └────┬─────┘            └──────────┘  └────┬─────┘
                     │                                      │
                     ▼                                      ▼
              ┌──────────┐                          ┌──────────┐
              │  kraken  │                          │ validate │
              │  (fix)   │                          │ (check)  │
              └────┬─────┘                          └────┬─────┘
                   │                                      │
                   ▼                                      ▼
              ┌──────────┐                          ┌──────────┐
              │  arbiter │                          │  kraken  │
              │ (test)   │                          │(implement│
              └────┬─────┘                          └────┬─────┘
                   │                                      │
                   ▼                                      ▼
              ┌──────────┐                          ┌──────────┐
              │  commit  │                          │  commit  │
              └──────────┘                          └──────────┘
```

### 5.2 Natural Language Examples

| What You Say | What Activates |
|--------------|----------------|
| "Fix the broken login" | `/fix` workflow → debug-agent, scout |
| "Build a user dashboard" | `/build` workflow → plan-agent, kraken |
| "I want to understand this codebase" | `/explore` + scout agent |
| "What could go wrong with this plan?" | `/premortem` |
| "Help me figure out what I need" | `/discovery-interview` |
| "Done for today" | `create_handoff` (critical) |
| "Resume where we left off" | `resume_handoff` |
| "Research auth patterns" | oracle agent + perplexity |
| "Find all usages of this API" | scout agent + ast-grep |

### 5.3 Skill vs Workflow vs Agent

| Type | Purpose | Example |
|------|---------|---------|
| **Skill** | Single-purpose tool | `commit`, `tldr-code`, `qlty-check` |
| **Workflow** | Multi-step process | `/fix` (sleuth → premortem → kraken → commit) |
| **Agent** | Specialized sub-session | scout (exploration), oracle (research) |

---

## 6. Quick Start

### 6.1 Prerequisites

- Python 3.11+
- [uv](https://github.com/astral-sh/uv) package manager
- Docker (for PostgreSQL)
- Claude Code CLI

### 6.2 Installation

```bash
# Clone
git clone https://github.com/parcadei/Continuous-Claude-v3.git
cd Continuous-Claude-v3/opc

# Run setup wizard (12 steps)
uv run python -m scripts.setup.wizard
```

### 6.3 What the Wizard Does

| Step | What It Does |
|------|--------------|
| 1 | Backup existing .claude/ config (if present) |
| 2 | Check prerequisites (Docker, Python, uv) |
| 3-5 | Database + API key configuration |
| 6-7 | Start Docker stack, run migrations |
| 8 | Install Claude Code integration (32 agents, 109 skills, 30 hooks) |
| 9 | Math features (SymPy, Z3, Pint - optional) |
| 10 | TLDR code analysis tool |
| 11-12 | Diagnostics tools + Loogle (optional) |

### 6.4 First Session

```bash
# Start Claude Code
claude

# Try a workflow
> /workflow
```

### 6.5 First Session Commands

| Command | What it does |
|---------|--------------|
| `/workflow` | Goal-based routing (Research/Plan/Build/Fix) |
| `/fix bug <description>` | Investigate and fix a bug |
| `/build greenfield <feature>` | Build a new feature from scratch |
| `/explore` | Understand the codebase |
| `/premortem` | Risk analysis before implementation |

---

## 7. Usage Guide

### 7.1 Configuration

#### .claude/settings.json

Central configuration for hooks, tools, and workflows.

```json
{
  "hooks": {
    "SessionStart": [...],
    "PreToolUse": [...],
    "PostToolUse": [...],
    "UserPromptSubmit": [...]
  }
}
```

#### .claude/skills/skill-rules.json

Skill activation triggers.

```json
{
  "rules": [
    {
      "skill": "fix",
      "keywords": ["fix this", "broken", "not working"],
      "intentPatterns": ["fix.*(bug|issue|error)"]
    }
  ]
}
```

### 7.2 Environment Variables

| Variable | Purpose | Required |
|----------|---------|----------|
| `DATABASE_URL` | PostgreSQL connection string | Yes |
| `BRAINTRUST_API_KEY` | Session tracing | No |
| `PERPLEXITY_API_KEY` | Web search | No |
| `NIA_API_KEY` | Documentation search | No |

Services without API keys still work:
- Continuity system (ledgers, handoffs)
- TLDR code analysis
- Local git operations
- TDD workflow

### 7.3 Directory Structure

```
continuous-claude/
├── .claude/
│   ├── agents/           # 32 specialized agents
│   ├── hooks/            # 30 lifecycle hooks
│   │   ├── src/          # TypeScript source
│   │   └── dist/         # Compiled JavaScript
│   ├── skills/           # 109 modular capabilities
│   ├── rules/            # System policies
│   ├── scripts/          # Python utilities
│   └── settings.json     # Hook configuration
├── opc/
│   ├── packages/
│   │   └── tldr-code/    # 5-layer code analysis
│   ├── scripts/
│   │   ├── setup/        # Wizard, Docker, integration
│   │   └── core/         # recall_learnings, store_learning
│   └── docker/
│       └── init-schema.sql  # 4-table PostgreSQL schema
├── thoughts/
│   ├── ledgers/          # Continuity ledgers (CONTINUITY_*.md)
│   └── shared/
│       ├── handoffs/     # Session handoffs (*.yaml)
│       └── plans/        # Implementation plans
└── docs/                 # Documentation
```

### 7.4 Updating

Pull latest changes and sync your installation:

```bash
cd continuous-claude/opc
uv run python -m scripts.setup.update
```

This will:
- Pull latest from GitHub
- Update hooks, skills, rules, agents
- Upgrade TLDR if installed
- Rebuild TypeScript hooks if changed

### 7.5 For Brownfield Projects

After installation, start Claude and run:
```
> /onboard
```

This analyzes the codebase and creates an initial continuity ledger.

---

## 8. Technology Stack

### 8.1 Core Technologies

| Component | Technology |
|-----------|------------|
| **Languages** | Python 3.11+, TypeScript |
| **AI Models** | Claude 3.5 Sonnet, Claude 3 Opus |
| **Database** | PostgreSQL + pgvector |
| **Vector Embeddings** | BGE-large-en-v1.5 (1024-dim) |
| **Code Parsing** | tree-sitter |
| **Package Management** | uv (Python), npm (TypeScript) |
| **Containerization** | Docker, Docker Compose |

### 8.2 Dependencies and Tools

**Core Tools**
- **[uv](https://github.com/astral-sh/uv)** - Python packaging
- **[tree-sitter](https://tree-sitter.github.io/)** - Code parsing
- **[Braintrust](https://braintrust.dev)** - LLM evaluation, logging, and session tracing
- **[qlty](https://github.com/qltysh/qlty)** - Universal code quality CLI (70+ linters)
- **[ast-grep](https://github.com/ast-grep/ast-grep)** - AST-based code search and refactoring

**Optional Services**
- **[Nia](https://trynia.ai)** - Library documentation search
- **[Morph](https://www.morphllm.com)** - WarpGrep fast code search
- **[Firecrawl](https://www.firecrawl.dev)** - Web scraping API
- **[RepoPrompt](https://repoprompt.com)** - Token-efficient codebase maps

### 8.3 Math and Formal Verification

| Tool | Purpose | Example |
|------|---------|---------|
| **SymPy** | Symbolic math | Solve equations, integrals, matrix operations |
| **Z3** | Constraint solving | Prove inequalities, SAT problems |
| **Pint** | Unit conversion | Convert miles to km, dimensional analysis |
| **Lean4** | Formal proofs | Machine-verified theorems |
| **Mathlib** | 100K+ theorems | Pre-formalized lemmas to build on |
| **Loogle** | Type-aware search | Find Mathlib lemmas by signature |

### 8.4 Database Schema

**PostgreSQL Tables (4)**

```sql
-- Session tracking
CREATE TABLE sessions (
    session_id TEXT PRIMARY KEY,
    heartbeat TIMESTAMPTZ,
    terminal_id TEXT,
    status TEXT
);

-- File locking
CREATE TABLE file_claims (
    file_path TEXT PRIMARY KEY,
    session_id TEXT,
    claimed_at TIMESTAMPTZ
);

-- Long-term memory (with vector embeddings)
CREATE TABLE archival_memory (
    id SERIAL PRIMARY KEY,
    session_id TEXT,
    content TEXT,
    embedding vector(1024),  -- BGE-large-en-v1.5
    created_at TIMESTAMPTZ,
    confidence TEXT,
    type TEXT
);

-- Session handoffs
CREATE TABLE handoffs (
    id SERIAL PRIMARY KEY,
    session_name TEXT,
    content TEXT,
    embedding vector(1024),
    created_at TIMESTAMPTZ
);
```

### 8.5 File Formats

**Continuity Ledger (Markdown)**
```markdown
# Session: <name>
Updated: <timestamp>

## Goal
...

## Completed
- [x] Task 1

## In Progress
- [ ] Task 2
```

**Handoff (YAML + Markdown)**
```yaml
---
date: 2026-01-08T15:26:01+0000
session_name: feature-x
status: complete
---

# Handoff: <title>

## Task(s)
| Task | Status |
|------|--------|
| ... | ... |
```

---

## 9. Key Statistics

| Metric | Count | Notes |
|--------|-------|-------|
| Python functions | 2,328 | Across all `opc/scripts/` |
| TypeScript hooks | 34 | Active in `.claude/hooks/src/` |
| Skills | 109 | In `.claude/skills/` |
| Agents | 32 | Defined in system prompt |
| Tests | 265+ | TLDR-code alone |
| Token savings | 95% | TLDR vs raw file reads |
| Circular dependencies | 0 | Resolved by archiving scope modules |
| Most called function | get_connection | 38 callers |

---

## 10. Achievements

### 10.1 Sylvester-Gallai Theorem

Used the `/prove` skill to create the **first Lean formalization** of the Sylvester-Gallai theorem.

### 10.2 Token Efficiency

- **95% token savings**: TLDR 5-layer analysis vs reading raw files
- **1,200 tokens** vs **23,000 tokens**

### 10.3 System Complexity

- **109 skills** triggered by natural language
- **32 agents** as specialized sub-assistants
- **30 hooks** for automatic behaviors
- **5-layer code analysis**

---

## 11. Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on:

- Adding new skills
- Creating agents
- Developing hooks
- Extending TLDR

---

## 12. License

[MIT](LICENSE) - Use freely, contribute back.

---

## 13. Summary

**Continuous Claude** is not just a coding assistant—it's a persistent, learning, multi-agent development environment that gets smarter with every session.

### Core Advantages

1. **Persistent Memory**: Cross-session learning and context retention
2. **Token Efficiency**: 95% token savings through intelligent code analysis
3. **Specialized Agents**: 32 expert agents for complex tasks
4. **Natural Interaction**: No need to memorize commands, just describe your goals
5. **Automatic Learning**: Daemon extracts insights and builds knowledge base

### Use Cases

- ✅ Large codebase exploration and understanding
- ✅ Complex feature development (requiring planning and coordination)
- ✅ Bug investigation and fixing
- ✅ Code refactoring and architecture improvements
- ✅ Formal verification and mathematical proofs
- ✅ Long-term projects across sessions

### Get Started

```bash
git clone https://github.com/parcadei/Continuous-Claude-v3.git
cd Continuous-Claude-v3/opc
uv run python -m scripts.setup.wizard
```

In just 5 minutes, you'll have an intelligent, persistent, continuously learning AI development partner.

---

*This report was generated based on analysis of the Continuous Claude v3 codebase, covering architecture, components, workflows, and usage methods.*

*Generated: 2026-01-12*
