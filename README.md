## StepMate Facet-Graph Onboarding Architecture

### 1. High-Level Onboarding Flow Architecture

```mermaid
graph TD
    A[User Starts Onboarding] --> B[Mission Selection Phase]
    B --> C{User Chooses Mission}
    C -->|Study Partner| D[student_peer Mission]
    C -->|Find Tutor| E[tutor Mission]
    C -->|Student to Teach| F[student Mission]
    C -->|Student to Mentor| G[resident Mission]
    D --> H[Core Facets Collection]
    E --> H
    F --> H
    G --> H
    H --> I[Role-Specific Facets]
    I --> J[Mission Overrides]
    J --> K[Profile Summary & Confirmation]
    K --> L[Onboarding Complete]
    L --> M[Ready for Matching]
    style A fill:#e1f5fe
    style L fill:#c8e6c9
    style M fill:#4caf50
```

### 2. Detailed Mission Selection Flow

```mermaid
graph TD
    A[AI ask: Who are you looking for right now?] --> B[User Response]
    B --> C[AI analyzes response]
    C --> D[Extract mission type]
    C --> E[Ask clarifying question]
    C --> F[Help user choose]
    D --> G[Validate mission selection]
    G --> H[Update session: mission + roleCapabilities]
    G --> I[Ask for valid choice]
    H --> J[Move to core facets phase]
    I --> A
    E --> A
    F --> A
    style A fill:#fff3e0
    style J fill:#e8f5e8
    style H fill:#e3f2fd
```

### 3. Progressive Facet Collection Flow

```mermaid
graph TD
    A[Current Phase: Core Facets] --> B[Calculate Facet Priority]
    B --> C[Get Next High-Priority Facet]
    C --> D[Check Dependencies]
    D -->|Dependencies Met| E[Ask Question]
    D -->|Dependencies Missing| F[Ask Dependency Question First]
    E --> G[User Responds]
    G --> H[AI Extracts & Validates Answer]
    H -->|Valid| I[Update Facet Value]
    H -->|Invalid| J[Ask for Clarification]
    I --> K[Check Phase Completion]
    K -->|Phase Complete| L[Move to Next Phase]
    K -->|Phase Incomplete| M[Get Next Facet Question]
    L --> N[Update Session State]
    M --> C
    J --> E
    F --> C
    style A fill:#e8f5e8
    style L fill:#e3f2fd
    style I fill:#f3e5f5
```

### 4. Facet Dependency & Priority Flow

```mermaid
graph TD
    A[Facet Registry] --> B[Get Facet Dependencies]
    B --> C{Check Dependencies}
    C -->|No Dependencies| D[Ready to Ask]
    C -->|Has Dependencies| E[Check Dependency Status]
    E --> F{All Dependencies Met?]
    F -->|Yes| D
    F -->|No| G[Get Missing Dependencies]
    G --> H[Calculate Priority Score]
    H --> I[Sort by Priority]
    I --> J[Return Next Ready Facet]
    D --> J
    style A fill:#e1f5fe
    style J fill:#c8e6c9
    style G fill:#fff3e0
```

### 5. Mission-Specific Question Flow

```mermaid
graph TD
    A[Selected Mission] --> B[Load Mission Configuration]
    B --> C[Get Required Facets]
    C --> D[Get Optional Facets]
    D --> E[Build Question Flow]
    E --> F[Prioritize Questions]
    F --> G[Start Progressive Questioning]
    G --> H{Current Facet Type}
    H -->|Core| I[Ask Core Facet Question]
    H -->|Role| J[Ask Role-Specific Question]
    H -->|Mission| K[Ask Mission Override Question]
    I --> L[Process Answer]
    J --> L
    K --> L
    L --> M[Update Progress]
    M --> N{Phase Complete?}
    N -->|Yes| O[Move to Next Phase]
    N -->|No| P[Get Next Question]
    O --> Q[Update Session State]
    P --> H
    style A fill:#e3f2fd
    style Q fill:#e8f5e8
    style L fill:#f3e5f5
```

### 6. Session State Management Flow

```mermaid
graph TD
    A[User Session] --> B[Session State Machine]
    B --> C{Current Phase}
    C -->|mission_selection| D[Phase: Mission Selection]
    C -->|core_facets| E[Phase: Core Facets]
    C -->|role_facets| F[Phase: Role Facets]
    C -->|mission_overrides| G[Phase: Mission Overrides]
    C -->|complete| H[Phase: Complete]
    D --> I[State: Choosing Mission]
    E --> J[State: Collecting Core Info]
    F --> K[State: Collecting Role Info]
    G --> L[State: Setting Preferences]
    H --> M[State: Ready for Matching]
    I --> N{User Responds}
    J --> O{User Responds}
    K --> P{User Responds}
    L --> Q{User Responds}
    N --> R[Update: mission + roleCapabilities]
    O --> S[Update: core facets progress]
    P --> T[Update: role facets progress]
    Q --> U[Update: mission overrides]
    R --> V[Transition: mission_selection → core_facets]
    S --> W[Check: core_facets completion]
    T --> X[Check: role_facets completion]
    U --> Y[Check: mission_overrides completion]
    W -->|Complete| Z[Transition: core → role]
    X -->|Complete| AA[Transition: role → mission]
    Y -->|Complete| BB[Transition: mission → complete]
    style A fill:#e1f5fe
    style M fill:#4caf50
    style V fill:#e3f2fd
    style Z fill:#e3f2fd
    style AA fill:#e3f2fd
    style BB fill:#e3f2fd
```

### 7. Data Flow & Storage Architecture

```mermaid
graph TD
    A[User Input] --> B[AI Processing]
    B --> C[Facet Extraction]
    C --> D[Validation & Standardization]
    D --> E{Validation Result}
    E -->|Valid| F[Update Session State]
    E -->|Invalid| G[Request Clarification]
    F --> H[Update Firestore Collections]
    H --> I[users/uid/facets]
    H --> J[users/uid/missions]
    H --> K[onboarding_sessions/sessionId]
    I --> L[Core Facets]
    I --> M[Role Facets]
    J --> N[Mission Configurations]
    J --> O[Mission Overrides]
    K --> P[Session Progress]
    K --> Q[Current State]
    K --> R[Completion Status]
    L --> S[Vector Store Metadata]
    M --> S
    N --> S
    S --> T[Matching Engine]
    style A fill:#e1f5fe
    style T fill:#4caf50
    style H fill:#e3f2fd
    style S fill:#fff3e0
```

### 8. Complete Onboarding Journey (Tutor mission → student capability)

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
    AI->>U: "Who are you looking for right now?"
    U->>AI: "I need a tutor for Step 1"
    AI->>M: Analyze mission choice
    M->>AI: Mission: tutor, RoleCapabilities: [student]
    AI->>S: Update session: mission=tutor, roleCapabilities=[student]
    AI->>U: "Great! You're looking for a tutor. Let's get to know you better."
    AI->>F: Get next core facet
    F->>AI: location (priority 1)
    AI->>U: "Where are you currently located?"
    U->>AI: "I'm in Seattle, Washington"
    AI->>F: Validate location
    F->>AI: Valid: "Seattle, WA, USA"
    AI->>S: Update facet: location="Seattle, WA, USA"
    AI->>F: Get next core facet
    F->>AI: gender (priority 2)
    AI->>U: "What is your gender?"
    U->>AI: "Female"
    AI->>F: Validate gender
    F->>AI: Valid: "female"
    AI->>S: Update facet: gender="female"
    AI->>F: Get next core facet
    F->>AI: commsPref (priority 3)
    AI->>U: "How would you like to connect with your tutor?"
    U->>AI: "Video calls and text messages"
    AI->>F: Validate commsPref
    F->>AI: Valid: ["video", "text"]
    AI->>S: Update facet: commsPref=["video", "text"]
    AI->>S: Check core facets completion
    S->>AI: Core facets complete (100%)
    AI->>U: "Perfect! Now let's get specific about your tutoring needs."
    AI->>F: Get next role facet
    F->>AI: step (priority 1)
    AI->>U: "Which USMLE Step are you preparing for?"
    U->>AI: "Step 1"
    AI->>F: Validate step
    F->>AI: Valid: "1"
    AI->>S: Update facet: step="1"
    AI->>F: Get next role facet
    F->>AI: budget (priority 2)
    AI->>U: "What's your budget range for tutoring?"
    U->>AI: "Around $50-100 per hour"
    AI->>F: Validate budget
    F->>AI: Valid: "$$"
    AI->>S: Update facet: budget="$$"
    AI->>S: Check role facets completion
    S->>AI: Role facets complete (75%)
    AI->>U: "Excellent! Now let's find the right tutor for you."
    AI->>S: Check overall completion
    S->>AI: Overall: 87.5% (Ready for matching)
    AI->>U: "Perfect! I have enough information to start finding tutors."
    AI->>S: Mark onboarding complete
    S->>AI: Session completed
```

### 9. Key Decision Points & Transitions

```mermaid
graph TD
    A[Onboarding Start] --> B{User Has Existing Profile?}
    B -->|Yes| C[Load Existing Facets]
    B -->|No| D[Start Fresh]
    C --> E[Identify Missing Facets]
    D --> F[Begin Mission Selection]
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

### 10. Error Handling & Recovery Flow

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


