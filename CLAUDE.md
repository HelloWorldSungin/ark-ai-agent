# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

ark-ai-agent is a Claude Code plugin providing skills, slash commands, subagents, and a playbook system for agentic development workflows. It's a prompt engineering system — no build, test, or lint steps. Validation is done through auditor agents (`/audit-skill`, `/audit-slash-command`, `/audit-subagent`).

## Installation

```bash
# Plugin install (recommended)
/plugin marketplace add HelloWorldSungin/ark-ai-agent
/plugin install ark-ai-agent@ark-ai-agent
```

## Architecture

### Component Types

**Slash Commands** (`commands/*.md`) — YAML frontmatter + markdown prompt. Invoked as `/<name>`. Categories: meta-prompting, todo management, context handoff, create/audit extensions, thinking models (`consider/`), research workflows (`research/`).

**Skills** (`skills/*/SKILL.md`) — Autonomous workflows using the router pattern: SKILL.md (intake + routing, <500 lines) → `workflows/` (procedures) → `references/` (domain knowledge) → `templates/` (output structures). Progressive disclosure keeps context efficient.

**Subagents** (`agents/*.md`) — Specialized validation/curation agents: skill-auditor, slash-command-auditor, subagent-auditor, playbook-curator. **Team agents** (`agents/team/*.md`) — builder (focused execution) and validator (read-only verification) for team-orchestrated plans.

**Domain Expertise** (`skills/expertise/`) — Large knowledge bases (5k-10k+ lines) loaded by create-plans for framework-specific context (macOS, iOS, n8n).

**Plugin Config** (`.claude-plugin/`) — `plugin.json` (metadata) and `marketplace.json` (listing).

### Key Systems

**Playbook (ACE Pattern)** — Cross-session knowledge in JSON files at `.ai-agents/playbook/`. Each bullet has `id`, `helpful`/`harmful` counters, `content`, `context`. Flow: session work → `/session-reflect` (extract learnings) → `/playbook-update` (curator agent applies updates with deduplication).

**Meta-Prompting** — Separates analysis (main context generates rigorous prompt) from execution (fresh sub-agent runs generated prompt). Ensures context quality.

**Planning** — `/create-plan` produces PLAN.md files that ARE the prompt (not docs). `/run-plan` executes them solo. Scope rule: 2-3 tasks per plan, 50% context max (Claude degrades at ~40-50% context).

**Team Orchestration** — `/plan-w-team` generates plans with Team Members and Step by Step Tasks sections (task IDs, dependencies, parallel flags, agent assignments). `/build` executes them by creating TaskCreate entries, dispatching builder agents via Task tool, and running validators. Builder/Validator separation ensures independent verification. `/run-plan` remains for solo execution (backward compatible).

**Linear Integration** (v2.1.0) — `/add-to-todos`, `/check-todos`, `/create-plan`, `/whats-next`, `/run-plan` integrate with Linear. Team resolution checks CLAUDE.md for `linear-team`, then mcporter CLI.

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
