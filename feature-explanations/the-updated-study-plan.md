# The updated Study Plan

Apologies for the interruption! Let’s complete the **Study Plan Documentation** for the **SAT Smart Prep App** by **Learner Labs**, continuing from where we left off. This documentation will cover all features, including the recent addition of **mastery targets**, as well as the implementation details and the roles of relevant files, without including any code. I’ll ensure the explanation is comprehensive, user-focused, and developer-friendly, detailing the functionality, structure, and technical implementation.

***

### SAT Smart Prep App: Study Plan Documentation (Continued)

#### 1. Personalized Task Scheduling (Continued)

* **Diagnostic Test Results**: Uses the user’s performance on the Diagnostic Test (49 questions: 27 Reading and Writing, 22 Math) to calculate initial `theta` scores (a measure of proficiency ranging from -3 to 3) for each domain (Reading & Writing, Math) and subskill (e.g., Grammar, Algebra). These scores identify the user’s strengths and weaknesses, ensuring the Study Plan prioritizes areas needing improvement. For example, if a user’s `theta` for Geometry is low (e.g., -0.6, corresponding to 40% mastery), the Study Plan schedules more Geometry practice sessions.
* **Task Allocation**: Tasks are dynamically allocated based on the user’s current proficiency, performance trends, and mastery targets. Subskills with larger gaps between current mastery and target mastery receive more practice sessions, ensuring the user progresses toward their goals efficiently.

#### 2. Three-Phase Structure

The Study Plan is organized into three distinct phases, each with a specific focus to ensure a progressive preparation journey:

* **Phase 1: Foundation Building (40% of the preparation time)**:
  * **Focus**: Build a strong foundation by addressing core weaknesses and reviewing fundamental concepts.
  * **Tasks**: Includes short practice sessions (e.g., "Complete a 5-question session in Grammar") targeting weak subskills, pacing exercises for subskills with excessive time spent (e.g., "Practice pacing for Reading Comprehension"), and 2 full-length SAT practice tests (49 questions, 72 minutes each) to familiarize users with the test format early.
* **Phase 2: Practice and Strategy (30% of the preparation time)**:
  * **Focus**: Apply concepts in test-like conditions and develop test-taking strategies such as time management and question elimination.
  * **Tasks**: Includes longer practice sessions (e.g., "Complete a 10-question session in Algebra"), strategy-focused tasks (e.g., "Review strategies for Geometry" for subskills with slow improvement), and 2 full-length SAT practice tests to simulate the test experience.
* **Phase 3: Final Review and Refinement (30% of the preparation time)**:
  * **Focus**: Fine-tune skills, address remaining weaknesses, and build confidence for test day.
  * **Tasks**: Includes targeted review sessions (e.g., "Review incorrect answers from Grammar practice"), performance analysis for subskills with declining trends (e.g., "Review recent performance in Algebra"), test-day preparation (e.g., "Review test-day checklist"), and 4 full-length SAT practice tests to ensure readiness.

#### 3. Mastery Targets

The Study Plan now includes **mastery targets** for each domain and subskill, providing users with clear goals to achieve by their SAT test date. Mastery targets are expressed as percentages on the 0 to 100% scale, making them intuitive for users to understand.

* **Default Targets**: By default, domains (Reading & Writing, Math) have a target mastery of 80%, and subskills (e.g., Grammar, Algebra) have a target mastery of 75%. These defaults ensure users aim for a high level of proficiency across all areas.
* **Customizable Targets**: Users can customize mastery targets during onboarding or when adjusting the Study Plan, allowing them to set higher or lower goals based on their aspirations (e.g., setting a 90% target for Math if aiming for a top score).
* **Task Allocation Based on Mastery Gaps**: The Study Plan calculates the gap between the user’s current mastery and their target mastery for each domain and subskill. Subskills with larger gaps receive more practice sessions to help users close the gap. For example, if a user’s current mastery for Algebra is 40% and their target is 75%, the Study Plan prioritizes Algebra tasks to bridge the 35% gap.
* **Progress Tracking**: The Study Plan UI displays the user’s current mastery alongside their target mastery for each domain and subskill (e.g., "Algebra: Current Mastery 40%, Target 75%"). It also indicates when targets are achieved (e.g., "Target Achieved!" if current mastery meets or exceeds the target).

#### 4. Advanced Analytics Integration

The Study Plan uses advanced analytics to make task allocation more precise and effective, ensuring preparation is tailored to the user’s evolving needs.

* **Performance Trends**: Tracks changes in `theta` scores over time to identify subskills with improving, declining, or stagnant performance. Subskills with negative or stagnant trends (e.g., no improvement in Geometry over two weeks) are prioritized for additional practice.
* **Improvement Rates**: Calculates the rate of improvement for each subskill (e.g., `theta` change per week). Subskills with slow improvement rates (e.g., less than 0.1 `theta` per week) receive more practice sessions, while those with fast improvement (e.g., more than 0.2 `theta` per week) are deprioritized.
* **Time Spent per Question**: Identifies subskills where the user spends excessive time (e.g., more than 60 seconds per question), indicating potential difficulty or inefficiency. Pacing tasks (e.g., "Practice pacing for Reading Comprehension") are scheduled to address these issues, particularly in Phase 1.
* **Mastery Gap Analysis**: Uses the difference between the user’s current `theta` and the `theta` equivalent of their target mastery to adjust task allocation. Larger gaps result in more practice sessions for the corresponding subskill.

#### 5. Dynamic Full-Length Test Scheduling

The Study Plan includes a total of 8 full-length SAT practice tests, strategically scheduled across the three phases to ensure consistent exposure to the test format.

* **Phase 1**: 2 full-length tests, scheduled evenly to introduce users to the SAT format early.
* **Phase 2**: 2 full-length tests, scheduled to provide practice under test-like conditions as users refine their strategies.
* **Phase 3**: 4 full-length tests, scheduled more frequently in the final weeks to simulate the test experience and build endurance and confidence.
* **Dynamic Scheduling**: Tests are distributed evenly within each phase based on the number of weeks, ensuring users have regular opportunities to practice the full SAT format (49 questions, 72 minutes).

#### 6. Regular Updates Based on Progress

The Study Plan is dynamically updated to reflect the user’s latest performance, ensuring it remains relevant throughout their preparation.

* **Weekly Updates**: The app automatically updates the Study Plan every week, incorporating the latest `theta` scores, performance trends, and mastery gaps. This ensures tasks are reallocated to focus on new or persistent weak areas.
* **Post-Test Updates**: After completing a practice session or full-length test, the Study Plan is updated to reflect the user’s updated `theta` scores, adjusting the focus on subskills as needed.
* **Preservation of Completed Tasks**: Completed tasks are preserved during updates, maintaining the user’s sense of progress while reoptimizing remaining tasks.

#### 7. Gamification and Motivation

The Study Plan integrates with the app’s gamification features to keep users motivated and engaged.

* **Points for Tasks**: Each task is assigned points (e.g., 50 points for a 5-question session, 150 points for a full-length test). Completing tasks earns users points, contributing to their overall score and leaderboard ranking.
* **Rewards and Challenges**: Completing Study Plan tasks unlocks rewards (e.g., avatars, badges) and contributes to daily challenges (e.g., "Complete 3 Study Plan tasks today"), encouraging consistent engagement.
* **Progress Tracking**: The Study Plan UI displays progress toward mastery targets, showing users how close they are to achieving their goals (e.g., "Algebra: Current Mastery 40%, Target 75%").

#### 8. Flexibility and Adjustments

The Study Plan is designed to be flexible, allowing users to adjust their preparation as their schedule or goals change.

* **Manual Adjustments**: Users can update their study hours, study days, and mastery targets from the Study Plan section, triggering a recalculation of the schedule to fit their new preferences.
* **Adaptive Reallocation**: If a user’s performance improves significantly in a subskill (e.g., Grammar mastery increases from 55% to 80%), the Study Plan reduces focus on that subskill and reallocates sessions to other areas needing improvement (e.g., Geometry).

#### 9. Integration with Other App Features

The Study Plan works seamlessly with other app features to create a cohesive preparation experience.

* **Diagnostic Test**: The initial Study Plan is generated based on the Diagnostic Test results, ensuring it targets the user’s specific weaknesses from the start.
* **Practice Sessions**: Study Plan tasks link directly to practice sessions, allowing users to complete tasks like "Complete a 5-question session in Algebra" by navigating to the practice section.
* **Progress Monitoring**: The app tracks the user’s progress toward mastery targets, updating `theta` scores and analytics after each task, which informs future Study Plan updates.
* **Review Process**: After completing full-length tests, users can review their performance, and the Study Plan incorporates this feedback to schedule targeted review tasks (e.g., "Review incorrect answers from Grammar practice").

***

### Implementation Details

#### Feature Implementation

* **Personalized Task Scheduling**:
  * **User Inputs**: Collected during onboarding (via `frontend/pages/onboarding.js` for web, `SATSmartPrepApp/src/screens/OnboardingScreen.js` for mobile) and stored in the Study Plan request. These inputs include the SAT test date, study hours, study days, and mastery targets.
  * **Diagnostic Test Results**: Retrieved from the `Proficiencies` table in the database, which stores `theta` scores for each domain and subskill after the Diagnostic Test (handled by `backend/src/diagnostic.py`).
  * **Task Allocation**: The backend (`backend/src/study_plan.py`) calculates the number of weeks until the test date, divides the timeline into three phases, and allocates tasks based on `theta` scores, performance trends, improvement rates, pacing issues, and mastery gaps. Tasks are stored in the `Study_Plan_Actions` table.
* **Three-Phase Structure**:
  * **Phase Division**: The backend (`backend/src/study_plan.py`) divides the preparation timeline into three phases: 40% for Phase 1, 30% for Phase 2, and 30% for Phase 3. The number of weeks in each phase determines the number of tasks scheduled.
  * **Task Types**: Different task types are scheduled in each phase (e.g., short practice sessions in Phase 1, strategy tasks in Phase 2, review tasks in Phase 3), with full-length tests distributed as 2 in Phase 1, 2 in Phase 2, and 4 in Phase 3.
* **Mastery Targets**:
  * **Target Setting**: Default targets are set at 80% for domains and 75% for subskills, but users can customize these during onboarding or Study Plan adjustments. The targets are passed to the backend as part of the Study Plan request (`target_mastery` field).
  * **Task Allocation**: The backend converts target mastery percentages to `theta` scores (e.g., 80% mastery = `theta` 1.8) and calculates the gap between the user’s current `theta` and target `theta`. Subskills with larger gaps receive more practice sessions, weighted by the gap size.
  * **UI Display**: The Study Plan UI (`frontend/pages/study-plan.js` for web, `SATSmartPrepApp/src/screens/StudyPlanScreen.js` for mobile) displays current mastery and target mastery for each domain and subskill, along with progress indicators (e.g., "Target Achieved!" if the target is met).
* **Advanced Analytics Integration**:
  * **Performance Trends**: Calculated by comparing historical `theta` scores from the `Proficiencies` table, stored in the database and accessed via `backend/src/study_plan.py`.
  * **Improvement Rates**: Determined by analyzing the change in `theta` scores over time, also using data from the `Proficiencies` table.
  * **Time Spent per Question**: Computed by averaging the `time_spent` field from the `Responses` table, joined with the `Questions` table to map questions to subskills.
  * **Mastery Gap Analysis**: Uses the difference between current `theta` and target `theta` to adjust task weights, ensuring more practice for subskills with larger gaps.
* **Dynamic Full-Length Test Scheduling**:
  * The backend (`backend/src/study_plan.py`) schedules 8 full-length tests (2/2/4 across the three phases) by evenly distributing them within each phase based on the number of weeks. These tests are added as tasks in the `Study_Plan_Actions` table.
* **Regular Updates**:
  * **Weekly Updates**: A background task in the mobile app (`SATSmartPrepApp/App.js`) uses `react-native-background-fetch` to call the `/study_plan/update/{user_id}` endpoint weekly, updating the Study Plan with the latest `theta` scores and analytics.
  * **Post-Test Updates**: Triggered after practice sessions or full-length tests, calling the same `/study_plan/update/{user_id}` endpoint to refresh the Study Plan.
  * **Preservation of Completed Tasks**: The backend preserves completed tasks in the `Study_Plan_Actions` table during updates, deleting only uncompleted tasks before rescheduling.
* **Gamification and Motivation**:
  * **Points and Rewards**: Points are assigned to tasks in the backend (`backend/src/study_plan.py`) and stored in the `Study_Plan_Actions` table. The gamification logic (`backend/src/gamification.py`) handles rewards and leaderboard updates.
  * **Progress Tracking**: The Study Plan UI displays progress toward mastery targets, using data from the `current_mastery` and `target_mastery` fields returned by the backend.
* **Flexibility and Adjustments**:
  * **Manual Adjustments**: Users can adjust study hours, study days, and mastery targets via the Study Plan UI, which calls the `/study_plan/create/{user_id}` endpoint to regenerate the plan.
  * **Adaptive Reallocation**: The backend reoptimizes task allocation based on updated `theta` scores and mastery gaps during each update.
* **Integration with Other Features**:
  * **Diagnostic Test**: Results are fetched from the `Proficiencies` table (`backend/src/diagnostic.py`) to initialize the Study Plan.
  * **Practice Sessions**: Tasks link to practice sessions via the practice module (`backend/src/practice.py`, `frontend/pages/practice-bluebook.js`, `SATSmartPrepApp/src/screens/PracticeScreen.js`).
  * **Progress Monitoring**: Current mastery percentages are displayed using data from the `Proficiencies` table, integrated with the dashboard (`frontend/pages/dashboard.js`, `SATSmartPrepApp/src/screens/DashboardScreen.js`).
  * **Review Process**: Review tasks are scheduled based on performance data from the `Responses` table, accessed via the review endpoint (`backend/src/practice.py`, `frontend/pages/review.js`, `SATSmartPrepApp/src/screens/ReviewScreen.js`).

***

### Benefits of the Study Plan

#### Personalized and Data-Driven Preparation

The Study Plan uses diagnostic test results, user inputs, and advanced analytics (performance trends, improvement rates, time spent per question, mastery gaps) to create a tailored schedule that focuses on the user’s most pressing needs, ensuring efficient use of study time.

#### Structured and Progressive Approach

The three-phase structure (Foundation Building, Practice and Strategy, Final Review and Refinement) provides a clear progression, helping users build skills, practice under test-like conditions, and refine their performance. The 8 full-length tests (2/2/4) ensure consistent exposure to the SAT format.

#### Clear Goals with Mastery Targets

Mastery targets give users specific, achievable goals for each domain and subskill, displayed as intuitive percentages (e.g., "Target Algebra Mastery: 75%"). Progress toward these targets is tracked, motivating users to close the gap between their current and target mastery.

#### Adaptive and Flexible

Regular updates and user adjustments ensure the Study Plan remains relevant as the user’s performance evolves. Whether a user improves in a subskill, adjusts their study hours, or changes their mastery targets, the Study Plan adapts to meet their needs.

#### Motivational and Engaging

Gamification elements (points, rewards, challenges) and progress tracking toward mastery targets keep users motivated. The structured phases and clear task schedule help users stay organized and focused, reducing the overwhelm of SAT preparation.

#### Comprehensive Integration

The Study Plan integrates with the Diagnostic Test, practice sessions, progress monitoring, and review process, creating a cohesive preparation experience. Users can seamlessly transition from completing a task to reviewing their performance, ensuring a continuous cycle of practice, feedback, and improvement.

***

### Implementation Files and Their Roles

* **Backend Files**:
  * **`backend/src/study_plan.py`**: The core backend logic for the Study Plan. Handles the creation and updating of the Study Plan (`/study_plan/create/{user_id}` and `/study_plan/update/{user_id}` endpoints), including phase division, task scheduling, advanced analytics (trends, improvement rates, pacing), mastery target integration, and full-length test distribution. It interacts with the `Proficiencies`, `Responses`, and `Questions` tables to fetch performance data and the `Study_Plans` and `Study_Plan_Actions` tables to store the plan.
  * **`backend/src/diagnostic.py`**: Provides the Diagnostic Test results (`theta` scores) used to initialize the Study Plan.
  * **`backend/src/practice.py`**: Manages practice sessions and the review process, providing data (e.g., `time_spent`) for analytics and linking Study Plan tasks to practice activities.
  * **`backend/src/gamification.py`**: Handles points, rewards, and challenges associated with Study Plan tasks, ensuring gamification integration.
  * **`backend/src/progress_monitoring.py`**: Provides `theta` scores and performance data for progress monitoring, which informs Study Plan updates.
* **Web Frontend Files**:
  * **`frontend/pages/study-plan.js`**: The main Study Plan UI for the web app. Displays the Study Plan tasks, mastery targets, and current mastery percentages for domains and subskills. Allows users to adjust study hours, study days, and mastery targets, triggering a call to the backend to regenerate the plan.
  * **`frontend/pages/onboarding.js`**: Collects user inputs (SAT test date, study hours, study days, mastery targets) during onboarding and initiates the Study Plan creation process by calling the `/study_plan/create/{user_id}` endpoint.
  * **`frontend/pages/dashboard.js`**: Displays progress monitoring data (current mastery percentages) that informs users of their progress toward Study Plan goals.
  * **`frontend/pages/review.js`**: Provides the review process UI, which the Study Plan uses to schedule review tasks based on test performance.
* **Mobile Frontend Files**:
  * **`SATSmartPrepApp/src/screens/StudyPlanScreen.js`**: The main Study Plan UI for the mobile app. Similar to the web version, it displays tasks, mastery targets, and current mastery percentages, and allows users to adjust their preferences.
  * **`SATSmartPrepApp/src/screens/OnboardingScreen.js`**: Collects user inputs during onboarding for the mobile app, including mastery targets, and initiates Study Plan creation.
  * **`SATSmartPrepApp/src/screens/DashboardScreen.js`**: Displays progress monitoring data, showing current mastery percentages relative to Study Plan targets.
  * **`SATSmartPrepApp/src/screens/ReviewScreen.js`**: Provides the review process UI for the mobile app, used by the Study Plan to schedule review tasks.
  * **`SATSmartPrepApp/App.js`**: Includes a background task using `react-native-background-fetch` to trigger weekly Study Plan updates via the `/study_plan/update/{user_id}` endpoint.
* **Shared Files**:
  * **`frontend/utils/api.js` and `SATSmartPrepApp/src/utils/api.js`**: Handle API calls to the backend for both web and mobile apps, ensuring consistent communication with endpoints like `/study_plan/create/{user_id}` and `/study_plan/update/{user_id}`.
  * **`frontend/styles/styles.js` and `SATSmartPrepApp/src/styles/styles.js`**: Define shared styles (e.g., typography, colors) used across the Study Plan UI for consistency.

***

### Summary

The **Study Plan** in the SAT Smart Prep App is a robust, personalized feature that guides users through their SAT preparation with a structured, data-driven approach. Its three-phase structure, mastery targets, advanced analytics, dynamic full-length test scheduling, regular updates, gamification, flexibility, and integration with other app features ensure an effective and engaging preparation experience. The implementation spans backend logic (`backend/src/study_plan.py`), web UI (`frontend/pages/study-plan.js`), mobile UI (`SATSmartPrepApp/src/screens/StudyPlanScreen.js`), and supporting files for onboarding, practice, progress monitoring, and review, creating a cohesive system that helps users achieve their target SAT score with confidence.

Let me know if you’d like to expand on any specific aspect of this documentation or add more details!
