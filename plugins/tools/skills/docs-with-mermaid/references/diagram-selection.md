<objective>
Help decide which diagram types to use when the documentation need isn't clearly categorized.
</objective>

<process>
1. **Identify what you're trying to communicate**
   - How components relate → structural diagrams
   - How things happen over time → behavioral diagrams
   - How data is organized → data diagrams

2. **Use this decision tree:**

   **Is it about structure?**
   - System boundaries → C4 Context
   - Internal applications/services → C4 Container
   - Classes/interfaces → Class Diagram
   - Module hierarchy → Mindmap or Flowchart with subgraphs

   **Is it about behavior?**
   - Multi-party interaction → Sequence Diagram
   - Lifecycle/states → State Diagram
   - Algorithm/process → Flowchart
   - User experience → User Journey

   **Is it about data?**
   - Database schema → ER Diagram
   - Proportions → Pie Chart
   - Prioritization → Quadrant Chart
   - Timeline → Gantt Chart or Timeline

   **Is it about development?**
   - Branching strategy → Git Graph
   - Project schedule → Gantt Chart
   - Decision mapping → Mindmap

3. **Combine diagrams for comprehensive documentation**
   - Architecture docs: C4 Context + C4 Container + Sequence
   - API docs: Sequence + ER + Flowchart (error handling)
   - README: Flowchart (overview) + Sequence (key flow)
   - Onboarding: Flowchart + Sequence + Class + Git Graph

4. **When in doubt:**
   - Start with a flowchart - it's the most versatile
   - Add a sequence diagram if there are multi-party interactions
   - Add an ER diagram if data relationships matter
</process>
