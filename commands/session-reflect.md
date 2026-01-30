---
description: Analyze session work and extract reusable knowledge for playbook (ACE pattern)
allowed-tools: [Read, Glob, Grep, Bash]
---

Invoke the session-reflect skill to analyze this session's work and extract reusable knowledge.

The skill will:
1. Review git log and completed work
2. Load existing playbook bullets
3. Identify which bullets were helpful or harmful
4. Extract new strategies, pitfalls, code patterns, and decisions
5. Output structured JSON for /playbook-update to process

After completion, run `/playbook-update` to apply the changes.
