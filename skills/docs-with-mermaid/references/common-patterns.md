# Common Mermaid Diagram Patterns

Practical examples organized by use case. Copy and adapt these patterns for your documentation.

---

## API Request Flow

**Use case:** Documenting how requests flow through your system

```mermaid
sequenceDiagram
    autonumber
    participant Client as Client App
    participant Gateway as API Gateway
    participant Auth as Auth Service
    participant API as REST API
    participant Cache as Redis Cache
    participant DB as PostgreSQL

    Client->>Gateway: POST /api/orders
    Gateway->>Auth: Validate JWT Token

    alt Token Valid
        Auth-->>Gateway: Token Valid + User Context
        Gateway->>API: Forward Request + User Context

        API->>Cache: Check product availability

        alt Cache Hit
            Cache-->>API: Product data (cached)
        else Cache Miss
            API->>DB: SELECT products
            DB-->>API: Product data
            API->>Cache: SET product data (TTL: 5min)
        end

        API->>DB: INSERT order
        DB-->>API: Order created
        API-->>Gateway: 201 Created + Order ID
        Gateway-->>Client: 201 Created + Order Response

    else Token Invalid/Expired
        Auth-->>Gateway: 401 Unauthorized
        Gateway-->>Client: 401 Unauthorized
    end
```

---

## System Architecture Overview

**Use case:** High-level architecture for README or onboarding

```mermaid
flowchart TB
    subgraph Clients["Client Applications"]
        Web[Web App<br/>React]
        Mobile[Mobile App<br/>React Native]
        CLI[CLI Tool<br/>Node.js]
    end

    subgraph Gateway["API Layer"]
        LB[Load Balancer<br/>nginx]
        API[API Gateway<br/>Kong]
    end

    subgraph Services["Microservices"]
        direction TB
        UserSvc[User Service<br/>Go]
        OrderSvc[Order Service<br/>Java]
        PaymentSvc[Payment Service<br/>Python]
        NotifSvc[Notification Service<br/>Node.js]
    end

    subgraph Data["Data Layer"]
        PG[(PostgreSQL<br/>Primary DB)]
        Redis[(Redis<br/>Cache)]
        Kafka[Kafka<br/>Events]
        S3[(S3<br/>File Storage)]
    end

    subgraph External["External Services"]
        Stripe[Stripe<br/>Payments]
        SendGrid[SendGrid<br/>Email]
    end

    Web --> LB
    Mobile --> LB
    CLI --> LB
    LB --> API

    API --> UserSvc
    API --> OrderSvc
    API --> PaymentSvc

    UserSvc --> PG
    UserSvc --> Redis
    OrderSvc --> PG
    OrderSvc --> Kafka
    PaymentSvc --> Stripe
    PaymentSvc --> Kafka

    Kafka --> NotifSvc
    NotifSvc --> SendGrid
    NotifSvc --> S3
```

---

## Database Schema

**Use case:** Documenting database relationships

```mermaid
erDiagram
    users {
        uuid id PK
        varchar email UK
        varchar password_hash
        varchar full_name
        enum role
        timestamp created_at
        boolean is_active
    }

    organizations {
        uuid id PK
        varchar name
        varchar slug UK
        jsonb settings
    }

    organization_members {
        uuid id PK
        uuid organization_id FK
        uuid user_id FK
        enum role
        timestamp joined_at
    }

    projects {
        uuid id PK
        uuid organization_id FK
        varchar name
        text description
        enum status
        uuid created_by FK
    }

    tasks {
        uuid id PK
        uuid project_id FK
        varchar title
        enum priority
        enum status
        uuid assignee_id FK
        timestamp due_date
    }

    users ||--o{ organization_members : "belongs to"
    organizations ||--o{ organization_members : "has"
    organizations ||--o{ projects : "contains"
    users ||--o{ projects : "creates"
    projects ||--o{ tasks : "has"
    users ||--o{ tasks : "assigned to"
```

---

## Order State Machine

**Use case:** Documenting lifecycle states

```mermaid
stateDiagram-v2
    [*] --> Draft: Create Order

    Draft --> Pending: Submit Order
    Draft --> Cancelled: Cancel

    state Pending {
        [*] --> AwaitingPayment
        AwaitingPayment --> PaymentReceived: Payment Success
        AwaitingPayment --> PaymentFailed: Payment Failed
        PaymentFailed --> AwaitingPayment: Retry Payment
    }

    Pending --> Processing: Payment Confirmed
    Pending --> Cancelled: Cancel / Timeout

    state Processing {
        [*] --> Picking
        Picking --> Packing: Items Picked
        Packing --> ReadyToShip: Packed
    }

    Processing --> Shipped: Dispatch
    Shipped --> Delivered: Complete
    Delivered --> [*]
    Cancelled --> [*]

    note right of Draft: Customer can edit order
    note right of Processing: Warehouse operations
```

---

## Authentication Flow (OAuth2/PKCE)

**Use case:** Documenting OAuth2 authentication

```mermaid
sequenceDiagram
    participant User as User Browser
    participant App as Web Application
    participant Auth as Auth Server
    participant API as Resource API

    User->>App: Click "Login"
    App->>App: Generate state & code_verifier
    App->>Auth: Redirect to /authorize<br/>(client_id, redirect_uri, state, code_challenge)

    Auth->>User: Show Login Page
    User->>Auth: Enter Credentials
    Auth->>Auth: Validate Credentials

    alt Credentials Valid
        Auth->>User: Redirect to redirect_uri<br/>(code, state)
        User->>App: Follow Redirect
        App->>App: Verify state
        App->>Auth: POST /token<br/>(code, code_verifier, client_secret)
        Auth-->>App: access_token, refresh_token, id_token
        App-->>User: Login Success

        User->>App: Request Protected Resource
        App->>API: GET /resource (Bearer token)
        API->>API: Validate JWT
        API-->>App: Protected Data
        App-->>User: Display Data
    else Credentials Invalid
        Auth-->>User: Error: Invalid Credentials
    end
```

---

## CI/CD Pipeline

**Use case:** Documenting deployment pipeline

```mermaid
flowchart LR
    subgraph Trigger
        Push[Git Push]
        PR[Pull Request]
    end

    subgraph Build["Build Stage"]
        Checkout[Checkout] --> Install[Install Deps]
        Install --> Lint[Lint] --> TypeCheck[Types] --> Compile[Build]
    end

    subgraph Test["Test Stage"]
        Unit[Unit Tests]
        Integration[Integration]
        E2E[E2E Tests]
    end

    subgraph Security
        SAST[SAST Scan]
        Deps[Dep Audit]
    end

    subgraph Deploy
        Docker[Docker Build] --> Registry[Push Registry]
        Registry --> Staging[Deploy Staging]
        Staging --> Smoke[Smoke Tests]
        Smoke -->|Pass| Prod[Deploy Prod]
    end

    Push --> Checkout
    PR --> Checkout
    Compile --> Unit --> Integration --> E2E
    Compile --> SAST
    Install --> Deps
    E2E --> Docker
    SAST --> Docker
    Deps --> Docker
```

---

## Domain Model (Class Diagram)

**Use case:** Documenting entity relationships in code

```mermaid
classDiagram
    class User {
        -UUID id
        -String email
        -String passwordHash
        -UserRole role
        +authenticate(password: String) bool
        +hasPermission(permission: Permission) bool
    }

    class Organization {
        -UUID id
        -String name
        +addMember(user: User, role: MemberRole) void
        +removeMember(user: User) void
    }

    class Project {
        -UUID id
        -String name
        -ProjectStatus status
        +addTask(task: Task) void
        +getActiveTasks() Task[]
    }

    class Task {
        -UUID id
        -String title
        -TaskPriority priority
        -TaskStatus status
        +assign(user: User) void
        +updateStatus(status: TaskStatus) void
    }

    class TaskStatus {
        <<enumeration>>
        TODO
        IN_PROGRESS
        IN_REVIEW
        DONE
    }

    User "1" --> "*" Organization : member of
    Organization "1" --> "*" Project : contains
    Project "1" --> "*" Task : has
    User "1" --> "*" Task : assigned
```

---

## Microservices Communication Patterns

**Use case:** Documenting service-to-service patterns

```mermaid
flowchart TB
    subgraph sync["Synchronous"]
        direction LR
        A1[Service A] -->|REST/gRPC| B1[Service B]
        B1 -->|Response| A1
    end

    subgraph async["Asynchronous (Event-Driven)"]
        direction LR
        A2[Service A] -->|Publish| MQ[Message Queue]
        MQ -->|Subscribe| B2[Service B]
        MQ -->|Subscribe| C2[Service C]
    end

    subgraph saga["Saga Pattern"]
        direction LR
        Order[Order Svc] -->|1. Create| OS[(Order DB)]
        Order -->|2. Reserve| Inventory[Inventory Svc]
        Inventory -->|3. Success| Order
        Order -->|4. Charge| Payment[Payment Svc]
        Payment -.->|5. Failure| Order
        Order -.->|6. Compensate| Inventory
    end
```

---

## Error Handling Flow

**Use case:** Documenting error classification and handling

```mermaid
flowchart TB
    Start([Request]) --> Validate{Validate Input}
    Validate -->|Invalid| E400[400 Bad Request]
    Validate -->|Valid| Auth{Authenticate}
    Auth -->|No Token| E401[401 Unauthorized]
    Auth -->|Valid| Authz{Authorize}
    Authz -->|Forbidden| E403[403 Forbidden]
    Authz -->|Allowed| Process{Process}
    Process -->|Not Found| E404[404 Not Found]
    Process -->|Success| OK[2xx Success]
    Process -->|Error| E500[500 Internal Error]

    E400 --> Handler[Error Handler]
    E401 --> Handler
    E403 --> Handler
    E404 --> Handler
    E500 --> Handler
    Handler --> Log[Log] --> Respond([Response])
    OK --> Respond

    style E400 fill:#ffcccc
    style E401 fill:#ffcccc
    style E403 fill:#ffcccc
    style E404 fill:#ffcccc
    style E500 fill:#ff9999
    style OK fill:#ccffcc
```

---

## Component Hierarchy (Frontend)

**Use case:** Documenting React/Vue component structure

```mermaid
flowchart TB
    subgraph App
        Router[Router]
    end
    subgraph Layout
        Main[MainLayout] --> Header & Sidebar & Footer
    end
    subgraph Pages
        Dashboard & Projects & Settings
    end
    subgraph Features
        ProjectList --> ProjectCard
        TaskBoard --> TaskColumn --> TaskCard
    end

    Router --> Main
    Main --> Dashboard & Projects & Settings
    Dashboard --> ProjectList
    Projects --> ProjectList & TaskBoard
```

---

## User Journey

**Use case:** Documenting user experience

```mermaid
journey
    title E-Commerce Checkout
    section Discovery
        Visit homepage: 5: Customer
        Browse categories: 4: Customer
        View product: 5: Customer
    section Selection
        Add to cart: 5: Customer
        Apply coupon: 3: Customer
    section Checkout
        Enter shipping: 3: Customer
        Enter payment: 2: Customer
        Place order: 5: Customer
    section Post-Purchase
        View confirmation: 5: Customer
        Track order: 4: Customer
```

---

## Git Branching Strategy

**Use case:** Documenting team Git workflow

```mermaid
gitGraph
    commit id: "Initial"
    branch develop
    commit id: "Setup CI/CD"
    branch feature/auth
    commit id: "Add login"
    commit id: "Add registration"
    checkout develop
    merge feature/auth id: "Merge auth" tag: "auth-complete"
    branch release/1.0
    commit id: "Version bump"
    checkout main
    merge release/1.0 id: "Release" tag: "v1.0.0"
    checkout develop
    merge release/1.0 id: "Sync"
```

---

## Feature Prioritization

**Use case:** Documenting priority decisions

```mermaid
quadrantChart
    title Feature Priority Matrix
    x-axis Low Effort --> High Effort
    y-axis Low Impact --> High Impact
    quadrant-1 Do Now
    quadrant-2 Plan Carefully
    quadrant-3 Consider Dropping
    quadrant-4 Quick Wins
    SSO Integration: [0.75, 0.9]
    Mobile App: [0.85, 0.85]
    Dark Mode: [0.15, 0.25]
    Search: [0.35, 0.75]
    Notifications: [0.30, 0.65]
    Audit Logging: [0.40, 0.80]
```

---

## Architecture Decision Mindmap

**Use case:** Documenting ADRs

```mermaid
mindmap
    root((API Design))
        Protocol
            REST - Familiar, good tooling
            GraphQL - Flexible queries
            gRPC - High performance
            **Decision: REST**
        Auth
            JWT - Stateless
            Session - Easy revocation
            OAuth2 - Third-party auth
            **Decision: JWT + OAuth2**
        Database
            PostgreSQL - ACID, JSON support
            MongoDB - Flexible schema
            **Decision: PostgreSQL**
```

---

*These patterns are part of the docs-with-mermaid skill. Mix and match to create comprehensive documentation.*
