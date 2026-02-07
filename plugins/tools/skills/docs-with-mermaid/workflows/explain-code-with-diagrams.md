<required_reading>
Before starting, read these references as needed:
- `references/diagram-selection.md` - to choose the right diagram types
- `references/mermaid-syntax.md` - for syntax of chosen diagram types
</required_reading>

<objective>
Explain how code works using a combination of prose and Mermaid diagrams to make complex logic easy to understand.
</objective>

<process>
1. **Read and understand the code**
   - If a file path is provided, read the file(s)
   - Identify the main abstractions, classes, and functions
   - Trace the primary execution flow
   - Note any state management, external calls, or complex logic

2. **Choose diagram types based on what's in the code**
   - Classes/interfaces → **Class diagram**
   - Request handling / multi-service flow → **Sequence diagram**
   - State management / lifecycle → **State diagram**
   - Algorithm / decision logic → **Flowchart**
   - Data relationships → **ER diagram**
   - Module dependencies → **Flowchart with subgraphs**

3. **Create a high-level overview diagram first**
   - Show the main components and how they relate
   - Use a flowchart or class diagram depending on the code

4. **Create detailed diagrams for complex parts**
   - Sequence diagram for the main execution path
   - State diagram if there's lifecycle management
   - Flowchart for complex algorithms or decision trees

5. **Write the explanation**
   - Start with a 2-3 sentence overview of what the code does and why
   - Present the high-level diagram with explanation
   - Walk through the detailed flow, referencing diagrams
   - Highlight key design patterns or non-obvious behavior
   - Keep prose focused - let diagrams carry the visual weight

6. **Validate**
   - Verify diagrams match the actual code behavior
   - Ensure all important code paths are covered
   - Check that explanations add value beyond what diagrams show
</process>

<success_criteria>
- High-level overview diagram present
- Complex logic has supporting diagrams
- Prose complements (not duplicates) the diagrams
- Reader can understand the code without reading it
</success_criteria>
