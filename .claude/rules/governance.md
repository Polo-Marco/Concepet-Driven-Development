# Global Governance & Safety (Inviolable Rules)

These rules apply at ALL times, regardless of mode or model tier.

## 1. Strict Version Control (Git)

- Execute a git commit with a semantic message (`feat:`, `fix:`, `chore:`, `refactor:`, `revise:`) at the end of every completed phase.
- Forbidden from modifying multiple unrelated domains in a single commit.

## 2. Security & Secrets Policy

- **NO HARDCODING:** Forbidden from hardcoding API keys, database URIs, IP addresses, or credentials in any file.
- All configuration routes through `.env` (which must be in `.gitignore`) loaded via environment variables.
- Never leak raw sensitive data into persistent public logs or UI traces.

## 3. Context Window Protection (The Halt Protocol)

- You suffer from context degradation in long sessions. You MUST respect `[Halt here]` flags.
- When you encounter `[Halt here]` at the end of a plan/triage phase:
  1. Commit all work.
  2. Write a checkpoint to STATE.md.
  3. Tell the user: "Phase complete. To protect context, please start a new session and type `continue plan`."

## 4. Test-Driven Philosophy

- Write tests *before* application logic.
- Specific execution (Full-Stack TDD vs. Throwaway Sandboxes) is delegated to the active Mode Skill.

## 5. The Trackers

- Update `CHANGELOG.md` and `README.md` at the end of every major phase.
- These files are the primary state inheritance mechanism between sessions.