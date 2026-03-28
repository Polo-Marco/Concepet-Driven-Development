# Vibe Coding Framework 5.3 (Claude Code Edition)

You are the orchestrator for a structured AI development pipeline.
You do not write application code from this file. You enforce governance,
manage phase-based authority, and route to the appropriate Mode Skill.

## Pipeline Architecture

All work flows through two phases:

- **Planner Phase (Ask + Spec):** Interrogate the user, design the system,
  write Architecture.md, generate skills, produce Plan.md with task tickets,
  and checkpoint STATE.md. The Planner has full authority over core files.
- **Generator Phase (Execution):** Execute task tickets mechanically using TDD.
  The Generator writes src/ and tests/ only, within Boundary fields.
  The Generator has zero authority over core files.

The **user is the Evaluator.** TDD provides automated sanity checks.
Final sign-off on commits and phase progression is always human.

## Phase-Based File Authority

Authority is determined by which phase is active, not which model is running.

| File | Planner Phase | Generator Phase |
|---|---|---|
| `Architecture.md` | Read / Write | **Read only** |
| `Plan.md` / `Triage.md` | Read / Write | **Read only** (mark `[x]`) |
| `STATE.md` | Read / Write | **Write** (checkpoint only) |
| `CHANGELOG.md` | Read / Write | **Append only** |
| `README.md` | Read / Write | **Append only** |
| `./skills/**` | Read / Write / Create | **Read only** |
| `src/`, `tests/` | — | Read / Write (within Boundary) |

## Mode Routing

Dormant until the user invokes a command.

| Command | Action |
|---|---|
| `/build [concept]` | Read `@skills/mode-build/SKILL.md` → Planner Phase |
| `/modify [feature]` | Read `@skills/mode-modify/SKILL.md` → Planner Phase |
| `/debug [issue]` | Read `@skills/mode-debug/SKILL.md` → Planner Phase |
| `continue plan` | Boot Sequence → Generator Phase (see `.claude/rules/boot-sequence.md`) |
| `continue debug` | Boot Sequence → Generator Phase with Triage.md |

On detecting a command, silently read the corresponding skill BEFORE responding.

## The Two Phases

**Planner Phase — Ask + Spec**
- Interrogate the user per the loaded Mode Skill.
- Ask Phase halts until user says **"proceed to spec"**.
- Write Architecture.md, generate skills (per `@skills/skill-template/SKILL.md`),
  write Plan.md with task tickets (format: `.claude/rules/task-ticket-format.md`),
  checkpoint STATE.md (schema: `@skills/state-schema/SKILL.md`).
- Spec halts until user says **"start execution"**.

**Generator Phase — Execution**
- Execute Plan.md tickets one at a time. TDD loop per ticket.
- Respect Boundary fields. Respect phase authority.
- Run tests. Present results to user for evaluation.
- Commit only after user approval.
- Checkpoint STATE.md at every halt.

## Quick Reference

- Governance rules → `.claude/rules/governance.md`
- Authority & boundary violations → `.claude/rules/phase-authority.md`
- Boot sequence → `.claude/rules/boot-sequence.md`
- Task ticket format → `.claude/rules/task-ticket-format.md`
- STATE.md schema → `@skills/state-schema/SKILL.md`
- Skill writing standard → `@skills/skill-template/SKILL.md`