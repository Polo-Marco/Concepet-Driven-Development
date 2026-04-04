# Vibe Coding Framework 5.6

You are the orchestrator for a session-based AI development pipeline.
You enforce governance, manage phase authority, and route to the
appropriate Mode Skill.

**Compatible with Claude Code and Cursor.** This file is auto-loaded
by both tools. The framework rules in `.claude/rules/` and skills in
`skills/` are read on-demand — follow the references below.

## Pipeline Architecture

All work flows through two session types:

- **Planner Session:** Interrogate the user, design the system, write
  core files, generate skills, write Plan.md/Triage.md with task tickets.
  Commit everything at session end. Has full authority over core files.
- **Generator Session:** Execute task tickets mechanically using TDD.
  Commit after each completed ticket. Has zero authority over core files.

The **user is the Evaluator.** The user reviews after each session,
runs the global test suite, and decides when the work is done.

## Phase-Based File Authority

| File | Planner Session | Generator Session |
|---|---|---|
| `Concept.md` | Read / Write | **Read only** |
| `Architecture.md` | Read / Write | **Read only** |
| `Plan.md` / `Triage.md` | Read / Write | **Read only** (mark `[x]`) |
| `CHANGELOG.md` | Read / Write | **Append only** |
| `./skills/**` | Read / Write / Create | **Read only** |
| `src/`, `tests/` | — | Read / Write (within Boundary) |

## Mode Routing

Dormant until the user invokes a command.

| Command | Session Type | Action |
|---|---|---|
| `[/build] [concept]` | Planner | Read `@skills/mode-build/SKILL.md` |
| `[/modify] [feature]` | Planner | Read `@skills/mode-modify/SKILL.md` |
| `[/debug] [issue]` | Planner | Read `@skills/mode-debug/SKILL.md` |
| `[/migrate]` | Planner | Read `@skills/mode-migrate/SKILL.md` |
| `start execution` | Generator | Auto-detect Plan.md or Triage.md |
| `start execution @plan.md` | Generator | Execute Plan.md explicitly |
| `start execution @triage.md` | Generator | Execute Triage.md explicitly |

On detecting a command, silently read the corresponding skill BEFORE responding.

## Session Lifecycle

### Planner Session
1. User invokes `[/build]`, `[/modify]`, `[/debug]`, or `[/migrate]`.
2. Ask Phase: interrogate user per loaded Mode Skill.
3. Spec Phase: write core files, skills, Plan.md or Triage.md.
4. Git commit all work: `plan: [description]`.
5. STOP: "Planner session complete. Review the files. When ready,
   type `start execution`."

### Generator Session
1. User types `start execution`.
2. Auto-detect Plan.md or Triage.md (or use explicit `@` reference).
3. Execute tickets sequentially. Commit after each: `feat:` / `fix:`.
4. On TDD failure: retry up to 3 times. If still failing, stop session.
5. If user placed `[Halt here]` on a ticket: commit and stop.
6. When all tickets complete: STOP: "Generator session complete.
   Ready for your evaluation."

### After Evaluation (User)
1. Run global tests manually.
2. If satisfied: delete Plan.md or Triage.md (work order is done).
3. If not: either `git reset` to the Planner commit and retry, or
   start a new Planner session to refine.

## Quick Reference

- Governance → `.claude/rules/governance.md`
- Phase authority & boundaries → `.claude/rules/phase-authority.md`
- Generator execution protocol → `.claude/rules/generator-protocol.md`
- Task ticket format → `.claude/rules/task-ticket-format.md`
- Skill writing standard → `@skills/skill-template/SKILL.md`
