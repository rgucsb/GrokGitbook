# Learner Lab Data Model Documentation

##

This document provides a comprehensive overview of the Learner Lab data model, including database schema, relationships, and implementation details.

### Database Schema Overview

```mermaid
erDiagram
    User ||--o{ PracticeSession : "creates"
    User ||--o{ StudyPlan : "has"
    User ||--o{ DiagnosticResult : "takes"
    Question ||--o{ QuestionResponse : "generates"
    PracticeSession ||--o{ QuestionResponse : "contains"

    %% User Table
    User {
        string id PK "UUID"
        string name "User's full name"
        string email "Unique email"
        string password "Hashed password"
        int targetScore "Optional target"
        datetime testDate "Planned test date"
        datetime createdAt "Record creation"
        datetime updatedAt "Last update"
    }

    %% Question Table
    Question {
        string id PK "UUID"
        string domain "Subject domain"
        string skill "Specific skill"
        string text "Question content"
        json options "Multiple choice"
        string correctAnswer "Right answer"
        string rationale "Explanation"
        json metadata "IRT params etc"
        json imageUrls "Optional images"
        json tables "Optional tables"
        datetime createdAt "Record creation"
        datetime updatedAt "Last update"
    }

    %% Practice Session Table
    PracticeSession {
        string id PK "UUID"
        string userId FK "User reference"
        datetime startTime "Session start"
        datetime endTime "Session end"
        string domain "Subject focus"
        string skill "Optional focus"
        int questionsAnswered "Progress"
        int correctAnswers "Performance"
        float abilityEstimate "IRT theta"
        datetime createdAt "Record creation"
        datetime updatedAt "Last update"
    }

    %% Question Response Table
    QuestionResponse {
        string id PK "UUID"
        string sessionId FK "Session reference"
        string questionId FK "Question reference"
        string userAnswer "Student answer"
        boolean isCorrect "Correctness"
        int timeSpent "Response time"
        datetime createdAt "Record creation"
    }

    %% Study Plan Table
    StudyPlan {
        string id PK "UUID"
        string userId FK "User reference"
        datetime targetDate "Goal date"
        json dailyTasks "Daily goals"
        json weeklyGoals "Weekly goals"
        json monthlyTargets "Monthly goals"
        datetime createdAt "Record creation"
        datetime updatedAt "Last update"
    }

    %% Diagnostic Result Table
    DiagnosticResult {
        string id PK "UUID"
        string userId FK "User reference"
        datetime completedAt "Test completion"
        json domains "Domain scores"
        int overallScore "Total score"
        json recommendations "Study advice"
        datetime createdAt "Record creation"
        datetime updatedAt "Last update"
    }
```

### Key Components and Relationships

#### User Management

The User model is the central entity in the system, connecting to all learning activities:

```mermaid
graph TD
    U[User] --> PS[Practice Sessions]
    U --> SP[Study Plans]
    U --> DR[Diagnostic Results]
    
    subgraph "User Management"
        U
        style U fill:#f9f,stroke:#333,stroke-width:2px
    end
    
    subgraph "Learning Activities"
        PS
        SP
        DR
    end
```

#### Question Bank Structure

Questions are organized by domain and skill, with rich content support:

```mermaid
graph TD
    D[Domain] --> S[Skill]
    S --> Q[Question]
    Q --> R[Response]
    
    subgraph "Content Organization"
        D
        S
        Q
    end
    
    subgraph "Performance Data"
        R
    end
```

#### Practice Session Flow

Practice sessions track user interactions with questions:

```mermaid
graph LR
    PS[Practice Session] --> QR[Question Response]
    Q[Question] --> QR
    
    subgraph "Session Management"
        PS --> |tracks| M[Metrics]
        M --> P[Progress]
        M --> S[Score]
        M --> A[Ability]
    end
```

### Adaptive Learning Implementation

The system uses Item Response Theory (IRT) with a sophisticated adaptive question selection mechanism.

#### Adaptive Learning Architecture

```mermaid
flowchart TD
    A[Student Starts Practice] --> B[Initialize Ability]
    B --> C[Question Selection]
    C --> D[Student Response]
    D --> E[Update Ability]
    E --> |Next Question| C
    
    subgraph "IRT Parameters"
        P1[a: Discrimination]
        P2[b: Difficulty]
        P3[c: Guessing]
    end
    
    subgraph "Ability Tracking"
        T1[Domain Level]
        T2[Skill Level]
        T3[Confidence Level]
    end
```

#### IRT Model Implementation

The system uses a 3-Parameter Logistic Model (3PL) for adaptive question selection:

```mermaid
graph LR
 A[3PL Model] --> B[P(θ) = c + (1-c)/(1 + e^(-Da(θ-b)))]
 B --> C[Probability]
 B --> D[Information]
 
 subgraph "Parameters"
     E[θ: Student Ability]
     F[a: Discrimination]
     G[b: Difficulty]
     H[c: Guessing]
 end
```

Key components:

* **Ability range**: -3 to 3 (standard IRT scale)
* **Discrimination (a)**: How well the question differentiates abilities
* **Difficulty (b)**: Ability level where P(correct) = 0.5
* **Guessing (c)**: Lower asymptote probability

#### Question Selection Algorithm

```mermaid
flowchart TD
 A[Current Ability] --> B[Calculate Information]
 B --> C[Filter Questions]
 C --> D[Sort by Information]
 D --> E[Select Highest Info]
 
 subgraph "Filters"
     F[Domain Match]
     G[Skill Match]
     H[Not Answered]
 end
```

The algorithm:

1. Calculates information value for each question at current ability
2. Filters questions by domain/skill and previous answers
3. Selects the question with maximum information value

#### Ability Estimation

```mermaid
graph TD
 A[Response] --> B[EAP Estimation]
 B --> C[Update Ability]
 C --> D[Update Confidence]
 
 E[Learning Rate] --> B
 F[Prior Ability] --> B
 G[Response Pattern] --> B
```

The process:

1. Uses Expected a Posteriori (EAP) estimation
2. Adjusts learning rate based on confidence
3. Updates ability based on response correctness
4. Increases confidence with each response

### Session Management Implementation

#### Session Lifecycle

```mermaid
sequenceDiagram
    participant U as User
    participant S as Session Controller
    participant A as Adaptive Engine
    participant D as Database

    U->>S: Start Session
    S->>D: Create Session Record
    S->>A: Initialize Ability
    
    loop Question Cycle
        A->>A: Select Question
        S->>U: Present Question
        U->>S: Submit Response
        S->>A: Process Response
        A->>A: Update Ability
        S->>D: Store Response
        S->>S: Update Statistics
    end
    
    U->>S: End Session
    S->>D: Update Session Record
    S->>U: Return Results
```

#### Session State Management

```mermaid
stateDiagram-v2
    [*] --> Active: Start Session
    Active --> InProgress: First Question
    InProgress --> InProgress: Answer Question
    InProgress --> Completed: End Session
    InProgress --> Paused: Timeout/Interrupt
    Paused --> InProgress: Resume
    Completed --> [*]
```

### JSON Field Structures

#### Question Options

```json
{
  "A": "Option text for A",
  "B": "Option text for B",
  "C": "Option text for C",
  "D": "Option text for D"
}
```

#### Question Metadata

```json
{
  "irt_parameters": {
    "a": 1.2,
    "b": 0.5,
    "c": 0.25
  },
  "irt_difficulty": 0.65,
  "cognitive_level": "application",
  "tags": ["algebra", "equations", "linear"]
}
```

#### Diagnostic Domains

```json
{
  "math": {
    "score": 650,
    "subsections": {
      "algebra": 680,
      "geometry": 620,
      "statistics": 650
    }
  },
  "verbal": {
    "score": 700,
    "subsections": {
      "reading": 720,
      "writing": 680
    }
  }
}
```

#### Study Plan Tasks

```json
{
  "monday": [
    { "topic": "Linear equations", "duration": 30, "completed": false },
    { "topic": "Reading comprehension", "duration": 45, "completed": true }
  ],
  "tuesday": [
    { "topic": "Quadratic equations", "duration": 30, "completed": false },
    { "topic": "Vocabulary", "duration": 20, "completed": false }
  ]
}
```

### Database Design Considerations

#### Primary Keys & Indexes

* All tables use UUID primary keys for security and distribution
* Foreign keys are indexed for query performance
* Composite indexes on frequently queried combinations

#### Relationship Types

* User → Practice Session: One-to-Many
* Practice Session → Response: One-to-Many
* Question → Response: One-to-Many
* User → Study Plan: One-to-Many
* User → Diagnostic Result: One-to-Many

#### Data Access Patterns

* Session-based access for practice activities
* User-centric queries for personalized content
* Domain/skill filtering for targeted practice
* Temporal analysis for progress tracking

### Backend Service Architecture

```mermaid
classDiagram
    class PracticeSession {
        +string id
        +string userId
        +DateTime startTime
        +DateTime endTime
        +string domain
        +string skill
        +int questionsAnswered
        +int correctAnswers
        +float abilityEstimate
    }

    class AdaptiveQuestionService {
        -AbilityStore abilities
        -Record~string,boolean~ sessionAnswers
        -AdaptiveQuestion[] questionBank
        +setQuestionBank()
        +getAbility()
        +selectNextQuestionForStudent()
        +recordResponse()
        +resetSession()
        +getSessionStats()
    }

    class SessionController {
        +startPracticeSession()
        +getPracticeSession()
        +submitQuestionResponse()
        +endPracticeSession()
    }

    PracticeSession --> SessionController
    SessionController --> AdaptiveQuestionService
```

This document provides a comprehensive overview of the Learner Lab data model and its implementation. The system is designed to deliver personalized, adaptive learning experiences through sophisticated algorithms and a well-structured database schema.
