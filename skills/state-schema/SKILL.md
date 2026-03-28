# Plan.md Task Ticket Format

Each execution step in Plan.md MUST follow this format. The Generator
executes these tickets mechanically. The user evaluates the results.

## Template

```markdown
### Phase [N], Step [M]: [Descriptive Title]

**Input:** [Files/schemas/APIs the Generator reads from]
**Output:** [Exact files to create or modify]
**Spec:**
- [Concrete behavioral specification]
- [Function signatures with types]
- [Explicit error handling: what to catch, what to raise]
- [Edge cases to handle]

**Test Contract:**
- [test_name]: [input] → [expected output/behavior]
- [test_name]: [input] → [expected error type]
- [test_name]: [edge case] → [expected behavior]

**Manual Verification:**
- [What the user should visually inspect or click through]
- [Expected behavior from the user's perspective]
- [UX or integration checks that automated tests cannot cover]

**Skills to Load:** @skills/[relevant-skill]/SKILL.md
**Boundary:** [Exact directories/files the Generator may touch]
**Run Command:** [Exact command to run tests for this step]
```

## Field Definitions

**Input:** What exists before the Generator starts. References to
Architecture.md sections, existing source files, or schema definitions.

**Output:** Exact files to create or modify. Be specific.

**Spec:** Behavioral requirements precise enough that the Generator
doesn't need judgment calls. Include typed function signatures, explicit
error types, and edge cases.

**Test Contract:** Named test cases with input → output expectations.
Generator writes these tests first (TDD), then implements.
Minimum: 1 success + 1 validation error + 1 edge case.

**Manual Verification:** What the user (as Evaluator) should check
after tests pass. This captures UX, visual, and integration concerns
that automated tests cannot. The Generator presents these criteria
to the user alongside test results before requesting commit approval.

**Skills to Load:** `@skills/` file references for patterns and rules.

**Boundary:** Every file the Generator may touch. Anything outside
triggers a mandatory halt.

**Run Command:** Exact shell command. Copy-pasteable.

## Planner Self-Check

Before leaving Spec Phase, verify for each ticket:
- Could the Generator execute without asking clarifying questions?
- Is every function signature typed?
- Is every error case explicit?
- Does the Boundary list every file that will be touched?
- Is the Test Contract specific enough to write tests from?
- Does Manual Verification tell the user exactly what to check?

Every gap becomes a potential Generator hallucination.

## Halt Flag Placement

Place `[Halt here]` when:
- The next step shifts domains (backend → frontend).
- 3-5 steps have accumulated since the last halt.
- The next step depends on reviewing what was just built.
- Context degradation is likely.

The final phase is always the "Global Test Phase" with `[Halt here]`
before it. The user runs the final test suite manually.