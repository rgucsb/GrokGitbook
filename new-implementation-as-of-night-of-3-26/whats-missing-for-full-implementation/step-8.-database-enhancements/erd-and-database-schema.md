# ERD & Database Schema

To provide a clear understanding of the **SAT Prep Suite** database schema and its Entity-Relationship Diagram (ERD) as implemented in `database.py`, I’ll first present the schema in a structured textual format, detailing each table’s columns, data types, constraints, and relationships. Then, I’ll describe the ERD in a textual representation (since I can’t generate visual diagrams directly), showing entities, attributes, and relationships. This reflects the fully enhanced database from **Item 8** as of March 27, 2025.

***

### Database Schema

#### Tables and Columns

1. **`users`**
   * **Purpose**: Stores user information (students, tutors, parents).
   * **Columns**:
     * `user_id` (String, Primary Key, Indexed): Unique identifier.
     * `email` (String, Unique, Indexed): User’s email.
     * `hashed_password` (String): Password (hashed in production).
     * `role` (String, Default: "student"): Role (student, tutor, parent).
     * `study_hours` (Integer): Weekly study hours.
     * `points` (Integer, Default: 0): Gamification points.
     * `streak` (Integer, Default: 0): Daily streak count.
     * `league` (String, Default: "Bronze"): Gamification league.
     * `linked_user_id` (String, Foreign Key: `users.user_id`, Nullable): Links tutors/parents to students.
   * **Relationships**:
     * One-to-Many: `responses`, `proficiencies`, `sessions`, `badges`, `notifications`, `tutor_interactions`, `posts`, `comments`.
     * Many-to-Many: `friends` (via `user_id` and `friend_id`).
2. **`questions`**
   * **Purpose**: Stores SAT questions.
   * **Columns**:
     * `question_id` (String, Primary Key, Indexed): Unique identifier.
     * `domain` (String): Category (e.g., Math, Reading & Writing).
     * `skill` (String): Specific skill (e.g., Algebra).
     * `text` (String): Question content.
     * `options` (String, Nullable): JSON string for MCQ options.
     * `correct_answer` (String): Correct answer.
     * `a_param` (Float, Default: 1.0): IRT discrimination.
     * `b_param` (Float, Default: 0.0): IRT difficulty.
     * `c_param` (Float, Default: 0.25): IRT guessing.
   * **Relationships**:
     * One-to-Many: `responses`.
3. **`responses`**
   * **Purpose**: Records user answers to questions.
   * **Columns**:
     * `id` (Integer, Primary Key, Indexed): Unique identifier.
     * `session_id` (String, Foreign Key: `practice_sessions.session_id`): Links to session.
     * `user_id` (String, Foreign Key: `users.user_id`): Links to user.
     * `question_id` (String, Foreign Key: `questions.question_id`): Links to question.
     * `answer` (String): User’s answer.
     * `time_spent` (Integer): Time in seconds.
     * `timestamp` (DateTime, Default: `now()`): When answered.
     * `review_text` (String, Nullable): AI-generated feedback.
   * **Relationships**:
     * Many-to-One: `user`, `question`, `session`.
4. **`proficiencies`**
   * **Purpose**: Tracks user skill proficiency.
   * **Columns**:
     * `id` (Integer, Primary Key, Indexed): Unique identifier.
     * `user_id` (String, Foreign Key: `users.user_id`): Links to user.
     * `domain` (String): Category.
     * `skill` (String): Specific skill.
     * `theta` (Float): IRT proficiency score.
     * `timestamp` (DateTime, Default: `now()`): When recorded.
   * **Relationships**:
     * Many-to-One: `user`.
5. **`practice_sessions`**
   * **Purpose**: Manages practice/test sessions.
   * **Columns**:
     * `session_id` (String, Primary Key, Indexed): Unique identifier.
     * `user_id` (String, Foreign Key: `users.user_id`): Links to user.
     * `theta` (Float, Default: 0.0): Session proficiency score.
     * `test_type` (String): "diagnostic", "full", "practice".
     * `section` (String, Nullable): Sections (e.g., "Math").
     * `plan_id` (String, Foreign Key: `study_plans.plan_id`, Nullable): Links to study plan.
   * **Relationships**:
     * Many-to-One: `user`, `plan`.
     * One-to-Many: `responses`.
6. **`study_plans`**
   * **Purpose**: Stores user study plans.
   * **Columns**:
     * `plan_id` (String, Primary Key, Indexed): Unique identifier.
     * `user_id` (String, Foreign Key: `users.user_id`): Links to user.
     * `test_date` (DateTime): SAT test date.
   * **Relationships**:
     * One-to-Many: `study_plan_actions`, `sessions`.
7. **`study_plan_actions`**
   * **Purpose**: Tracks tasks in study plans.
   * **Columns**:
     * `id` (Integer, Primary Key, Indexed): Unique identifier.
     * `plan_id` (String, Foreign Key: `study_plans.plan_id`): Links to plan.
     * `task` (String): Task name (e.g., "Practice Algebra").
     * `action` (String): Task description.
     * `due_date` (DateTime): Due date.
     * `points` (Integer): Points awarded.
     * `completed` (DateTime, Nullable): Completion timestamp.
   * **Relationships**:
     * Many-to-One: `plan`.
8. **`badges`**
   * **Purpose**: Stores user badges for gamification.
   * **Columns**:
     * `id` (Integer, Primary Key, Indexed): Unique identifier.
     * `user_id` (String, Foreign Key: `users.user_id`): Links to user.
     * `name` (String): Badge name (e.g., "Beginner").
     * `awarded_at` (DateTime, Default: `now()`): When awarded.
   * **Relationships**:
     * Many-to-One: `user`.
9. **`notifications`**
   * **Purpose**: Stores user notifications.
   * **Columns**:
     * `id` (Integer, Primary Key, Indexed): Unique identifier.
     * `user_id` (String, Foreign Key: `users.user_id`): Links to user.
     * `message` (String): Notification text.
     * `timestamp` (DateTime, Default: `now()`): When sent.
     * `read` (Boolean, Default: False): Read status.
   * **Relationships**:
     * Many-to-One: `user`.
10. **`tutor_interactions`**
    * **Purpose**: Logs AI tutor interactions.
    * **Columns**:
      * `id` (Integer, Primary Key, Indexed): Unique identifier.
      * `user_id` (String, Foreign Key: `users.user_id`): Links to user.
      * `query` (String): User’s question/input.
      * `response` (String): AI response.
      * `timestamp` (DateTime, Default: `now()`): When occurred.
      * `duration` (Integer): Time in seconds.
    * **Relationships**:
      * Many-to-One: `user`.
11. **`tutor_actions`**
    * **Purpose**: Tracks tutor actions on student data.
    * **Columns**:
      * `id` (Integer, Primary Key, Indexed): Unique identifier.
      * `tutor_id` (String, Foreign Key: `users.user_id`): Tutor’s ID.
      * `student_id` (String, Foreign Key: `users.user_id`): Student’s ID.
      * `action` (String): Action type (e.g., "Viewed analytics").
      * `details` (String, Nullable): Additional info.
      * `timestamp` (DateTime, Default: `now()`): When occurred.
    * **Relationships**:
      * Many-to-One: `tutor`, `student` (both link to `users`).
12. **`posts`**
    * **Purpose**: Stores social posts.
    * **Columns**:
      * `id` (Integer, Primary Key, Indexed): Unique identifier.
      * `user_id` (String, Foreign Key: `users.user_id`): Links to user.
      * `content` (String): Post text.
      * `timestamp` (DateTime, Default: `now()`): When posted.
    * **Relationships**:
      * Many-to-One: `user`.
      * One-to-Many: `comments`.
13. **`comments`**
    * **Purpose**: Stores comments on posts.
    * **Columns**:
      * `id` (Integer, Primary Key, Indexed): Unique identifier.
      * `post_id` (Integer, Foreign Key: `posts.id`): Links to post.
      * `user_id` (String, Foreign Key: `users.user_id`): Links to user.
      * `content` (String): Comment text.
      * `timestamp` (DateTime, Default: `now()`): When commented.
    * **Relationships**:
      * Many-to-One: `post`, `user`.
14. **`friends`**
    * **Purpose**: Manages friendships.
    * **Columns**:
      * `id` (Integer, Primary Key, Indexed): Unique identifier.
      * `user_id` (String, Foreign Key: `users.user_id`): Requester’s ID.
      * `friend_id` (String, Foreign Key: `users.user_id`): Recipient’s ID.
      * `status` (String, Default: "pending"): "pending", "accepted".
      * `timestamp` (DateTime, Default: `now()`): When requested.
    * **Relationships**:
      * Many-to-One: `user` (requester), `friend` (recipient).

***

### Entity-Relationship Diagram (ERD) - Textual Representation

#### Entities and Attributes

1. **User**
   * Attributes: `user_id (PK)`, `email`, `hashed_password`, `role`, `study_hours`, `points`, `streak`, `league`, `linked_user_id (FK)`
   * Relationships:
     * 1:N with `responses`, `proficiencies`, `practice_sessions`, `badges`, `notifications`, `tutor_interactions`, `posts`, `comments`
     * N:N with `friends` (self-referencing via `user_id` and `friend_id`)
     * 1:N with `tutor_actions` (as `tutor_id` or `student_id`)
2. **Question**
   * Attributes: `question_id (PK)`, `domain`, `skill`, `text`, `options`, `correct_answer`, `a_param`, `b_param`, `c_param`
   * Relationships:
     * 1:N with `responses`
3. **Response**
   * Attributes: `id (PK)`, `session_id (FK)`, `user_id (FK)`, `question_id (FK)`, `answer`, `time_spent`, `timestamp`, `review_text`
   * Relationships:
     * N:1 with `user`, `question`, `practice_session`
4. **Proficiency**
   * Attributes: `id (PK)`, `user_id (FK)`, `domain`, `skill`, `theta`, `timestamp`
   * Relationships:
     * N:1 with `user`
5. **PracticeSession**
   * Attributes: `session_id (PK)`, `user_id (FK)`, `theta`, `test_type`, `section`, `plan_id (FK)`
   * Relationships:
     * N:1 with `user`, `study_plan`
     * 1:N with `responses`
6. **StudyPlan**
   * Attributes: `plan_id (PK)`, `user_id (FK)`, `test_date`
   * Relationships:
     * 1:N with `study_plan_actions`, `practice_sessions`
7. **StudyPlanAction**
   * Attributes: `id (PK)`, `plan_id (FK)`, `task`, `action`, `due_date`, `points`, `completed`
   * Relationships:
     * N:1 with `study_plan`
8. **Badge**
   * Attributes: `id (PK)`, `user_id (FK)`, `name`, `awarded_at`
   * Relationships:
     * N:1 with `user`
9. **Notification**
   * Attributes: `id (PK)`, `user_id (FK)`, `message`, `timestamp`, `read`
   * Relationships:
     * N:1 with `user`
10. **TutorInteraction**
    * Attributes: `id (PK)`, `user_id (FK)`, `query`, `response`, `timestamp`, `duration`
    * Relationships:
      * N:1 with `user`
11. **TutorAction**
    * Attributes: `id (PK)`, `tutor_id (FK)`, `student_id (FK)`, `action`, `details`, `timestamp`
    * Relationships:
      * N:1 with `user` (via `tutor_id` and `student_id`)
12. **Post**
    * Attributes: `id (PK)`, `user_id (FK)`, `content`, `timestamp`
    * Relationships:
      * N:1 with `user`
      * 1:N with `comments`
13. **Comment**
    * Attributes: `id (PK)`, `post_id (FK)`, `user_id (FK)`, `content`, `timestamp`
    * Relationships:
      * N:1 with `post`, `user`
14. **Friend**
    * Attributes: `id (PK)`, `user_id (FK)`, `friend_id (FK)`, `status`, `timestamp`
    * Relationships:
      * N:1 with `user` (via `user_id` and `friend_id`)

#### Relationships

* **User ↔ Responses**: One user has many responses; one response belongs to one user.
* **User ↔ PracticeSessions**: One user has many sessions; one session belongs to one user.
* **User ↔ StudyPlans**: One user has many plans; one plan belongs to one user.
* **User ↔ Badges**: One user has many badges; one badge belongs to one user.
* **User ↔ Notifications**: One user has many notifications; one notification belongs to one user.
* **User ↔ TutorInteractions**: One user has many interactions; one interaction belongs to one user.
* **User ↔ Posts**: One user has many posts; one post belongs to one user.
* **User ↔ Comments**: One user has many comments; one comment belongs to one user.
* **User ↔ Friends**: Many-to-many self-referencing (user sends/receives friend requests).
* **Question ↔ Responses**: One question has many responses; one response belongs to one question.
* **PracticeSession ↔ Responses**: One session has many responses; one response belongs to one session.
* **StudyPlan ↔ StudyPlanActions**: One plan has many actions; one action belongs to one plan.
* **StudyPlan ↔ PracticeSessions**: One plan has many sessions; one session belongs to one plan.
* **Post ↔ Comments**: One post has many comments; one comment belongs to one post.
* **TutorAction**: Links two users (tutor and student).

#### ERD Textual Diagram

```
[User] ---1:N---> [Responses] ---N:1---> [Question]
   |                |
   |                N:1
   |                |
   1:N            [PracticeSession] ---1:N---> [Responses]
   |                |
   |                N:1
   |                |
   1:N            [StudyPlan] ---1:N---> [StudyPlanActions]
   |
   1:N
   |
[Badges] ---N:1---> [User] ---1:N---> [Notifications]
   |
   1:N
   |
[TutorInteractions] ---N:1---> [User] <---N:1--- [TutorActions] ---N:1---> [User]
   |
   1:N
   |
[Posts] ---1:N---> [Comments] ---N:1---> [User]
   |
   N:N
   |
[Friends] ---N:1---> [User]
```

***

### Verification

* **Schema Completeness**: All 14 tables match feature requirements:
  * User management: `users`.
  * Practice/tests: `questions`, `responses`, `proficiencies`, `practice_sessions`.
  * Study plans: `study_plans`, `study_plan_actions`.
  * Gamification: `users.points`, `users.streak`, `users.league`, `badges`.
  * AI: `tutor_interactions`, `responses.review_text`.
  * Tutor/parent: `tutor_actions`, `notifications`.
  * Social: `posts`, `comments`, `friends`.
* **Relationships**: All foreign keys and ORM relationships implemented (e.g., `User.responses`, `Post.comments`).
* **Testing**:
  * Run `python migrations/init_db.py` → All tables created.
  * Test endpoints (e.g., `/practice/submit`, `/community/posts`) → Data populates correctly.

***

### Conclusion

* **Schema**: Fully defined with 14 tables, all fields, and relationships.
* **ERD**: Represents entities and relationships supporting all features.
* **Next Steps**: Move to **Item 9: Configuration and Scalability**.

The database is complete—review the schema/ERD and let me know if you need adjustments (e.g., additional indexes) before proceeding!
