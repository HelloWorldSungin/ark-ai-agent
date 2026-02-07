---
description: Execute a plan from .planning/ with context monitoring — self-checkpoints at ~50% usage
argument-hint: [path-to-plan]
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - Task
  - WebSearch
  - WebFetch
  - AskUserQuestion
---

You are executing the `/execute` command — loads a plan from `.planning/` and executes it with context window monitoring.

**Input:** $ARGUMENTS

---

## Step 1: Load Plan

1. **Resolve plan path:**
   - If `$ARGUMENTS` provides a path: use it directly
   - If no path provided: find the most recently modified `.planning/*.md` file
   - If no plans found: error — "No plans found in `.planning/`. Run `/plan` first to create one."

2. **Read the plan file** and parse its sections:
   - **Objective** — what we're building
   - **Context Files** — files to read before starting
   - **Tasks** — numbered implementation tasks with completion criteria
   - **Extensions to Use** — available extensions referenced in the plan
   - **Verification** — how to verify completion

3. **Identify task status:**
   - Tasks prefixed with `[DONE]` have already been completed (skip them)
   - All other tasks are pending execution
   - If ALL tasks are `[DONE]`: report "All tasks already completed" and exit

4. **Load context files** listed in the plan (read them to establish working context).

---

## Step 2: Execute Tasks

Process pending tasks sequentially:

1. **For each task:**
   - Read the task description and completion criteria
   - Implement the task using available tools
   - Verify completion criteria are met before moving on
   - If a task fails or is blocked: note the issue and continue to the next task if possible

2. **Deviation rules:**
   - **Auto-fix:** Bugs, type errors, and blockers encountered during implementation — fix them and note the deviation
   - **Ask user:** Architectural changes, scope additions, or significant departures from the plan — use AskUserQuestion
   - **Log:** Enhancements or nice-to-haves discovered during execution — append to `.planning/ISSUES.md` (create if needed)

---

## Step 3: Context Monitoring

After completing each task, assess your context window usage:

- **Below ~50%:** Continue to the next task normally
- **At ~50% or above:** **STOP** execution and report:

```
Context approaching limit (~50% used).

Completed:
- [x] Task 1: description
- [x] Task 2: description

Remaining:
- [ ] Task 3: description
- [ ] Task 4: description

Next steps:
1. Run: /update-plan .planning/YYYY-MM-DD_plan-name.md
2. Start a fresh session
3. Run: /execute .planning/YYYY-MM-DD_plan-name.md
```

**Context estimation heuristic:** Consider the total volume of text exchanged so far (all prompts, tool results, and responses). If you've had many large file reads, extensive tool output, or numerous back-and-forth exchanges, you're likely approaching the limit.

---

## Step 4: Completion

If all tasks are completed without hitting the context limit:

1. **Run verification** checks from the plan's Verification section
2. **Report results:**

```
Plan executed successfully: .planning/YYYY-MM-DD_plan-name.md

Completed:
- [x] Task 1: description
- [x] Task 2: description
- [x] Task 3: description

Verification:
- [result of each verification check]

Files modified:
- [list of files created/modified]

Deviations:
- [any deviations from the plan, or "None"]
```

---

**Critical Rules:**
- Skip `[DONE]` tasks — never re-execute completed work
- Context monitoring is mandatory — degraded output at high context usage is worse than stopping early
- Deviations are tracked, not hidden — always report what changed from the plan
- Execute in the user's current session — this command assumes the user started a fresh session for clean context
