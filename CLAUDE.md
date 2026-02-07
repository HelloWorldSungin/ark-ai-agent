# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

ark-ai-agent is a modular Claude Code plugin marketplace providing skills, slash commands, subagents, and a playbook system for agentic development workflows. It's a prompt engineering system — no build, test, or lint steps. Validation is done through auditor agents (`/audit-skill`, `/audit-slash-command`, `/audit-subagent`).

## Installation

```bash
# Add marketplace
/plugin marketplace add HelloWorldSungin/ark-ai-agent

# Install individual plugins (pick what you need)
/plugin install consider@ark-ai-agent
/plugin install planning@ark-ai-agent
/plugin install reflection@ark-ai-agent
```

## Architecture

### Plugin Structure (v3.0.0)

The repository is organized as 9 independent plugins under `plugins/`. Users install selectively to minimize context footprint.

```
plugins/
├── consider/        # 12 thinking model commands
├── research/        # 8 research workflow commands
├── reflection/      # ACE playbook system (cmds + skills + agent)
├── extensibility/   # Create/audit/heal extensions (cmds + skills + agents)
├── planning/        # Solo + team plan execution (cmds + skill + team agents)
├── todos/           # Linear-integrated task management (cmds)
├── tools/           # Utilities: docs, debug, Ralph (cmds + skills)
├── prompting/       # Prompt engineering (cmds)
└── expertise/       # Domain knowledge: iOS, macOS, n8n (skills)
```

Each plugin has:
```
plugins/<name>/
├── .claude-plugin/plugin.json   # Plugin metadata
├── commands/                    # Slash commands (*.md)
├── skills/                      # Skills (optional)
└── agents/                      # Subagents (optional)
```

### Plugin Catalog

| Plugin | Commands | Skills | Agents | Category |
|--------|----------|--------|--------|----------|
| **consider** | 12 (thinking models) | 0 | 0 | productivity |
| **research** | 8 (research workflows) | 0 | 0 | productivity |
| **reflection** | 2 (session-reflect, playbook-update) | 2 | 1 (playbook-curator) | productivity |
| **extensibility** | 9 (create-*, audit-*, heal-skill) | 6 (create-*) | 3 (auditors) | development |
| **planning** | 4 (create-plan, run-plan, plan-w-team, build) | 1 (create-plans) | 2 (team/) | productivity |
| **todos** | 3 (add-to-todos, check-todos, whats-next) | 0 | 0 | productivity |
| **tools** | 3 (docs-with-mermaid, debug, setup-ralph) | 3 | 0 | development |
| **prompting** | 3 (create-prompt, run-prompt, ask-me-questions) | 0 | 0 | productivity |
| **expertise** | 0 | 1 (3 domain sub-skills: iOS, macOS, n8n) | 0 | development |

### Cross-Plugin Dependencies

All hard dependencies (Skill tool calls, agent references) are within the same plugin. Two soft references:
- `planning` → `todos`: run-plan.md references `/whats-next` (advisory)
- `planning` → `expertise`: domain-expertise.md references expertise skills (optional companion)

### Component Types

**Slash Commands** (`plugins/<name>/commands/*.md`) — YAML frontmatter + markdown prompt. Invoked as `<plugin>:<name>` (e.g., `consider:first-principles`).

**Skills** (`plugins/<name>/skills/*/SKILL.md`) — Autonomous workflows using the router pattern: SKILL.md (intake + routing, <500 lines) → `workflows/` (procedures) → `references/` (domain knowledge) → `templates/` (output structures). Progressive disclosure keeps context efficient.

**Subagents** (`plugins/<name>/agents/*.md`) — Specialized validation/curation agents. **Team agents** (`plugins/planning/agents/team/*.md`) — builder (focused execution) and validator (read-only verification) for team-orchestrated plans.

**Plugin Config** — Root `.claude-plugin/marketplace.json` (9 plugin entries) + per-plugin `.claude-plugin/plugin.json` (metadata).

### Key Systems

**Playbook (ACE Pattern)** — Cross-session knowledge in JSON files at `.ai-agents/playbook/`. Each bullet has `id`, `helpful`/`harmful` counters, `content`, `context`. Flow: session work → `/session-reflect` (extract learnings) → `/playbook-update` (curator agent applies updates with deduplication).

**Meta-Prompting** — Separates analysis (main context generates rigorous prompt) from execution (fresh sub-agent runs generated prompt). Ensures context quality.

**Planning** — `/create-plan` produces PLAN.md files that ARE the prompt (not docs). `/run-plan` executes them solo. Scope rule: 2-3 tasks per plan, 50% context max (Claude degrades at ~40-50% context).

**Team Orchestration** — `/plan-w-team` generates plans with Team Members and Step by Step Tasks sections (task IDs, dependencies, parallel flags, agent assignments). `/build` executes them by creating TaskCreate entries, dispatching builder agents via Task tool, and running validators. Builder/Validator separation ensures independent verification. `/run-plan` remains for solo execution (backward compatible).

**Linear Integration** — `/add-to-todos`, `/check-todos`, `/create-plan`, `/whats-next`, `/run-plan` integrate with Linear. Team resolution checks CLAUDE.md for `linear-team`, then MCP.

### YAML Frontmatter Format

Commands and agents use:
```yaml
---
description: What it does
argument-hint: <optional-args>
allowed-tools: [Read, Write, Bash, etc.]
---
```

### Conventions

- Plans target solo developer + Claude, not enterprise teams
- Checkpoints are only for: human-verify (visual), decision (architectural), human-action (no CLI exists)
- Deviation rules in plans: auto-fix bugs/blockers, ask on architecture, log enhancements to ISSUES.md
- Playbook bullets use IDs like `strat-NNNNN`, `pitfall-NNNNN`, `code-NNNNN`, `dec-NNNNN`
- Hierarchical playbooks: root level (infrastructure) + project level (project-specific)
