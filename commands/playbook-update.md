---
description: Apply playbook updates from session reflection via curator sub-agent
argument-hint: [reflection-json]
allowed-tools: [Read, Write, Edit, Bash, Glob, Task]
---

Invoke the playbook-update skill to apply reflection output to playbook files.

Prerequisites: Run `/session-reflect` first to generate the reflection data.

The skill will:
1. Load reflection output (from argument or previous step)
2. Delegate to playbook-curator sub-agent
3. Increment helpful/harmful counters on existing bullets
4. Add new bullets with unique IDs
5. Commit changes to git
