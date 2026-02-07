# Mermaid Diagram Complete Syntax Reference

Mermaid is a text-based diagramming tool that renders from markdown-like syntax. It integrates natively with GitHub, GitLab, Notion, Obsidian, and most modern documentation platforms.

Every Mermaid diagram starts with a diagram type declaration inside a fenced code block with `mermaid` language.

---

## Flowcharts

Visualize processes, workflows, decision trees, and algorithms.

### Declaration and Direction

```mermaid
flowchart TD
    A --> B
```

**Direction Options:**
- `TD` or `TB` - Top to Bottom
- `BT` - Bottom to Top
- `LR` - Left to Right
- `RL` - Right to Left

### Node Shapes

```mermaid
flowchart LR
    A[Rectangle]
    B(Rounded Rectangle)
    C([Stadium/Pill])
    D[[Subroutine]]
    E[(Database/Cylinder)]
    F((Circle))
    G>Asymmetric/Flag]
    H{Diamond/Decision}
    I{{Hexagon}}
    J[/Parallelogram/]
    K[\Parallelogram Alt\]
    L[/Trapezoid\]
    M[\Trapezoid Alt/]
```

### Arrow/Edge Types

```mermaid
flowchart LR
    A --> B           %% Arrow
    B --- C           %% Line (no arrow)
    C -.-> D          %% Dotted arrow
    D ==> E           %% Thick arrow
    E --text--> F     %% Arrow with text
    F -->|text| G     %% Alternative text syntax
    G -.text.-> H     %% Dotted with text
    H ==text==> I     %% Thick with text
    I <--> J          %% Bidirectional
    J o--o K          %% Circle endpoints
    K x--x L          %% Cross endpoints
```

### Subgraphs

```mermaid
flowchart TB
    subgraph Frontend["Frontend Layer"]
        direction LR
        A[React App] --> B[Redux Store]
        B --> C[API Client]
    end

    subgraph Backend["Backend Services"]
        direction LR
        D[API Gateway] --> E[Auth Service]
        D --> F[User Service]
        D --> G[Data Service]
    end

    subgraph Database["Data Layer"]
        H[(PostgreSQL)]
        I[(Redis Cache)]
    end

    C --> D
    F --> H
    E --> I
```

### Styling

```mermaid
flowchart LR
    A[Start]:::startNode --> B{Decision}:::decisionNode
    B -->|Yes| C[Process]:::processNode
    B -->|No| D[End]:::endNode

    classDef startNode fill:#90EE90,stroke:#006400
    classDef decisionNode fill:#FFD700,stroke:#B8860B
    classDef processNode fill:#87CEEB,stroke:#4682B4
    classDef endNode fill:#FFA07A,stroke:#CD5C5C
```

---

## Sequence Diagrams

Show interactions between components/actors over time.

### Basic Syntax

```mermaid
sequenceDiagram
    participant C as Client
    participant S as Server
    participant D as Database

    C->>S: HTTP Request
    activate S
    S->>D: Query Data
    activate D
    D-->>S: Result Set
    deactivate D
    S-->>C: JSON Response
    deactivate S
```

### Message Types

```
A->>B     Solid line with arrowhead
A-->>B    Dotted line with arrowhead
A-)B      Solid line with open arrow
A--)B     Dotted line with open arrow
A-xB      Solid line with cross (async)
A--xB     Dotted line with cross
```

### Control Flow

```mermaid
sequenceDiagram
    participant User
    participant Auth
    participant API

    User->>Auth: Login Request

    alt Valid Credentials
        Auth->>API: Generate Token
        API-->>Auth: JWT Token
        Auth-->>User: Success + Token
    else Invalid Credentials
        Auth-->>User: Error: Invalid credentials
    end

    opt Remember Me
        Auth->>Auth: Store Session
    end

    loop Health Check
        API->>API: Ping every 30s
    end

    par Parallel Operations
        User->>API: Fetch Profile
    and
        User->>API: Fetch Settings
    end
```

### Notes and Activation

```mermaid
sequenceDiagram
    participant A as Service A
    participant B as Service B

    Note over A: Initialize connection
    A->>+B: Request
    Note over A,B: Processing...
    B-->>-A: Response
    Note right of B: Log response
```

### Auto-numbering

Add `autonumber` after `sequenceDiagram` to auto-number messages.

---

## Class Diagrams

Show object-oriented structure with classes, attributes, methods, and relationships.

### Class Definition

```mermaid
classDiagram
    class Animal {
        +String name
        +int age
        -String species
        #boolean isAlive
        +makeSound() void
        +move(distance: int) void
        -digest() void
        #reproduce()* Animal
    }
```

**Visibility:** `+` Public, `-` Private, `#` Protected, `~` Package/Internal
**Classifiers:** `*` Abstract, `$` Static

### Relationships

```
Animal <|-- Dog           %% Inheritance (is-a)
Flyable <|.. Bird         %% Implementation (implements)
Car *-- Engine            %% Composition (lifecycle dependent)
University o-- Student    %% Aggregation (independent lifecycle)
Teacher --> Student       %% Association (uses)
Client ..> Service        %% Dependency (depends on)
```

### Cardinality

```mermaid
classDiagram
    User "1" --> "*" Order : places
    Order "1" *-- "*" OrderItem : contains
    OrderItem "*" --> "1" Product : references
```

---

## State Diagrams

Model state machines and transitions.

### Basic States

```mermaid
stateDiagram-v2
    [*] --> Idle
    Idle --> Processing : start
    Processing --> Success : complete
    Processing --> Error : fail
    Success --> [*]
    Error --> Idle : retry
    Error --> [*] : abort
```

### Composite States

```mermaid
stateDiagram-v2
    [*] --> Active

    state Active {
        [*] --> Running
        Running --> Paused : pause
        Paused --> Running : resume
        Running --> [*] : stop
    }

    Active --> Inactive : deactivate
    Inactive --> Active : activate
```

### Choice Points and Forks

```mermaid
stateDiagram-v2
    state check_result <<choice>>
    [*] --> CheckInput
    CheckInput --> check_result
    check_result --> Valid : is valid
    check_result --> Invalid : is invalid

    state fork_state <<fork>>
    Valid --> fork_state
    fork_state --> Task1
    fork_state --> Task2

    state join_state <<join>>
    Task1 --> join_state
    Task2 --> join_state
    join_state --> Complete
```

---

## Entity Relationship Diagrams

Model database schemas and entity relationships.

### Basic Syntax

```mermaid
erDiagram
    CUSTOMER ||--o{ ORDER : places
    ORDER ||--|{ ORDER_ITEM : contains
    PRODUCT ||--o{ ORDER_ITEM : "ordered in"
```

### Cardinality (Crow's Foot)

```
||--||  One-to-One (exactly one)
||--o|  One-to-Zero-or-One
||--|{  One-to-One-or-More
||--o{  One-to-Zero-or-More
}|--|{  One-or-More to One-or-More
}o--o{  Zero-or-More to Zero-or-More
```

**Symbols:** `|` Exactly one, `o` Zero or one, `{` One or more, `}` Zero or more

### Entities with Attributes

```mermaid
erDiagram
    USER {
        int id PK
        string email UK
        string password_hash
        datetime created_at
    }

    POST {
        int id PK
        int user_id FK
        string title
        text content
    }

    USER ||--o{ POST : writes
```

**Attribute Keys:** `PK` Primary Key, `FK` Foreign Key, `UK` Unique Key

### Identifying vs Non-Identifying

```
ORDER ||--|{ ORDER_LINE : contains     %% Identifying (solid) - child needs parent
PERSON }|..|{ CAR : drives            %% Non-identifying (dashed) - independent
```

---

## C4 Architecture Diagrams

Model software architecture at different abstraction levels.

### System Context (C4Context)

```mermaid
C4Context
    title System Context Diagram

    Person(customer, "Customer", "A user who purchases products")
    System(ecommerce, "E-Commerce Platform", "Allows browsing and purchasing")
    System_Ext(payment, "Payment Gateway", "Processes payments")
    System_Ext(email, "Email Service", "Sends emails")

    Rel(customer, ecommerce, "Uses", "HTTPS")
    Rel(ecommerce, payment, "Processes payments", "API")
    Rel(ecommerce, email, "Sends emails", "SMTP")
```

### Container Diagram (C4Container)

```mermaid
C4Container
    title Container Diagram

    Person(customer, "Customer")

    System_Boundary(ecommerce, "E-Commerce Platform") {
        Container(web, "Web App", "React", "Serves SPA")
        Container(api, "API Gateway", "Node.js", "Routes requests")
        Container(catalog, "Catalog Service", "Python", "Product catalog")
        ContainerDb(db, "Database", "PostgreSQL", "Application data")
        ContainerDb(cache, "Cache", "Redis", "Frequent queries")
    }

    Rel(customer, web, "Uses", "HTTPS")
    Rel(web, api, "Calls", "JSON/HTTPS")
    Rel(api, catalog, "Reads", "gRPC")
    Rel(catalog, db, "Reads/Writes", "SQL")
    Rel(catalog, cache, "Reads/Writes")
```

### Component Diagram (C4Component)

```mermaid
C4Component
    title Component Diagram

    Container_Boundary(orders, "Order Service") {
        Component(controller, "Controller", "Spring MVC", "HTTP requests")
        Component(service, "Service", "Spring Bean", "Business logic")
        Component(repo, "Repository", "JPA", "Data access")
        Component(events, "Publisher", "Kafka Client", "Domain events")
    }

    ContainerDb(db, "Database", "PostgreSQL")

    Rel(controller, service, "Uses")
    Rel(service, repo, "Uses")
    Rel(repo, db, "Reads/Writes")
```

---

## User Journey Maps

Visualize user experiences across touchpoints.

```mermaid
journey
    title User Onboarding
    section Sign Up
        Visit site: 5: User
        Create account: 4: User
        Verify email: 3: User
    section First Use
        Complete tutorial: 4: User
        Create first project: 5: User
    section Retention
        Invite team: 3: User
        Daily usage: 4: User
```

**Score Scale:** 1 (worst) to 5 (best)

---

## Gantt Charts

Project schedules and timelines.

```mermaid
gantt
    title Project Timeline
    dateFormat YYYY-MM-DD

    section Phase 1
        Requirements    :done, a1, 2024-01-01, 14d
        Design          :active, a2, after a1, 10d

    section Phase 2
        Development     :b1, after a2, 30d
        Testing         :b2, after b1, 14d
```

**Task markers:** `done`, `active`, `crit` (critical), `milestone`

---

## Other Diagram Types

### Pie Charts

```mermaid
pie showData
    title Distribution
    "Category A" : 35
    "Category B" : 25
    "Category C" : 20
    "Other" : 20
```

### Mindmaps

```mermaid
mindmap
    root((Topic))
        Branch A
            Leaf 1
            Leaf 2
        Branch B
            Leaf 3
```

### Timeline

```mermaid
timeline
    title Milestones
    section Q1
        January : Kickoff
        March : MVP
    section Q2
        June : Launch
```

### Git Graphs

```mermaid
gitGraph
    commit id: "Initial"
    branch develop
    commit id: "Feature"
    checkout main
    merge develop id: "Release" tag: "v1.0"
```

### Quadrant Charts

```mermaid
quadrantChart
    title Priority Matrix
    x-axis Low Effort --> High Effort
    y-axis Low Impact --> High Impact
    quadrant-1 Do First
    quadrant-2 Plan
    quadrant-3 Drop
    quadrant-4 Quick Wins
    Feature A: [0.3, 0.8]
    Feature B: [0.7, 0.9]
```

### Block Diagrams

```mermaid
block-beta
    columns 3
    A["Frontend"]:1 B["API"]:1 C["Services"]:1
    D[("Database")]:1 E[("Cache")]:1 F[("Queue")]:1
    A --> B
    B --> C
    C --> D
```

---

## Troubleshooting

- **Reserved words:** Avoid `end` as node ID in lowercase. Use `End` or wrap in quotes.
- **Special characters:** Wrap text with parentheses/brackets in quotes: `A["Node with (parens)"]`
- **Long labels:** Use markdown strings: `` A["`**Bold** text`"] ``

*This reference is part of the docs-with-mermaid skill.*
