---
name: session-reflect
description: Analyzes completed session to extract reusable knowledge for the playbook system. Identifies strategies that helped, mistakes to avoid, reusable code patterns, and key decisions. Outputs structured JSON for the playbook-curator sub-agent to process.
allowed-tools:
  - Read
  - Bash
  - Glob
  - Grep
  - mcp__plugin_linear_linear__create_issue
  - mcp__plugin_linear_linear__create_issue_label
  - mcp__plugin_linear_linear__list_teams
  - mcp__plugin_linear_linear__list_issue_labels
---

<objective>
Analyze the current session's work to extract reusable knowledge for the playbook system (ACE pattern). Produces structured output for the playbook-curator sub-agent to apply.
</objective>

<quick_start>
Run at the end of a session before creating a handoff:
1. Reviews work completed in the session
2. Identifies what strategies were helpful vs harmful
3. Extracts new learnings for the playbook
4. Outputs structured JSON for playbook-curator

This skill does NOT modify playbook files directly. It outputs proposals for review.
</quick_start>

<process>
<step_1>
<title>Gather Session Context</title>
<description>Understand what was accomplished this session.</description>
<actions>
<action>Review git log for recent commits in this session</action>
<action>Review any task lists or progress tracking</action>
<action>Identify major accomplishments and issues encountered</action>
</actions>
</step_1>

<step_2>
<title>Load Current Playbook</title>
<description>Read existing playbook files to avoid duplicates and find bullets to update.</description>
<actions>
<action>Find playbook directory: look for .ai-agents/playbook/ in project root or parent directories</action>
<action>Read all JSON files: strategies.json, pitfalls.json, code-patterns.json, decisions.json</action>
<action>If hierarchical (root + project levels exist), load both levels</action>
</actions>
</step_2>

<step_3>
<title>Identify Helpful Bullets Used</title>
<description>Review which existing playbook bullets were applied during this session.</description>
<questions>
<question>Which strategies from the playbook were explicitly followed?</question>
<question>Which pitfalls were successfully avoided?</question>
<question>Which code patterns were reused?</question>
<question>Which past decisions guided current work?</question>
</questions>
<output>List of bullet IDs that were helpful this session</output>
</step_3>

<step_4>
<title>Identify Harmful or Outdated Bullets</title>
<description>Identify any playbook bullets that caused issues or are now outdated.</description>
<questions>
<question>Did any strategy recommendations lead to problems?</question>
<question>Were any pitfall warnings inaccurate or misleading?</question>
<question>Did any code patterns fail or need modification?</question>
<question>Did any past decisions prove incorrect in hindsight?</question>
</questions>
<output>List of bullet IDs that were harmful or need revision</output>
</step_4>

<step_5>
<title>Extract New Learnings</title>
<description>Identify new knowledge gained this session.</description>
<categories>
<category name="strategies">
<description>New approaches that worked well and should be reused</description>
<criteria>Must be generalizable beyond this specific task</criteria>
<criteria>Must be actionable guidance, not just observations</criteria>
</category>
<category name="pitfalls">
<description>Mistakes encountered that should be avoided in future</description>
<criteria>Must describe both the mistake AND how to avoid it</criteria>
<criteria>Must be a pattern that could recur, not a one-off error</criteria>
</category>
<category name="code_patterns">
<description>Reusable code snippets or patterns</description>
<criteria>Must be generic enough to apply to multiple situations</criteria>
<criteria>Should include context for when to use the pattern</criteria>
</category>
<category name="decisions">
<description>Significant decisions made with rationale</description>
<criteria>Must be architectural or strategic, not trivial</criteria>
<criteria>Must include rationale that helps future sessions understand why</criteria>
</category>
</categories>
</step_5>

<step_6>
<title>Generate Reflection Output</title>
<description>Produce structured JSON output for the playbook-curator.</description>
<output_format>
```json
{
  "session_id": "NNN",
  "timestamp": "ISO-8601",
  "bullets_used": {
    "helpful": ["strat-00007"],
    "harmful": [],
    "neutral": []
  },
  "new_bullets": {
    "strategies": [
      {
        "content": "Description of strategy",
        "context": "category",
        "rationale": "Why this is valuable"
      }
    ],
    "pitfalls": [
      {
        "content": "Description of pitfall and how to avoid",
        "context": "category",
        "rationale": "Why this matters"
      }
    ],
    "code_patterns": [
      {
        "content": "Description of pattern",
        "snippet": "code here",
        "context": "category",
        "rationale": "When to use this"
      }
    ],
    "decisions": [
      {
        "content": "Decision and rationale",
        "context": "category",
        "expected_outcome": "What we expect to happen"
      }
    ]
  },
  "summary": "Brief summary of session learnings"
}
```

Iterative refinement: If the initial extraction seems incomplete or low-quality, review your own output and refine up to 3 iterations before finalizing.
</output_format>
</step_6>

<step_7>
<title>Present Reflection for Review</title>
<description>Present the reflection output to the user for review before curator processing.</description>
<actions>
<action>Display the structured JSON output</action>
<action>Highlight any harmful bullets that need attention</action>
<action>Summarize count of new bullets proposed</action>
<action>Ask user if they want to proceed with /playbook-update</action>
</actions>
</step_7>

<step_8>
<title>Offer to Create Linear Issues for Discovered Work</title>
<description>If the reflection identified actionable improvements, bugs, or tech debt that require code changes, offer to create Linear issues.</description>
<actions>
<action>Review the new_bullets output — filter for items that represent actionable code work (not just knowledge). Look for pitfalls that need fixing, strategies that imply missing features, or decisions that require implementation.</action>
<action>If actionable items exist (at least 1), present them and ask: "Create Linear issues for [N] discovered items? (y/n)"</action>
<action>If user declines or no actionable items: skip to completion</action>
<action>If user accepts:
  1. Resolve Linear team (same pattern as /add-to-todos): check CLAUDE.md → list_teams → auto-select or ask
  2. Ensure "discovered" label exists: check via list_issue_labels, create with create_issue_label if missing (name: "discovered", color: "#FF6B35")
  3. For each actionable item, create a Linear issue via create_issue:
     - **title**: Action verb + component (3-8 words)
     - **team**: Resolved team
     - **state**: "Backlog"
     - **labels**: ["discovered"]
     - **description**: Markdown formatted:
       ```
       **Discovery:** [What was found]

       **Context:** [From which playbook category — strategy/pitfall/pattern/decision]

       **Rationale:** [Why this needs action]

       ---
       *Discovered during /session-reflect (Session [N])*
       ```
  4. Report created issues: "Created [N] Linear issues: [LINEAR_ID_1], [LINEAR_ID_2], ..."
</action>
</actions>
</step_8>
</process>

<playbook_format>
Playbooks use JSON files with this bullet format:

```json
{
  "id": "strat-00001",
  "helpful": 5,
  "harmful": 0,
  "content": "Strategy description",
  "context": "category",
  "created_session": 1,
  "last_used_session": 10
}
```

ID prefixes by type:
- strategies: strat-NNNNN
- pitfalls: pitfall-NNNNN
- code-patterns: code-NNNNN
- decisions: dec-NNNNN

Files: strategies.json, pitfalls.json, code-patterns.json, decisions.json
Each file has: `{"_schema": {"version": "1.0"}, "bullets": [...]}`
</playbook_format>

<success_criteria>
<criterion>Session context is fully loaded and understood</criterion>
<criterion>Existing playbook bullets are checked for relevance</criterion>
<criterion>Helpful bullets are identified with specific usage context</criterion>
<criterion>Harmful or outdated bullets are flagged for review</criterion>
<criterion>New learnings are extracted and categorized correctly</criterion>
<criterion>Output JSON is valid and complete</criterion>
<criterion>User has opportunity to review before curator processes</criterion>
</success_criteria>

<integration_notes>
<note>This skill outputs JSON but does NOT modify playbook files</note>
<note>After reflection, run /playbook-update to apply changes via curator</note>
<note>Works with hierarchical playbooks (root + project) or single-level</note>
</integration_notes>
