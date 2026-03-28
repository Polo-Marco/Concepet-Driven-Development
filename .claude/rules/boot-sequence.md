# Boot Sequence (Re-hydration Protocol)

When the user types `continue plan` or `continue debug`, execute these
steps in exact order. Do not skip steps. Do not start coding before
completing all steps.

## The Sequence

### Step 1: Read STATE.md
Identify: active mode, current phase, project summary, tech stack,
and whether a `## Blocked` section exists.

### Step 2: Check for Blocks
If `## Blocked` exists and is unresolved:
- If you are in **Planner Phase** (user invoked a planning command or
  is resolving a block): resolve it — update Architecture.md and/or
  Plan.md, clear the Blocked section, append to Key Decisions Log.
  Commit with `revise: resolve block`.
- If you are in **Generator Phase** (user typed `continue plan`):
  inform the user that unresolved blocks require a Planner session. STOP.

### Step 3: Read the Project Summary (in STATE.md)
The `## Project Summary` section gives you compressed context about the
system — what it does, major components, current scope. This is your
fast path. Only read the full `Architecture.md` if the current task
ticket involves structural changes or cross-cutting concerns.

### Step 4: Read Plan.md (or Triage.md for debug)
Find the next incomplete step (first unchecked `[ ]` item).

### Step 5: Read Active Skills
Load skill files listed in STATE.md's `## Active Skills` using
`@skills/<name>/SKILL.md`. Load ONLY listed skills.

### Step 6: Read CHANGELOG.md (last 3 entries)
Understand recent momentum.

### Step 7: Announce Re-hydration Summary

> **Re-hydration complete.**
> - **Last completed:** [phase/step name]
> - **Next up:** [phase/step name]
> - **Risks/Notes:** [blocks resolved, relevant context]
>
> Reply 'confirmed' to proceed, or correct me if this is wrong.

**Wait for confirmation.** This is a verification handshake.

## Command Routing

| User says | Phase | Reads from |
|---|---|---|
| `continue plan` | Generator | STATE.md → Plan.md |
| `continue debug` | Generator | STATE.md → Triage.md |
| `/build`, `/modify`, `/debug` | Planner | STATE.md (if exists) → Mode Skill |