---
description: Prepare a native plan for execution — scan extensions, copy plan, allocate tools, suggest execution mode
allowed-tools:
  - Task
  - Read
  - Glob
  - Grep
  - Bash(ls *)
  - Bash(mkdir *)
  - Bash(date *)
  - Bash(cp *)
  - Write
  - Edit
  - AskUserQuestion
argument-hint: [plan-name]
---

<objective>
Prepare a user-created plan from `~/.claude/plans/` for execution: scan extensions, let the user pick a plan (optionally filtered by `$ARGUMENTS`), copy it to `.planning/`, allocate extensions to tasks, and recommend an execution mode.
</objective>

<context>
Available plans: !`ls -lt ~/.claude/plans/*.md 2>/dev/null | head -5`
Current date: !`date +%Y-%m-%d`
</context>

<process>

<phase name="1_extension_discovery" label="Scan available extensions">
1. **Launch extension scanner:** Dispatch the `extension-scanner` agent via Task tool:
   - subagent_type: `native-workflow:extension-scanner`
   - model: haiku
   - prompt: "Scan all available Claude Code extensions at project, user, and plugin levels. Return a formatted markdown summary of all commands, skills, and agents found."

2. **Store the returned markdown summary** (commands/skills/agents tables) in context for Phase 4.
</phase>

<phase name="2_plan_selection" label="User picks a plan">
1. **List recent plans:** Run `ls -lt ~/.claude/plans/*.md | head -5` to find the 5 most recently modified plan files.

2. **Extract titles:** For each file, read the first 10 lines to find the first `#` heading (the plan title).

3. **Argument matching:** If `$ARGUMENTS` is provided, try to match it against filenames (partial, case-insensitive). If exactly one match is found, skip to step 5 with that file.

4. **Present choices:** Use `AskUserQuestion` with options formatted as:
   ```
   filename.md — "First heading line"
   ```
   Let the user select which plan to prepare.

5. **Read the selected plan file** fully into context.
</phase>

<phase name="3_copy_plan" label="Copy plan to .planning/">
1. **Derive plan name:**
   - Extract the first `#` heading from the plan content
   - Convert to kebab-case: lowercase, replace spaces with hyphens, strip special characters
   - Example: "Redesign Native Workflow" becomes `redesign-native-workflow`

2. **Generate filename:** `YYYY-MM-DD_${PLAN_NAME}.md` using today's date (from `date +%Y-%m-%d`).

3. **Create directory:** Run `mkdir -p .planning/` in the project root.

4. **Write the plan:** Write the full plan content to `.planning/${DATE}_${PLAN_NAME}.md`.

5. **Confirm** the file was written successfully.
</phase>

<phase name="4_extension_allocation" label="Map extensions to tasks">
1. **Read the copied plan** from `.planning/`.

2. **For each task in the plan**, analyze what it does and match against the extension summary from Phase 1:
   - Which **skills** could assist (e.g., `create-agent-skills` for agent work, `mcp-github` for GitHub operations)
   - Which **agents** could be dispatched (e.g., `code-reviewer` for review steps)
   - Which **commands** might be invoked during execution

3. **Add an `## Extensions to Use` section** to the plan file (or update if one exists):
   ```markdown
   ## Extensions to Use
   - Task 1: `/mcp-github` for repo operations, `code-reviewer` agent for validation
   - Task 3: `create-agent-skills` skill for agent creation
   ```

4. **Allocation criteria:**
   - Include only if the task explicitly needs the extension's functionality
   - Exclude general-purpose thinking commands (e.g., `/first-principles`, `/sos`)
   - Exclude if native tools (Read, Write, Bash) are sufficient on their own
   - Do not force-map every extension to every task
</phase>

<phase name="5_complexity_analysis" label="Recommend execution mode">
1. **Analyze plan structure:**
   - **Task count**: How many tasks?
   - **Dependencies**: Which tasks depend on others? Which are independent?
   - **Scope per task**: Single file? Multi-file? Cross-system?
   - **Risk level**: Does any task require human verification?

2. **Apply heuristics:**
   - **Sequential** (recommend when): 1-3 tasks, mostly dependent chain, or tasks need human decisions
   - **Parallel sub-agents** (recommend when): 3-5 independent tasks, medium complexity, no shared state between tasks
   - **Agent teams** (recommend when): 5+ tasks, multiple independent work streams, complex scope with builder+validator benefit

3. **Present recommendation** via `AskUserQuestion` with 3 options:
   - Sequential
   - Parallel sub-agents
   - Agent teams
   Mark the recommended option based on the heuristics above.

4. **Embed in plan file:**
   ```markdown
   ## Execution Mode
   Mode: sequential | parallel | teams

   Rationale: [1-2 sentence explanation of why this mode was chosen]
   ```
</phase>

</process>

<success_criteria>
- Extension scanner agent completed and returned non-empty markdown summary with commands, skills, and agents tables
- User selected a plan from `~/.claude/plans/`
- Plan copied to `.planning/YYYY-MM-DD_plan-name.md`
- Extensions allocated to tasks where they genuinely fit
- Execution mode embedded in plan file with rationale
- User informed of next step: `/native-workflow:execute .planning/YYYY-MM-DD_plan-name.md`
</success_criteria>

<constraints>
- NEVER skip Phase 1 (extension discovery) — it provides context for better extension allocation
- NEVER modify task descriptions, objectives, or goals — adding metadata sections (Extensions to Use, Execution Mode) is permitted and required
- If `~/.claude/plans/` is empty or missing, error with: "No plans found in ~/.claude/plans/. Create a plan in native plan mode first."
- The copied plan file IS the execution prompt — ensure it is self-contained and actionable after setup
- If $ARGUMENTS matches exactly one plan file, still confirm the selection with the user before copying
</constraints>
