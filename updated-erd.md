# Updated ERD

<div data-full-width="true"><figure><img src=".gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure></div>

Let’s create an **Entity-Relationship Diagram (ERD)** for the **SAT Smart Prep App** by **Learner Labs** using **Mermaid** syntax, reflecting the latest database schema as of the most recent updates. The ERD will include all tables, their fields, and relationships, capturing the structure of the database after incorporating changes such as the **IRT 3-Parameter Logistic (3PL) model** support, **mastery targets**, and new tables like `Question_IRT_Updates` and `Qualified_Questions_Reports`. This ERD will provide a visual representation of the database schema, making it easier to understand the relationships between entities.

***

### ERD in Mermaid Syntax

Below is the ERD for the SAT Smart Prep App, including all tables, their fields, and relationships. You can render this diagram using a Mermaid-compatible tool (e.g., Mermaid Live Editor, GitHub, or a compatible IDE).

```mermaid
erDiagram
    Users ||--o{ Study_Plans : has
    Users ||--o{ Proficiencies : has
    Users ||--o{ Responses : submits
    Users ||--o{ Scores : achieves
    Users ||--o{ Challenges : participates
    Users ||--o{ Achievements : earns
    Users ||--o{ Rewards : owns
    Users ||--o{ Feedback : provides
    Users ||--o{ Community_Posts : creates
    Users ||--o{ Community_Comments : posts
    Users ||--o{ Diagnostic_Sessions : takes
    Users ||--o{ Practice_Sessions : takes
    Study_Plans ||--o{ Study_Plan_Actions : contains
    Questions ||--o{ Responses : answered_in
    Questions ||--o{ Proficiencies : measures
    Questions ||--o{ Question_IRT_Updates : updates
    Questions ||--o{ Qualified_Questions_Reports : reported_in
    Challenges ||--o{ Achievements : unlocks
    Rewards ||--o{ Achievements : granted_by
    Community_Posts ||--o{ Community_Comments : has
    Diagnostic_Sessions ||--o{ Responses : generates
    Practice_Sessions ||--o{ Responses : generates

    Users {
        string user_id PK
        string name
        string email
        string device_token
        int coins
    }

    Study_Plans {
        string plan_id PK
        string user_id FK
        string test_date
    }

    Study_Plan_Actions {
        string id PK
        string plan_id FK
        string task
        string action
        string due_date
        int points
        string completed
        string subskill
        int target_mastery
    }

    Proficiencies {
        string id PK
        string user_id FK
        string domain
        string skill
        float theta
        string timestamp
    }

    Questions {
        string id PK
        string domain
        string skill
        string question_text
        json options
        string correct_answer
        string explanation
        float irt_a
        float irt_b
        float irt_c
        boolean has_image
        boolean has_table
        boolean is_mcq
        string qualification_status
        string created_at
    }

    Question_IRT_Updates {
        string id PK
        string question_id FK
        float irt_a
        float irt_b
        float irt_c
        float confidence_level
        int response_count
        string updated_at
    }

    Qualified_Questions_Reports {
        string id PK
        string question_id FK
        float irt_a
        float irt_b
        float irt_c
        float confidence_level
        int response_count
        string qualified_at
    }

    Responses {
        string id PK
        string user_id FK
        string question_id FK
        string user_answer
        boolean is_correct
        int time_spent
        string timestamp
    }

    Scores {
        string id PK
        string user_id FK
        int score
        float theta
        string timestamp
    }

    Challenges {
        string id PK
        string user_id FK
        string challenge_type
        int target
        int progress
        boolean completed
    }

    Achievements {
        string id PK
        string user_id FK
        string name
        int target
        int progress
        boolean completed
    }

    Rewards {
        string id PK
        string user_id FK
        string name
        string type
    }

    Feedback {
        string id PK
        string user_id FK
        string content
        string timestamp
    }

    Community_Posts {
        string id PK
        string user_id FK
        string content
        string timestamp
    }

    Community_Comments {
        string id PK
        string user_id FK
        string post_id FK
        string content
        string timestamp
    }

    Diagnostic_Sessions {
        string session_id PK
        string user_id FK
        json questions
    }

    Practice_Sessions {
        string session_id PK
        string user_id FK
        json questions
    }
```

***

### Explanation of the ERD

#### Entities and Fields

* **Users**: The central entity, storing user information such as `user_id`, `name`, `email`, `device_token`, and `coins` for gamification.
* **Study\_Plans**: Stores user-specific study plans, linked to `Users` via `user_id`, with a `test_date` for scheduling.
* **Study\_Plan\_Actions**: Represents individual tasks in a study plan, including `task`, `due_date`, `points`, `completed`, `subskill`, and `target_mastery` (added for mastery targets).
* **Proficiencies**: Stores `theta` scores for each user, domain, and subskill, used for progress monitoring and Study Plan task allocation.
* **Questions**: Stores SAT questions, updated with IRT 3PL parameters (`irt_a`, `irt_b`, `irt_c`), metadata (`has_image`, `has_table`, `is_mcq`), and `qualification_status`.
* **Question\_IRT\_Updates**: Stores updated IRT parameters as responses are collected, with `confidence_level` and `response_count` to determine qualification.
* **Qualified\_Questions\_Reports**: Stores weekly reports of questions that have reached the 0.90 confidence threshold for qualification.
* **Responses**: Stores user responses to questions, linked to `Users` and `Questions`, with `time_spent` for pacing analytics.
* **Scores**: Tracks overall scores from full-length tests, used for progress monitoring and score improvement guarantee.
* **Challenges**, **Achievements**, **Rewards**: Support gamification features, linked to `Users`.
* **Feedback**, **Community\_Posts**, **Community\_Comments**: Support user feedback and social features.
* **Diagnostic\_Sessions**, **Practice\_Sessions**: Store sessions for the Diagnostic Test and practice/full-length tests, respectively, with a JSON array of question IDs.

#### Relationships

* **Users** has a one-to-many relationship with `Study_Plans`, `Proficiencies`, `Responses`, `Scores`, `Challenges`, `Achievements`, `Rewards`, `Feedback`, `Community_Posts`, `Community_Comments`, `Diagnostic_Sessions`, and `Practice_Sessions`.
* **Study\_Plans** has a one-to-many relationship with `Study_Plan_Actions`.
* **Questions** has a one-to-many relationship with `Responses`, `Proficiencies` (via responses), `Question_IRT_Updates`, and `Qualified_Questions_Reports`.
* **Challenges** has a one-to-many relationship with `Achievements` (unlocking achievements).
* **Rewards** has a one-to-many relationship with `Achievements` (granted by achievements).
* **Community\_Posts** has a one-to-many relationship with `Community_Comments`.
* **Diagnostic\_Sessions** and **Practice\_Sessions** have a one-to-many relationship with `Responses` (generating responses).

***

### Summary

This ERD provides a comprehensive visual representation of the latest database schema for the SAT Smart Prep App, including all tables, fields, and relationships. It reflects the recent updates for the IRT 3PL model, mastery targets, and new tables for IRT parameter updates and reporting. You can render this diagram in a Mermaid-compatible tool to visualize the database structure. Let me know if you’d like to make any adjustments or explore specific aspects further!
