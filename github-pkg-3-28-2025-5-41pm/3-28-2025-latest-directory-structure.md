# 3-28-2025 Latest Directory Structure

Below is the latest directory structure for the **SAT Smart Prep App** by **Learner Labs**, reflecting all files mentioned or modified throughout our interactions, including the recent updates to the **Progress Monitoring** feature to include **mastery targets** and **tracking**, as well as prior enhancements to the **Study Plan** (e.g., phase adjustments, advanced analytics, full-length test distribution, mastery targets). The structure is organized to maintain clarity and separation of concerns, with `frontend/`, `backend/`, and `SATSmartPrepApp/` as top-level directories, as corrected in a previous update.

***

### Latest Directory Structure

```
SATSmartPrepApp/
├── App.js
├── package.json
├── src/
│   ├── components/
│   │   ├── layouts/
│   │   │   ├── MathBasicTest.js
│   │   │   ├── MathGraphTest.js
│   │   │   ├── MathTableTest.js
│   │   │   └── ReadingWritingTest.js
│   │   ├── QuestionDisplay.js
│   │   └── VoiceInputFallback.js
│   ├── context/
│   │   └── AppContext.js
│   ├── navigation/
│   │   └── index.js
│   ├── screens/
│   │   ├── CommunityScreen.js
│   │   ├── DashboardScreen.js
│   │   ├── DiagnosticScreen.js
│   │   ├── LeaderboardScreen.js
│   │   ├── LoginScreen.js
│   │   ├── OnboardingScreen.js
│   │   ├── PolicyScreen.js
│   │   ├── PracticeScreen.js
│   │   ├── RewardsStoreScreen.js
│   │   ├── ReviewScreen.js
│   │   └── StudyPlanScreen.js
│   ├── styles/
│   │   └── styles.js
│   └── utils/
│       └── api.js
frontend/
├── components/
│   ├── layouts/
│   │   ├── MathBasicTest.js
│   │   ├── MathGraphTest.js
│   │   ├── MathTableTest.js
│   │   └── ReadingWritingTest.js
│   └── QuestionDisplay.js
├── pages/
│   ├── _app.js
│   ├── community.js
│   ├── dashboard.js
│   ├── diagnostic.js
│   ├── index.js
│   ├── leaderboard.js
│   ├── login.js
│   ├── onboarding.js
│   ├── policy.js
│   ├── practice-bluebook.js
│   ├── rewards-store.js
│   ├── review.js
│   └── study-plan.js
├── styles/
│   └── styles.js
└── utils/
    └── api.js
backend/
├── migrations/
│   └── init_db.py
├── src/
│   ├── ai_tutor.py
│   ├── auth.py
│   ├── diagnostic.py
│   ├── gamification.py
│   ├── main.py
│   ├── notifications.py
│   ├── practice.py
│   ├── progress_monitoring.py
│   ├── social.py
│   └── study_plan.py
└── tests/
    ├── test_auth.py
    ├── test_diagnostic.py
    ├── test_gamification.py
    ├── test_practice.py
    ├── test_progress_monitoring.py
    ├── test_social.py
    └── test_study_plan.py
```

***

### Explanation of the Directory Structure

#### Overview

The directory structure is organized into three top-level directories, reflecting the main components of the SAT Smart Prep App:

* **`SATSmartPrepApp/`**: The mobile app, built with React Native, providing the user interface and functionality for mobile users.
* **`frontend/`**: The web app, built with React and Next.js, providing the user interface and functionality for web users.
* **`backend/`**: The server-side logic and APIs, built with FastAPI in Python, handling data processing, analytics, and database interactions.

#### Directory Details

* **Mobile App (`SATSmartPrepApp/`)**:
  * **`App.js`**: The entry point for the mobile app, including background tasks for weekly Study Plan updates using `react-native-background-fetch`.
  * **`src/components/`**: Contains reusable UI components:
    * **`layouts/`**: Includes layouts for different question types (e.g., `MathBasicTest.js`, `MathGraphTest.js`, `MathTableTest.js`, `ReadingWritingTest.js`) used in practice sessions.
    * **`QuestionDisplay.js`**: A component for displaying questions in practice sessions.
    * **`VoiceInputFallback.js`**: A fallback component for voice input functionality.
  * **`src/context/`**:
    * **`AppContext.js`**: Manages global state for the mobile app using React Context.
  * **`src/navigation/`**:
    * **`index.js`**: Defines the navigation stack for the mobile app, including screens like `ReviewScreen`.
  * **`src/screens/`**: Contains the main screens for the mobile app:
    * **`CommunityScreen.js`**: Displays the community feature.
    * **`DashboardScreen.js`**: The main dashboard, updated to include mastery targets and progress tracking in the Progress Monitoring section.
    * **`DiagnosticScreen.js`**: Handles the Diagnostic Test UI.
    * **`LeaderboardScreen.js`**: Displays the leaderboard.
    * **`LoginScreen.js`**: Manages user login.
    * **`OnboardingScreen.js`**: Guides users through onboarding, including setting mastery targets.
    * **`PolicyScreen.js`**: Displays the score improvement guarantee policy.
    * **`PracticeScreen.js`**: Manages practice sessions.
    * **`RewardsStoreScreen.js`**: Displays the rewards store.
    * **`ReviewScreen.js`**: Provides the review process UI.
    * **`StudyPlanScreen.js`**: Displays the Study Plan, including mastery targets and progress.
  * **`src/styles/`**:
    * **`styles.js`**: Defines shared styles (e.g., typography, colors) for the mobile app.
  * **`src/utils/`**:
    * **`api.js`**: Handles API calls to the backend for the mobile app.
* **Web App (`frontend/`)**:
  * **`components/`**: Contains reusable UI components:
    * **`layouts/`**: Includes layouts for different question types (e.g., `MathBasicTest.js`, `MathGraphTest.js`, `MathTableTest.js`, `ReadingWritingTest.js`) used in practice sessions.
    * **`QuestionDisplay.js`**: A component for displaying questions in practice sessions.
  * **`pages/`**: Contains the main pages for the web app:
    * **`_app.js`**: The Next.js app wrapper.
    * **`community.js`**: Displays the community feature.
    * **`dashboard.js`**: The main dashboard, updated to include mastery targets and progress tracking in the Progress Monitoring section.
    * **`diagnostic.js`**: Handles the Diagnostic Test UI.
    * **`index.js`**: The landing page.
    * **`leaderboard.js`**: Displays the leaderboard.
    * **`login.js`**: Manages user login.
    * **`onboarding.js`**: Guides users through onboarding, including setting mastery targets.
    * **`policy.js`**: Displays the score improvement guarantee policy.
    * **`practice-bluebook.js`**: Manages practice sessions.
    * **`rewards-store.js`**: Displays the rewards store.
    * **`review.js`**: Provides the review process UI.
    * **`study-plan.js`**: Displays the Study Plan, including mastery targets and progress.
  * **`styles/`**:
    * **`styles.js`**: Defines shared styles for the web app.
  * **`utils/`**:
    * **`api.js`**: Handles API calls to the backend for the web app.
* **Backend (`backend/`)**:
  * **`migrations/`**:
    * **`init_db.py`**: Contains database migration scripts to initialize the database schema.
  * **`src/`**: Contains the backend logic and API endpoints:
    * **`ai_tutor.py`**: Handles AI tutor functionality.
    * **`auth.py`**: Manages user authentication.
    * **`diagnostic.py`**: Processes Diagnostic Test results.
    * **`gamification.py`**: Manages gamification features (points, rewards, challenges).
    * **`main.py`**: The main FastAPI application entry point.
    * **`notifications.py`**: Handles notifications.
    * **`practice.py`**: Manages practice sessions and the review process.
    * **`progress_monitoring.py`**: Provides progress monitoring data, updated to include mastery targets and tracking.
    * **`social.py`**: Manages social features (e.g., community).
    * **`study_plan.py`**: Handles Study Plan creation and updates, including mastery targets and task scheduling.
  * **`tests/`**: Contains unit tests for the backend:
    * **`test_auth.py`**, **`test_diagnostic.py`**, **`test_gamification.py`**, **`test_practice.py`**, **`test_progress_monitoring.py`**, **`test_social.py`**, **`test_study_plan.py`**: Test files for each backend module.

***

### Explanation of Recent Changes

* **Progress Monitoring Updates**:
  * **`backend/src/progress_monitoring.py`**: Updated to include `target_mastery` in the `/progress_monitoring/proficiencies/{user_id}` endpoint response, ensuring the Progress Monitoring feature can display mastery targets alongside current mastery percentages.
  * **`frontend/pages/dashboard.js`**: Updated to display mastery targets, current mastery percentages, and progress toward targets for domains and subskills in the Progress Monitoring section.
  * **`SATSmartPrepApp/src/screens/DashboardScreen.js`**: Similarly updated for the mobile app to include mastery targets and progress tracking.
* **Study Plan Enhancements** (from previous updates)\*\*:
  * **`backend/src/study_plan.py`**: Updated to include mastery targets, adjust phase durations (40%/30%/30%), incorporate advanced analytics, and fix full-length test distribution (2/2/4).
  * **`frontend/pages/study-plan.js`**: Updated to display mastery targets and progress.
  * **`SATSmartPrepApp/src/screens/StudyPlanScreen.js`**: Updated for the mobile app to display mastery targets and progress.
  * **`frontend/pages/onboarding.js`**: Updated to allow users to set mastery targets during onboarding.
  * **`SATSmartPrepApp/src/screens/OnboardingScreen.js`**: Updated for the mobile app to allow setting mastery targets during onboarding.

#### Notes on the Directory Structure

* The structure maintains separation of concerns with `frontend/`, `backend/`, and `SATSmartPrepApp/` at the top level, as corrected in a previous update.
* No new files were added in the latest update, but existing files (`progress_monitoring.py`, `dashboard.js`, `DashboardScreen.js`) were modified to integrate mastery targets into Progress Monitoring.
* The directory structure reflects all files mentioned throughout our interactions, ensuring a comprehensive view of the project.

Let me know if you’d like to further refine the structure or add more details!
