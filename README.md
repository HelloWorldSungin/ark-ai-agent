# ark-ai-agent

A modular marketplace of Claude Code plugins for planning, prompting, reflection, and productivity workflows. Install only what you need — each plugin is independent with its own commands, skills, and agents.

Forked from [TACHES](https://github.com/glittercowboy/taches-cc-resources) with added playbook system (ACE pattern) for cross-session knowledge accumulation.

## Plugin Catalog

| Plugin | What it does | Commands | Skills | Agents |
|--------|-------------|----------|--------|--------|
| **[consider](#consider)** | Mental model thinking frameworks | 12 | — | — |
| **[research](#research)** | Structured research workflows | 8 | — | — |
| **[reflection](#reflection)** | ACE playbook system | 2 | 2 | 1 |
| **[extensibility](#extensibility)** | Create/audit/heal Claude Code extensions | 9 | 6 | 3 |
| **[planning](#planning)** | Project planning + team orchestration | 4 | 1 | 2 |
| **[todos](#todos)** | Linear-integrated task management | 3 | — | — |
| **[tools](#tools)** | Docs, debugging, Ralph loops | 3 | 3 | — |
| **[prompting](#prompting)** | Prompt engineering for delegation | 3 | — | — |
| **[expertise](#expertise)** | Domain knowledge (iOS, macOS, n8n) | — | 3 | — |

## Installation

### Selective Install (Recommended)

From within Claude Code:

```
# Add the marketplace
/plugin marketplace add HelloWorldSungin/ark-ai-agent

# Install only plugins you need
/plugin install consider@ark-ai-agent
/plugin install planning@ark-ai-agent
/plugin install reflection@ark-ai-agent
```

Or from a local submodule:

```
/plugin marketplace add ./external/ark-ai-agent
/plugin install consider@ark-ai-agent
```

Use the interactive `/plugin` UI to browse all available plugins.

### Migration from v2.x

v3.0.0 replaces the monolithic `ark-ai-agent` plugin with 9 independent plugins. To migrate:

1. Remove the old plugin: `/plugin uninstall ark-ai-agent@ark-ai-agent`
2. Install the plugins you need (see catalog above)
3. Commands that were `ark-ai-agent:consider:first-principles` are now `consider:first-principles`

---

## consider

Apply mental frameworks to decisions and problems.

| Command | Description |
|---------|-------------|
| `first-principles` | Break down to fundamentals and rebuild |
| `inversion` | Solve backwards — what guarantees failure? |
| `pareto` | Apply 80/20 rule to focus on what matters |
| `second-order` | Think through consequences of consequences |
| `5-whys` | Drill to root cause |
| `occams-razor` | Find simplest explanation |
| `one-thing` | Identify highest-leverage action |
| `swot` | Map strengths, weaknesses, opportunities, threats |
| `eisenhower-matrix` | Prioritize by urgent/important |
| `10-10-10` | Evaluate across time horizons |
| `opportunity-cost` | Analyze what you give up |
| `via-negativa` | Improve by removing |

## research

Structured research workflows with web search and analysis.

| Command | Description |
|---------|-------------|
| `competitive` | Competitive analysis |
| `deep-dive` | Deep dive into a topic |
| `feasibility` | Feasibility study |
| `history` | Historical research |
| `landscape` | Landscape survey |
| `open-source` | Open source research |
| `options` | Options analysis |
| `technical` | Technical research |

## reflection

Accumulate reusable knowledge across sessions using the ACE (Agentic Context Engineering) pattern.

| Component | Name | Description |
|-----------|------|-------------|
| Command | `session-reflect` | Analyze session and extract learnings |
| Command | `playbook-update` | Apply reflection to playbook files |
| Skill | session-reflect | End-of-session knowledge extraction |
| Skill | playbook-update | Curator-managed playbook updates |
| Agent | playbook-curator | ACE-style playbook curation with dedup |

**Workflow:** End of session → `/session-reflect` → `/playbook-update` → knowledge persists to next session.

See [playbook system docs](./docs/playbook-system.md) for details.

## extensibility

The complete Claude Code extension lifecycle: create, audit, and heal.

| Component | Name | Description |
|-----------|------|-------------|
| Command | `create-agent-skill` | Create a new skill |
| Command | `create-meta-prompt` | Create staged workflow prompts |
| Command | `create-slash-command` | Create a new slash command |
| Command | `create-subagent` | Create a new subagent |
| Command | `create-hook` | Create a new hook |
| Command | `audit-skill` | Audit skill for best practices |
| Command | `audit-slash-command` | Audit command structure |
| Command | `audit-subagent` | Audit subagent configuration |
| Command | `heal-skill` | Fix skills based on execution issues |
| Skill | create-agent-skills | Build skills from descriptions |
| Skill | create-hooks | Build event-driven automation |
| Skill | create-mcp-servers | Build MCP servers (Python/TypeScript) |
| Skill | create-meta-prompts | Build staged workflow prompts |
| Skill | create-slash-commands | Build commands with proper structure |
| Skill | create-subagents | Build specialized Claude instances |
| Agent | skill-auditor | Reviews skills for best practices |
| Agent | slash-command-auditor | Reviews commands for structure |
| Agent | subagent-auditor | Reviews agent configurations |

## planning

Project planning and execution — solo and team-orchestrated, with Linear integration.

| Component | Name | Description |
|-----------|------|-------------|
| Command | `create-plan` | Create hierarchical project plans |
| Command | `run-plan` | Execute plans solo or auto-dispatch from Linear |
| Command | `plan-w-team` | Create team-orchestrated plans |
| Command | `build` | Execute team plans with builder/validator agents |
| Skill | create-plans | Full planning lifecycle (brief → roadmap → plan → execute) |
| Agent | team/builder | Focused execution agent |
| Agent | team/validator | Read-only verification agent |

**Cross-plugin:** Optionally uses `expertise` plugin for domain-specific planning, and `todos` plugin for `/whats-next` handoffs.

## todos

Capture tasks mid-conversation and resume with full context. Integrates with Linear.

| Command | Description |
|---------|-------------|
| `add-to-todos` | Capture tasks with full context |
| `check-todos` | Resume work on captured tasks |
| `whats-next` | Create handoff document for fresh context |

## tools

Standalone utility workflows.

| Component | Name | Description |
|-----------|------|-------------|
| Command | `docs-with-mermaid` | Generate technical docs with Mermaid diagrams |
| Command | `debug` | Apply expert debugging methodology |
| Command | `setup-ralph` | Set up Ralph Wiggum coding loop |
| Skill | docs-with-mermaid | Documentation with diagram generation |
| Skill | debug-like-expert | Systematic debugging with hypothesis testing |
| Skill | setup-ralph | Ralph autonomous coding loop setup |

## prompting

Meta-prompting: separate analysis from execution for higher quality output.

| Command | Description |
|---------|-------------|
| `create-prompt` | Generate optimized prompts with XML structure |
| `run-prompt` | Execute saved prompts in sub-agent contexts |
| `ask-me-questions` | Guided questioning for requirements gathering |

## expertise

Domain knowledge bases loaded by planning for framework-specific context. Each sub-skill is an exhaustive knowledge base (5k-10k+ lines).

| Sub-skill | Description |
|-----------|-------------|
| iphone-apps | iOS/SwiftUI development patterns |
| macos-apps | macOS app development patterns |
| n8n-automations | n8n workflow automation |

Create new domain expertise with the `extensibility` plugin's `/create-agent-skill` command.

---

## Recommended Workflow

1. **Install what you need:** Start with `planning` + `reflection` + `todos` for a complete project lifecycle
2. **Build projects:** `/create-plan` → `/run-plan` (solo) or `/plan-w-team` → `/build` (team)
3. **Accumulate knowledge:** `/session-reflect` → `/playbook-update` at end of session
4. **Extend Claude:** Use `extensibility` to create new skills, commands, agents, hooks, and MCP servers

---

**Based on:** [TACHES CC Resources](https://github.com/glittercowboy/taches-cc-resources) by Lex Christopherson
**Community Ports:** [OpenCode](https://github.com/stephenschoettler/taches-oc-prompts)
