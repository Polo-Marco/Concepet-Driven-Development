---
name: skill-template
description: Meta-skill defining the standard for bespoke execution skills. The Good Model reads this during Spec Phase to write skills the Ok Model can follow mechanically.
author: framework
version: 5.2-cc
---

# Skill Template Standard (v5.2 — Claude Code)

## Purpose

This defines the mandatory structure for every skill the Good Model generates.
Skills are the primary interface between the Good Model's architectural intent
and the Ok Model's mechanical execution.

**Design Principle:** Write every skill as if the reader is a competent but
literal-minded junior engineer who will follow instructions exactly and
improvise dangerously when instructions are missing.

---

## 1. Mandatory YAML Frontmatter

```yaml
---
name: [kebab-case-name]
description: [one-line purpose]
author: good-model
project: [project name]
stack: [comma-separated tools]
version: [semver or date]
---
```

## 2. Mandatory Sections (All Six, In Order)

### Section 1: File Structure (Canonical)

The exact directory tree for this domain. The Ok Model must not create files
outside this tree. Use `←` arrows for inline annotations.

### Section 2: Patterns (Copy-Paste Templates)

Concrete, working code templates using the project's **actual types and names**.
Not pseudocode. Not generic placeholders. Real code.

Requirements:
- Minimum 3 patterns: create, read/query, and test.
- Include full import blocks at the top of every pattern.
- Show complete functions, not truncated snippets.

### Section 3: Hard Rules (DO / DO NOT)

Binary rules. No nuance. Minimum 5 DO and 5 DO NOT.
Every rule must be specific and testable.

Example of good rule: "Never use `from module import *`"
Example of bad rule: "Write clean code"

### Section 4: Error Handling Contract

Exactly how errors should be raised, caught, and surfaced.
Define patterns for each layer (routes, services, etc.).
Specify what NOT to catch.

### Section 5: Testing Contract

Framework, naming conventions, minimum coverage per component,
fixture patterns with real code, and exact run commands.

### Section 6: Environment & Commands

Every command the Ok Model might need, ready to copy-paste.
Setup, dev server, tests, linting, migrations — all of them.

---

## 3. Quality Checklist (Good Model Self-Review)

Before finalizing a skill, verify:

- [ ] No ambiguity: could a literal-minded model read any rule two ways?
- [ ] No missing imports: every code pattern has its full import block?
- [ ] Real names: patterns use the project's actual types, not `Item` or `Thing`?
- [ ] Boundary-safe: File Structure makes it unambiguous where every file goes?
- [ ] Error handling explicit: Ok Model can determine what to raise/catch without judgment?
- [ ] Commands copy-pasteable: every command in Section 6 runs verbatim?
- [ ] DO NOT list exhaustive: every common mistake a weaker model might make is listed?

## 4. Versioning

When the Good Model revises a skill:
1. Increment `version` in frontmatter.
2. Add entry under `## Revision Log` at the bottom.
3. Verify all Patterns still match updated Architecture.md.

The Ok Model must NEVER modify skill files. If a skill seems wrong, halt and
report as Blocked in STATE.md.