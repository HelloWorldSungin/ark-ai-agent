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

### Step 3: Ensure Labels Exist

1. Collect all unique agent names from ROADMAP.md phase assignments
2. Determine the **plan-name label**: use the project name from `.planning/BRIEF.md` (the same name used for the Linear project in Step 2)
3. List existing labels via `mcp__plugin_linear_linear__list_issue_labels`
4. Ensure the following labels exist (create any that are missing via `mcp__plugin_linear_linear__create_issue_label`):

| Label | Color | Purpose |
|-------|-------|---------|
| `plan` | red | Identifies plan-created project issues (reused across plans) |
| `project-plan` | red | Identifies the top-level project issue (reused across plans) |
| `phase` | indigo | Identifies phase issues (reused across plans) |
| `<plan-name>` | cyan | Unique label matching project name — enables filtering all related issues |
| Agent labels | varies (see below) | One per unique agent name from ROADMAP.md |

   - Agent label colors:
     - `general-purpose` → blue
     - `feature-dev:code-architect` → purple
     - `feature-dev:code-explorer` → green
     - `feature-dev:code-reviewer` → orange
     - `Explore` → teal
     - `Plan` → yellow
     - Other → gray
5. Store all label IDs mapped by name

### Step 4: Create Project Issue and Phase Issues

**First, create (or update) the project issue:**

If a new project was created in Step 2, find or create an issue to represent the plan itself:
1. Create a Linear issue via `mcp__plugin_linear_linear__create_issue`:
   - **title**: Project name from BRIEF.md
   - **team**: Resolved team from Step 0
   - **state**: "Backlog"
   - **project**: Project ID from Step 2
   - **labels**: `["<plan-label-id>", "<project-plan-label-id>", "<plan-name-label-id>"]` — the `plan` label + `project-plan` label + the unique plan-name label from Step 3
   - **description**: Problem statement and success criteria from BRIEF.md
2. Store the project issue ID

**Then, create phase issues:**

For each phase in ROADMAP.md:

1. Create a Linear issue via `mcp__plugin_linear_linear__create_issue`:
   - **title**: Phase title from roadmap (e.g., "Phase 01: Foundation")
   - **team**: Resolved team from Step 0
   - **state**: "Backlog" (first phase gets "Todo")
   - **project**: Project ID from Step 2
   - **labels**: `["<phase-label-id>", "<plan-name-label-id>", "<agent-label-id>"]` — the `phase` label + plan-name label + agent label from Step 3
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
- **labels**: `["<plan-name-label-id>"]` — the plan-name label only (no `phase` label, no agent label)
- Same project and team

### Step 6: Summary

Display created issues:

```
Linear project: [name] (ID: [id])
Plan-name label: [plan-name]
Labels created: [list of new labels]

Created [N] Linear issues from ROADMAP.md:

- ARK-49 — [project name] (Backlog) [labels: plan, plan-name]
- ARK-50 — Phase 01: Foundation (Todo) [labels: phase, plan-name, code-architect]
- ARK-51 — Phase 02: Core Logic (Backlog) [labels: phase, plan-name, general-purpose]
- ARK-52 — Phase 03: Integration (Backlog) [labels: phase, plan-name, code-reviewer]

[+ N child tasks if created (labeled with plan-name only)]
```
