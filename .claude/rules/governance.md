# Global Governance & Safety

These rules apply at ALL times, regardless of mode or session type.

## 1. Version Control (Git)

### Commit Conventions
- Semantic prefixes: `plan:`, `feat:`, `fix:`, `refactor:`, `migrate:`.
- **Planner Session:** one commit at session end covering all core files.
- **Generator Session:** one commit per completed ticket.
- Commit messages must be detailed and informative — they serve as the
  project's progress log. Include what was built, which files changed,
  and any notable decisions.

### Commit Message Format
```
feat: implement PDF extraction pipeline

- Created src/pipeline/extractor.py with extract_text()
- Handles PDF, DOCX, plain text formats
- Raises ExtractionError on corrupt/unsupported files
- Added tests: test_extractor.py (3 passing)
```

One domain per commit. No unrelated changes bundled.

## 2. Security & Secrets

- **NO HARDCODING.** API keys, URIs, credentials → `.env` (in `.gitignore`).
- Never leak sensitive data into logs or UI traces.

## 3. Test-Driven Development

- Write tests before application logic.
- TDD is the automated sanity check. It catches regressions and proves
  basic functionality.
- The user is the final evaluator. TDD does not replace human judgment.

## 4. Core File Lifecycle

| File | Lifecycle | Purpose |
|---|---|---|
| `Concept.md` | Persistent, evolving | Vision, scope, principles |
| `Architecture.md` | Persistent, evolving | System design source of truth |
| `CHANGELOG.md` | Persistent, append-only | What changed, when |
| `Plan.md` | **Ephemeral** — deleted after full loop | Task tickets |
| `Triage.md` | **Ephemeral** — deleted after full loop | Bug hypotheses |
| `skills/` | Persistent, evolving | Execution patterns and rules |

Only one of Plan.md or Triage.md exists at a time. When the full loop
(Planner + Generator + user evaluation) completes successfully, the
user deletes the work order file. It served its purpose.