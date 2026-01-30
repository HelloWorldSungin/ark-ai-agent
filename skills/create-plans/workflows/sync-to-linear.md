# Sync Plan to Linear

Creates Linear issues from ROADMAP.md phases with project creation and agent label support.

## Required Tools

- mcp__plugin_linear_linear__create_issue
- mcp__plugin_linear_linear__list_projects
- mcp__plugin_linear_linear__list_teams
- mcp__plugin_linear_linear__list_issue_labels
- mcp__plugin_linear_linear__create_issue_label
- mcp__plugin_linear_linear__create_project
- mcp__plugin_linear_linear__update_project
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

Parse the phase structure and agent assignments. Each phase has:

```markdown
### Phase 1: [Name]
**Goal**: [description]
**Agent**: [agent-name]
```

Also extract agent assignments from the Phases summary line format: `agent:[name]`

### Step 2: Smart Project Detection

1. Search existing projects by name via `mcp__plugin_linear_linear__list_projects`
2. Extract project name from `.planning/BRIEF.md` or ROADMAP.md title

**If matching project found:**
- Ask user: "Found existing project '[name]'. Use this, or create new?"
- If use existing: store project ID, continue to Step 3

**If no matching project found:**
- Read `.planning/BRIEF.md` for problem statement and success criteria
- Create project via `mcp__plugin_linear_linear__create_project`:

| Field | Source |
|-------|--------|
| `name` | BRIEF.md project name |
| `teamIds` | Resolved team ID |
| `description` | Problem statement + success criteria from BRIEF.md |
| `state` | "planned" |

- Store created project ID

### Step 3: Ensure Agent Labels Exist

1. Collect all unique agent names from ROADMAP.md phase assignments
2. List existing labels via `mcp__plugin_linear_linear__list_issue_labels`
3. For each unique agent name not already a label:
   - Create via `mcp__plugin_linear_linear__create_issue_label`:
     - `name`: agent name (e.g., "feature-dev:code-architect")
     - `teamId`: resolved team ID
   - Assign distinct colors per agent type:
     - `general-purpose` → blue
     - `feature-dev:code-architect` → purple
     - `feature-dev:code-explorer` → green
     - `feature-dev:code-reviewer` → orange
     - `Explore` → teal
     - `Plan` → yellow
     - Other → gray
4. Store label IDs mapped to agent names

### Step 4: Create Issues for Each Phase

For each phase in ROADMAP.md:

1. Create a Linear issue via `mcp__plugin_linear_linear__create_issue`:
   - **title**: Phase title from roadmap (e.g., "Phase 01: Foundation")
   - **team**: Resolved team from Step 0
   - **state**: "Backlog" (first phase gets "Todo")
   - **project**: Project ID from Step 2
   - **labels**: `["<agent-label-id>"]` — the agent label for this phase from Step 3
   - **description**: Markdown formatted:
     ```
     Phase from project roadmap.

     **Scope:** [phase description from ROADMAP.md]

     **Agent:** [agent-name]

     **Planning files:** `.planning/phases/XX-name/`

     ---
     *Synced from ROADMAP.md via /create-plan*
     ```

2. Record the issue identifier for each phase

### Step 5: (Optional) Create Sub-Issues from PLAN.md

If PLAN.md files exist for phases, offer to create child issues:

"Found detailed plans for [N] phases. Create child issues for individual tasks? (y/n)"

If yes, for each task in PLAN.md:
- Create issue with `parentId` set to the phase issue
- **title**: Task description
- **state**: "Backlog"
- Same project, team, and agent label

### Step 6: Summary

Display created issues:

```
Linear project: [name] (ID: [id])
Agent labels created: [list of new labels]

Created [N] Linear issues from ROADMAP.md:

- ARK-50 — Phase 01: Foundation (Todo) [agent: code-architect]
- ARK-51 — Phase 02: Core Logic (Backlog) [agent: general-purpose]
- ARK-52 — Phase 03: Integration (Backlog) [agent: code-reviewer]

[+ N child tasks if created]
```
