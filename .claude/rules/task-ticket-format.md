# Plan.md Task Ticket Format

When the Good Model writes Plan.md during Spec Phase, each execution step
MUST follow this format. The Ok Model executes these tickets mechanically.

## Template

```markdown
### Phase [N], Step [M]: [Descriptive Title]

**Input:** [Files/schemas/APIs the Ok Model reads from]
**Output:** [Exact files to create or modify]
**Spec:**
- [Concrete behavioral specification]
- [Function signatures with parameter and return types]
- [Explicit error handling: what to catch, what to raise]
- [Edge cases to handle]

**Test Contract:**
- [test_name]: [input] → [expected output/behavior]
- [test_name]: [input] → [expected error type]
- [test_name]: [edge case] → [expected behavior]

**Skills to Load:** @skills/[relevant-skill]/SKILL.md
**Boundary:** [Exact directories/files the Ok Model may touch. Nothing else.]
**Run Command:** [Exact command to run tests for this step]
```

## Field Definitions

**Input:** What exists before the Ok Model starts. Can reference Architecture.md
sections, existing source files, or schema definitions. The Ok Model reads these
but must not modify them unless they appear in the Boundary.

**Output:** The exact files the Ok Model will create or modify. Be specific —
`src/services/document_service.py` not "the service layer."

**Spec:** Behavioral requirements precise enough that the Ok Model doesn't need
to make judgment calls. Include function signatures with types. State what errors
to raise and what to catch. List edge cases explicitly.

**Test Contract:** Named test cases with clear input → output expectations.
The Ok Model writes these tests first (TDD), then implements the logic.
Minimum per endpoint/function: 1 success + 1 validation error + 1 edge case.

**Skills to Load:** The `@skills/` file reference(s) the Ok Model reads for
patterns, hard rules, and conventions. Use Claude Code's `@` file reference.

**Boundary:** The most important safety field. Lists every directory and file
the Ok Model is allowed to touch. If a file isn't listed here, the Ok Model
must halt and report Blocked rather than create or modify it.

**Run Command:** The exact shell command to run tests. Must be copy-pasteable.

## Good Model Self-Check

Before leaving Spec Phase, verify for each ticket:
- Could the Ok Model execute this without asking a single clarifying question?
- Is every function signature typed?
- Is every error case explicit (not "handle errors appropriately")?
- Does the Boundary list every file that will be touched?
- Is the Test Contract specific enough to write tests from?

If the answer to any of these is no, the ticket is underspecified.
Every gap becomes a potential Ok Model hallucination.

## Halt Flag Placement Guidelines

Place `[Halt here]` at the end of a step when:
- The next step shifts domains (Backend → Frontend, or vice versa).
- 3-5 steps have accumulated since the last halt.
- The next step depends on reviewing what was just built.
- Cumulative token usage risks context degradation.

The **final phase** must always be "Global Test Phase" with a `[Halt here]`
immediately before it. The user runs the final test suite manually.