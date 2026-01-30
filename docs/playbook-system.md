# Playbook System (ACE Pattern)

The playbook system accumulates reusable strategies and learnings across sessions using the ACE (Agentic Context Engineering) pattern.

## Overview

Playbooks store structured knowledge as "bullets" in JSON files. Each bullet tracks how often it has been helpful or harmful, enabling continuous improvement across sessions.

## Files

Each playbook directory contains four JSON files:

| File | Content | ID Format |
|------|---------|-----------|
| `strategies.json` | Approaches that work well | `strat-NNNNN` |
| `pitfalls.json` | Mistakes to avoid | `pitfall-NNNNN` |
| `code-patterns.json` | Reusable code snippets | `code-NNNNN` |
| `decisions.json` | Key decisions with rationale | `dec-NNNNN` |

## Bullet Format

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

## Hierarchical Playbooks

Playbooks can be organized hierarchically:

- **Root level** (`/.ai-agents/playbook/`) - Infrastructure, cross-project patterns
- **Project level** (`/project/.ai-agents/playbook/`) - Project-specific strategies

## Workflow

1. **Session start** - Load relevant playbook bullets for context
2. **During work** - Track which bullets are applied
3. **Session end** - Run `/session-reflect` to extract learnings
4. **Apply updates** - Run `/playbook-update` to update counters and add new bullets

## Commands

- `/session-reflect` - Analyze session, propose playbook updates
- `/playbook-update` - Apply updates via playbook-curator sub-agent

## Setup

Create the playbook directory structure:

```bash
mkdir -p .ai-agents/playbook
for file in strategies pitfalls code-patterns decisions; do
  [ -f ".ai-agents/playbook/$file.json" ] || \
    echo '{"_schema":{"version":"1.0"},"bullets":[]}' > ".ai-agents/playbook/$file.json"
done
```

## ACE Pattern Reference

The ACE (Agentic Context Engineering) pattern uses iterative reflection to build a knowledge base that improves agent performance over time. Key concepts:

- **Bullets** - Atomic units of knowledge with helpful/harmful counters
- **Reflection** - End-of-session analysis extracting new learnings
- **Curation** - Deduplication and counter management
- **Loading** - Session-start context injection of relevant bullets
