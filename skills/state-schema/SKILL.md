---
name: state-schema
description: Defines the STATE.md checkpoint format. The handoff document between sessions and model tiers.
author: framework
version: 5.2-cc
---

# STATE.md Schema

## Purpose

STATE.md is the save game file. It enables:
1. Session continuity after `[Halt here]` resets.
2. Model tier handoff (Ok Model halts → Good Model resolves).
3. Audit trail via Phase History and Key Decisions Log.

## Write Rules

| Actor | Allowed Operations |
|---|---|
| Good Model | Read/Write all. Resolve `## Blocked`. |
| Ok Model | Write checkpoint at `[Halt here]`. Write `## Blocked` on boundary violation. |
| Human | May annotate any section. |

## Template

```markdown
# System State
<!-- Last updated: [ISO 8601 timestamp] -->

## Active Mode
[mode-build | mode-modify | mode-debug]

## Current Phase
Phase [N]: [Name] — Step [M]: [Title]

## Tech Stack
- Language: [e.g., Python 3.12]
- Backend: [e.g., FastAPI]
- Frontend: [e.g., React 18, Vite]
- Database: [e.g., PostgreSQL via SQLAlchemy]
- Testing: [e.g., pytest (backend), Vitest (frontend)]
- Package Manager: [e.g., uv]

## Phase History
- [x] Phase 1: Discovery (good-model)
- [x] Phase 2: Architecture & Planning (good-model)
- [x] Phase 3, Step 1: Scaffolding (ok-model)
- [ ] Phase 3, Step 2: Auth routes ← NEXT

## Key Decisions Log
<!-- Append-only. Never delete. -->
- [YYYY-MM-DD]: [decision]: [rationale]

## Active Skills
- @skills/fastapi-backend/SKILL.md
- @skills/react-frontend/SKILL.md

## Files Modified Last Session
- src/models/user.py (created)
- tests/conftest.py (modified)

## Blocked
<!-- ONLY present when Ok Model halted on boundary violation. -->
<!-- Good Model: resolve, then delete this section. -->
**Phase:** [phase and step]
**Problem:** [factual — no suggestions]
**Evidence:** [reference to spec that's insufficient]
**Files touched before halt:** [list]
```

## Checkpoint Timing

| Event | Who | What |
|---|---|---|
| `[Halt here]` reached | Ok Model | Full checkpoint |
| Boundary violation halt | Ok Model | Add `## Blocked` |
| Block resolved | Good Model | Clear `## Blocked`, update specs |
| Spec Phase done | Good Model | Create/rewrite STATE.md |
| Major Revision | Good Model | Append Key Decisions, update history |

## Anti-Patterns

- **No code in STATE.md.** Metadata only.
- **No suggestions in Blocked.** Ok Model states the problem, not the solution.
- **No duplicated spec.** Reference Architecture.md, don't copy it.
- **Delete resolved Blocked sections.** Log the resolution in Key Decisions instead.
- **Always update Phase History.** Next session uses it to find the starting point.
- **Always update Files Modified.** Modified files are where regressions hide.