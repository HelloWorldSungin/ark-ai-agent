---
description: List outstanding todos from Linear and select one to work on
allowed-tools:
  - Read
  - Edit
  - Glob
  - mcp__plugin_linear_linear__list_issues
  - mcp__plugin_linear_linear__get_issue
  - mcp__plugin_linear_linear__update_issue
  - mcp__plugin_linear_linear__list_teams
---

# Check Todos

## Instructions

1. Resolve Linear team:
   - Check if the project's CLAUDE.md specifies a `linear-team` value
   - If not, use `mcp__plugin_linear_linear__list_teams` to list available teams
   - If only one team exists, use it automatically
   - If multiple teams, ask user which team to use

2. Fetch todos from Linear:
   - Use `mcp__plugin_linear_linear__list_issues` with label "todo", state "Todo", using the resolved team
   - If Linear returns results, use those as the primary source
   - If Linear is unavailable or returns an error, fall back to reading TO-DOS.md in the working directory
   - If neither source has items, say "No outstanding todos" and exit

3. Parse and display todos:
   - Display compact numbered list showing:
     - Number (for selection)
     - Issue identifier (e.g., ARK-42)
     - Issue title (bold)
     - Created date
   - Prompt: "Reply with the number of the todo you'd like to work on."
   - Wait for user to reply with a number

4. Load full context for selected todo:
   - Fetch full issue details via `mcp__plugin_linear_linear__get_issue`
   - Display complete description (Problem, Files, Solution)
   - Read and briefly summarize relevant files mentioned

5. Check for established workflows:
   - Read CLAUDE.md (if exists) to understand project-specific workflows and rules
   - Look for `.claude/skills/` directory
   - Match file paths in todo to domain patterns (`plugins/` → plugin workflow, `mcp-servers/` → MCP workflow)
   - Check CLAUDE.md for explicit workflow requirements for this type of work

6. Present action options to user:
   - **If matching skill/workflow found**: "This looks like [domain] work. Would you like to:\n\n1. Invoke [skill-name] skill and start\n2. Work on it directly\n3. Brainstorm approach first\n4. Put it back and browse other todos\n\nReply with the number of your choice."
   - **If no workflow match**: "Would you like to:\n\n1. Start working on it\n2. Brainstorm approach first\n3. Put it back and browse other todos\n\nReply with the number of your choice."
   - Wait for user response

7. Handle user choice:
   - **Option "Invoke skill" or "Start working"**:
     - Update Linear issue to "In Progress" via `mcp__plugin_linear_linear__update_issue`
     - Remove todo from TO-DOS.md if it exists there (and h2 heading if section becomes empty)
     - Begin work (invoke skill if applicable, or proceed directly)
   - **Option "Brainstorm approach"**: Keep as "Todo", invoke `/brainstorm` with the todo description as argument
   - **Option "Put it back"**: Keep as "Todo", return to step 2 to display the full list again

8. When work is completed:
   - Update Linear issue to "Done" via `mcp__plugin_linear_linear__update_issue`

## Display Format

```
Outstanding Todos:

1. ARK-42 — **Add structured format to add-to-todos** (2025-11-15)
2. ARK-43 — **Create check-todos command** (2025-11-15)
3. ARK-38 — **Fix cookie-extractor MCP workflow** (2025-11-14)

Reply with the number of the todo you'd like to work on.
```
