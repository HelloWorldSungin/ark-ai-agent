---
description: Add todo item to Linear and TO-DOS.md with context from conversation
argument-hint: <todo-description> (optional - infers from conversation if omitted)
allowed-tools:
  - Read
  - Edit
  - Write
  - Bash
---

# Add Todo Item

## Context

- Current timestamp: !`date "+%Y-%m-%d %H:%M"`

## Instructions

1. Read TO-DOS.md in the working directory (create with Write tool if it doesn't exist)

2. Resolve Linear team:
   - Check if the project's CLAUDE.md specifies a `linear-team` value
   - If not, use `mcporter call linear.list_teams --output json` to list available teams
   - If only one team exists, use it automatically
   - If multiple teams, ask user which team to use
   - Cache the resolved team name for steps below

3. Check for duplicates:
   - Extract key concept/action from the new todo
   - Search existing todos for similar titles or overlapping scope (check both TO-DOS.md and Linear via `mcporter call linear.list_issues --output json` with label "todo", state "Todo", using the resolved team)
   - If found, ask user: "A similar todo already exists: [title]. Would you like to:\n\n1. Skip adding (keep existing)\n2. Replace existing with new version\n3. Add anyway as separate item\n\nReply with the number of your choice."
   - Wait for user response before proceeding

4. Extract todo content:
   - **With $ARGUMENTS**: Use as the focus/title for the todo and context heading
   - **Without $ARGUMENTS**: Analyze recent conversation to extract:
     - Specific problem or task discussed
     - Relevant file paths that need attention
     - Technical details (line numbers, error messages, conflicting specifications)
     - Root cause if identified

5. Create Linear issue:
   - Use `mcporter call 'linear.create_issue(...)' --output json` with:
     - **title**: Action verb + component (3-8 words, same as the bold title)
     - **team**: Resolved team from step 2
     - **state**: "Todo"
     - **labels**: ["todo"]
     - **description**: Markdown formatted with:
       ```
       **Problem:** [What's wrong/why needed]

       **Files:** [Comma-separated paths with line numbers]

       **Solution:** [Approach hints or constraints, if applicable]

       ---
       *Created via /add-to-todos*
       ```
   - Capture the returned issue identifier (e.g., "ARKSIG-8")

6. Append to TO-DOS.md as local reference:
   - **Heading**: `## Brief Context Title - YYYY-MM-DD HH:MM` (3-8 word title, current timestamp)
   - **Todo format**: `- **[Action verb] [Component]** (LINEAR_ID) - [Brief description]. **Problem:** [What's wrong/why needed]. **Files:** [Comma-separated paths with line numbers]. **Solution:** [Approach hints or constraints, if applicable].`
   - **Required fields**: Problem and Files (with line numbers like `path/to/file.ts:123-145`)
   - **Optional field**: Solution
   - Make each section self-contained for future Claude to understand weeks later
   - Use simple list items (not checkboxes) - todos are removed when work begins

7. Confirm and offer to continue with original work:
   - Identify what the user was working on before `/add-to-todos` was called
   - Confirm: "Saved to Linear ([LINEAR_ID]) and TO-DOS.md."
   - Ask if they want to continue with the original work: "Would you like to continue with [original task]?"
   - Wait for user response

## Format Example

```markdown
## Add Todo Command Improvements - 2025-11-15 14:23

- **Add structured format to add-to-todos** (ARK-42) - Standardize todo entries with Problem/Files/Solution pattern. **Problem:** Current todos lack consistent structure, making it hard for Claude to have enough context when revisiting tasks later. **Files:** `commands/add-to-todos.md:22-29`. **Solution:** Use inline bold labels with required Problem and Files fields, optional Solution field.
```
