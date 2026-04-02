# Phase-Based Authority & Boundary Rules

## Core Principle

Authority is determined by the active session type, not the model.
The Planner Session has full authority over core files.
The Generator Session has zero authority over core files.

## File Authority Matrix

| File | Planner Session | Generator Session |
|---|---|---|
| `Concept.md` | Read / Write | **Read only** |
| `Architecture.md` | Read / Write | **Read only** |
| `Plan.md` / `Triage.md` | Read / Write | **Read only** (mark `[x]`) |
| `CHANGELOG.md` | Read / Write | **Append only** |
| `./skills/**` | Read / Write / Create | **Read only** |
| `src/`, `tests/` | — | Read / Write (within Boundary) |

## Generator Boundary Rules

During a Generator Session, the agent MUST stop the session if ANY of
these are true:

1. The task requires creating a file, endpoint, component, or DB table
   not in `Architecture.md`.
2. The task requires modifying a file outside the ticket's **Boundary**.
3. The test contract is ambiguous or missing a case encountered.
4. Two parts of the spec contradict each other.
5. A dependency or tool is needed that is not in the tech stack.
6. The code requires an architectural decision not already documented.

**On stop:** Commit progress so far with a descriptive message explaining
what was completed and what failed. The user can then start a Planner
session to resolve the gap, or `git reset` and retry.

## Generator Prohibitions

During a Generator Session, the agent must NEVER:
- Propose architectural solutions.
- Work around a spec gap.
- Add TODO placeholders as a substitute for stopping.
- Create files outside the Boundary.
- Modify Concept.md, Architecture.md, Plan.md/Triage.md, or skills.