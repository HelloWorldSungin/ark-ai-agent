---
description: Update an existing plan — marks completed tasks and revises remaining work using native plan mode
argument-hint: <path-to-plan>
allowed-tools:
  - Read
  - Bash
  - Glob
  - Grep
  - Write
  - EnterPlanMode
  - ExitPlanMode
---

You are executing the `/update-plan` command — a three-phase workflow that assesses progress on an existing plan, then uses native plan mode to revise it.

**Input:** $ARGUMENTS

---

## Phase 1: Assess Progress (BEFORE plan mode)

1. **Resolve plan path:**
   - If `$ARGUMENTS` provides a path: use it directly
   - If no path provided: find the most recently modified `.planning/*.md` file
   - If no plans found: error — "No plans found in `.planning/`. Nothing to update."

2. **Read the existing plan** and parse its sections (Objective, Tasks, Verification, etc.)

3. **Extract date context** from the plan filename:
   - Parse `YYYY-MM-DD` from the filename (e.g., `2026-02-07_rest-api.md` → `2026-02-07`)
   - This is used to scope git history analysis

4. **Analyze actual progress** using git history:
   ```bash
   git log --since="YYYY-MM-DD" --oneline
   ```
   ```bash
   git diff --name-only HEAD~10
   ```
   (Adjust the diff range based on commit count)

5. **Cross-reference** plan tasks with actual changes:
   - For each task, check if the files it references were modified
   - Check if completion criteria can be verified from the current state
   - Build a status assessment: `[DONE]`, `[PARTIAL]`, or `[TODO]` for each task

6. **Check for issues** — read `.planning/ISSUES.md` if it exists (logged during `/execute`)

---

## Phase 2: Revise Plan (IN native plan mode)

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

---

## Phase 3: Save Updated Plan (AFTER user approval)

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

---

**Critical Rules:**
- Never delete tasks — mark them `[DONE]`, don't remove them (preserves history)
- Git analysis is best-effort — if git history is limited, rely on file existence checks and plan cross-referencing
- Always use native plan mode for revision — ensures user reviews changes before they're saved
- Overwrite the original file — don't create a new file (the cycle is plan → execute → update → execute on the SAME file)
