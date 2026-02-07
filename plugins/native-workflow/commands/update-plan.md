---
description: Re-assess plan progress by marking completed tasks, incorporating issues, and revising remaining work using native plan mode
argument-hint: [path-to-plan]
allowed-tools:
  - Read
  - Bash(git log *)
  - Bash(git diff *)
  - Glob
  - Grep
  - Write
  - EnterPlanMode
  - ExitPlanMode
---

<objective>
Update the plan at $ARGUMENTS (or most recent plan in `.planning/`) — assess progress via git history, mark completed tasks `[DONE]`, revise remaining work, and incorporate discovered issues.
</objective>

<process>

<phase name="1_assess_progress" label="BEFORE plan mode">
1. **Resolve plan path:**
   - If `$ARGUMENTS` provides a path: use it directly
   - If no path provided: find the most recently modified `.planning/*.md` file
   - If no plans found: error — "No plans found in `.planning/`. Nothing to update."

2. **Read the existing plan** and parse its sections (Objective, Tasks, Verification, etc.)

3. **Extract date context** from the plan filename:
   - Parse `YYYY-MM-DD` from the filename (e.g., `2026-02-07_rest-api.md` → `2026-02-07`)
   - This is used to scope git history analysis

4. **Analyze actual progress** using git history:
   - `git log --since="YYYY-MM-DD" --oneline`
   - `git diff --name-only HEAD~10` (adjust range based on commit count)

5. **Cross-reference** plan tasks with actual changes:
   - For each task, check if the files it references were modified
   - Check if completion criteria can be verified from the current state
   - Build a status assessment: `[DONE]`, `[PARTIAL]`, or `[TODO]` for each task

6. **Check for issues** — read `.planning/ISSUES.md` if it exists (logged during `/execute`)
</phase>

<phase name="2_revise_plan" label="IN native plan mode">
1. **Enter plan mode:** Call `EnterPlanMode`

2. **Write the updated plan** to the system plan file with these rules:
   - **Preserve** the original Objective (unchanged)
   - **Mark completed tasks** with `[DONE]` prefix:
     ```
     1. [DONE] Set up project scaffolding
        - What: Created directory structure and config files
        - Completed: verified via git log
     ```
   - **Update remaining tasks** with current context:
     - Adjust descriptions based on what was learned during execution
     - Reorder if priorities changed
     - Split complex tasks that proved too large
   - **Add new tasks** if issues were discovered during execution (from ISSUES.md or git analysis)
   - **Update Context Files** section if new files are now relevant
   - **Update Verification** to cover remaining work only (completed verifications removed)
   - **Preserve Extensions to Use** section (update if new extensions are relevant)

3. **Exit plan mode:** Call `ExitPlanMode` — user reviews and approves the updated plan
</phase>

<phase name="3_save_updated_plan" label="AFTER user approval">
1. **Read the approved updated plan** from the system plan file

2. **Overwrite the original plan file** — same path, updated content:
   - Write to the same `.planning/YYYY-MM-DD_plan-name.md` path
   - The filename does NOT change (preserves the original date for history)

3. **Report to user:**
   ```
   Plan updated: .planning/YYYY-MM-DD_plan-name.md

   Progress:
   - Completed: N tasks marked [DONE]
   - Remaining: M tasks
   - New: K tasks added

   To continue: start a fresh session and run /execute .planning/YYYY-MM-DD_plan-name.md
   ```
</phase>

</process>

<verification>
Before reporting completion, verify:
- All completed tasks correctly marked `[DONE]` based on git evidence
- ISSUES.md items incorporated as new tasks (if ISSUES.md exists)
- Updated plan is self-contained and executable by `/execute`
- Original plan file overwritten (not a new file)
</verification>

<success_criteria>
- Progress assessed against git history and file changes
- Completed tasks marked `[DONE]` with evidence
- Remaining tasks updated with current context
- New tasks added from ISSUES.md or discovered issues
- User approved updated plan via ExitPlanMode
- Original plan file overwritten with updated content
</success_criteria>

<constraints>
- NEVER delete tasks — mark them `[DONE]`, don't remove them (preserves history)
- Git analysis is best-effort — if git history is limited, rely on file existence checks and plan cross-referencing
- ALWAYS use native plan mode for revision — ensures user reviews changes before they're saved
- ALWAYS overwrite the original file — the cycle is plan → execute → update → execute on the SAME file
</constraints>
