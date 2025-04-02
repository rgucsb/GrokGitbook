# ERD

Below is a detailed **Entity-Relationship Diagram (ERD)** for the **SAT Smart Prep App** by **Learner Labs**, covering both the web and mobile app. The ERD represents the database schema used by the FastAPI backend, which both the web app (`frontend/`) and mobile app (`SATSmartPrepApp/`) interact with. I’ll describe the entities, their attributes, relationships, and provide a textual representation of the ERD, as I cannot generate visual diagrams directly. You can use this description to create a visual ERD using tools like Lucidchart, Draw.io, or DBeaver.

***

### ERD Overview

The SAT Smart Prep App’s database is designed to support personalized study plans, adaptive practice, gamification, social features, and progress monitoring. The schema is implemented in PostgreSQL, as defined in `frontend/backend/migrations/init_db.py`. The ERD includes entities for users, questions, practice sessions, responses, proficiencies, study plans, tutor interactions, social features, and gamification elements.

***

### Entities and Attributes

#### 1. **Users**

* **Description**: Represents students, tutors, or parents using the app.
* **Attributes**:
  * `user_id` (VARCHAR(36), Primary Key): Unique identifier for the user.
  * `email` (VARCHAR(255), Unique, Not Null): User’s email address.
  * `hashed_password` (VARCHAR(255), Not Null): Hashed password for authentication.
  * `full_name` (VARCHAR(255)): User’s full name.
  * `grade` (VARCHAR(50)): User’s grade level (e.g., 9th, 10th).
  * `school` (VARCHAR(255)): User’s school (optional).
  * `sat_taken` (BOOLEAN): Whether the user has taken the SAT before.
  * `sat_math_score` (INTEGER): Previous SAT Math score (if applicable).
  * `sat_reading_score` (INTEGER): Previous SAT Reading & Writing score (if applicable).
  * `sat_test_date` (VARCHAR(50)): Date of the user’s SAT test.
  * `study_hours_per_week` (INTEGER): Number of hours the user can study per week.
  * `study_days_per_week` (INTEGER): Number of days the user can study per week.
  * `preferred_study_time` (VARCHAR(50)): Preferred study time (e.g., Morning, Afternoon).
  * `onboarding_completed` (BOOLEAN, Default: FALSE): Whether onboarding is completed.
  * `theme` (VARCHAR(50), Default: 'light'): User’s theme preference (e.g., light, dark).
  * `device_token` (VARCHAR(255)): Device token for push notifications.
  * `points` (INTEGER, Default: 0): Points earned through practice.
  * `streak` (INTEGER, Default: 0): Consecutive days of activity.
  * `league` (VARCHAR(50), Default: 'Bronze'): User’s league based on points.
  * `coins` (INTEGER, Default: 0): Coins earned for gamification.
  * `referral_code` (VARCHAR(8)): Unique referral code.
  * `referral_redeemed` (BOOLEAN, Default: FALSE): Whether the referral has been redeemed.
  * `subscription_status` (VARCHAR(50), Default: 'inactive'): Subscription status (e.g., active, inactive).

#### 2. **Questions**

* **Description**: Stores SAT practice questions.
* **Attributes**:
  * `id` (VARCHAR(36), Primary Key): Unique identifier for the question.
  * `domain` (VARCHAR(50)): Domain (e.g., Math, Reading & Writing).
  * `skill` (VARCHAR(50)): Specific skill (e.g., Algebra, Grammar).
  * `difficulty` (INTEGER): Difficulty level (e.g., 1-5).
  * `question_text` (TEXT): The question text.
  * `options` (JSONB): Answer options in JSON format.
  * `correct_answer` (VARCHAR(255)): Correct answer.
  * `explanation` (TEXT): Explanation for the correct answer.
  * `test_type` (VARCHAR(50), Default: 'SAT'): Type of test (e.g., SAT, PSAT).
  * `question_type` (VARCHAR(50)): Type of question (e.g., multiple-choice, grid-in).

#### 3. **Practice\_Sessions**

* **Description**: Tracks practice or diagnostic sessions.
* **Attributes**:
  * `session_id` (VARCHAR(36), Primary Key): Unique identifier for the session.
  * `user_id` (VARCHAR(36), Foreign Key → Users(user\_id)): User who started the session.
  * `test_type` (VARCHAR(50)): Type of test (e.g., SAT, PSAT).
  * `section` (VARCHAR(50)): Section (e.g., Math, Reading & Writing).
  * `device` (VARCHAR(50)): Device used (e.g., web, mobile).
  * `theta` (FLOAT): Adaptive scoring metric (IRT theta value).
  * `timestamp` (TIMESTAMP, Default: CURRENT\_TIMESTAMP): Session start time.

#### 4. **Responses**

* **Description**: Stores user responses to questions in a session.
* **Attributes**:
  * `response_id` (VARCHAR(36), Primary Key): Unique identifier for the response.
  * `session_id` (VARCHAR(36), Foreign Key → Practice\_Sessions(session\_id)): Session the response belongs to.
  * `question_id` (VARCHAR(36), Foreign Key → Questions(id)): Question answered.
  * `answer` (VARCHAR(255)): User’s answer.
  * `time_spent` (INTEGER): Time spent on the question (in seconds).
  * `domain` (VARCHAR(50)): Domain of the question.
  * `theta` (FLOAT): Theta value for the response.

#### 5. **Proficiencies**

* **Description**: Tracks user proficiency in specific skills.
* **Attributes**:
  * `id` (VARCHAR(36), Primary Key): Unique identifier for the proficiency record.
  * `user_id` (VARCHAR(36), Foreign Key → Users(user\_id)): User associated with the proficiency.
  * `domain` (VARCHAR(50)): Domain (e.g., Math, Reading & Writing).
  * `skill` (VARCHAR(50)): Specific skill (e.g., Algebra, Grammar).
  * `theta` (FLOAT): Proficiency score (IRT theta value).
  * `timestamp` (TIMESTAMP, Default: CURRENT\_TIMESTAMP): When the proficiency was recorded.

#### 6. **Study\_Plans**

* **Description**: Stores user study plans.
* **Attributes**:
  * `plan_id` (VARCHAR(36), Primary Key): Unique identifier for the study plan.
  * `user_id` (VARCHAR(36), Foreign Key → Users(user\_id)): User associated with the plan.
  * `test_date` (VARCHAR(50)): Target SAT test date.

#### 7. **Study\_Plan\_Actions**

* **Description**: Stores tasks within a study plan.
* **Attributes**:
  * `id` (VARCHAR(36), Primary Key): Unique identifier for the action.
  * `plan_id` (VARCHAR(36), Foreign Key → Study\_Plans(plan\_id)): Study plan the action belongs to.
  * `task` (VARCHAR(255)): Task description (e.g., "Complete 5 Algebra sessions").
  * `action` (VARCHAR(50)): Action type (e.g., Complete).
  * `due_date` (VARCHAR(50)): Due date for the task.
  * `points` (INTEGER): Points awarded for completing the task.
  * `completed` (TIMESTAMP): When the task was completed (null if not completed).

#### 8. **Tutor\_Interactions**

* **Description**: Logs interactions with the AI tutor.
* **Attributes**:
  * `id` (VARCHAR(36), Primary Key): Unique identifier for the interaction.
  * `user_id` (VARCHAR(36), Foreign Key → Users(user\_id)): User who interacted with the tutor.
  * `query` (TEXT): User’s query to the AI tutor.
  * `response` (TEXT): AI tutor’s response.
  * `duration` (INTEGER): Duration of the interaction (in seconds).
  * `timestamp` (TIMESTAMP, Default: CURRENT\_TIMESTAMP): When the interaction occurred.

#### 9. **Posts**

* **Description**: Stores community posts.
* **Attributes**:
  * `id` (VARCHAR(36), Primary Key): Unique identifier for the post.
  * `user_id` (VARCHAR(36), Foreign Key → Users(user\_id)): User who created the post.
  * `content` (TEXT): Post content.
  * `timestamp` (TIMESTAMP, Default: CURRENT\_TIMESTAMP): When the post was created.

#### 10. **Friends**

* **Description**: Manages friendships between users.
* **Attributes**:
  * `id` (VARCHAR(36), Primary Key): Unique identifier for the friendship.
  * `user_id` (VARCHAR(36), Foreign Key → Users(user\_id)): User who initiated the friendship.
  * `friend_id` (VARCHAR(36), Foreign Key → Users(user\_id)): User who is the friend.
  * `status` (VARCHAR(50)): Friendship status (e.g., pending, accepted).
  * `timestamp` (TIMESTAMP, Default: CURRENT\_TIMESTAMP): When the friendship was initiated.

#### 11. **Teams**

* **Description**: Stores study teams for collaborative learning.
* **Attributes**:
  * `id` (VARCHAR(36), Primary Key): Unique identifier for the team.
  * `team_name` (VARCHAR(255)): Name of the team.
  * `user_id` (VARCHAR(36), Foreign Key → Users(user\_id)): User who created the team.
  * `points` (INTEGER, Default: 0): Team’s total points.
  * `timestamp` (TIMESTAMP, Default: CURRENT\_TIMESTAMP): When the team was created.

***

### Relationships

1. **Users ↔ Practice\_Sessions**
   * **Type**: One-to-Many
   * **Description**: A user can have multiple practice sessions.
   * **Foreign Key**: `Practice_Sessions.user_id` references `Users.user_id`.
2. **Practice\_Sessions ↔ Responses**
   * **Type**: One-to-Many
   * **Description**: A practice session can have multiple responses.
   * **Foreign Key**: `Responses.session_id` references `Practice_Sessions.session_id`.
3. **Questions ↔ Responses**
   * **Type**: One-to-Many
   * **Description**: A question can have multiple responses (from different users/sessions).
   * **Foreign Key**: `Responses.question_id` references `Questions.id`.
4. **Users ↔ Proficiencies**
   * **Type**: One-to-Many
   * **Description**: A user can have multiple proficiency records.
   * **Foreign Key**: `Proficiencies.user_id` references `Users.user_id`.
5. **Users ↔ Study\_Plans**
   * **Type**: One-to-Many
   * **Description**: A user can have multiple study plans.
   * **Foreign Key**: `Study_Plans.user_id` references `Users.user_id`.
6. **Study\_Plans ↔ Study\_Plan\_Actions**
   * **Type**: One-to-Many
   * **Description**: A study plan can have multiple actions (tasks).
   * **Foreign Key**: `Study_Plan_Actions.plan_id` references `Study_Plans.plan_id`.
7. **Users ↔ Tutor\_Interactions**
   * **Type**: One-to-Many
   * **Description**: A user can have multiple interactions with the AI tutor.
   * **Foreign Key**: `Tutor_Interactions.user_id` references `Users.user_id`.
8. **Users ↔ Posts**
   * **Type**: One-to-Many
   * **Description**: A user can create multiple posts.
   * **Foreign Key**: `Posts.user_id` references `Users.user_id`.
9. **Users ↔ Friends**
   * **Type**: Many-to-Many (self-referential)
   * **Description**: Users can be friends with other users, managed through the `Friends` table.
   * **Foreign Keys**:
     * `Friends.user_id` references `Users.user_id`.
     * `Friends.friend_id` references `Users.user_id`.
10. **Users ↔ Teams**
    * **Type**: One-to-Many
    * **Description**: A user can create multiple teams.
    * **Foreign Key**: `Teams.user_id` references `Users.user_id`.

***

### Textual Representation of the ERD

Below is a textual representation of the ERD, showing entities, their attributes, and relationships. You can use this to create a visual diagram in a tool like Lucidchart.

```
[Users]
  user_id (PK)
  email (UNIQUE, NOT NULL)
  hashed_password (NOT NULL)
  full_name
  grade
  school
  sat_taken
  sat_math_score
  sat_reading_score
  sat_test_date
  study_hours_per_week
  study_days_per_week
  preferred_study_time
  onboarding_completed (DEFAULT: FALSE)
  theme (DEFAULT: 'light')
  device_token
  points (DEFAULT: 0)
  streak (DEFAULT: 0)
  league (DEFAULT: 'Bronze')
  coins (DEFAULT: 0)
  referral_code
  referral_redeemed (DEFAULT: FALSE)
  subscription_status (DEFAULT: 'inactive')
  |
  | 1:N
  |
[Practice_Sessions]
  session_id (PK)
  user_id (FK → Users.user_id)
  test_type
  section
  device
  theta
  timestamp (DEFAULT: CURRENT_TIMESTAMP)
  |
  | 1:N
  |
[Responses]
  response_id (PK)
  session_id (FK → Practice_Sessions.session_id)
  question_id (FK → Questions.id)
  answer
  time_spent
  domain
  theta
  |
  | N:1
  |
[Questions]
  id (PK)
  domain
  skill
  difficulty
  question_text
  options (JSONB)
  correct_answer
  explanation
  test_type (DEFAULT: 'SAT')
  question_type

[Users]
  |
  | 1:N
  |
[Proficiencies]
  id (PK)
  user_id (FK → Users.user_id)
  domain
  skill
  theta
  timestamp (DEFAULT: CURRENT_TIMESTAMP)

[Users]
  |
  | 1:N
  |
[Study_Plans]
  plan_id (PK)
  user_id (FK → Users.user_id)
  test_date
  |
  | 1:N
  |
[Study_Plan_Actions]
  id (PK)
  plan_id (FK → Study_Plans.plan_id)
  task
  action
  due_date
  points
  completed

[Users]
  |
  | 1:N
  |
[Tutor_Interactions]
  id (PK)
  user_id (FK → Users.user_id)
  query
  response
  duration
  timestamp (DEFAULT: CURRENT_TIMESTAMP)

[Users]
  |
  | 1:N
  |
[Posts]
  id (PK)
  user_id (FK → Users.user_id)
  content
  timestamp (DEFAULT: CURRENT_TIMESTAMP)

[Users]
  |
  | N:N (self-referential)
  |
[Friends]
  id (PK)
  user_id (FK → Users.user_id)
  friend_id (FK → Users.user_id)
  status
  timestamp (DEFAULT: CURRENT_TIMESTAMP)

[Users]
  |
  | 1:N
  |
[Teams]
  id (PK)
  team_name
  user_id (FK → Users.user_id)
  points (DEFAULT: 0)
  timestamp (DEFAULT: CURRENT_TIMESTAMP)
```

***

### Notes for Implementation

1. **Primary Keys**: All entities use `VARCHAR(36)` for primary keys, typically UUIDs generated by the app (e.g., using Python’s `uuid` library).
2. **Foreign Keys**: Relationships are enforced with foreign key constraints to maintain referential integrity.
3. **Data Types**:
   * `VARCHAR` and `TEXT` are used for strings.
   * `INTEGER` and `FLOAT` for numeric values.
   * `TIMESTAMP` for dates/times.
   * `JSONB` for structured data (e.g., question options).
4. **Indexes**: Consider adding indexes on frequently queried fields (e.g., `user_id`, `session_id`) to improve performance.
5. **Scalability**: The schema supports scalability, but for large datasets, you might need to shard the database or use a caching layer (e.g., Redis) for leaderboards and frequently accessed data.

***

### How to Use the ERD

* **For Testing**: The outsourcing team can use this ERD to understand the database structure, ensuring that API endpoints (e.g., `/practice/start`, `/study_plan/create`) correctly interact with the database. They can also verify data integrity (e.g., foreign key constraints) during testing.
* **For Implementation**: If additional features are added, the team can extend the schema (e.g., adding a `Comments` table for post comments) while maintaining relationships. The ERD ensures that new tables integrate seamlessly with existing ones.

You can visualize this ERD in a tool by creating boxes for each entity, listing their attributes, and drawing lines to represent relationships (e.g., 1:N, N:N). Let me know if you’d like further assistance with visualizing or extending the schema!
