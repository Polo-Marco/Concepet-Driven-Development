# Two-Tier Authority & Boundary Rules

## File Authority Matrix

| File | Good Model (Tier 1) | Ok Model (Tier 2) |
|---|---|---|
| `Concept.md` | Read / Write | **Read only** |
| `Architecture.md` | Read / Write | **Read only** |
| `Plan.md` | Read / Write | **Read only** (mark `[x]` only) |
| `STATE.md` | Read / Write | **Write** (checkpoint only) |
| `CHANGELOG.md` | Read / Write | **Append only** |
| `README.md` | Read / Write | **Append only** |
| `./skills/**` | Read / Write / Create | **Read only** |
| `src/`, `tests/` | — | Read / Write (within Boundary) |

**Cardinal Rule:** The Ok Model must never create, modify, or delete any file not explicitly listed in the current Plan.md phase's **Boundary** field.

## Execution Boundary Violation Rule (Ok Model)

During Phase 3, the Ok Model MUST immediately halt if ANY of these are true:

1. The task requires creating a file, endpoint, component, or DB table not in `Architecture.md`.
2. The task requires modifying a file outside the current phase's **Boundary** field.
3. The test contract in `Plan.md` is ambiguous, incomplete, or missing a case encountered during implementation.
4. Two parts of the spec (Architecture.md, Plan.md, or Skills) contradict each other.
5. A dependency or tool is needed that is not listed in the tech stack.
6. The code requires an architectural decision not already documented.

**On halt:** Write the `## Blocked` section in STATE.md, commit with `chore: halt — blocked on [reason]`, and tell the user to start a Good Model session.

### Blocked Section Format

```markdown
## Blocked
**Phase:** [current phase and step]
**Problem:** [factual description — no suggestions, no opinions]
**Evidence:** [specific reference to Architecture.md, Plan.md, or Skill that is insufficient]
**Files touched before halt:** [list]
```

## The Ok Model Must NEVER:

- Propose architectural solutions in the Blocked section.
- Attempt to "work around" a spec gap.
- Add placeholder implementations with TODO comments as a substitute for halting.
- Create files outside the Boundary, even if "obviously needed."
- Modify Concept.md, Architecture.md, Plan.md, or any skill file.