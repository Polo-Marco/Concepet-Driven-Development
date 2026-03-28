# Phase-Based Authority & Boundary Rules

## Core Principle

Authority is determined by the active phase, not the model running.
The same model can have full authority in Planner Phase and zero authority
in Generator Phase. The rules bind to the phase, not the identity.

## File Authority Matrix

| File | Planner Phase | Generator Phase |
|---|---|---|
| `Architecture.md` | Read / Write | **Read only** |
| `Plan.md` / `Triage.md` | Read / Write | **Read only** (mark `[x]`) |
| `STATE.md` | Read / Write | **Write** (checkpoint only) |
| `CHANGELOG.md` | Read / Write | **Append only** |
| `README.md` | Read / Write | **Append only** |
| `./skills/**` | Read / Write / Create | **Read only** |
| `src/`, `tests/` | — | Read / Write (within Boundary) |

## Generator Phase Boundary Rules

During Generator Phase, the agent MUST immediately halt if ANY of these
are true:

1. The task requires creating a file, endpoint, component, or DB table
   not described in `Architecture.md`.
2. The task requires modifying a file outside the current ticket's
   **Boundary** field.
3. The test contract is ambiguous, incomplete, or missing a case
   encountered during implementation.
4. Two parts of the spec contradict each other.
5. A dependency or tool is needed that is not in the tech stack.
6. The code requires an architectural decision not already documented.

**On halt:** Write `## Blocked` in STATE.md, present the problem to the
user, and wait for instructions.

### Blocked Section Format

```markdown
## Blocked
**Phase:** [current phase and step]
**Problem:** [factual description — no suggestions, no opinions]
**Evidence:** [specific reference to Architecture.md, Plan.md, or Skill]
**Files touched before halt:** [list]
```

## Generator Phase Prohibitions

During Generator Phase, the agent must NEVER:
- Propose architectural solutions in the Blocked section.
- Work around a spec gap.
- Add TODO placeholders as a substitute for halting.
- Create files outside the Boundary.
- Modify Architecture.md, Plan.md, or any skill file.