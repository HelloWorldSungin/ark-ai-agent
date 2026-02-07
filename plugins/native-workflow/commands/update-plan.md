---
description: Update a plan with current progress, roadblocks, and issues encountered during execution
allowed-tools:
  - Read
  - Bash(git log *)
  - Bash(git diff *)
  - Bash(ls *)
  - Glob
  - Grep
  - Write
  - EnterPlanMode
  - ExitPlanMode
argument-hint: [path-to-plan]
---

<objective>
Update the plan at `$ARGUMENTS` (or most recent plan in `.planning/`) — assess progress via git history, identify roadblocks and issues, mark completed tasks `[DONE]`, and revise remaining work.
</objective>

<context>
Recent plans: !`ls -lt .planning/*.md 2>/dev/null | head -3 || echo "No plans found"`
</context>

<process>

<phase name="1_assess_progress" label="BEFORE plan mode">
1. **Resolve plan path:**
   - If `$ARGUMENTS` provides a path: use it directly
   - If no path provided: find the most recently modified `.planning/*.md` file
   - If no plans found: error — "No plans found in `.planning/`. Nothing to update."

2. **Read the existing plan** and parse its sections (Objective, Tasks, Extensions to Use, Execution Mode, Verification, etc.)

3. **Extract date context** from the plan filename:
   - Parse `YYYY-MM-DD` from the filename (e.g., `2026-02-07_rest-api.md` -> `2026-02-07`)
   - If no date found in filename: use `--since="1 week ago"` as fallback
   - This scopes the git history analysis

4. **Analyze actual progress** using git history:
   - `git log --since="YYYY-MM-DD" --oneline`
   - `git diff --name-only HEAD~10` (adjust range based on commit count)

5. **Cross-reference** plan tasks with actual changes:
   - For each task, check if the files it references were modified
   - Check if completion criteria can be verified from the current state
   - Build a status assessment: `[DONE]`, `[PARTIAL]`, or `[TODO]` for each task

6. **Check for roadblocks and issues:**
   - If `.planning/ISSUES.md` exists: read and incorporate issues logged during `/native-workflow:execute`
   - If `.planning/ISSUES.md` does not exist: rely solely on git history and file state analysis
   - Check if any tasks were partially completed (look for `[PARTIAL]` markers in the plan)
   - Note any error patterns from execution logs or git history
   - Assess: what went wrong, what's blocking progress
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
   - **Preserve `## Extensions to Use` section** — update if new extensions are relevant
   - **Preserve `## Execution Mode` section** — update if the mode should change based on experience
   - **Add `## Roadblocks & Issues` section:**
     ```
     ## Roadblocks & Issues
     - [Issue 1]: description and status
     - [Issue 2]: description and resolution
     ```

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

   To continue: start a fresh session and run /native-workflow:execute .planning/YYYY-MM-DD_plan-name.md
   ```
</phase>

</process>

<verification>
Before reporting completion, verify:
- All completed tasks correctly marked `[DONE]` based on git evidence
- ISSUES.md items incorporated as new tasks or noted in Roadblocks & Issues (if ISSUES.md exists)
- Updated plan is self-contained and executable by `/native-workflow:execute`
- Original plan file overwritten (not a new file)
- `## Execution Mode` and `## Extensions to Use` sections preserved (updated if needed)
</verification>

<success_criteria>
- Progress assessed against git history and file changes
- Completed tasks marked `[DONE]` with evidence
- Remaining tasks updated with current context
- Roadblocks & Issues section added with any discovered problems
- New tasks added from ISSUES.md or discovered issues
- User approved updated plan via ExitPlanMode
- Original plan file overwritten with updated content
</success_criteria>

<constraints>
- NEVER delete tasks — mark them `[DONE]`, don't remove them (preserves history)
- Git analysis is best-effort — if git history is limited, rely on file existence checks and plan cross-referencing
- ALWAYS use native plan mode for revision — ensures user reviews changes before they're saved
- ALWAYS overwrite the original file — the cycle is plan-setup -> execute -> update -> execute on the SAME file
- ALWAYS preserve `## Execution Mode` and `## Extensions to Use` sections (update content if needed, never remove)
</constraints>
