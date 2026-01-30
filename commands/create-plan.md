---
description: Create hierarchical project plans for solo agentic development (briefs, roadmaps, phase plans)
argument-hint: [what to plan | plan-file-path]
allowed-tools:
  - Skill(create-plans)
  - Read
  - Bash
  - Write
---

Invoke the create-plans skill for: $ARGUMENTS

**Import mode:** If the argument is a file path (ends in `.md`, contains `/`, or starts with `@`), this imports an existing plan file into the `.planning/` hierarchy and syncs to Linear. The plan file can come from Claude Code planning mode or any structured plan document.
