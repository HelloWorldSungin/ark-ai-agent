# [System Name] Architecture

## System Context

[What is this system? Who uses it? What external systems does it interact with?]

```mermaid
C4Context
    title System Context - [System Name]

    Person(user, "User", "Description")
    System(system, "System Name", "What it does")
    System_Ext(ext1, "External System", "What it provides")

    Rel(user, system, "Uses", "HTTPS")
    Rel(system, ext1, "Calls", "API")
```

## Container Architecture

[What applications and data stores make up the system?]

```mermaid
C4Container
    title Container Diagram - [System Name]

    Person(user, "User")

    System_Boundary(system, "System Name") {
        Container(web, "Web App", "Technology", "Description")
        Container(api, "API", "Technology", "Description")
        ContainerDb(db, "Database", "Technology", "Description")
    }

    Rel(user, web, "Uses", "HTTPS")
    Rel(web, api, "Calls", "JSON/HTTPS")
    Rel(api, db, "Reads/Writes", "SQL")
```

## Key Flows

### [Primary Flow Name]

[Explain the most important flow through the system]

```mermaid
sequenceDiagram
    autonumber
    participant User
    participant API
    participant DB

    User->>API: Request
    API->>DB: Query
    DB-->>API: Result
    API-->>User: Response
```

## Data Model

[If applicable]

```mermaid
erDiagram
    ENTITY_A ||--o{ ENTITY_B : relationship
```

## Design Decisions

| Decision | Rationale |
|----------|-----------|
| [Choice made] | [Why this choice] |

## Deployment

[How the system is deployed, if relevant]
