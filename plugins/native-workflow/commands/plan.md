---
description: Create a plan using native Claude Code plan mode with awareness of available project extensions
argument-hint: <what-to-build>
allowed-tools:
  - Task
  - Read
  - Glob
  - Grep
  - Bash
  - Write
  - EnterPlanMode
  - ExitPlanMode
---

You are executing the `/plan` command — a three-phase workflow that creates an implementation plan using Claude Code's native plan mode, with awareness of available extensions.

**Input:** $ARGUMENTS

---

## Phase 1: Extension Discovery (BEFORE plan mode)

1. **Validate input:** If `$ARGUMENTS` is empty or missing, ask the user what they want to build. Do not proceed without a clear objective.

2. **Scan available extensions:** Launch the `extension-scanner` agent via Task tool:
   ```
   Task tool:
     subagent_type: "extension-scanner"
     model: haiku
     prompt: "Scan all available Claude Code extensions at project, user, and plugin levels. Return a formatted markdown summary of all commands, skills, and agents found."
   ```

3. **Store the scanner output** — this extension summary stays in your context for Phase 2.

---

## Phase 2: Planning (IN native plan mode)

1. **Enter plan mode:** Call `EnterPlanMode`. The system will activate plan mode with read-only tools and a system plan file.

2. **Explore the codebase** using Read, Glob, and Grep to understand:
   - Project structure and architecture
   - Existing patterns and conventions
   - Files that will need to be created or modified

3. **Design the implementation plan** with these sections:

   ```markdown
   ## Objective
   [What we're building and why]

   ## Context Files
   [Key files to read before executing — paths only]

   ## Tasks
   1. [Task name]
      - What: [specific implementation details]
      - Files: [files to create/modify]
      - Completion criteria: [how to verify this task is done]

   2. [Next task...]

   ## Extensions to Use
   [Reference discovered extensions where applicable, e.g.:]
   - Use `/debug` if issues arise during implementation
   - Use `extension-scanner` agent if new extensions are needed
   [Only include extensions that are genuinely useful for this plan]

   ## Verification
   [How to verify the entire plan is complete — tests, manual checks, etc.]
   ```

4. **Key planning rules:**
   - Reference discovered extensions in the plan where they add value (don't force it)
   - Keep tasks concrete with clear completion criteria
   - Target 2-5 tasks per plan (scope appropriately)
   - Each task should be independently verifiable

5. **Exit plan mode:** Call `ExitPlanMode` — the user reviews and approves the plan.

---

## Phase 3: Save (AFTER user approval)

1. **Read the approved plan** from the system plan file (the path is provided in the plan mode system message).

2. **Generate filename:**
   - Date: today's date in `YYYY-MM-DD` format
   - Name: derive from the objective, converted to kebab-case (lowercase, hyphens, no special chars)
   - Result: `YYYY-MM-DD_objective-name.md`

3. **Save the plan:**
   ```bash
   mkdir -p .planning/
   ```
   Write the plan to `.planning/${filename}.md`

4. **Report to user:**
   ```
   Plan saved: .planning/YYYY-MM-DD_plan-name.md

   To execute: start a fresh session and run /execute .planning/YYYY-MM-DD_plan-name.md
   ```

---

**Critical Rules:**
- Never skip Phase 1 (extension discovery) — it provides context for better plans
- Always use native plan mode (EnterPlanMode/ExitPlanMode) — this is what differentiates this from `/create-plan`
- Save only AFTER user approval — ExitPlanMode handles the approval flow
- The saved plan file IS the execution prompt — write it to be self-contained and actionable
