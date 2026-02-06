---
description: Execute a team-orchestrated plan — creates tasks, dispatches builders, runs validators
argument-hint: <path-to-plan>
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - Task
  - TaskCreate
  - TaskUpdate
  - TaskList
  - TaskGet
  - TaskOutput
  - AskUserQuestion
---

# Build

Execute a team-orchestrated plan by reading the plan, creating tasks, dispatching builders and validators, and reporting results.

## Variables

PATH_TO_PLAN: $ARGUMENTS

## Workflow

- If no `PATH_TO_PLAN` is provided, STOP immediately and ask the user to provide it (AskUserQuestion).

### Step 1: Read and Parse Plan

1. Read the plan at `PATH_TO_PLAN`
2. Extract these sections:
   - **Team Members** — list of builders/validators with names, roles, agent types
   - **Step by Step Tasks** — ordered tasks with IDs, dependencies, assignments, parallel flags
   - **Acceptance Criteria** — what constitutes completion
   - **Validation Commands** — commands to verify the work

### Step 2: Linear Integration (optional, non-blocking)

1. Resolve Linear team: check CLAUDE.md for `linear-team`, else `mcporter call linear.list_teams --output json` (auto-select if one team)
2. Search Linear for a matching issue based on the plan title
3. If found: update state to "In Progress" and store issue ID for Step 6
4. If not found: skip silently

### Step 3: Create Task List

For each task in **Step by Step Tasks**:

1. Call `TaskCreate` with:
   - `subject`: task name
   - `description`: full task details including specific actions from the plan
   - `activeForm`: present continuous form of the task name
2. After all tasks created, set dependencies with `TaskUpdate` + `addBlockedBy` based on each task's **Depends On** field

### Step 4: Execute Tasks

Process tasks in dependency order. For each unblocked task:

1. Mark task as `in_progress` via `TaskUpdate`
2. Determine agent type from the task's **Agent Type** field:
   - `team:builder` → use subagent_type `builder` (from `agents/team/builder.md`)
   - `team:validator` → use subagent_type `validator` (from `agents/team/validator.md`)
   - Any other value → use that value directly as subagent_type
3. Dispatch via `Task` tool:
   ```
   Task({
     description: "[task name]",
     prompt: "You are assigned task [TaskCreate ID]. Use TaskGet to read your task details.

     Plan context: [PATH_TO_PLAN]
     Read the plan for full context about the objective and implementation approach.

     Execute your assigned task completely, then mark it completed via TaskUpdate.",
     subagent_type: [resolved agent type],
     model: "opus"
   })
   ```
4. **Parallel tasks**: If a task has `Parallel: true` AND its dependencies are met AND another unblocked task also has `Parallel: true`, dispatch both using `run_in_background: true` and wait for both via `TaskOutput`
5. **Sequential tasks**: Dispatch and wait for completion before proceeding

### Step 5: Handle Results

After each agent completes:

1. Check `TaskList` to verify task was marked `completed`
2. If the next task is a validation task (Agent Type: `team:validator`):
   - Dispatch validator with the same task context
   - If validator reports FAIL: resume the original builder via `Task` with `resume` parameter to fix issues, then re-validate
3. Continue to next unblocked task

### Step 6: Completion

After all tasks complete:

1. Run any **Validation Commands** from the plan
2. Verify **Acceptance Criteria** are met
3. **Linear Integration** — If a Linear issue was found in Step 2:
   - Update state to "Done" via `mcporter call 'linear.update_issue(...)' --output json`
   - Add a comment via `mcporter call 'linear.create_comment(...)' --output json`:
     ```
     Plan executed successfully.

     **Summary:** [2-3 sentence summary]
     **Tasks completed:** [count]

     ---
     *Updated via /build*
     ```
4. Present completion report

### Context Window Guard

After each task completes, assess context usage. If above 45%, stop and tell the user:
```
Context window at ~[X]%. Stopping to maintain quality.

Tasks completed: [list]
Tasks remaining: [list]

Run /build [PATH_TO_PLAN] to continue remaining tasks.
```

## Report

After all tasks complete, present:

```
Build Complete

Plan: [PATH_TO_PLAN]
Tasks: [completed]/[total]

Results:
- [task 1]: [status + summary]
- [task 2]: [status + summary]
- ...

Validation: [pass/fail]
Linear: [issue updated or N/A]
```
