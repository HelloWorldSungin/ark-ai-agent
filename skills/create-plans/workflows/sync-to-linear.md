# Sync Plan to Linear

Creates Linear issues from ROADMAP.md phases and optionally from PLAN.md tasks.

## Required Tools

- mcp__plugin_linear_linear__create_issue
- mcp__plugin_linear_linear__list_projects
- mcp__plugin_linear_linear__list_teams
- Read

## Workflow

### Step 0: Resolve Linear Team

- Check if the project's CLAUDE.md specifies a `linear-team` value
- If not, use `mcp__plugin_linear_linear__list_teams` to list available teams
- If only one team exists, use it automatically
- If multiple teams, ask user which team to use

### Step 1: Read ROADMAP.md

```bash
cat .planning/ROADMAP.md
```

Parse the phase structure. Each phase typically looks like:

```markdown
## Phase 01: Foundation
Description of what this phase covers...

## Phase 02: Core Logic
...
```

### Step 2: Determine Linear Project

- Use `mcp__plugin_linear_linear__list_projects` with the resolved team to find active projects
- If only one project exists, use it automatically
- If multiple projects, ask user which project to use
- If no projects exist, create issues without a project

### Step 3: Create Issues for Each Phase

For each phase in ROADMAP.md:

1. Create a Linear issue via `mcp__plugin_linear_linear__create_issue`:
   - **title**: Phase title from roadmap (e.g., "Phase 01: Foundation")
   - **team**: Resolved team from Step 0
   - **state**: "Backlog" (first phase gets "Todo")
   - **project**: Determined in Step 2
   - **description**: Markdown formatted:
     ```
     Phase from project roadmap.

     **Scope:** [phase description from ROADMAP.md]

     **Planning files:** `.planning/phases/XX-name/`

     ---
     *Synced from ROADMAP.md via /create-plan*
     ```

2. Record the issue identifier for each phase

### Step 4: (Optional) Create Sub-Issues from PLAN.md

If PLAN.md files exist for phases, offer to create child issues:

"Found detailed plans for [N] phases. Create child issues for individual tasks? (y/n)"

If yes, for each task in PLAN.md:
- Create issue with `parentId` set to the phase issue
- **title**: Task description
- **state**: "Backlog"
- Same project and team

### Step 5: Summary

Display created issues:

```
Created [N] Linear issues from ROADMAP.md:

- ARK-50 — Phase 01: Foundation (Todo)
- ARK-51 — Phase 02: Core Logic (Backlog)
- ARK-52 — Phase 03: Integration (Backlog)

[+ N child tasks if created]
```
