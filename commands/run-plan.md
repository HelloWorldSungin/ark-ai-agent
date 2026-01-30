---
type: prompt
description: Execute a PLAN.md file or auto-dispatch next phase from Linear
arguments:
  - name: plan_path
    description: Path to PLAN.md file (optional — omit to auto-dispatch next phase from Linear)
    required: false
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - Task
  - AskUserQuestion
  - mcp__plugin_linear_linear__list_issues
  - mcp__plugin_linear_linear__list_teams
  - mcp__plugin_linear_linear__list_projects
  - mcp__plugin_linear_linear__update_issue
  - mcp__plugin_linear_linear__update_project
  - mcp__plugin_linear_linear__create_comment
  - mcp__plugin_linear_linear__get_issue
---

{{#if plan_path}}

## Mode: Direct Plan Execution (path provided)

Execute the plan at {{plan_path}} using **intelligent segmentation** for optimal quality.

**Process:**

0. **Linear Integration — Resolve issue for this plan (optional, non-blocking):**
   - Resolve Linear team: check CLAUDE.md for `linear-team`, else `mcp__plugin_linear_linear__list_teams` → auto-select if one team
   - Extract the plan's phase title from the `<objective>` section or filename (e.g., "Phase 03: Integration")
   - Search Linear for a matching issue: `mcp__plugin_linear_linear__list_issues` with `query` matching the phase title and the resolved team
   - If a matching issue is found:
     - Update its state to "In Progress" via `mcp__plugin_linear_linear__update_issue`
     - Store the issue ID for use in step 4
   - If no matching issue found: skip Linear updates silently (no error, no user prompt)

1. **Verify plan exists and is unexecuted:**
   - Read {{plan_path}}
   - Check if corresponding SUMMARY.md exists in same directory
   - If SUMMARY exists: inform user plan already executed, ask if they want to re-run
   - If plan doesn't exist: error and exit

2. **Parse plan and determine execution strategy:**
   - Extract `<objective>`, `<execution_context>`, `<context>`, `<tasks>`, `<verification>`, `<success_criteria>` sections
   - Analyze checkpoint structure: `grep "type=\"checkpoint" {{plan_path}}`
   - Determine routing strategy:

   **Strategy A: Fully Autonomous (no checkpoints)**
   - Spawn single subagent to execute entire plan
   - Subagent reads plan, executes all tasks, creates SUMMARY, commits
   - Main context: Orchestration only (~5% usage)
   - Go to step 3A

   **Strategy B: Segmented Execution (has verify-only checkpoints)**
   - Parse into segments separated by checkpoints
   - Check if checkpoints are verify-only (checkpoint:human-verify)
   - If all checkpoints are verify-only: segment execution enabled
   - Go to step 3B

   **Strategy C: Decision-Dependent (has decision/action checkpoints)**
   - Has checkpoint:decision or checkpoint:human-action checkpoints
   - Following tasks depend on checkpoint outcomes
   - Must execute sequentially in main context
   - Go to step 3C

3. **Execute based on strategy:**

   **3A: Fully Autonomous Execution**
   ```
   Spawn Task tool (subagent_type="general-purpose"):

   Prompt: "Execute plan at {{plan_path}}

   This is a fully autonomous plan (no checkpoints).

   - Read the plan for full objective, context, and tasks
   - Execute ALL tasks sequentially
   - Follow all deviation rules and authentication gate protocols
   - Create SUMMARY.md in same directory as PLAN.md
   - Update ROADMAP.md plan count
   - Commit with format: feat({phase}-{plan}): [summary]
   - Report: tasks completed, files modified, commit hash"

   Wait for completion → Done
   ```

   **3B: Segmented Execution (verify-only checkpoints)**
   ```
   For each segment (autonomous block between checkpoints):

     IF segment is autonomous:
       Spawn subagent:
         "Execute tasks [X-Y] from {{plan_path}}
          Read plan for context and deviation rules.
          DO NOT create SUMMARY or commit.
          Report: tasks done, files modified, deviations"

       Wait for subagent completion
       Capture results

     ELSE IF task is checkpoint:
       Execute in main context:
         - Load checkpoint task details
         - Present checkpoint to user (action/verify/decision)
         - Wait for user response
         - Continue to next segment

   After all segments complete:
     - Aggregate results from all segments
     - Create SUMMARY.md with aggregated data
     - Update ROADMAP.md
     - Commit all changes
     - Done
   ```

   **3C: Decision-Dependent Execution**
   ```
   Execute in main context:

   Read execution context from plan <execution_context> section
   Read domain context from plan <context> section

   For each task in <tasks>:
     IF type="auto": execute in main, track deviations
     IF type="checkpoint:*": execute in main, wait for user

   After all tasks:
     - Create SUMMARY.md
     - Update ROADMAP.md
     - Commit
     - Done
   ```

4. **Summary and completion:**
   - Verify SUMMARY.md created
   - Verify commit successful
   - **Linear Integration** — If a Linear issue was found in step 0:
     - Update state to "Done" via `mcp__plugin_linear_linear__update_issue`
     - Add a comment with `mcp__plugin_linear_linear__create_comment`:
       ```
       ✅ Plan executed successfully.

       **Summary:** [2-3 sentence summary from SUMMARY.md]

       **Commit:** [commit hash]

       ---
       *Updated via /run-plan*
       ```
   - Present completion message with next steps

{{else}}

## Mode: Auto-Dispatch Next Phase from Linear (no args)

Automatically find the next phase to execute from Linear, read its agent assignment, and dispatch to the correct subagent.

**Process:**

### Step 1: Resolve Linear Team and Project

1. Resolve Linear team: check CLAUDE.md for `linear-team`, else `mcp__plugin_linear_linear__list_teams` → auto-select if one team
2. List projects via `mcp__plugin_linear_linear__list_projects`
   - If one project: use it automatically
   - If multiple: ask user which project to use via AskUserQuestion
   - If none: error — "No Linear projects found. Run `/create-plan` first to set up a project."

### Step 2: Find Next Phase

1. List issues for the selected project: `mcp__plugin_linear_linear__list_issues` filtered to "Todo" state
2. Sort by title (phase numbering ensures correct order: "Phase 01" < "Phase 02")
3. Take the first "Todo" issue — this is the next phase to execute
4. If no "Todo" issues found:
   - Check for "In Progress" issues — if found: "Phase [X] is already in progress. Check its status or wait for completion."
   - If all "Done": "All phases complete! Project finished."
   - Exit

### Step 3: Read Agent Assignment

1. Read the issue's labels to find the agent assignment
2. The agent label indicates which subagent_type to use (e.g., "feature-dev:code-architect", "general-purpose")
3. If no agent label found: default to "general-purpose"

### Step 4: Map to Plan File

1. Parse the issue title to extract phase number (e.g., "Phase 01: Foundation" → `01`)
2. Find the corresponding plan directory: `.planning/phases/01-*/`
3. Find the first unexecuted PLAN.md: look for `XX-YY-PLAN.md` without a matching `XX-YY-SUMMARY.md`
4. If no plan file found: error — "No PLAN.md found for this phase. Run `/create-plan` to generate plans."

### Step 5: Confirm with User

Present to user:

```
Next phase: [issue title]
Agent: [agent-name]
Plan file: [path to PLAN.md]

Dispatching to [agent-name] in background. Proceed? (Y/n)
```

Wait for confirmation.

### Step 6: Update Linear State

1. **First execution check:** If this is the first phase being started (no "Done" or "In Progress" issues exist):
   - Update the Linear project state from "planned" to "started" via `mcp__plugin_linear_linear__update_project`

2. Update the phase issue state to "In Progress" via `mcp__plugin_linear_linear__update_issue`

### Step 7: Dispatch to Subagent

Launch the subagent in background via Task tool:

```
Task tool:
  subagent_type: [agent-label from Step 3]
  prompt: |
    Execute the plan at [plan-file-path].

    **Context:**
    - Read .planning/BRIEF.md for project context
    - Read the PLAN.md file for full objective, tasks, and verification

    **Instructions:**
    - Execute ALL tasks in the plan sequentially
    - Follow all deviation rules (auto-fix bugs, auto-add critical, ask on architectural)
    - Create SUMMARY.md in the same directory as PLAN.md when complete
    - Update .planning/ROADMAP.md progress
    - Commit with format: feat({phase}-{plan}): [summary]

    **Linear completion (do this AFTER committing):**
    - Linear issue ID: [issue-id from Step 2]
    - Linear team: [resolved team from Step 1]
    - Update the issue state to "Done" via mcp__plugin_linear_linear__update_issue
    - Add a comment via mcp__plugin_linear_linear__create_comment:
      "✅ Phase executed successfully.

      **Agent:** [agent-name]
      **Summary:** [2-3 sentence summary from SUMMARY.md]
      **Commit:** [commit hash]

      ---
      *Updated via /run-plan (auto-dispatch)*"

    - Report back: tasks completed, files modified, commit hash, any deviations
```

**Note:** Subagent runs in foreground (fresh context, parent waits). The subagent handles its own Linear updates on completion, then reports results back to parent.

### Step 8: Report Results

When the subagent returns, present the results to the user:

```
Phase complete: [phase title]
Agent: [agent-name]
Result: [summary from subagent report]
Commit: [hash]
Linear: [issue-id] → Done

Your options:
- /run-plan — dispatch next phase
- /whats-next — pause and create handoff
- Continue with other work
```

{{/if}}

**Critical Rules:**

- **Read execution_context first:** Always load files from `<execution_context>` section before executing
- **Minimal context loading:** Only read files explicitly mentioned in `<execution_context>` and `<context>` sections
- **No skill invocation:** Execute directly using native tools - don't invoke create-plans skill
- **All deviations tracked:** Apply deviation rules from execute-phase.md, document everything in Summary
- **Checkpoints are blocking:** Never skip user interaction for checkpoint tasks
- **Verification is mandatory:** Don't mark complete without running verification checks
- **Follow execute-phase.md protocol:** Loaded context contains all execution instructions

**Context Efficiency Target:**
- Execution context: ~5-7k tokens (execute-phase.md, summary.md, checkpoints.md if needed)
- Domain context: ~10-15k tokens (BRIEF, ROADMAP, codebase files)
- Total overhead: <30% context, reserving 70%+ for workspace and implementation
