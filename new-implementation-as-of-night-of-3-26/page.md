# Page

Below is an updated **Markdown file** that reflects the current state of the **SAT Prep Suite** as of March 26, 2025, incorporating all recent enhancements (e.g., 3-phase study plan, 6-8 full-length tests, dynamic updates, proficiency-based practices). It includes:

1. **Updated Feature List**: All functionalities, including recent additions.
2. **Component List**: Backend, frontend, and supporting tools.
3. **Key Programs**: Descriptions and purposes of critical programs.
4. **Architecture**: Frontend and backend structure.
5. **User Journeys**: Two examples (9-week and 12-week prep) showing component usage.



## SAT Prep Suite: Updated Features, Components, Architecture, and User Journeys

### Overview

The **SAT Prep Suite** is an adaptive, scalable SAT preparation platform designed to guide students from signup to test day. This document updates the feature list, components, and architecture based on enhancements as of March 26, 2025, and provides user journeys for 9-week and 12-week prep timelines.

***

### Updated Feature List

#### 1. User Management

* Signup, login, roles (student, tutor, parent), profile settings (hours, target score, test date).
* Tutor/parent linking for oversight.

#### 2. Diagnostic Testing

* Math (22 questions, 35 min), R\&W (27 questions, 32 min), adaptive via IRT 3PL.
* Scores: 200–800/section, AI-generated review.

#### 3. Full-Length Testing

* Math (44 questions, 70 min), R\&W (54 questions, 64 min), Both (98 questions, 134 min + break).
* Adaptive modules, scores 400–1600, AI review.

#### 4. Adaptive Practice

* 5–10 questions per skill, IRT-driven, based on current (\theta).
* No question repetition across sessions.

#### 5. Study Plan

* **3-Phase Structure**:
  * **Foundation (30%)**: Weak skills ((\theta < 4)), lessons, easy/medium practice, goal (\theta \geq 4).
  * **Skill Building (40%)**: Moderate skills ((\theta = 4–6)), intermediate/advanced practice, goal (\theta \geq 5–6).
  * **Test Readiness (30%)**: Strategy/stamina, full tests, goal target score ± 50.
* Inputs: Diagnostic scores, target score, test date, skill gaps, learning style, weekly hours.
* Dynamic updates post-full-test based on (\theta).
* 6-8 full-length tests, frequency increases (2-week gap → 2/week).

#### 6. Gamification

* Points (e.g., 200/test, 50/practice), badges, leaderboards, streaks.

#### 7. AI Tutor

* Real-time chat (WebSocket), voice input, predictive scoring, interaction logging.

#### 8. Detailed Analytics & Insights

* Trends ((\theta) history), pacing, accuracy, predicted scores, actionable insights.

#### 9. Tutor & Parent Integration

* Analytics access, notifications, tutor action logging (e.g., suggestions).

#### 10. Offline Mode

* Cached questions, plans, chat history, sync on reconnect.

#### 11. Scalability

* PostgreSQL, Docker, Redis for high concurrency.

#### 12. UX Improvements

* Light/dark themes, animations, accessibility.

***

### Component List

#### Backend (FastAPI, Python)

* **Database**: PostgreSQL (`database.py`)
  * Tables: `users`, `questions`, `practice_sessions`, `responses`, `proficiencies`, `study_plans`, `study_plan_actions`, `badges`, `notifications`, `tutor_interactions`, `tutor_actions`.
* **Modules**:
  * `auth.py`: Authentication, roles.
  * `diagnostic.py`: Diagnostic tests.
  * `full_length_test.py`: Full SAT simulations.
  * `study_plan.py`: 3-phase study plan, dynamic updates.
  * `practice_module.py`: Adaptive practice sessions.
  * `gamification.py`: Points, badges, leaderboards.
  * `progress_monitoring.py`: Analytics, insights.
  * `ai_tutor.py`: Conversational AI, interaction logging.
  * `question_review.py`: AI test reviews.
  * `tutor_parent.py`: Tutor/parent access, notifications.
  * `sync.py`: Offline sync.
  * `utils.py`: Helpers (e.g., `select_next_question`, `get_ai_feedback`).
* **Config**: `main.py`, `.env`, `Dockerfile`, `docker-compose.yml`.

#### Frontend (Web: Next.js, Mobile: React Native)

* **Web**:
  * `login.js`: Signup/login UI.
  * `practice.js`: Practice interface.
  * `diagnostic.js`: Diagnostic UI.
  * `full-test.js`: Full-test UI.
  * `study-plan.js`: Plan display.
  * `dashboard/analytics.js`: Analytics dashboard.
  * `review.js`: Test review UI.
  * `tutor-parent.js`: Tutor/parent dashboard.
  * `api.js`: API client, caching.
  * `theme.js`: Theme management.
* **Mobile**:
  * `Login.js`, `Practice.js`, `Diagnostic.js`, `FullTest.js`, `StudyPlan.js`, `Review.js`: Mobile equivalents.
  * `App.js`: Entry point.
* **Dependencies**: `axios`, `framer-motion`, `react-chartjs-2`, `react-native-voice`, `AsyncStorage`.

#### Supporting Tools

* **Redis**: Caches chat, streaks.
* **Docker**: Containers (app, PostgreSQL, Redis).
* **Migrations**: `init_db.py`, `seed_data.py`.

***

### Key Programs

1. **`auth.py`**: Manages user authentication, roles.
2. **`diagnostic.py`**: Runs initial adaptive test, sets baseline (\theta).
3. **`full_length_test.py`**: Simulates SAT, updates plan via `study_plan.py`.
4. **`study_plan.py`**: Creates/updates 3-phase plan, schedules 6-8 tests, adapts tasks to (\theta).
5. **`practice_module.py`**: Delivers adaptive practice, updates (\theta).
6. **`gamification.py`**: Tracks points, badges, motivates engagement.
7. **`progress_monitoring.py`**: Analyzes (\theta), pacing, predicts scores.
8. **`ai_tutor.py`**: Provides real-time help, logs interactions.
9. **`question_review.py`**: Generates AI feedback post-test.
10. **`tutor_parent.py`**: Shares analytics, logs tutor actions.
11. **`sync.py`**: Handles offline functionality.
12. **`utils.py`**: Core logic (e.g., IRT question selection).

***

### Architecture

#### Backend (FastAPI)

* **Structure**:
  * **API Layer**: FastAPI routes (`auth.py`, `study_plan.py`, etc.) handle requests.
  * **Business Logic**: Modules process data (e.g., `utils.py` for IRT, `study_plan.py` for scheduling).
  * **Database Layer**: SQLAlchemy ORM (`database.py`) interacts with PostgreSQL.
  * **Caching**: Redis stores chat context, streaks.
* **Flow**:
  * Request → Route → Logic → DB/Redis → Response.
  * E.g., `/study_plan/create` → `study_plan.py` → `proficiencies` → Plan generation → Response.
* **Scalability**: Docker containers, PostgreSQL for persistence, Redis for speed.

#### Frontend (Next.js/React Native)

* **Structure**:
  * **Web (Next.js)**:
    * Pages (`login.js`, `study-plan.js`) render UI.
    * Components (`dashboard/analytics.js`) display data.
    * API client (`api.js`) fetches data, caches offline.
  * **Mobile (React Native)**:
    * Screens (`Login.js`, `Practice.js`) mirror web.
    * `App.js` manages navigation.
    * `AsyncStorage` for offline caching.
* **Flow**:
  * User action → API call → State update → UI render.
  * E.g., Complete practice → `Practice.js` → `/practice/submit` → Update (\theta) → Refresh UI.
* **UX**: Themes (`theme.js`), animations (`framer-motion`), voice input (`react-native-voice`).

***

### User Journey: 9-Week Prep (Alex)

* **Profile**: Alex, goal 1400, 15 hrs/week, test date June 1, 2025, starts March 26.
* **Baseline**: Math 620 ((\theta = 0.8)), R\&W 600 ((\theta = 0.6)), weak in Geometry (-0.5), Vocabulary (-1.0).

#### Week 1-3: Foundation (March 26 - April 15)

* **Signup**: `login.js` → `auth.py` → Account `alex123`, linked to `parent123`, `tutor123`.
* **Diagnostic**: `diagnostic.js` → `diagnostic.py` → 620/600, `question_review.py` → Feedback: "Review Geometry."
* **Plan**: `study-plan.js` → `study_plan.py` → Tasks:
  * "Lesson: Geometry": `Practice.js` → `practice_module.py` → Triangles lesson, 25 pts.
  * "Practice: Vocabulary": 10 easy MCQs ((\theta = -1.0)), 50 pts.
* **Full Test (Week 3)**: `full-test.js` → `full_length_test.py` → April 14, 650/620, `study_plan.py` updates (\theta).

#### Week 4-6: Skill Building (April 16 - May 6)

* **Practice**: `Practice.js` → `practice_module.py` → "Algebra" 10 intermediate questions ((\theta = 0.8)), 50 pts.
* **Analytics**: `analytics.js` → `progress_monitoring.py` → (\theta\_{\text{Geometry\}} = 0.2), "Focus on pacing."
* **Full Tests**: 3 tests (April 28, May 5, May 12), `full_length_test.py` → Scores rise to 1320, `tutor_parent.py` notifies parent.
* **AI Tutor**: `Practice.js` → `ai_tutor.py` → "How to factor?" → Logged in `tutor_interactions`.

#### Week 7-9: Test Readiness (May 7 - June 1)

* **Strategy**: `Practice.js` → `practice_module.py` → 5 advanced Math questions ((\theta = 1.5)), 25 pts.
* **Full Tests**: 3 tests (May 16, May 18, May 30), `full_length_test.py` → 1380, `question_review.py` → "Mastered Algebra."
* **Tutor**: `tutor-parent.js` → `tutor_parent.py` → Suggests "Vocabulary drills," logged in `tutor_actions`.
* **SAT**: June 1, scores 1410.

***

### User Journey: 12-Week Prep (Bella)

* **Profile**: Bella, goal 1500, 20 hrs/week, test date June 15, 2025, starts March 25.
* **Baseline**: Math 700 ((\theta = 1.5)), R\&W 650 ((\theta = 1.0)), weak in Evidence-Based Analysis (0.5).

#### Week 1-4: Foundation (March 25 - April 21)

* **Signup**: `Login.js` → `auth.py` → `bella456`, linked to tutor.
* **Diagnostic**: `Diagnostic.js` → `diagnostic.py` → 700/650, `question_review.py` → "Focus on Evidence."
* **Plan**: `StudyPlan.js` → `study_plan.py` → Tasks:
  * "Lesson: Evidence-Based Analysis": `Practice.js` → `practice_module.py` → Reading Comprehension, 25 pts.
  * "Practice: Evidence": 10 easy SPRs ((\theta = 0.5)), 50 pts.
* **Full Test (Week 4)**: `FullTest.js` → `full_length_test.py` → April 20, 720/670, `study_plan.py` updates.

#### Week 5-9: Skill Building (April 22 - May 26)

* **Practice**: `Practice.js` → `practice_module.py` → "Geometry" 10 intermediate questions ((\theta = 1.5)), 50 pts.
* **Analytics**: `analytics.js` → `progress_monitoring.py` → Predicted 1400, "Improve R\&W stamina."
* **Full Tests**: 3 tests (May 3, May 17, May 24), `full_length_test.py` → 1420, `gamification.py` → "Skill Master" badge.
* **Offline**: `sync.py` → Practices cached during travel.

#### Week 10-12: Test Readiness (May 27 - June 15)

* **Strategy**: `Practice.js` → `practice_module.py` → 5 advanced R\&W questions ((\theta = 1.8)), 25 pts.
* **Full Tests**: 3 tests (June 2, June 4, June 13), `full_length_test.py` → 1480, `ai_tutor.py` → "Evidence tips."
* **Tutor**: `tutor-parent.js` → `tutor_parent.py` → Reviews progress, suggests focus.
* **SAT**: June 15, scores 1500.

***

### Conclusion

The updated **SAT Prep Suite** ensures 6-8 full-length tests (7 in both journeys), integrates all components across a 3-phase plan, and scales via a robust architecture. Alex (9 weeks) and Bella (12 weeks) leverage every feature to hit their goals—deploy-ready for testing!

***

***

### Notes

* **Full Tests**: Both journeys have 7 tests, dynamically scheduled (e.g., Alex: April 14, April 28, May 5, May 12, May 16, May 18, May 30; Bella: adjusted for 12 weeks).
* **Components**: All key programs are utilized, with `study_plan.py` driving the 3-phase dynamic updates.
* **File**: Save as `updated_features_components_journeys.md`.

Let me know if you’d like to tweak test counts (e.g., exactly 6 or 8) or add more detail to any section!
