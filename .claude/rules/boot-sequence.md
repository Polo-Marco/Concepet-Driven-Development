# Boot Sequence (Re-hydration Protocol)

When the user types `continue plan` or `continue debug`, execute these steps
in this exact order. Do not skip steps. Do not start coding before completing
all steps.

## The Sequence

### Step 1: Read STATE.md
Identify: active mode, current phase, tech stack, model tier history,
and whether a `## Blocked` section exists.

### Step 2: Check for Blocks
If a `## Blocked` section is present and unresolved:
- **If you are the Good Model (Tier 1):** Resolve the block — update
  Architecture.md and/or Plan.md as needed. Clear the Blocked section.
  Append resolution to Key Decisions Log. Commit with `revise: resolve block`.
- **If you are the Ok Model (Tier 2):** Tell the user unresolved blocks
  require a Good Model session. STOP. Do not proceed.

### Step 3: Read Architecture.md
Rebuild the structural mental model — data flow, components, endpoints, models.

### Step 4: Read Plan.md (or Triage.md for debug mode)
Find the next incomplete step (the first unchecked `[ ]` item).

### Step 5: Read Active Skills
Load the skill files listed in STATE.md's `## Active Skills` section using
`@skills/<skill-name>/SKILL.md`. Load ONLY the skills listed — do not load
all skills.

### Step 6: Read CHANGELOG.md (last 3 entries)
Understand recent momentum and what was built in the prior session.

### Step 7: Announce the Re-hydration Summary
Output a brief summary to the user:

> **Re-hydration complete.**
> - **Last completed:** [phase/step name]
> - **Next up:** [phase/step name]
> - **Risks/Notes:** [any blocks resolved, or relevant context]
>
> Reply 'confirmed' to proceed, or correct me if this is wrong.

**Wait for the user's confirmation before executing.** This is a verification
handshake, not a formality. If the user corrects you, adjust your understanding
before proceeding.

## When to Use Which Command

| User says | Read from |
|---|---|
| `continue plan` | STATE.md → Plan.md |
| `continue debug` | STATE.md → Triage.md |