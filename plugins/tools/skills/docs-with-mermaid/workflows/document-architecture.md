<required_reading>
Before starting, read these references as needed:
- `references/documentation-types.md` - for architecture doc best practices
- `references/mermaid-syntax.md` - sections on C4 diagrams, flowcharts
- `references/diagram-selection.md` - if unsure which diagrams to include
</required_reading>

<objective>
Create comprehensive architecture documentation with layered diagrams showing system context, containers, and components.
</objective>

<process>
1. **Understand the system scope**
   - Identify all major components, services, and external dependencies
   - Determine the audience (team members, new developers, stakeholders)
   - If a codebase is provided, explore it to map the architecture

2. **Create the system context diagram (C4 Context)**
   - Show the system as a single box
   - Add all users/personas
   - Add all external systems it integrates with
   - Label all relationships with protocol/purpose

3. **Create the container diagram (C4 Container)**
   - Break the system into applications, services, data stores
   - Show technology choices (e.g., "React", "PostgreSQL")
   - Show communication patterns between containers

4. **Create component diagrams for complex containers** (if needed)
   - Only for containers with non-obvious internal structure
   - Show major classes/modules and their responsibilities

5. **Add supporting diagrams as needed**
   - **Sequence diagram** for the most important request flow
   - **State diagram** if there are lifecycle-dependent resources
   - **ER diagram** if the data model is central to understanding

6. **Write the documentation**
   - Use the template from `templates/architecture-doc.md`
   - Introduce each diagram with 2-3 sentences of context
   - Explain key design decisions inline
   - Highlight important relationships and data flows

7. **Validate**
   - Run through `templates/quality-checklist.md`
   - Ensure all Mermaid syntax is correct
   - Verify diagrams render at appropriate complexity
</process>

<success_criteria>
- Has C4 Context diagram showing system boundaries
- Has C4 Container diagram showing internal structure
- Key flows are documented with sequence diagrams
- Every diagram has explanatory prose
- Design decisions are documented
</success_criteria>
