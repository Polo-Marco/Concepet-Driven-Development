# Vibe Coding Framework 5.2 — Claude Code Edition

A structured AI development framework that splits system design from implementation across two model tiers. The Good Model architects. The Ok Model executes. The framework is the contract between them.

Adapted for [Claude Code](https://code.claude.com), Anthropic's terminal-based coding agent.

## Why This Exists

Standard AI coding suffers from three failure modes:

1. **Context collapse.** The AI forgets its own decisions mid-session and starts contradicting itself.
2. **Architectural hallucination.** Without a planning phase, the AI guesses your architecture and writes spaghetti.
3. **Cost inefficiency.** Using a top-tier model for every line of boilerplate code burns budget on work that doesn't require intelligence.

This framework solves all three by enforcing a strict planning-then-execution pipeline, externalizing the AI's "working memory" into structured files, and splitting work between an expensive model (for design) and a cheap model (for implementation).

## How It Works

### The Two-Tier Model

| | Good Model (Tier 1) | Ok Model (Tier 2) |
|---|---|---|
| **Role** | Architect, Spec Author, Skill Creator | Code Implementer, Test Writer |
| **Phases** | Phase 1 (Ask) + Phase 2 (Spec) | Phase 3 (Execution) |
| **Cost** | Expensive, used sparingly | Cheap, used for volume work |
| **Example** | Sonnet 4.6 | Haiku 4.5 |

The Good Model writes the blueprints. The Ok Model follows them mechanically. The Ok Model is **never allowed** to make architectural decisions — if it encounters a gap in the spec, it halts and escalates.

### The Three Phases

```
Phase 1: Ask ──── Phase 2: Spec ──── Phase 3: Execution
(Good Model)      (Good Model)       (Ok Model)

 Interrogate        Architecture.md     TDD loop per
 the user    ──→    Plan.md         ──→ task ticket
 Document in        Skills              Commit + checkpoint
 Concept.md         STATE.md            Halt at [Halt here]
```

**Phase 1 — Ask:** The Good Model interviews you about requirements, tech stack, testing frameworks, data models, and edge cases. Nothing gets built until ambiguity is eliminated.

**Phase 2 — Spec:** The Good Model writes `Architecture.md` (system design), generates bespoke skills with copy-paste code patterns for the Ok Model, and creates `Plan.md` with detailed task tickets. Each ticket specifies exact inputs, outputs, function signatures, test contracts, and file boundaries.

**Phase 3 — Execution:** The Ok Model picks up `Plan.md` and executes one task ticket at a time using TDD (write failing test → implement → pass). It commits after each step and halts at `[Halt here]` flags to prevent context degradation.

### The Three Modes

| Command | Persona | Use When |
|---|---|---|
| `/build [concept]` | The Architect | Starting a new project from scratch |
| `/modify [feature]` | The Refactoring Engineer | Adding features or changing existing code |
| `/debug [issue]` | The QA Lead | Hunting and fixing bugs |

Each mode loads a dedicated skill file that defines the specific workflow, audit checks, and testing strategy for that type of work.

## File Structure

```
your-project/
├── CLAUDE.md                 ← Router (auto-loaded every session, ~55 lines)
├── .claude/
│   └── rules/
│       ├── governance.md     ← Git, security, TDD, halt protocol (auto-loaded)
│       ├── two-tier-authority.md  ← File authority matrix, boundary rules
│       ├── boot-sequence.md  ← Re-hydration protocol for continue plan
│       └── task-ticket-format.md  ← Plan.md step format reference
├── skills/
│   ├── skill-template/       ← Meta-skill: how to write skills for the Ok Model
│   │   └── SKILL.md
│   ├── state-schema/         ← STATE.md checkpoint format reference
│   │   └── SKILL.md
│   ├── mode-build/           ← The Architect workflow
│   │   └── SKILL.md
│   ├── mode-modify/          ← The Refactoring Engineer workflow
│   │   └── SKILL.md
│   └── mode-debug/           ← The QA Lead workflow
│       └── SKILL.md
│
│   (Generated during Spec phase — examples:)
│   ├── fastapi-backend/
│   │   └── SKILL.md
│   └── react-frontend/
│       └── SKILL.md
│
├── Concept.md                ← Raw requirements + Project Summary
├── Architecture.md           ← System design (source of truth)
├── Plan.md                   ← Task tickets for the Ok Model
├── STATE.md                  ← Session checkpoint / save game file
├── CHANGELOG.md              ← What was built, when
├── Triage.md                 ← Bug investigation (debug mode only)
└── README.md
```

### Why the Split Architecture?

Claude Code auto-loads `CLAUDE.md` and everything in `.claude/rules/` at session start. Files under 200 lines achieve over 90% instruction adherence; longer files degrade significantly. So the monolithic rules file is split into a lean 55-line router plus four focused rule modules, each under 80 lines. Skills load on demand via `@` file references — only the skills needed for the current task enter the context window.

## Prerequisites

- **Claude Code** installed ([setup guide](https://code.claude.com/docs/en/setup))
- **Claude Pro, Max, Teams, or Enterprise** account (free plan does not include Claude Code)
- **Git** initialized in your project

### Install Claude Code

**macOS / Linux:**

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

**Windows (PowerShell):**

```powershell
irm https://claude.ai/install.ps1 | iex
```

Verify:

```bash
claude --version
```

First run opens your browser for one-time authentication:

```bash
claude
```

## Setup

```bash
# Create your project
mkdir my-project && cd my-project
git init

# Copy the framework into your project
cp CLAUDE.md my-project/
cp -r .claude my-project/
cp -r skills my-project/
```

Verify the structure:

```
my-project/
├── CLAUDE.md
├── .claude/rules/ (4 files)
└── skills/ (5 directories)
```

That's it. Claude Code auto-detects `CLAUDE.md` and `.claude/rules/` when you launch a session from this directory.

## Usage

### Your First Build

**Session 1 — Good Model:**

```bash
# Launch with your strong model
claude --model claude-sonnet-4-6-20250514
```

Inside the session:

```
You:     /build A FastAPI app that uploads PDFs, extracts text, and
         summarizes them with an LLM. React frontend for upload and results.

Claude:  [Asks detailed questions about stack, testing, data model, edge cases]

You:     proceed to spec

Claude:  [Writes Architecture.md, Plan.md, generates bespoke skills, STATE.md]
Claude:  "Spec Phase complete. Review the files. Start an Ok Model session
          and type start execution."
```

**Session 2 — Ok Model:**

```bash
# Launch with your cheap model
claude --model claude-haiku-4-5-20251001
```

Inside the session:

```
You:     start execution

Claude:  [Reads STATE.md, loads skills, finds first task ticket]
Claude:  [Writes failing test → implements code → test passes → commits]
Claude:  "Phase 3, Step 3 complete. Halting. Start a new session,
          type continue plan."
```

**Session 3+ — Ok Model (continuation):**

```bash
claude --model claude-haiku-4-5-20251001
```

```
You:     continue plan

Claude:  "Re-hydration complete.
          Last completed: Phase 3, Step 3 — Auth routes
          Next up: Phase 3, Step 4 — Extraction pipeline
          Reply 'confirmed' to proceed."

You:     confirmed

Claude:  [Continues executing task tickets...]
```

### Adding Features

Switch back to the Good Model:

```bash
claude --model claude-sonnet-4-6-20250514
```

```
You:     /modify Add batch upload that processes multiple PDFs concurrently
         with a progress bar.
```

The Refactoring Engineer checks for architectural conflicts, updates the spec surgically, and generates new task tickets.

### Debugging

```bash
claude --model claude-sonnet-4-6-20250514
```

```
You:     /debug The extraction pipeline returns empty text for scanned PDFs.
         Stack trace: [paste error]
```

The QA Lead triages, classifies the bug (permanent test vs throwaway sandbox), formulates hypotheses, and writes verification tickets for the Ok Model.

## Key Concepts

### Task Tickets

Every step in `Plan.md` is a structured ticket the Ok Model follows literally:

```markdown
### Phase 3, Step 4: Build Extraction Pipeline

**Input:** src/models/document.py (Document schema)
**Output:** src/pipeline/extractor.py, tests/test_extractor.py
**Spec:**
- Function `extract_text(doc: Document) -> ExtractedContent`
- Must handle: PDF, DOCX, plain text
- On failure: raise `ExtractionError` with source format in message

**Test Contract:**
- test_extract_pdf_success: valid PDF → ExtractedContent with .text populated
- test_extract_unsupported: .xlsx → raises ExtractionError
- test_extract_corrupt: truncated PDF → raises ExtractionError

**Skills to Load:** @skills/fastapi-backend/SKILL.md
**Boundary:** src/pipeline/, tests/test_extractor.py
**Run Command:** uv run pytest tests/test_extractor.py -v
```

The **Boundary** field is critical — it tells the Ok Model exactly which files it may touch. Anything outside the boundary triggers a mandatory halt.

### The Halt Protocol

`[Halt here]` flags in `Plan.md` mark safe stopping points placed by the Good Model:

- Before domain shifts (backend → frontend)
- Every 3–5 execution steps
- When the context window is at risk of degradation
- Before the final Global Test Phase (which you run manually)

When the Ok Model hits `[Halt here]`, it checkpoints `STATE.md`, commits, and tells you to start a fresh session.

### Blocked Escalations

If the Ok Model encounters something the spec doesn't cover, it **cannot improvise**. It writes a factual problem statement in `STATE.md` and halts:

```markdown
## Blocked
**Phase:** Phase 3, Step 4
**Problem:** Architecture.md specifies REST endpoints but the frontend
  component in Step 7 expects WebSocket push for real-time progress.
**Evidence:** Architecture.md Section 3.2 defines only GET/POST routes.
**Files touched before halt:** src/pipeline/extractor.py (partial)
```

You then start a Good Model session. It reads the Blocked section, resolves the architectural issue, updates the spec, clears the block, and the Ok Model can resume.

### Bespoke Skills

During Spec phase, the Good Model generates project-specific skill files with:

- **Canonical file structure** (the Ok Model can't create files outside it)
- **Copy-paste code patterns** using your project's actual types and names
- **DO / DO NOT rules** (binary, no judgment calls)
- **Error handling contracts** (exactly what to raise and catch)
- **Testing conventions** (naming, fixtures, minimum coverage)
- **Exact commands** (every CLI command, ready to paste)

Skills are loaded on demand using Claude Code's `@` file reference syntax (e.g., `@skills/fastapi-backend/SKILL.md`). Only the skills relevant to the current task ticket enter the context window.

### STATE.md — The Save Game

`STATE.md` is the handoff document between sessions and between model tiers. It records:

- Active mode and current phase/step
- Tech stack summary
- Phase history with model tier annotations (who did what)
- Key decisions log (append-only, never deleted)
- Active skill file paths
- Files modified in the last session
- Blocked sections (when the Ok Model has halted on a boundary violation)

Written automatically at every `[Halt here]` checkpoint.

## File Authority

| File | Good Model | Ok Model |
|---|---|---|
|Model Switching

Use the `--model` flag when launching Claude Code:

```bash
# Good Model session
claude --model claude-sonnet-4-6-20250514

# Ok Model session
claude --model claude-haiku-4-5-20251001
```

Or switch interactively inside a session with the `/model` command.

### How Rules Load

Claude Code has a specific loading order:

1. `CLAUDE.md` (project root) — loaded first, every session
2. `.claude/rules/*.md` — loaded automatically alongside CLAUDE.md
3. `~/.claude/CLAUDE.md` (user-level, optional) — personal preferences across all projects
4. Skills (`@skills/...`) — loaded on demand when referenced

The framework's rules in `.claude/rules/` auto-load without you doing anything. Skills load only when a task ticket references them, saving context window space.

### Line Count Budget

All framework files are sized for Claude Code's adherence sweet spot:

| File | Lines |
|---|---|
| CLAUDE.md | 55 |
| governance.md | 32 |
| two-tier-authority.md | 47 |
| boot-sequence.md | 54 |
| task-ticket-format.md | 76 |
| Mode skills | 103–126 each |

Every file is under the 200-line threshold where adherence drops significantly.

### Differences from the Cursor Version

| Aspect | Cursor | Claude Code |
|---|---|---|
| Rules file | `.cursorrules` (single file) | `CLAUDE.md` + `.claude/rules/` (split) |
| Auto-loading | `.cursorrules` only | CLAUDE.md + all rules/ files |
| Skill references | `./skills/path` | `@skills/path` |
| Model switching | UI model picker | `--model` flag or `/model` |
| Slash commands | Native Cursor feature | Plain text pattern matching |

## Version History

| Version | Key Change |
|---|---|
| 1.0–3.0 | Standard prompt-in, code-out. Suffered context collapse. |
| 3.5 | Decoupled design from implementation. Introduced HITL gates. |
| 4.0 | 3-Phase Pipeline (Ask → Spec → Execute). Git, TDD, CHANGELOG. |
| 4.1 | Dynamic skill creation replaced static skill banks. |
| 4.2 | Programmable `[Halt here]` flags for context-aware chunking. |
| 5.0 | Strategy Pattern: `/build`, `/modify`, `/debug` personas. |
| 5.1 | Two-Tier Bug Classification (permanent tests vs throwaway sandboxes). |
| **5.2** | **Two-Tier Model Architecture, STATE.md re-hydration, task tickets with Boundary fields, Blocked escalation protocol. Claude Code adaptation with split rules architecture.** |

## Tips

- **Start small.** Your first project should be something you could build manually in a day. This lets you verify the framework works before trusting it with complex multi-session builds.
- **Review the generated skills.** If the Ok Model keeps deviating, the skills are probably underspecified. Tighten the DO NOT rules.
- **Don't skip the handshake.** When the Ok Model announces its re-hydration summary after `continue plan`, actually read it. This is where you catch drift before it costs you hours.
- **The Global Test Phase is yours.** The framework deliberately makes you run the final test suite manually. This is your last line of defense.
- **Check context consumption.** Run `/memory` in Claude Code to see how much of your context window the framework files consume. If it feels heavy, move rarely-used skills out of `skills/` and reference them explicitly with `@` only when needed.
- **Use `/compact` sparingly.** Claude Code's `/compact` command summarizes the session. CLAUDE.md survives compaction (re-read from disk), but in-conversation instructions don't. The framework's file-based architecture means most state survives compaction naturally.

## License

This framework is open for personal and commercial use. Attribution appreciated but not required.
