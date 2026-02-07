# Best Practices by Documentation Type

## README Documentation

For README files, include:
1. **Architecture overview diagram** (flowchart or C4 Context)
2. **Key workflow diagram** showing main user/system flow
3. Keep diagrams simple - link to detailed docs for complexity

**Goal:** First-time visitors understand the project in 30 seconds.

## API Documentation

For API docs, include:
1. **Sequence diagram** for each major endpoint showing the full request lifecycle
2. **State diagram** if the API manages stateful resources
3. **ER diagram** for data models returned by the API

**Goal:** Developers can integrate without reading source code.

## Architecture Documentation

For architecture docs, include:
1. **C4 Context diagram** - system and its external dependencies
2. **C4 Container diagram** - applications and data stores
3. **C4 Component diagram** - internal structure of complex containers
4. **Deployment diagram** if infrastructure is complex

**Goal:** New team members understand the system in one sitting.

## Onboarding Documentation

For new developer onboarding:
1. **High-level flowchart** of the system
2. **Sequence diagram** of a typical request flow
3. **Class diagram** of core domain models
4. **Git graph** showing branching strategy

**Goal:** New developers are productive within their first week.

## Code Explanation Documentation

For explaining how code works:
1. **Class diagram** for object relationships
2. **Sequence diagram** for execution flow
3. **State diagram** for lifecycle management
4. **Flowchart** for algorithms and decision logic

**Goal:** Complex code is understandable without reading every line.

## Decision Documentation (ADRs)

For architecture decision records:
1. **Mindmap** for option exploration
2. **Quadrant chart** for trade-off analysis
3. **Flowchart** for decision process
4. **C4 diagrams** for before/after architecture comparison

**Goal:** Future team members understand WHY decisions were made.
