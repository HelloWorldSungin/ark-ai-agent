---
name: docs-with-mermaid
description: Create excellent technical documentation with Mermaid diagrams. Use when documenting code architecture, API flows, database schemas, state machines, system design, or any technical concept that benefits from visual diagrams. Also use when asked to explain code, create documentation, write README files, or document how systems work.
argument-hint: [topic or file to document]
---

<objective>
Create technical documentation with strategically placed Mermaid diagrams. Provides workflows for documenting architecture, APIs, and code with appropriate diagram selection, syntax guidance, and quality validation. Use when documentation would benefit from visual explanations of flows, relationships, and structure.
</objective>

<quick_start>
**"A diagram is worth a thousand lines of code."**

Good technical documentation:
1. **Explains the WHY** before the WHAT and HOW
2. **Uses visuals strategically** - diagrams should clarify, not decorate
3. **Layers information** - overview first, then details
4. **Stays current** - diagrams as code can be versioned and updated

**Diagram selection quick reference:**

| Documenting... | Use This Diagram |
|----------------|------------------|
| Process flow, algorithms, decision logic | **Flowchart** |
| API calls, service interactions, protocols | **Sequence Diagram** |
| Object-oriented design, class relationships | **Class Diagram** |
| Lifecycle, state machines, workflows | **State Diagram** |
| Database schema, data models | **ER Diagram** |
| System architecture (high level) | **C4 Context Diagram** |
| Application architecture (containers) | **C4 Container Diagram** |
| Component internals | **C4 Component Diagram** |
| User experience flows | **User Journey** |
| Project timelines | **Gantt Chart** |
| Prioritization matrices | **Quadrant Chart** |
| Hierarchical concepts | **Mindmap** |
| Historical events | **Timeline** |
| Git workflows | **Git Graph** |
| Proportions/percentages | **Pie Chart** |

**Diagram creation guidelines:**
- Start simple, add complexity gradually
- Use `PascalCase` for services/components, `camelCase` for actions
- Group related elements with subgraphs
- Use `LR` for timelines/sequential, `TB` for hierarchies
- Add notes in sequence diagrams for non-obvious behavior
- Apply color purposefully (boundaries, status, priority)
</quick_start>

<intake>
What would you like to document?

1. **Architecture** - System design, component diagrams, deployment
2. **API** - Endpoint flows, request lifecycles, data models
3. **Code explanation** - Explain how code works with diagrams
4. **README** - Project README with architecture overview
5. **Other** - Custom documentation with appropriate diagrams

If you provided a specific topic or file path, I'll determine the best approach automatically.

**Wait for response before proceeding.**
</intake>

<routing>
| Response | Workflow |
|----------|----------|
| 1, "architecture", "system", "design", "deploy" | workflows/document-architecture.md |
| 2, "api", "endpoint", "request", "flow" | workflows/document-api.md |
| 3, "explain", "code", "how does", "understand" | workflows/explain-code-with-diagrams.md |
| 4, "readme", "README" | workflows/create-readme.md |
| 5, "other", custom topic | workflows/document-architecture.md (default) |

**Intent-based routing (if user provides clear intent without selecting menu):**
- File path provided → workflows/explain-code-with-diagrams.md
- "document the API" → workflows/document-api.md
- "create a README" → workflows/create-readme.md
- System/architecture terms → workflows/document-architecture.md

**If unsure which diagram types to use** → Read references/diagram-selection.md first

**After reading the workflow, follow it exactly.**
</routing>

<reference_index>
All in `references/`:

| Reference | When to Read | Size |
|-----------|-------------|------|
| mermaid-syntax.md | When creating any diagram - full syntax reference | ~900 lines |
| common-patterns.md | When looking for diagram examples by use case | ~850 lines |
| diagram-best-practices.md | When reviewing diagram quality | ~200 lines |
| documentation-types.md | When deciding what to include by doc type | ~150 lines |
| diagram-selection.md | When unsure which diagram type to use | ~60 lines |

**Progressive disclosure:** Only read references when the workflow instructs you to, or when you need syntax help for a specific diagram type.
</reference_index>

<templates_index>
All in `templates/`:

| Template | Purpose |
|----------|---------|
| documentation-structure.md | General documentation template |
| architecture-doc.md | Architecture documentation template |
| api-doc.md | API documentation template |
| readme-structure.md | README template with diagram placeholders |
| quality-checklist.md | Pre-publish validation checklist |
</templates_index>

<success_criteria>
Documentation is complete when:
- Every major concept has an appropriate diagram
- Diagrams render correctly in target platform
- Prose introduces and explains each diagram
- Level of detail matches the audience
- Quality checklist in templates/quality-checklist.md passes
</success_criteria>
