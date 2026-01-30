---
name: whats-next
description: Analyze the current conversation and create a handoff document for continuing this work in a fresh context
allowed-tools:
  - Read
  - Write
  - Bash
  - WebSearch
  - WebFetch
  - mcp__plugin_linear_linear__create_issue
  - mcp__plugin_linear_linear__create_issue_label
  - mcp__plugin_linear_linear__list_teams
  - mcp__plugin_linear_linear__list_issue_labels
---

Create a comprehensive, detailed handoff document that captures all context from the current conversation. This allows continuing the work in a fresh context with complete precision.

## Instructions

**PRIORITY: Comprehensive detail and precision over brevity.** The goal is to enable someone (or a fresh Claude instance) to pick up exactly where you left off with zero information loss.

Adapt the level of detail to the task type (coding, research, analysis, writing, configuration, etc.) but maintain comprehensive coverage:

1. **Original Task**: Identify what was initially requested (not new scope or side tasks)

2. **Work Completed**: Document everything accomplished in detail
   - All artifacts created, modified, or analyzed (files, documents, research findings, etc.)
   - Specific changes made (code with line numbers, content written, data analyzed, etc.)
   - Actions taken (commands run, APIs called, searches performed, tools used, etc.)
   - Findings discovered (insights, patterns, answers, data points, etc.)
   - Decisions made and the reasoning behind them

3. **Work Remaining**: Specify exactly what still needs to be done
   - Break down remaining work into specific, actionable steps
   - Include precise locations, references, or targets (file paths, URLs, data sources, etc.)
   - Note dependencies, prerequisites, or ordering requirements
   - Specify validation or verification steps needed

4. **Attempted Approaches**: Capture everything tried, including failures
   - Approaches that didn't work and why they failed
   - Errors encountered, blockers hit, or limitations discovered
   - Dead ends to avoid repeating
   - Alternative approaches considered but not pursued

5. **Critical Context**: Preserve all essential knowledge
   - Key decisions and trade-offs considered
   - Constraints, requirements, or boundaries
   - Important discoveries, gotchas, edge cases, or non-obvious behaviors
   - Relevant environment, configuration, or setup details
   - Assumptions made that need validation
   - References to documentation, sources, or resources consulted

6. **Current State**: Document the exact current state
   - Status of deliverables (complete, in-progress, not started)
   - What's committed, saved, or finalized vs. what's temporary or draft
   - Any temporary changes, workarounds, or open questions
   - Current position in the workflow or process

Write to `whats-next.md` in the current working directory using the format below.

## Linear Integration

After writing `whats-next.md`, if there is remaining work (i.e., `<work_remaining>` is non-empty), create a Linear issue to track the handoff:

1. **Resolve Linear team** (same pattern as /add-to-todos):
   - Check if the project's CLAUDE.md specifies a `linear-team` value
   - If not, use `mcp__plugin_linear_linear__list_teams` to list available teams
   - If only one team exists, use it automatically
   - If multiple teams, ask user which team to use

2. **Ensure "handoff" label exists:**
   - Use `mcp__plugin_linear_linear__list_issue_labels` with the resolved team to check for a label named "handoff"
   - If it doesn't exist, create it with `mcp__plugin_linear_linear__create_issue_label` (name: "handoff", color: "#7B68EE")

3. **Create Linear issue** via `mcp__plugin_linear_linear__create_issue`:
   - **title**: "Resume: [1-sentence summary of original task]"
   - **team**: Resolved team
   - **state**: "Todo"
   - **labels**: ["handoff"]
   - **description**: Markdown formatted:
     ```
     **Original Task:** [original task summary]

     **Work Remaining:**
     [Bullet list of remaining items from work_remaining section]

     **Handoff File:** `[absolute or relative path to whats-next.md]`

     ---
     *Created via /whats-next*
     ```

4. **Append to handoff file:**
   - Add a `<linear_issue>` section at the end of `whats-next.md`:
     ```xml
     <linear_issue>
     [LINEAR_ID] — Resume: [title]
     </linear_issue>
     ```

5. **Confirm:** "Handoff saved to `whats-next.md` and tracked in Linear ([LINEAR_ID])."

If there is NO remaining work, skip Linear issue creation and just confirm the handoff file was written.

## Output Format

```xml
<original_task>
[The specific task that was initially requested - be precise about scope]
</original_task>

<work_completed>
[Comprehensive detail of everything accomplished:
- Artifacts created/modified/analyzed (with specific references)
- Specific changes, additions, or findings (with details and locations)
- Actions taken (commands, searches, API calls, tool usage, etc.)
- Key discoveries or insights
- Decisions made and reasoning
- Side tasks completed]
</work_completed>

<work_remaining>
[Detailed breakdown of what needs to be done:
- Specific tasks with precise locations or references
- Exact targets to create, modify, or analyze
- Dependencies and ordering
- Validation or verification steps needed]
</work_remaining>

<attempted_approaches>
[Everything tried, including failures:
- Approaches that didn't work and why
- Errors, blockers, or limitations encountered
- Dead ends to avoid
- Alternative approaches considered but not pursued]
</attempted_approaches>

<critical_context>
[All essential knowledge for continuing:
- Key decisions and trade-offs
- Constraints, requirements, or boundaries
- Important discoveries, gotcas, or edge cases
- Environment, configuration, or setup details
- Assumptions requiring validation
- References to documentation, sources, or resources]
</critical_context>

<current_state>
[Exact state of the work:
- Status of deliverables (complete/in-progress/not started)
- What's finalized vs. what's temporary or draft
- Temporary changes or workarounds in place
- Current position in workflow or process
- Any open questions or pending decisions]
</current_state>
```
