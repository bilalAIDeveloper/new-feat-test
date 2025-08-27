## StepMate Facet-Graph Onboarding Architecture

### 1. High-Level Onboarding Flow (Simplified)

```mermaid
graph TD
    A[User Starts] --> B[Collect Core Facets]
    B --> C[Mission Selection]
    C --> D[Role-Specific Facets]
    D --> E[Mission Overrides]
    E --> F[Complete & Ready]
    
    style A fill:#e1f5fe
    style F fill:#4caf50
    style B fill:#fff3e0
```

### 2. Complete Question Flow (Combines Core, Mission, Dependencies)

```mermaid
graph TD
    A[Start Onboarding] --> B[Collect Core Facets]
    B --> C[Ask: Where are you located?]
    C --> D[User: Location]
    D --> E[Ask: What's your gender?]
    E --> F[User: Gender]
    F --> G[Ask: How do you prefer to communicate?]
    G --> H[User: Communication]
    H --> I[Core Profile Complete]
    
    I --> J[Mission Selection]
    J --> K[AI: Who are you looking for?]
    K --> L[User: Mission Choice]
    L --> M[Extract Mission + Role]
    M --> N[Load Mission Config]
    
    N --> O[Get Next Role Facet]
    O --> P[Check Dependencies]
    P -->|Ready| Q[Ask Question]
    P -->|Not Ready| R[Get Missing Dependencies]
    R --> S[Sort by Priority]
    S --> T[Return Next Ready Facet]
    T --> Q
    
    Q --> U[User Responds]
    U --> V[Process Answer]
    V --> W[Update Progress]
    W --> X{Phase Complete?}
    X -->|Yes| Y[Next Phase]
    X -->|No| O
    Y --> Z{All Complete?}
    Z -->|Yes| AA[Ready for Matching]
    Z -->|No| O
    
    style A fill:#e1f5fe
    style AA fill:#4caf50
    style I fill:#c8e6c9
    style Y fill:#e3f2fd
```

### 3. Session State Transitions

```mermaid
graph TD
    A[Session Start] --> B[Core Facets Collection]
    B --> C[Mission Selection]
    C --> D[Role Facets]
    D --> E[Mission Overrides]
    E --> F[Complete]
    
    B -->|User Responds| G[Update Core Profile]
    C -->|User Responds| H[Update Mission + Roles]
    D -->|User Responds| I[Update Role Progress]
    E -->|User Responds| J[Update Overrides]
    
    G --> C
    H --> D
    I --> E
    J --> F
    
    style A fill:#e1f5fe
    style F fill:#4caf50
    style G fill:#e3f2fd
    style H fill:#e3f2fd
    style I fill:#e3f2fd
    style J fill:#e3f2fd
```

### 4. Data Flow Architecture

```mermaid
graph TD
    A[User Input] --> B[AI Processing]
    B --> C[Facet Extraction]
    C --> D[Validation]
    D --> E{Valid?}
    E -->|Yes| F[Update Collections]
    E -->|No| G[Request Clarification]
    F --> H[Firestore: Facets + Missions + Sessions]
    H --> I[Vector Store Metadata]
    I --> J[Matching Engine]
    
    style A fill:#e1f5fe
    style J fill:#4caf50
    style F fill:#e3f2fd
```

### 5. Complete Onboarding Journey (Tutor mission → student capability)

```mermaid
sequenceDiagram
    participant U as User
    participant AI as StepMate AI
    participant S as Session Manager
    participant F as Facet Registry
    participant M as Mission Service
    U->>AI: Start onboarding
    AI->>S: Create session
    S->>AI: Session created
    AI->>U: "Tell me about yourself - where are you located?"
    U->>AI: "I'm in Seattle, Washington"
    AI->>F: Validate location
    F->>AI: Valid: "Seattle, WA, USA"
    AI->>S: Update facet: location="Seattle, WA, USA"
    AI->>U: "What's your gender?"
    U->>AI: "Female"
    AI->>F: Validate gender
    F->>AI: Valid: "female"
    AI->>S: Update facet: gender="female"
    AI->>U: "How do you prefer to communicate?"
    U->>AI: "Video calls and text messages"
    AI->>F: Validate commsPref
    F->>AI: Valid: ["video", "text"]
    AI->>S: Update facet: commsPref=["video", "text"]
    AI->>U: "Great! I see you're in Seattle and prefer video calls. Who are you looking for right now?"
    U->>AI: "I need a tutor for Step 1"
    AI->>M: Analyze mission choice
    M->>AI: Mission: tutor, RoleCapabilities: [student]
    AI->>S: Update session: mission=tutor, roleCapabilities=[student]
    AI->>U: "Perfect! I'll help you find Step 1 tutors in Seattle who offer video sessions."
    AI->>F: Get next role facet
    F->>AI: step (priority 1)
    AI->>U: "Which USMLE Step are you preparing for?"
    U->>AI: "Step 1"
    AI->>F: Validate step
    F->>AI: Valid: "1"
    AI->>S: Update facet: step="1"
    AI->>F: Get next role facet
    F->>AI: budget (priority 2)
    AI->>U: "What's your budget range for Step 1 tutoring in Seattle?"
    U->>AI: "Around $50-100 per hour"
    AI->>F: Validate budget
    F->>AI: Valid: "$$"
    AI->>S: Update facet: budget="$$"
    AI->>S: Check role facets completion
    S->>AI: Role facets complete (75%)
    AI->>U: "Excellent! Now let's find the right tutor for you."
    AI->>S: Check overall completion
    S->>AI: Overall: 87.5% (Ready for matching)
    AI->>U: "Perfect! I have enough information to start finding Step 1 tutors in Seattle who offer video sessions within your budget."
    AI->>S: Mark onboarding complete
    S->>AI: Session completed
```

### 6. Key Decision Points & Transitions

```mermaid
graph TD
    A[Onboarding Start] --> B{User Has Existing Profile?}
    B -->|Yes| C[Load Existing Facets]
    B -->|No| D[Start Fresh]
    C --> E[Identify Missing Facets]
    D --> F[Begin Core Facets Collection]
    E --> G[Progressive Facet Collection]
    F --> G
    G --> H{Current Facet Complete?}
    H -->|Yes| I[Check Phase Completion]
    H -->|No| J[Continue Current Facet]
    I --> K{Phase Complete?}
    K -->|Yes| L[Transition to Next Phase]
    K -->|No| M[Get Next Facet]
    L --> N{All Phases Complete?}
    N -->|Yes| O[Onboarding Complete]
    N -->|No| P[Continue Next Phase]
    O --> Q[Ready for Matching]
    J --> G
    M --> G
    P --> G
    style A fill:#e1f5fe
    style Q fill:#4caf50
    style L fill:#e3f2fd
    style O fill:#c8e6c9
```

### 7. Error Handling & Recovery Flow

```mermaid
graph TD
    A[User Response] --> B[AI Processing]
    B --> C[Response Valid?]
    C -->|Yes| D[Process Normally]
    C -->|No| E[Error Handling]
    E --> F{Error Type}
    F -->|Invalid Format| G[Ask for Clarification]
    F -->|Missing Information| H[Request Missing Data]
    F -->|Conflicting Data| I[Resolve Conflict]
    F -->|Ambiguous Response| J[Ask Specific Question]
    G --> K[User Clarifies]
    H --> L[User Provides Data]
    I --> M[User Resolves Conflict]
    J --> N[User Gives Clear Answer]
    K --> O[Re-process Response]
    L --> O
    M --> O
    N --> O
    O --> P{Now Valid?}
    P -->|Yes| D
    P -->|No| E
    D --> Q[Continue Flow]
    style A fill:#e1f5fe
    style Q fill:#4caf50
    style E fill:#ffebee
    style O fill:#fff3e0
```


