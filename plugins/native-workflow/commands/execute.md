---
description: Execute a plan from .planning/ with the specified execution mode — self-checkpoints at ~45% context usage
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - Task
  - Skill
  - WebSearch
  - WebFetch
  - AskUserQuestion
  - TeamCreate
  - TeamDelete
  - SendMessage
  - TaskCreate
  - TaskList
  - TaskUpdate
  - TaskGet
argument-hint: <path-to-plan>
---

<objective>
Load and execute the plan file at the path specified in `$ARGUMENTS` using the execution mode embedded in the plan, with context window monitoring that stops at ~45% usage to preserve output quality.
</objective>

<context>
Plan file: !`cat "$ARGUMENTS" 2>/dev/null | head -5 || echo "File not found"`
</context>

<process>

<step name="1_load_plan" label="Load and parse the plan">
1. **Validate input:**
   - `$ARGUMENTS` is REQUIRED — if empty, error with: "Usage: `/native-workflow:execute <path-to-plan>` — provide the path to a plan in `.planning/`"
   - Verify the file exists — if not, error with: "Plan not found at `$ARGUMENTS` — check the path and try again."

2. **Read the plan file** at the given path and parse its sections:
   - **Objective** — what we're building
   - **Context Files** — files to read before starting
   - **Tasks** — numbered implementation tasks with completion criteria
   - **Extensions to Use** — available extensions referenced in the plan
   - **Execution Mode** — `sequential`, `parallel`, or `teams` (default to `sequential` if missing)
   - **Verification** — how to verify completion

3. **Identify task status:**
   - Tasks prefixed with `[DONE]` have already been completed — skip them
   - All other tasks are pending execution
   - If ALL tasks are `[DONE]`: report "All tasks already completed" and exit

4. **Load context files** listed in the plan (read them to establish working context).

5. **Extract execution mode** from the `## Execution Mode` section. If the section is missing, default to `sequential`.
</step>

<deviation_rules>
All execution modes follow these deviation rules:
- **Auto-fix:** Bugs, type errors, and blockers — fix them and note the deviation
- **Ask user:** Architecture changes, scope additions, or significant departures — use AskUserQuestion
- **Log:** Enhancements or nice-to-haves — append to `.planning/ISSUES.md` (create if needed)
</deviation_rules>

<step name="2_execute" label="Execute tasks by mode">

<mode name="sequential">
Process pending tasks one by one in the main context:

1. **For each pending task:**
   - Read the task description and completion criteria
   - Check if any extensions were allocated in `## Extensions to Use` — invoke relevant skills or reference agents
   - Implement the task using available tools
   - Verify completion criteria are met
   - Mark the task `[DONE]` in the plan file via Edit tool
   - If a task fails or is blocked: note the issue and continue to the next task if possible

2. Apply deviation rules above for any departures from the plan.
</mode>

<mode name="parallel">
Dispatch independent tasks as sub-agents:

1. **Analyze task dependencies** from the plan:
   - Group tasks into **independent** (can run in parallel) and **dependent** (must wait for others)

2. **For each batch of independent tasks:**
   - Dispatch each as a Task tool sub-agent with `run_in_background: true`
   - Provide each sub-agent with: task description, relevant context files, allocated extensions, completion criteria
   - Use `subagent_type: general-purpose` for implementation tasks

3. **Monitor completion** via TaskOutput — check each background agent for results.

4. **For dependent tasks:** Wait for their blockers to complete, then dispatch the next batch.

5. **After all sub-agents complete:**
   - Collect results from each
   - Mark tasks `[DONE]` in the plan file
   - Aggregate deviations across all sub-agents
</mode>

<mode name="teams">
Orchestrate via agent teams with builder+validator separation:

1. **Create team:** `TeamCreate` with a descriptive name derived from the plan objective.

2. **Create task entries:** `TaskCreate` for each pending plan task, including:
   - Task description and completion criteria from the plan
   - Allocated extensions from `## Extensions to Use`

3. **Set dependencies:** Use `TaskUpdate` with `addBlockedBy` to mirror plan task dependencies.

4. **Dispatch workers:** For each unblocked task:
   - Dispatch a **builder** agent (`subagent_type: general-purpose`) via Task tool with `team_name`
   - After builder completes: dispatch a **validator** agent (`subagent_type: Explore`, read-only) to verify completion criteria

5. **Monitor progress** via `TaskList` — dispatch new tasks as dependencies resolve.

6. **On completion:** `TeamDelete` to clean up team resources.
</mode>

</step>

<step name="3_context_monitoring" label="Monitor context usage (ALL modes)">
After each task completion (or sub-agent batch), assess context window usage.

**STOP execution if you observe ANY of these signals:**
- Responses becoming noticeably shorter or less detailed than earlier in the session
- Difficulty tracking multiple file states or forgetting earlier changes
- Reading the same files multiple times because prior content was lost from context
- Completed **3+ tasks** with extensive tool output (many large file reads, long bash results)
- Uncertain whether enough context remains to complete the next task well

**When stopping:**
1. Mark all completed tasks `[DONE]` in the plan file
2. Report using the output format below with completed and remaining sections
3. Instruct user:
   ```
   Next steps:
   1. Run: /native-workflow:update-plan .planning/YYYY-MM-DD_plan-name.md
   2. Start a fresh session
   3. Run: /native-workflow:execute .planning/YYYY-MM-DD_plan-name.md
   ```
</step>

<step name="4_completion" label="Final verification">
If all tasks complete without hitting the context limit:

1. **Run verification** checks from the plan's `## Verification` section.
2. **Report results** using the output format below.
</step>

</process>

<output_format>
```
Execution complete: .planning/YYYY-MM-DD_plan-name.md
Mode: sequential | parallel | teams

Completed:
- [x] Task 1: description
- [x] Task 2: description

Remaining (if stopped early):
- [ ] Task 3: description

Verification:
- [result of each check]

Files modified:
- [list]

Deviations:
- [any deviations, or "None"]
```
</output_format>

<success_criteria>
- All pending tasks executed with completion criteria verified
- Execution mode from `## Execution Mode` section respected
- Plan verification checks passing (if all tasks completed)
- Deviations reported (or "None")
- Files modified listed
- Output format followed for either completion or early stop
</success_criteria>

<constraints>
- `$ARGUMENTS` is REQUIRED — do not fall back to most-recent file lookup
- NEVER re-execute `[DONE]` tasks — skip completed work
- Context monitoring is MANDATORY — degraded output at high context is worse than stopping early
- ALWAYS track and report deviations — never hide what changed from the plan
- Execute in the user's current session — this command assumes a fresh session for clean context
- For `parallel` and `teams` modes: ensure sub-agents receive sufficient context to work independently
</constraints>
