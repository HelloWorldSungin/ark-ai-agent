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

<objective>
Load and execute the plan at $ARGUMENTS (or most recent plan in `.planning/`) with context window monitoring, stopping at ~50% usage to preserve output quality.
</objective>

<process>

<step name="load_plan">
1. **Resolve plan path:**
   - If `$ARGUMENTS` provides a path: use it directly
   - If no path provided: find the most recently modified `.planning/*.md` file
   - If no plans found: error — "No plans found in `.planning/`. Run `/planning` first to create one."

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
</step>

<step name="execute_tasks">
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
</step>

<step name="context_monitoring">
After completing each task, assess your context window usage.

**Checkpoint triggers** — STOP execution if ANY of these apply:
- You have completed 3+ tasks AND had extensive tool output (many large file reads, long bash results)
- You have exchanged more than ~80 back-and-forth turns with tool calls
- You notice responses becoming slower or less detailed (signs of high context usage)
- You are uncertain whether you have enough context remaining to complete the next task well

When stopping, report:
```
Context approaching limit.

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
</step>

<step name="completion">
If all tasks are completed without hitting the context limit:

1. **Run verification** checks from the plan's Verification section.
2. **Report results.**
</step>

</process>

<verification>
Before reporting completion, verify:
- All pending tasks executed and completion criteria met
- Verification checks from the plan passing
- All deviations documented
- `.planning/ISSUES.md` updated if enhancements were discovered
</verification>

<success_criteria>
- All non-`[DONE]` tasks executed with completion criteria verified
- Plan verification checks passing
- Deviations reported (or "None")
- Files modified listed
- If context limit reached: clear stop report with completed/remaining task lists
</success_criteria>

<output_format>
```
Plan executed successfully: .planning/YYYY-MM-DD_plan-name.md

Completed:
- [x] Task 1: description
- [x] Task 2: description

Verification:
- [result of each verification check]

Files modified:
- [list of files created/modified]

Deviations:
- [any deviations from the plan, or "None"]
```
</output_format>

<constraints>
- NEVER re-execute `[DONE]` tasks — skip completed work
- Context monitoring is MANDATORY — degraded output at high context usage is worse than stopping early
- ALWAYS track and report deviations — never hide what changed from the plan
- Execute in the user's current session — this command assumes a fresh session for clean context
</constraints>
