# Import Plan File

Imports an existing plan file (from Claude Code planning mode or any structured plan) into the `.planning/` hierarchy with agent assignments, then syncs to Linear.

## Required Tools

- Read
- Write
- Bash

## Workflow

### Step 1: Read and Parse Plan File

Read the provided file path. Extract the following (flexible parsing — accept any reasonable format):

| Field | Look For |
|-------|----------|
| **Project name** | Title, `# Plan:`, first heading |
| **Problem statement** | "Problem", "Overview", "Summary", intro paragraph |
| **Phases** | Numbered sections, `## Phase N`, `### N.`, table rows with phase info |
| **Tasks per phase** | Bullet lists, sub-sections, table rows under each phase |
| **Agent assignments** | See agent parsing rules below |
| **Constraints** | "Constraints", "Rules", "Important" sections |
| **Success criteria** | "Verification", "Success criteria", "Done when" sections |

### Step 2: Parse Agent Assignments (Flexible Format)

Accept ANY of these formats for agent assignments:

**Table format:**
```markdown
| Phase | Agent |
|-------|-------|
| 1. Foundation | code-architect |
| 2. Implementation | general-purpose |
```

**Inline metadata:**
```markdown
## Phase 1: Foundation
Agent: code-architect
```

**Parenthetical:**
```markdown
## Phase 1: Foundation (agent: code-architect)
```

**YAML-like:**
```markdown
## Phase 1: Foundation
- agent: code-architect
- description: ...
```

**If no agent assignments found:** Default all phases to `general-purpose` and inform user.

Valid agent types (map to Task tool `subagent_type`):
- `general-purpose` — General implementation
- `code-architect` / `feature-dev:code-architect` — Architecture design
- `code-explorer` / `feature-dev:code-explorer` — Codebase analysis
- `code-reviewer` / `feature-dev:code-reviewer` — Code review
- `Explore` — Codebase exploration
- `Plan` — Planning agent
- Any custom agent name — passed through as-is

Normalize short names: `code-architect` → `feature-dev:code-architect`, `code-explorer` → `feature-dev:code-explorer`, `code-reviewer` → `feature-dev:code-reviewer`.

### Step 3: Generate `.planning/` Hierarchy

1. Create `.planning/` directory structure:

```bash
mkdir -p .planning/phases
```

2. **Generate `.planning/BRIEF.md`:**

```markdown
# Brief: [Project Name]

## Problem Statement

[Extracted from plan file]

## Constraints

[Extracted constraints, or "None specified"]

## Success Criteria

[Extracted success criteria]

---
*Imported from [original-file-path]*
```

3. **Generate `.planning/ROADMAP.md`** with agent metadata:

```markdown
# Roadmap: [Project Name]

## Overview

[One paragraph summary from plan file]

## Phases

- [ ] **Phase 1: [Name]** - [Description] `agent:[agent-name]`
- [ ] **Phase 2: [Name]** - [Description] `agent:[agent-name]`
...

## Phase Details

### Phase 1: [Name]
**Goal**: [Phase goal]
**Depends on**: Nothing (first phase)
**Agent**: [agent-name]
**Plans**: [N plans]

Plans:
- [ ] 01-01: [Task description]
- [ ] 01-02: [Task description]

### Phase 2: [Name]
**Goal**: [Phase goal]
**Depends on**: Phase 1
**Agent**: [agent-name]
**Plans**: [N plans]

Plans:
- [ ] 02-01: [Task description]

## Progress

| Phase | Agent | Plans Complete | Status | Completed |
|-------|-------|----------------|--------|-----------|
| 1. [Name] | [agent] | 0/N | Not started | - |
| 2. [Name] | [agent] | 0/N | Not started | - |
```

4. **Generate phase directories and PLAN.md files:**

For each phase, create:
- `.planning/phases/XX-name/` directory
- `.planning/phases/XX-name/XX-01-PLAN.md` with tasks from that phase

Each PLAN.md follows the standard plan format with `<objective>`, `<tasks>`, `<verification>`, `<success_criteria>` sections.

### Step 4: Confirm and Sync to Linear

Present summary to user:

```
Imported plan: [Project Name]
- [N] phases extracted
- Agent assignments: [list of phase→agent mappings]
- Generated: BRIEF.md, ROADMAP.md, [N] phase PLAN.md files

Sync to Linear? (Y/n)
```

If yes: proceed to `workflows/sync-to-linear.md` (which now handles project creation + agent labels).

### Step 5: Done

```
Import complete:
- .planning/BRIEF.md
- .planning/ROADMAP.md
- [N] phase plans
- Linear project: [name] (if synced)
- [N] phase issues with agent labels (if synced)

Next: /run-plan (auto-dispatches next phase to assigned agent)
```
