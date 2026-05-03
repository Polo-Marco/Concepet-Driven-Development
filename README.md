# Vibe Coding Framework 6.0

A structured AI development framework built on a session-based pipeline.
The Planner designs. The Generator builds. The Evaluator (optional)
audits. You sign off. Git is the checkpoint system.

**Works with Claude Code and Cursor.**

## The Problem

AI coding agents fail in predictable ways:

1. **Context collapse.** The agent forgets its own decisions mid-session.
2. **Architectural hallucination.** Without planning, the agent invents your architecture.
3. **Poor self-evaluation.** Agents approve their own mediocre work.
4. **Context bloat.** Reading every file every turn burns tokens on
   irrelevant material.
5. **Environment surprises.** The Generator wastes retries on missing
   tools, missing `.env`, missing package managers.
6. **Spec drift.** Implementations slowly diverge from external
   contracts (API specs, design systems) without anyone noticing.

This framework solves all six with a strict Planner → Generator →
Evaluator pipeline, structured files as external memory, layered
Architecture for selective loading, an environment audit baked into
the Planner, and `docs/` + `DEVIATIONS.md` for tracked spec drift.

## How It Works

### The Session-Based Pipeline

```
Planner Session              Generator Session            Evaluator Session
────────────────             ─────────────────            ──────────────────
[/build] / [/modify]         start execution              [/evaluate]
[/debug] / [/migrate]

Ask: interrogate user        Read Architecture Overview   Read plan + diff
Env audit                    Read ticket → load only      Run tests
Spec: write core files       the sections it lists        Write Evaluation.md
      write Plan/Triage      TDD loop per ticket          (no commit)
      log deviations         Commit after each ticket
Git commit all
STOP                         STOP when done               STOP

     ↓                            ↓                            ↓
User reviews plan           User runs tests            User follows checklist
User places [Halt here]     User can run [/evaluate]   User deletes Plan +
  (optional)                                              Evaluation
```

**Planner Session** has full authority over core files. It designs,
plans, audits the environment, and produces everything the Generator
needs.

**Generator Session** has zero authority over core files. It reads
task tickets and executes them literally. Selective context loading
keeps it focused on the sections each ticket actually needs.

**Evaluator Session** is optional. It cannot modify code — it produces
`Evaluation.md`, a checklist that lets a non-expert user audit the
output thoroughly. Useful when working in unfamiliar domains.

**You are the final Evaluator.** You sign off when the work is done.

### Recovery via Git

Git commits at each ticket give you clean recovery points:

- **Generator fails?** `git reset` to the Planner commit, switch
  to a better model, `start execution` again.
- **Plan was wrong?** `git reset` to the Planner commit, start a
  new Planner session, refine the plan.
- **Partial success?** Keep what worked, start a new Planner session
  to address what didn't.

### The Five Modes

| Command | Persona | Purpose |
|---|---|---|
| `[/build]` | The Architect | 0-to-1 creation from scratch |
| `[/modify]` | The Refactoring Engineer | Feature additions, refactoring |
| `[/debug]` | The QA Lead | Root-cause analysis, bug fixing |
| `[/migrate]` | Migration Specialist | Bring an existing codebase under the framework |
| `[/evaluate]` | The Evaluator | Audit a completed Generator session |

Planner modes (`build` / `modify` / `debug`) drive the Planner →
Generator pipeline. `[/migrate]` is Planner-only. `[/evaluate]` runs
after the Generator and is optional.

### Phase-Based Authority

Authority binds to the session type, not the model:

| File | Planner | Generator | Evaluator |
|---|---|---|---|
| Concept.md | Read / Write | Read only | Read only |
| Architecture.md | Read / Write | Read only (selective) | Read only |
| Plan.md / Triage.md | Read / Write | Read only (mark `[x]`) | Read only |
| CHANGELOG.md | Read / Write | Append only | Read only |
| skills/ | Read / Write / Create | Read only | Read only |
| docs/*.md | Read only | Read only | Read only |
| docs/DEVIATIONS.md | Read / Append | Read only | Read only |
| Evaluation.md | — | — | Read / Write |
| src/, tests/ | — | Read / Write (within Boundary) | Read only |

## Core Files

| File | Lifecycle | Purpose |
|---|---|---|
| `Concept.md` | Persistent | Vision — why it exists, scope, principles |
| `Architecture.md` | Persistent (layered) | System design source of truth |
| `CHANGELOG.md` | Persistent | What changed, when |
| `Plan.md` | **Ephemeral** | Task tickets — deleted after the loop |
| `Triage.md` | **Ephemeral** | Bug hypotheses — deleted after the loop |
| `Evaluation.md` | **Ephemeral** | Evaluator checklist — deleted after sign-off |
| `skills/` | Persistent | Execution patterns, rules, conventions |
| `docs/*.md` | User-maintained | External reference docs (immutable to agents) |
| `docs/DEVIATIONS.md` | Planner-appendable | Tracked departures from reference docs |

## What's New in 6.0

### 1. Layered Architecture.md (selective loading)

`Architecture.md` is now a layered document. The Generator always
reads `## Overview` (a self-contained 20–30-line snapshot) and only
the sections each ticket lists in its `**Architecture:**` field:

```markdown
# Architecture
## Overview            <!-- always read -->
## Environment         <!-- audit results -->
## API Surface
## Data Models
## Frontend Components
## Infrastructure
```

Tickets declare what they need:

```
**Architecture:** Overview, API Surface, Data Models
```

Use `Full` to load the entire document.

### 2. Environment Audit in the Planner

Before writing Plan.md the Planner runs CLI checks (`python --version`,
`uv --version`, `node --version`, …) and inspects config files
(`pyproject.toml`, `.env.example`, …). Findings land in
`Architecture.md ## Environment`, and the **first ticket of every Plan
is "Environment Setup"** — install missing tools, create `.env`, init
the package manager. The Generator stops failing on missing prereqs.

`[/modify]` does the same audit if the change introduces new
dependencies. `[/migrate]` audits during the codebase scan.

### 3. Evaluator Session (`[/evaluate]`)

A third session type. After the Generator finishes, run
`[/evaluate]`. The Evaluator reads the plan, the git diff, and
`docs/DEVIATIONS.md`; runs the test suite; and writes
`Evaluation.md` containing:

- Test results (pass / fail / coverage)
- Architecture compliance (every endpoint / model named in
  `Architecture.md` is checked)
- Code-quality smells (hardcoded secrets, leftover TODOs, generic
  excepts)
- Manual verification checklist — aggregated from every ticket's
  `Manual Verification` field, expanded into step-by-step instructions
  a non-expert user can follow
- Deviations from reference docs (flagged if not logged)

The Evaluator **cannot modify code**. Fixes go through a fresh
Planner → Generator cycle.

### 4. Reference Docs with Deviation Tracking

A `docs/` directory holds external specs the user wants the agents to
respect — API contracts, design systems, SDK manuals. Originals are
immutable to agents.

```
docs/
├── api-contract.md
├── design-system.md
├── development-manual.md
└── DEVIATIONS.md         ← Planner-appendable
```

Tickets opt in:

```
**Reference Docs:** @docs/api-contract.md (Section: Authentication)
```

When the Planner makes a decision that conflicts with a reference
doc, it appends to `docs/DEVIATIONS.md` in the same session — the
Generator then knows which parts of the spec are current. The
Evaluator flags any code that contradicts a reference doc without
a logged deviation.

`DEVIATIONS.md` format:

```markdown
## api-contract.md

### Section: Authentication — JWT lifetime
**Original spec:** 24-hour token TTL
**Current implementation:** 1-hour TTL with refresh token
**Reason:** Security review required short-lived access tokens
**Decided:** 2026-05-03
```

## File Structure

```
your-project/
├── CLAUDE.md                       ← Router (auto-loaded by Claude Code & Cursor)
├── claude/rules/
│   ├── governance.md               ← Git, security, TDD, file lifecycle
│   ├── phase-authority.md          ← Authority matrix, boundary rules
│   ├── generator-protocol.md       ← Selective context load, retry, halt
│   └── task-ticket-format.md       ← Ticket format with Architecture: + Reference Docs:
├── skills/
│   ├── skill-template/SKILL.md     ← How to write skills
│   ├── mode-build/SKILL.md         ← The Architect
│   ├── mode-modify/SKILL.md        ← The Refactoring Engineer
│   ├── mode-debug/SKILL.md         ← The QA Lead
│   ├── mode-migrate/SKILL.md       ← Migration Specialist
│   └── mode-evaluate/SKILL.md      ← The Evaluator (new in 6.0)
├── docs/                           ← User-maintained reference docs (new in 6.0)
│   ├── api-contract.md
│   ├── design-system.md
│   └── DEVIATIONS.md
├── Concept.md                      ← Vision (persistent)
├── Architecture.md                 ← Layered design (persistent)
├── Plan.md                         ← Work order (ephemeral)
├── Evaluation.md                   ← Evaluator output (ephemeral)
├── CHANGELOG.md                    ← History (persistent)
└── README.md
```

## Setup

### Prerequisites

- **Claude Code** or **Cursor** (or both)
- Claude Pro, Max, Teams, or Enterprise account
- Git initialized in your project

### Option A: Claude Code

Install Claude Code if you haven't:

```bash
# macOS / Linux
curl -fsSL https://claude.ai/install.sh | bash

# Windows (PowerShell)
irm https://claude.ai/install.ps1 | iex
```

Add the framework to your project:

```bash
mkdir my-project && cd my-project && git init
# Copy: CLAUDE.md, claude/, skills/ into project root
```

Claude Code auto-loads `CLAUDE.md` on startup. Rule files in
`claude/rules/` are read on-demand via the router.

### Option B: Cursor

Cursor auto-loads `CLAUDE.md` as a workspace rule. Copy the same
files into your project:

```bash
mkdir my-project && cd my-project && git init
# Copy: CLAUDE.md, claude/, skills/ into project root
```

### Both tools, same files

`CLAUDE.md` is the single entry point — the router for both Claude
Code and Cursor. No duplication. Switch tools mid-project freely;
git keeps everything in sync.

## Usage

### Building a New Project

**Planner session:**

```
You:     [/build] A FastAPI app that uploads PDFs, extracts text,
         and summarizes them with an LLM. React frontend.

Agent:   [Writes Concept.md, asks questions about stack and edge cases]

You:     proceed to spec

Agent:   [Runs environment audit: python ✓, uv ✓, node missing]
Agent:   [Writes layered Architecture.md, skills, Plan.md whose
          first ticket is "Environment Setup", CHANGELOG.md]
Agent:   "Planner session complete. Review the files. Place [Halt here]
          if needed. Type start execution when ready."
```

**Generator session:**

```
You:     start execution

Agent:   [Reads Architecture Overview only, then per-ticket sections]
Agent:   [Executes tickets, commits after each]
Agent:   "Generator session complete. Ready for your evaluation."
```

**Optional Evaluator session:**

```
You:     [/evaluate]

Agent:   [Reads plan + diff + reference docs, runs tests]
Agent:   [Writes Evaluation.md]
Agent:   "Evaluation complete. Review Evaluation.md."
```

**You sign off.** Run the checklist. If satisfied, delete Plan.md and
Evaluation.md.

### Modifying

```
You:     [/modify] Add batch upload with concurrent PDF processing

Agent:   [Reads Concept.md, Architecture sections, docs/DEVIATIONS.md]
Agent:   "Warning: Current pipeline is synchronous. Batch processing
          needs async. Options: ..."

You:     proceed to spec

Agent:   [Audits new deps, updates Architecture surgically + Overview,
          writes fresh Plan.md]
```

### Debugging

```
You:     [/debug] Extraction returns empty text for scanned PDFs

Agent:   [Creates Triage.md with hypotheses, lists Architecture sections
          and reference docs each hypothesis touches]

You:     start execution
```

### Migrating an Existing Codebase

```
You:     [/migrate]

Agent:   [Reads codebase, runs environment audit, asks about vision
          and reference docs]

You:     proceed to spec

Agent:   [Writes Concept.md, layered Architecture.md (with Environment
          section), skills matching existing patterns. Sets up empty
          docs/DEVIATIONS.md. No code changes.]
```

### Evaluating

```
You:     [/evaluate]

Agent:   [Reads Plan.md, Architecture Overview, docs/DEVIATIONS.md,
          git diff from planner commit. Runs the test suite.]
Agent:   [Writes Evaluation.md with test results, architecture
          compliance, code-quality checks, manual verification
          checklist, deviations.]
Agent:   "Evaluation complete. Review Evaluation.md."
```

## Key Concepts

### Task Tickets (v6.0)

```markdown
### Phase 2, Step 3: Build Extraction Pipeline

**Input:** src/models/document.py
**Output:** src/pipeline/extractor.py, tests/test_extractor.py
**Spec:**
- Function `extract_text(doc: Document) -> ExtractedContent`
- Handle: PDF, DOCX, plain text
- On failure: raise `ExtractionError`

**Test Contract:**
- test_extract_pdf_success: valid PDF → ExtractedContent
- test_extract_unsupported: .xlsx → ExtractionError
- test_extract_corrupt: truncated PDF → ExtractionError

**Manual Verification:**
- Upload multi-page PDF; confirm all pages extracted
- Upload scanned PDF; confirm OCR fallback works
- Upload 0-byte file; confirm clean error, no crash

**Architecture:** Overview, Data Models
**Skills to Load:** @skills/fastapi-backend/SKILL.md
**Reference Docs:** @docs/api-contract.md (Section: Documents)
**Boundary:** src/pipeline/, tests/test_extractor.py
**Run Command:** uv run pytest tests/test_extractor.py -v
```

### Generator Retry Logic

When a ticket's tests fail after implementation:
1. Attempt to fix (try 1).
2. If still failing, attempt again (try 2).
3. If still failing, final attempt (try 3).
4. If still failing: commit progress with a WIP message and stop.

### `[Halt here]` Flags

The Planner does NOT place halt flags. You place them after reviewing
Plan.md, wherever you want the Generator to pause. The Generator
commits and stops on hitting one.

### Bespoke Skills

Planner-generated, project-specific files with:
- Canonical file structure
- Copy-paste code patterns with your actual types
- DO / DO NOT rules (binary, no judgment)
- Error handling contracts, testing conventions, exact commands

### The Evaluation Model

**Layer 1 — TDD (automated):** Tests before code. Catches regressions.

**Layer 2 — Manual Verification:** Each ticket lists what to inspect.

**Layer 3 — Evaluator (optional, `[/evaluate]`):** Aggregates manual
verification into a step-by-step checklist, runs tests, flags
architecture and reference-doc drift.

**Layer 4 — You:** Sign off. The full loop isn't done until you say so.

## Claude Code vs Cursor

| Concern | Claude Code | Cursor |
|---|---|---|
| Router loading | `CLAUDE.md` auto-loaded on startup | `CLAUDE.md` auto-loaded as workspace rule |
| Rule files | `claude/rules/*.md` read on-demand | Read on-demand when referenced |
| Skill files | Read via `Read` tool when instructed | Read via `Read` tool when instructed |
| `@` references | Plain text (agent resolves) | File references (IDE resolves) |
| Mode commands | Typed in chat | Typed in chat |

## Version History

| Version | Key Change |
|---|---|
| 1.0–3.0 | Prompt-in, code-out. Context collapse. |
| 3.5 | Decoupled design from implementation. |
| 4.0 | 3-Phase Pipeline. Git, TDD, CHANGELOG. |
| 4.1 | Dynamic skill creation. |
| 4.2 | Programmable `[Halt here]` for context-aware chunking. |
| 5.0 | Strategy Pattern: personas via mode commands. |
| 5.1 | Two-Tier Bug Classification. |
| 5.2 | STATE.md, task tickets with Boundary, Blocked protocol. |
| 5.3 | Phase-based authority. Unified Planner/Generator pipeline. User as Evaluator. |
| 5.5 | Session-based development. Git as checkpoint system. Concept.md restored. STATE.md eliminated. Plan/Triage ephemeral. User-placed `[Halt here]` only. `[/migrate]` mode. 3-retry logic. |
| 5.6 | Dual-tool compatibility: Claude Code + Cursor. `CLAUDE.md` as unified router. |
| **6.0** | **Layered Architecture.md with selective loading. Environment audit baked into Planner. New Evaluator session via `[/evaluate]`. Reference docs in `docs/` with `DEVIATIONS.md` for tracked spec drift.** |

## Tips

- **Start small.** First project should be buildable in a day.
- **Keep the Architecture Overview tight.** It's the only section
  always loaded — anything that drifts here, drifts everywhere.
- **Drop your contracts into `docs/` early.** API specs, design
  systems, SDK manuals. Reference them from tickets.
- **Use `[/evaluate]` when you're not the domain expert.** The
  checklist makes you a better evaluator without making you a
  specialist.
- **Review the environment audit.** Catching missing tools at the
  Planner stage saves a Generator stop later.
- **Read git logs.** `git log --oneline` is the progress trail.
- **Switch tools freely.** Planner in Cursor, Generator in Claude
  Code, or vice versa. Files are tool-agnostic.

## License

Open for personal and commercial use. Attribution appreciated.
