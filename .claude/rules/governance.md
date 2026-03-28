# Global Governance & Safety (Inviolable Rules)

These rules apply at ALL times, regardless of mode or phase.

## 1. Strict Version Control (Git)

- Semantic commit messages: `feat:`, `fix:`, `chore:`, `refactor:`, `revise:`.
- One domain per commit. No unrelated changes bundled.
- **Commits require user approval.** Present your changes and test results,
  then wait for the user to confirm before committing.

## 2. Security & Secrets Policy

- **NO HARDCODING.** API keys, URIs, credentials → `.env` (in `.gitignore`).
- Never leak sensitive data into logs or UI traces.

## 3. Context Window Protection (The Halt Protocol)

- Respect `[Halt here]` flags unconditionally.
- On encountering `[Halt here]`:
  1. Present work and test results to user for evaluation.
  2. After user approval, commit.
  3. Write checkpoint to STATE.md.
  4. Tell user: "Phase complete. Start a new session and type `continue plan`."

## 4. Test-Driven Development (Automated Sanity Check)

- Write tests before application logic.
- TDD is the automated verification layer. It catches regressions and
  proves basic functionality.
- The user is the final evaluator. TDD does not replace human judgment —
  it supports it.

## 5. The Trackers

- Update `CHANGELOG.md` and `README.md` at the end of every major phase.
- These are the primary inheritance mechanism between sessions.