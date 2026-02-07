---
name: playbook-update
description: Applies playbook updates via the playbook-curator sub-agent. Receives reflection output and delegates to curator for counter increments, new bullet creation, and deduplication. Use after running /session-reflect.
---

<objective>
Process playbook updates from session reflection output. Delegates to the playbook-curator sub-agent to increment helpful/harmful counters on existing bullets, add new bullets with unique IDs, and detect semantic duplicates. Commits changes to git.
</objective>

<quick_start>
Run after /session-reflect to apply playbook changes:
1. Receives reflection JSON from previous step
2. Delegates to playbook-curator sub-agent
3. Reviews curator output for conflicts
4. Commits playbook changes to git

Prerequisites: Run /session-reflect first to generate reflection output.
</quick_start>

<process>
<step_1>
<title>Load Reflection Output</title>
<description>Get the reflection output to process.</description>
<options>
<option>Accept JSON directly from /session-reflect output</option>
<option>Read from file if saved</option>
</options>
<validation>
<check>Verify JSON has required fields: session_id, bullets_used, new_bullets</check>
<check>Verify bullets_used.helpful and bullets_used.harmful are arrays</check>
<check>Verify new_bullets has valid category arrays</check>
</validation>
</step_1>

<step_2>
<title>Delegate to Playbook Curator</title>
<description>Use Task tool to delegate playbook updates to curator sub-agent.</description>
<delegation>
```
Task tool parameters:
- subagent_type: "playbook-curator"
- description: "Update playbook from reflection"
- prompt: |
    Process the following reflection output and update playbook files.

    Tasks:
    1. Increment helpful counters for bullets in helpful list
    2. Increment harmful counters for bullets in harmful list
    3. Update last_used_session for all used bullets
    4. Add new bullets with auto-generated IDs
    5. Check for semantic duplicates before adding
    6. Return summary of changes made

    Full reflection data:
    {reflection_json}
```
</delegation>
</step_2>

<step_3>
<title>Review Curator Output</title>
<description>Review what changes the curator made.</description>
<actions>
<action>If duplicates found, present to user for decision</action>
<action>If conflicts detected, pause for user resolution</action>
</actions>
</step_3>

<step_4>
<title>Commit Playbook Changes</title>
<description>Commit all playbook file changes to git.</description>
<git_commands>
```bash
git add .ai-agents/playbook/*.json
git commit -m "chore: Update playbook from session {session_id}"
```
</git_commands>
</step_4>

<step_5>
<title>Report Summary</title>
<description>Provide summary of playbook updates to user.</description>
<output_format>
```
Playbook Updated (Session {session_id})

Counters Updated:
- Helpful incremented: {list}
- Harmful incremented: {list}

New Bullets Added:
- Strategies: {count}
- Pitfalls: {count}
- Code Patterns: {count}
- Decisions: {count}

Commit: {commit_hash}
```
</output_format>
</step_5>
</process>

<success_criteria>
<criterion>Reflection output is valid and complete</criterion>
<criterion>Curator sub-agent processes all updates successfully</criterion>
<criterion>Helpful/harmful counters are incremented correctly</criterion>
<criterion>New bullets have unique, sequential IDs</criterion>
<criterion>Duplicate checking is performed</criterion>
<criterion>Changes are committed with descriptive message</criterion>
</success_criteria>
