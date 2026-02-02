<required_reading>
Before starting, read these references as needed:
- `references/documentation-types.md` - for README best practices
- `references/diagram-selection.md` - for choosing overview diagrams
</required_reading>

<objective>
Create a project README with architecture overview diagrams and key workflow visualizations.
</objective>

<process>
1. **Understand the project**
   - Explore the codebase structure (package.json, main entry points, config files)
   - Identify the tech stack
   - Determine the project's purpose and primary use cases

2. **Create the architecture overview diagram**
   - Use a flowchart or C4 Context diagram
   - Show major components and their relationships
   - Keep it simple - link to detailed docs for complexity
   - Include external dependencies

3. **Create a key workflow diagram**
   - Pick the most important user or system flow
   - Sequence diagram for request flows
   - Flowchart for process/pipeline flows
   - Keep to 10-15 nodes maximum for readability

4. **Write the README**
   - Use the template from `templates/readme-structure.md`
   - Include: overview, architecture diagram, getting started, usage, workflow diagram
   - Keep diagrams simple - README should be scannable
   - Link to detailed documentation for complex topics

5. **Validate**
   - Run through `templates/quality-checklist.md`
   - Verify diagrams render on GitHub/GitLab
   - Check that README is useful for a first-time visitor
</process>

<success_criteria>
- Has architecture overview diagram
- Has at least one workflow/flow diagram
- Getting started section is complete
- README is scannable (not overwhelming)
- Diagrams are simple enough for the README context
</success_criteria>
