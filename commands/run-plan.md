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
  - mcp__plugin_linear_linear__list_issue_labels
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

### Step 1: Resolve Linear Team and Find Plan

1. Resolve Linear team: check CLAUDE.md for `linear-team`, else `mcp__plugin_linear_linear__list_teams` → auto-select if one team
2. Find plan issues via `mcp__plugin_linear_linear__list_issues` filtered by label `plan`
   - This returns only project-level issues created by `/create-plan`
   - If one plan issue: use it automatically
   - If multiple: ask user which plan to execute via AskUserQuestion
   - If none: error — "No plan issues found in Linear. Run `/create-plan` first."
3. Read the selected plan issue's labels to find the **plan-name label** (the label that is NOT `plan` — it's the unique plan-name)
4. Store the plan-name label for filtering in Step 2

### Step 2: Find Next Phase

1. List issues filtered by the plan-name label and "Todo" state: `mcp__plugin_linear_linear__list_issues` with label `<plan-name>` and state "Todo"
2. Filter to phase issues only (issues that also have the `phase` label)
3. Sort by title (phase numbering ensures correct order: "Phase 01" < "Phase 02")
4. Take the first "Todo" phase issue — this is the next phase to execute
5. If no "Todo" phase issues found:
   - Check for "In Progress" phase issues (filter by `phase` + plan-name labels, "In Progress" state) — if found: "Phase [X] is already in progress. Check its status or wait for completion."
   - If all "Done": "All phases complete! Project finished."
   - Exit

### Step 3: Read Agent Assignment

1. Read the issue's labels to find the agent assignment
2. The agent label indicates which subagent_type to use (e.g., "feature-dev:code-architect", "general-purpose")
3. **Fallback — parse from description:** If no agent label found, search the issue description for:
   - `**Sub-agent:** \`agent-name\`` or `**Agent:** \`agent-name\``
   - `**Sub-agents:** \`agent1\`, \`agent2\`` (use the first one)
   - Extract the agent name from the backtick-wrapped value
4. If still no agent found: default to "general-purpose"

### Step 4: Map to Plan File

1. Parse the issue title to extract phase number (e.g., "Phase 01: Foundation" → `01`)
2. Find the corresponding plan directory: `.planning/phases/01-*/`
3. Find the first unexecuted PLAN.md: look for `XX-YY-PLAN.md` without a matching `XX-YY-SUMMARY.md`
4. If no plan file found: error — "No PLAN.md found for this phase. Run `/create-plan` to generate plans."

### Step 5: Update Linear State

1. **First execution check:** If this is the first phase being started (no "Done" or "In Progress" issues exist):
   - Update the Linear project state from "planned" to "started" via `mcp__plugin_linear_linear__update_project`

2. Update the phase issue state to "In Progress" via `mcp__plugin_linear_linear__update_issue`

### Step 6: Dispatch to Subagent

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

### Step 7: Report Results and Auto-Continue

When the subagent returns, log the phase result:

```
✅ Phase complete: [phase title]
   Agent: [agent-name]
   Result: [summary from subagent report]
   Commit: [hash]
   Linear: [issue-id] → Done
```

Then apply the **context window guard**:

1. **Assess context usage.** After each phase completes, estimate how much of your context window you have consumed so far. Consider the total volume of text exchanged (prompts, tool results, responses) relative to your full context capacity.

2. **If context usage exceeds 45%**, stop the loop and tell the user:
   ```
   ⚠️ Context window is at ~[X]%. To maintain quality, stopping here.

   Run /whats-next to create a handoff, then start a fresh session and run /run-plan to continue.
   ```

3. **If context usage is below 45%**, loop back to **Step 2** — find the next "Todo" phase and dispatch it automatically. Do NOT ask for confirmation; proceed immediately.

4. **If no more "Todo" phases remain**, report project completion:
   ```
   🎉 All phases complete! Project finished.

   Run /whats-next to create a final handoff document.
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
