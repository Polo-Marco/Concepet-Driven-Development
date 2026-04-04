# Vibe Coding Framework 5.6

A structured AI development framework built on a session-based pipeline.
The Planner designs. The Generator builds. You evaluate. Git is the
checkpoint system.

**Works with Claude Code and Cursor.**

## The Problem

AI coding agents fail in predictable ways:

1. **Context collapse.** The agent forgets its own decisions mid-session.
2. **Architectural hallucination.** Without planning, the agent invents your architecture.
3. **Poor self-evaluation.** Agents approve their own mediocre work.

This framework solves all three with a strict Planner → Generator
pipeline, structured files as external memory, and you as the final
evaluator of every change.

## How It Works

### The Session-Based Pipeline

Every task flows through two sessions:

```
Planner Session                  Generator Session
────────────────                 ─────────────────
[/build] or [/modify]            start execution
or [/debug] or [/migrate]

Ask: interrogate user            Read Plan.md or Triage.md
Spec: write core files    ──→    Execute tickets sequentially
      write Plan/Triage          TDD loop per ticket
                                 Commit after each ticket
Git commit all               
STOP                             STOP when done

     ↓                               ↓
User reviews plan             User runs global tests
User places [Halt here]       User deletes Plan/Triage
  (optional)                    (work order complete)
```

**Planner Session** has full authority over core files. It designs,
plans, and produces everything the Generator needs.

**Generator Session** has zero authority over core files. It reads
task tickets and executes them literally. If it hits a spec gap,
it stops — no improvisation.

**You are the Evaluator.** You review the Planner's output. You run
the final tests. You decide when the work is done.

### Recovery via Git

Git commits at each ticket give you clean recovery points:

- **Generator fails?** `git reset` to the Planner commit, switch
  to a better model, `start execution` again.
- **Plan was wrong?** `git reset` to the Planner commit, start a
  new Planner session, refine the plan.
- **Partial success?** Keep what worked, start a new Planner session
  to address what didn't.

No checkpoint files needed. Git IS your save system.

### The Four Modes

| Command | Persona | Purpose |
|---|---|---|
| `[/build]` | The Architect | 0-to-1 creation from scratch |
| `[/modify]` | The Refactoring Engineer | Feature additions, refactoring |
| `[/debug]` | The QA Lead | Root-cause analysis, bug fixing |
| `[/migrate]` | Migration Specialist | Bring existing codebase under the framework |

All modes except `[/migrate]` follow the Planner → Generator pipeline.
`[/migrate]` is Planner-only — it documents, not changes.

### Phase-Based Authority

Authority binds to the session type, not the model:

| File | Planner Session | Generator Session |
|---|---|---|
| Concept.md | Read / Write | Read only |
| Architecture.md | Read / Write | Read only |
| Plan.md / Triage.md | Read / Write | Read only (mark `[x]`) |
| CHANGELOG.md | Read / Write | Append only |
| skills/ | Read / Write / Create | Read only |
| src/, tests/ | — | Read / Write (within Boundary) |

## Core Files

| File | Lifecycle | Purpose |
|---|---|---|
| `Concept.md` | Persistent | The project vision — why it exists, scope, principles |
| `Architecture.md` | Persistent | System design source of truth |
| `CHANGELOG.md` | Persistent | What changed, when |
| `Plan.md` | **Ephemeral** | Task tickets — deleted after work cycle completes |
| `Triage.md` | **Ephemeral** | Bug hypotheses — deleted after debug cycle completes |
| `skills/` | Persistent | Execution patterns, rules, conventions |

**Concept.md** is the north star. It can be vague and aspirational.
It captures *what the developer envisions* — without this, none of
the code matters.

**Plan.md and Triage.md are work orders.** Only one exists at a time.
When the full loop (Planner + Generator + your evaluation) completes,
you delete it. It served its purpose.

## File Structure

```
your-project/
├── CLAUDE.md                       ← Router (auto-loaded by Claude Code & Cursor)
├── .claude/rules/
│   ├── governance.md               ← Git, security, TDD, file lifecycle
│   ├── phase-authority.md          ← Authority matrix, boundary rules
│   ├── generator-protocol.md       ← Execution, retry, halt protocol
│   └── task-ticket-format.md       ← Ticket format + Manual Verification
├── skills/
│   ├── skill-template/SKILL.md     ← How to write skills
│   ├── mode-build/SKILL.md         ← The Architect
│   ├── mode-modify/SKILL.md        ← The Refactoring Engineer
│   ├── mode-debug/SKILL.md         ← The QA Lead
│   └── mode-migrate/SKILL.md       ← Migration Specialist
├── Concept.md                      ← Vision (persistent)
├── Architecture.md                 ← Design (persistent)
├── Plan.md                         ← Work order (ephemeral)
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
# Copy: CLAUDE.md, .claude/, skills/ into project root
```

Claude Code auto-loads `CLAUDE.md` and all files in `.claude/rules/`
on startup. The framework is active immediately.

### Option B: Cursor

Cursor auto-loads `CLAUDE.md` as a workspace rule. Copy the same
files into your project:

```bash
mkdir my-project && cd my-project && git init
# Copy: CLAUDE.md, .claude/, skills/ into project root
```

Open the project in Cursor. The `CLAUDE.md` router is picked up
automatically. The agent reads `.claude/rules/` and `skills/` files
on-demand when referenced by the router or mode skills.

### Both tools, same files

The framework uses the same file structure regardless of which tool
you use. `CLAUDE.md` is the single entry point — it serves as the
router for both Claude Code and Cursor. No duplication needed.

If you switch between tools on the same project, everything works.
Git history, skills, core files — all tool-agnostic.

## Usage

### Building a New Project

**Planner session:**

```
You:     [/build] A FastAPI app that uploads PDFs, extracts text,
         and summarizes them with an LLM. React frontend.

Agent:   [Writes Concept.md, asks questions about stack and edge cases]

You:     proceed to spec

Agent:   [Writes Architecture.md, Plan.md, skills, CHANGELOG.md]
Agent:   "Planner session complete. Review the files. Place [Halt here]
          if needed. Type start execution when ready."
```

**You review the plan.** Maybe add `[Halt here]` after Step 5 if
you want to check backend work before frontend starts.

**Generator session:**

```
You:     start execution

Agent:   [Reads Plan.md, executes tickets, commits after each]
Agent:   [Hits your [Halt here] → commits and stops]
         or
Agent:   [Completes all tickets → stops]
Agent:   "Generator session complete. Ready for your evaluation."
```

**You evaluate.** Run global tests. Click through the app. If
satisfied, delete Plan.md. Full loop done.

### Modifying

```
You:     [/modify] Add batch upload with concurrent PDF processing

Agent:   [Reads Concept.md and Architecture.md, detects conflicts]
Agent:   "Warning: Current pipeline is synchronous. Batch processing
          needs async. Options: ..."

You:     proceed to spec

Agent:   [Updates Architecture.md surgically, writes fresh Plan.md]
```

### Debugging

```
You:     [/debug] Extraction returns empty text for scanned PDFs

Agent:   [Creates Triage.md with hypotheses and test tickets]
Agent:   "Hypothesis 1: OCR fallback not triggered. Tier 1 — permanent test."

You:     start execution

Agent:   [Tests hypothesis, fixes if proven, commits]
```

### Migrating an Existing Codebase

```
You:     [/migrate]

Agent:   [Reads the codebase, asks about vision and principles]

You:     This is a customer support tool. We built it fast.
         It needs to stay simple — no user accounts, just ticket
         submission. We're thinking about adding AI responses later.

You:     proceed to spec

Agent:   [Writes Concept.md, Architecture.md (as-is), skills
          matching existing patterns. No code changes.]
Agent:   "Migration complete. Use [/modify] for changes,
          [/debug] for bugs."
```

After migration, start with a small `[/modify]` task to verify
the framework understands the codebase.

## Key Concepts

### Task Tickets

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

**Skills to Load:** @skills/fastapi-backend/SKILL.md
**Boundary:** src/pipeline/, tests/test_extractor.py
**Run Command:** uv run pytest tests/test_extractor.py -v
```

### Generator Retry Logic

When a ticket's tests fail after implementation:
1. Attempt to fix (try 1).
2. If still failing, attempt again (try 2).
3. If still failing, final attempt (try 3).
4. If still failing: commit progress with a WIP message explaining
   what worked and what didn't. Stop the session.

### `[Halt here]` Flags

The Planner does NOT place halt flags. You place them after reviewing
Plan.md, wherever you want the Generator to pause:

- Before domain shifts (backend → frontend)
- At natural review points
- Where you want to evaluate before continuing

When the Generator hits a `[Halt here]`, it commits and stops.

### Bespoke Skills

Planner-generated, project-specific files with:
- Canonical file structure (Generator can't create outside it)
- Copy-paste code patterns with your actual types and names
- DO / DO NOT rules (binary, no judgment)
- Error handling contracts
- Testing conventions
- Exact copy-pasteable commands

### The Evaluation Model

**Layer 1 — TDD (automated):** Tests before code. Catches regressions.

**Layer 2 — Manual Verification:** Each ticket lists what you should
inspect after tests pass — UX, visual, integration.

**Layer 3 — You:** Run global tests. Delete Plan.md/Triage.md when
satisfied. The full loop isn't done until you say so.

## Claude Code vs Cursor: What's Different?

The framework is tool-agnostic by design. Here's what each tool
does with the framework files:

| Concern | Claude Code | Cursor |
|---|---|---|
| Router loading | `CLAUDE.md` auto-loaded on startup | `CLAUDE.md` auto-loaded as workspace rule |
| Rule files | `.claude/rules/*.md` auto-loaded | Read on-demand when referenced by router/skills |
| Skill files | Read via `Read` tool when instructed | Read via `Read` tool when instructed |
| `@` references | Plain text (agent resolves paths) | File references (IDE resolves paths) |
| Git operations | Terminal commands | Terminal commands |
| Mode commands | Typed in chat | Typed in chat |

**No code changes needed** to switch tools. The same `CLAUDE.md`,
`.claude/rules/`, and `skills/` work everywhere.

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
| 5.3 | Phase-based authority. Unified Planner/Generator pipeline. User as Evaluator. Inspired by Anthropic harness research. |
| 5.5 | Session-based development. Git as checkpoint system. Concept.md restored as vision document. STATE.md eliminated. Plan/Triage ephemeral. User-placed `[Halt here]` only. `[/migrate]` mode. 3-retry logic. `[/mode]` command format. |
| **5.6** | **Dual-tool compatibility: Claude Code + Cursor. `CLAUDE.md` serves as unified router for both tools. Framework is fully tool-agnostic — same files, same workflow, any agent.** |

## Tips

- **Start small.** First project should be buildable in a day.
- **Review generated skills.** If the Generator deviates, skills are
  underspecified. Tighten DO NOT rules.
- **Place `[Halt here]` strategically.** Before domain shifts, at
  natural review points. You know your model's limits.
- **Read git logs.** `git log --oneline` gives you the progress trail.
  Each ticket commit tells you what was built.
- **Delete Plan.md when done.** It's a work order, not documentation.
  Architecture.md and git history are the permanent record.
- **Switch tools freely.** Start a Planner session in Cursor, run the
  Generator in Claude Code (or vice versa). Git keeps everything in sync.
- **Re-examine constraints.** Each rule assumes the model can't do
  something on its own. As models improve, strip what's not load-bearing.

## License

Open for personal and commercial use. Attribution appreciated.
