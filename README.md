# Vibe Coding Framework 5.3

A structured AI development framework that separates planning from execution and puts you in charge of evaluation. The Planner designs. The Generator builds. You decide when it's done.

## The Problem

AI coding agents fail in predictable ways:

1. **Context collapse.** The agent forgets its own decisions mid-session and contradicts itself.
2. **Architectural hallucination.** Without a planning phase, the agent invents your architecture and writes spaghetti.
3. **Poor self-evaluation.** Agents confidently approve their own mediocre work. Separating the judge from the builder is the fix.

This framework solves all three by enforcing a strict Planner → Generator pipeline, externalizing the agent's working memory into structured files, and making you the final evaluator of every change.

## How It Works

### The Pipeline

Every task — build, modify, or debug — flows through the same two-phase pipeline:

```
Planner Phase                    Generator Phase
(Ask + Spec)                     (Execution)

 Interrogate user                 TDD loop per ticket
 Design architecture      ──→    Present results to user
 Write task tickets               User evaluates
 Generate skills                  Commit after approval
 Checkpoint STATE.md              Halt at [Halt here]
```

**Planner Phase** has full authority over core files (Architecture.md, Plan.md, skills, STATE.md). It designs, plans, and produces blueprints precise enough for mechanical execution.

**Generator Phase** has zero authority over core files. It reads task tickets from Plan.md and executes them literally. If it encounters a gap in the spec, it halts — it never improvises.

**You are the Evaluator.** The Generator runs automated tests as a sanity check, then presents results and a manual verification checklist. You decide whether to approve the commit.

### Phase-Based Authority

Authority is determined by which phase is active, not which model is running. The same model can run both phases — the rules bind to the phase.

| File | Planner Phase | Generator Phase |
|---|---|---|
| Architecture.md | Read / Write | Read only |
| Plan.md / Triage.md | Read / Write | Read only (mark `[x]`) |
| STATE.md | Read / Write | Checkpoint only |
| CHANGELOG.md | Read / Write | Append only |
| skills/ | Read / Write / Create | Read only |
| src/, tests/ | — | Read / Write (within Boundary) |

### The Three Modes

All three modes use the same Planner → Generator pipeline. They differ in persona and focus:

| Command | Persona | Focus |
|---|---|---|
| `/build [concept]` | The Architect | 0-to-1 creation from scratch |
| `/modify [feature]` | The Refactoring Engineer | Feature additions, architectural shifts, backwards compatibility |
| `/debug [issue]` | The QA Lead | Root-cause analysis, hypothesis-driven bug fixing |

**The Architect** scopes a new project from a raw idea. The Planner interrogates you about requirements, designs the full architecture, generates project-specific skills, and writes a detailed execution plan.

**The Refactoring Engineer** modifies an existing system. The Planner reads the current architecture first, detects conflicts between your request and the existing design, updates the spec surgically, and adds regression-aware task tickets.

**The QA Lead** investigates bugs. The Planner classifies the bug (permanent regression test vs throwaway sandbox), formulates hypotheses with test tickets, and enforces one-hypothesis-at-a-time execution. It uses Triage.md instead of Plan.md.

## File Structure

```
your-project/
├── CLAUDE.md                       ← Router (auto-loaded, ~73 lines)
├── .claude/
│   └── rules/
│       ├── governance.md           ← Git, security, TDD, user evaluation
│       ├── phase-authority.md      ← File authority matrix, boundary rules
│       ├── boot-sequence.md        ← Re-hydration protocol for continue plan
│       └── task-ticket-format.md   ← Plan.md step format + Manual Verification
├── skills/
│   ├── skill-template/SKILL.md     ← How to write skills for the Generator
│   ├── state-schema/SKILL.md       ← STATE.md format (the operational hub)
│   ├── mode-build/SKILL.md         ← The Architect workflow
│   ├── mode-modify/SKILL.md        ← The Refactoring Engineer workflow
│   └── mode-debug/SKILL.md         ← The QA Lead workflow
│
│   (Generated during Spec — project-specific:)
│   ├── fastapi-backend/SKILL.md
│   └── react-frontend/SKILL.md
│
├── Architecture.md                 ← System design (source of truth)
├── Plan.md                         ← Task tickets for the Generator
├── STATE.md                        ← Operational hub: summary, tracking, decisions
├── CHANGELOG.md                    ← What was built, when
├── Triage.md                       ← Bug investigation (debug mode only)
└── README.md
```

### What Each File Does

**Architecture.md** — The structural source of truth. Every endpoint, model, component, and data flow lives here. If it's not in this file, the Generator will halt rather than invent it.

**Plan.md** — The work order. Each step is a task ticket with inputs, outputs, function signatures, test contracts, manual verification criteria, and boundary constraints. The Generator executes these mechanically.

**STATE.md** — The operational hub. Contains the Project Summary (compressed working memory), phase tracking, tech stack, key decisions log, active skills, and any blocked escalations. This is the first file read on every session start.

**Skills** — The interface between planning intent and mechanical execution. Project-specific files generated by the Planner with copy-paste code patterns, DO/DO NOT rules, error handling contracts, and test conventions — all using your project's actual types and names.

**Triage.md** — Debug mode only. The detective's notebook with hypotheses, bug classifications, and test tickets.

## Setup

### Prerequisites

- [Claude Code](https://code.claude.com/docs/en/setup) installed
- Claude Pro, Max, Teams, or Enterprise account
- Git initialized in your project

### Install Claude Code

```bash
# macOS / Linux
curl -fsSL https://claude.ai/install.sh | bash

# Windows (PowerShell)
irm https://claude.ai/install.ps1 | iex

# Verify
claude --version
```

### Add the Framework to Your Project

```bash
mkdir my-project && cd my-project && git init

# Copy framework files
cp CLAUDE.md my-project/
cp -r .claude my-project/
cp -r skills my-project/
```

Claude Code auto-detects `CLAUDE.md` and `.claude/rules/` when you launch from this directory.

## Usage

### Building a New Project

**Planner session:**

```bash
claude
```

```
You:     /build A FastAPI app that uploads PDFs, extracts text, and
         summarizes them with an LLM. React frontend.

Claude:  [Asks questions about stack, testing, data model, edge cases]

You:     proceed to spec

Claude:  [Writes Architecture.md, Plan.md, generates skills, STATE.md]
Claude:  "Spec Phase complete. Review the files. Type start execution."
```

**Generator session:**

```
You:     start execution

Claude:  [Reads STATE.md, loads skills, finds first task ticket]
Claude:  [Writes tests → implements → tests pass]
Claude:  "Tests passing. Here's what I built and the manual checks:
          - [x] test_create_document_success
          - [x] test_create_document_missing_field
          Manual verification:
          - POST /documents/ returns 201 with valid payload
          - POST /documents/ returns 422 with empty body
          Ready to commit?"

You:     [review, test manually if needed] yes

Claude:  [Commits, continues to next ticket or halts]
```

**Continuation:**

```
You:     continue plan

Claude:  "Re-hydration complete.
          Last completed: Phase 3, Step 3 — Auth routes
          Next up: Phase 3, Step 4 — Extraction pipeline
          Reply 'confirmed' to proceed."

You:     confirmed
```

### Modifying an Existing Project

```
You:     /modify Add batch upload for multiple PDFs with progress tracking

Claude:  [Reads existing architecture, detects conflicts]
Claude:  "Warning: Current pipeline processes PDFs synchronously.
          Batch upload needs async processing. Should we migrate the
          entire pipeline or add a separate batch path?"

You:     [discuss, decide] proceed to spec

Claude:  [Surgically updates Architecture.md, extends Plan.md]
```

### Debugging

```
You:     /debug Extraction returns empty text for scanned PDFs.
         Error: ExtractedContent.text is empty string.

Claude:  [Asks reproduction steps, classifies bug]

You:     proceed to spec

Claude:  [Writes hypotheses in Triage.md with test tickets]
Claude:  "Hypothesis 1: OCR fallback not triggered for image-only PDFs.
          Classification: Tier 1 — Permanent regression test.
          Type start execution to test this hypothesis."
```

## Key Concepts

### Task Tickets

Every step in Plan.md is a structured ticket:

```markdown
### Phase 3, Step 4: Build Extraction Pipeline

**Input:** src/models/document.py
**Output:** src/pipeline/extractor.py, tests/test_extractor.py
**Spec:**
- Function `extract_text(doc: Document) -> ExtractedContent`
- Handle: PDF, DOCX, plain text
- On failure: raise `ExtractionError` with source format

**Test Contract:**
- test_extract_pdf_success: valid PDF → ExtractedContent with .text
- test_extract_unsupported: .xlsx → ExtractionError
- test_extract_corrupt: truncated PDF → ExtractionError

**Manual Verification:**
- Upload a multi-page PDF via the API; confirm all pages extracted
- Upload a scanned PDF; confirm OCR fallback produces text
- Upload a 0-byte file; confirm clean error response, no crash

**Skills to Load:** @skills/fastapi-backend/SKILL.md
**Boundary:** src/pipeline/, tests/test_extractor.py
**Run Command:** uv run pytest tests/test_extractor.py -v
```

The **Boundary** field constrains which files the Generator can touch. The **Manual Verification** field tells you what to check after tests pass.

### The Halt Protocol

`[Halt here]` flags mark safe stopping points:

- Before domain shifts (backend → frontend)
- Every 3-5 execution steps
- Before the final Global Test Phase (you run this manually)

When the Generator hits `[Halt here]`, it checkpoints STATE.md, gets your commit approval, and tells you to start a fresh session.

### Blocked Escalations

When the Generator encounters a spec gap, it halts with a factual problem statement:

```markdown
## Blocked
**Phase:** Phase 3, Step 5
**Problem:** Architecture.md defines REST endpoints only but Step 7
  requires real-time progress updates.
**Evidence:** Architecture.md Section 3.2 — no streaming spec.
**Files touched before halt:** None
```

No suggestions, no workarounds. You start a Planner session to resolve the architectural question, then the Generator resumes.

### STATE.md — The Operational Hub

STATE.md replaced Concept.md as the single source of operational state. It includes:

- **Project Summary** — a 20-30 line compressed snapshot of the system (what it does, major components, current scope). This is the fast-path context — the Generator reads this instead of the full Architecture.md for most tasks.
- **Phase History** — what was completed, by which phase (planner/generator).
- **Tech Stack** — canonical list of all technologies.
- **Key Decisions Log** — append-only record of why things changed.
- **Active Skills** — which skill files to load for the current work.
- **Blocked** — present only when the Generator has halted on a boundary violation.

### Bespoke Sk— binary, no judgment calls required
- **Error handling contracts** — exactly what to raise and catch
- **Testing conventions** — naming, fixtures, minimum coverage
- **Exact commands** — every CLI command, ready to paste

Skills load on demand via `@skills/name/SKILL.md` — only the skills needed for the current ticket enter the context window.

## The Evaluation Model

This framework uses a three-layer evaluation approach:

**Layer 1 — Automated (TDD):** The Generator writes tests before code. Tests catch regressions and prove basic functionality. This is the automated sanity check.

**Layer 2 — Manual Verification:** Each task ticket includes a Manual Verification field specifying what you should inspect. The Generator presents this checklist alongside test results.

**Layer 3 — You:** No commit happens without your approval. You review test results, run manual checks, and decide whether the work meets your standards. The Global Test Phase at the end of every plan is exclusively yours to run.

The framework deliberately avoids automated self-evaluation because agents evaluate their own work poorly. Separating the builder from the judge — and making the judge human — produces better outcomes for projects at this complexity level.

## Applying to an Existing Codebase

The framework assumes it built the project from scratch. If you're
dropping it into an existing codebase — one you built manually, that
another AI generated, or that runs an older framework version — you
need a bootstrap step first.

### Migrating from an Older Framework Version (v5.0–5.2)

If your project already has framework artifacts (Concept.md, old-format
Plan.md, STATE.md without a Project Summary, skills using "Good Model /
Ok Model" terminology):

1. Drop the v5.3 framework files into the project root (`CLAUDE.md`,
   `.claude/rules/`, `skills/mode-*`, `skills/skill-template`,
   `skills/state-schema`), overwriting old framework infrastructure.
2. **Keep** your existing `Architecture.md`, `Plan.md`, and
   project-specific skills (like `skills/fastapi-backend/`) — don't
   delete them yet.
3. Start a Planner session:

```
/modify Migrate this project's framework artifacts from v5.x to v5.3.
Read the existing Architecture.md and any Concept.md, Plan.md, and
skills. Rewrite STATE.md with a Project Summary synthesized from the
existing docs. Update project-specific skills to match the v5.3
skill-template standard. Delete Concept.md after migrating its content
into the Project Summary.
```

The Planner reads everything, synthesizes it into the v5.3 format,
and you're running the new framework. Existing application code
is untouched — only the metadata layer changes.

### Bootstrapping a Codebase with No Framework

If your project has no framework artifacts at all — just source code,
maybe some tests, maybe a README:

1. Drop the v5.3 framework files into the project root.
2. Start a Planner session:

```
/build Bootstrap this existing codebase into the Vibe Coding 5.3
framework. Do NOT write any application code. Read the existing
source tree, then produce:
1. Architecture.md documenting the current system as-is
2. STATE.md with a Project Summary and tech stack
3. Bespoke skills for the existing stack and conventions
Mark all existing code as completed in STATE.md Phase History.
```

The Planner reverse-engineers the architecture from the code, documents
it, generates skills that match the existing patterns, and sets up
STATE.md as if the project had been built with the framework from the
start. No application code changes — just the metadata layer.

After bootstrapping, `/modify` and `/debug` work normally because
the Planner has Architecture.md to check conflicts against and skills
that reflect the actual codebase.

### Important: Skills Must Match Reality

When bootstrapping, the generated skills should describe patterns
the codebase **already uses**, not patterns the framework wishes
it used. If the existing code uses raw SQL instead of an ORM, the
skill documents raw SQL patterns. If it catches generic exceptions
everywhere, the skill documents that reality.

The DO NOT list in skills can flag existing anti-patterns as tech
debt to clean up later via `/modify`. But the skill's patterns must
match the code as it is today, or the Generator will fight the
codebase during execution.

### After Bootstrap: First Real Task

Once bootstrap is complete, your first real task should be small:

```
/modify Add input validation to the /users endpoint
```

This lets you verify that the Planner correctly understood the
architecture and that the Generator follows the generated skills
without deviating. If the Generator produces code that clashes with
existing conventions, the skills need tightening — better to discover
that on a small change than a large feature.

## Version History

| Version | Key Change |
|---|---|
| 1.0–3.0 | Standard prompt-in, code-out. Context collapse. |
| 3.5 | Decoupled design from implementation. HITL gates. |
| 4.0 | 3-Phase Pipeline (Ask → Spec → Execute). Git, TDD. |
| 4.1 | Dynamic skill creation replaced static skill banks. |
| 4.2 | Programmable `[Halt here]` for context-aware chunking. |
| 5.0 | Strategy Pattern: `/build`, `/modify`, `/debug` personas. |
| 5.1 | Two-Tier Bug Classification. |
| 5.2 | Two-Tier Model Architecture, STATE.md, task tickets, Blocked protocol. |
| **5.3** | **Phase-based authority (no model labels). Unified Planner → Generator pipeline. User as Evaluator. Manual Verification in tickets. Concept.md eliminated — STATE.md is the operational hub with Project Summary. Inspired by Anthropic's harness design research.** |

## Tips

- **Start small.** Your first project should be something you could build in a day. Verify the framework works before trusting it with multi-session builds.
- **Review generated skills.** If the Generator keeps deviating, the skills are underspecified. Tighten the DO NOT rules and add more concrete patterns.
- **Don't skip the handshake.** When the Generator announces its re-hydration summary, read it. This is where you catch drift early.
- **The Global Test Phase is yours.** You run the final test suite. This is your last line of defense.
- **Re-examine constraints as models improve.** Each rule encodes an assumption about what the model can't do on its own. As models get better, some constraints become unnecessary overhead. Strip what's no longer load-bearing.
- **Use `/compact` carefully in Claude Code.** CLAUDE.md survives compaction (re-read from disk), but in-conversation context doesn't. The framework's file-based architecture means most state survives naturally.

## Adapting to Other Agents

This README covers the Claude Code edition. The framework also works with:

- **Cursor** — use a single `.cursorrules` file instead of the split `CLAUDE.md` + `.claude/rules/` architecture. Skills and STATE.md work identically.
- **Other agents** — any agent that reads markdown files from the workspace can use this framework. The key requirement is that the agent can be instructed to read specific files before acting.

## License

Open for personal and commercial use. Attribution appreciated but not required.
