# Feature Testing Plan

Below is a detailed **step-by-step plan** to test each feature implemented in the **SAT Smart Prep App** by **Learner Labs**. The plan covers all major features across the web app (`frontend/`) and mobile app (`SATSmartPrepApp/`), ensuring comprehensive validation of functionality, user experience, and integration. Each step includes specific test cases, tools, and expected outcomes to guide the testing process.

***

### Step-by-Step Plan to Test Each Feature

#### Overview of Features to Test

The SAT Smart Prep App includes the following key features:

1. **User Authentication (Signup, Login, SSO)**
2. **Onboarding Process**
3. **Diagnostic Test**
4. **Personalized Study Plans**
5. **AI-Driven Practice Sessions (Adaptive, Bluebook-Like)**
6. **Gamification (Coins, Rewards, Challenges, Leaderboards)**
7. **Social Features (Posts, Friends, Teams)**
8. **Progress Monitoring (Proficiencies, Score History, Guarantee)**
9. **AI Tutor (Text and Voice Input)**
10. **Push Notifications**
11. **Accessibility (High-Contrast Mode, Voice Input Fallback)**
12. **Offline Support**

#### Testing Tools

* **Unit Testing**: Jest (web and mobile), Pytest (backend).
* **E2E Testing**: Cypress (web), Detox (mobile).
* **Manual Testing**: For UI/UX and features requiring real user interaction (e.g., push notifications).
* **Monitoring**: AWS CloudWatch (post-deployment), Google Play Console/App Store Connect (crash reports).

***

### Step-by-Step Testing Plan

#### Step 1: Test User Authentication

* **Feature Description**: Users can sign up, log in, or use Google SSO to authenticate.
* **Files Involved**:
  * `frontend/pages/login.js` (web)
  * `SATSmartPrepApp/src/screens/LoginScreen.js` (mobile)
  * `frontend/backend/src/auth.py` (backend)
* **Test Cases**:
  1. **Signup**:
     * **Test**: Sign up with a new email and password.
     * **Steps**:
       * Navigate to the login page (`/login` on web, `LoginScreen` on mobile).
       * Enter a new email (`testuser@example.com`) and password (`password123`).
       * Submit the signup form.
     * **Expected Outcome**: User is created, `user_id` is returned, and user is redirected to `/onboarding`.
     * **Tool**: Cypress (web), Detox (mobile).
     *   **Test Code** (Cypress):

         ```javascript
         describe('User Signup', () => {
           it('should sign up a new user', () => {
             cy.visit('/login');
             cy.get('input[placeholder="Email"]').type('testuser@example.com');
             cy.get('input[placeholder="Password"]').type('password123');
             cy.contains('Login').click();
             cy.url().should('include', '/onboarding');
           });
         });
         ```
  2. **Login**:
     * **Test**: Log in with existing credentials.
     * **Steps**:
       * Navigate to the login page.
       * Enter existing email (`testuser@example.com`) and password (`password123`).
       * Submit the login form.
     * **Expected Outcome**: User is authenticated, `user_id` is stored in `localStorage`/`AsyncStorage`, and user is redirected to `/onboarding`.
     * **Tool**: Pytest (backend).
     *   **Test Code** (Pytest):

         ```python
         from fastapi.testclient import TestClient
         from backend.src.main import app

         client = TestClient(app)

         def test_login_existing_user():
             client.post("/auth/signup", json={"email": "testuser@example.com", "password": "password123"})
             response = client.post("/auth/login", json={"email": "testuser@example.com", "password": "password123"})
             assert response.status_code == 200
             assert "user_id" in response.json()
         ```
  3. **Google SSO**:
     * **Test**: Log in using Google SSO.
     * **Steps**:
       * Navigate to the login page.
       * Click "Login with Google".
       * Complete the Google SSO flow (mocked in testing).
     * **Expected Outcome**: User is authenticated, and user is redirected to `/onboarding`.
     * **Tool**: Manual (requires Google SSO setup).

#### Step 2: Test Onboarding Process

* **Feature Description**: New users go through a step-by-step onboarding process to provide information, take a diagnostic test, and receive a study plan.
* **Files Involved**:
  * `frontend/pages/onboarding.js` (web)
  * `SATSmartPrepApp/src/screens/OnboardingScreen.js` (mobile)
  * `frontend/backend/src/auth.py`, `frontend/backend/src/diagnostic.py`, `frontend/backend/src/study_plan.py` (backend)
* **Test Cases**:
  1. **Complete Onboarding Flow**:
     * **Test**: Progress through all onboarding steps.
     * **Steps**:
       * Start onboarding after login.
       * Complete each step: Welcome, Basic Info (name: "Alex Smith", grade: "11th"), SAT Experience (Yes), SAT Scores (Math: 650, Reading: 600), Study Preferences (test date: "2025-12-01", hours: 10, days: 5, time: "Morning"), Diagnostic Test, Study Plan, Dashboard.
       * Verify post-onboarding rewards (50 coins, avatar, challenge).
     * **Expected Outcome**: User data is saved, diagnostic test is completed, study plan is created, and user is redirected to the dashboard.
     * **Tool**: Cypress (web).
     *   **Test Code** (Cypress):

         ```javascript
         describe('Onboarding Flow', () => {
           beforeEach(() => {
             cy.window().then(win => {
               win.localStorage.setItem('user_id', 'user1');
             });
             cy.visit('/onboarding');
           });

           it('should complete onboarding', () => {
             cy.contains('Next').click();
             cy.get('input[placeholder="Enter your full name"]').type('Alex Smith');
             cy.get('select').eq(0).select('11th');
             cy.contains('Next').click();
             cy.contains('Yes').click();
             cy.get('input[placeholder="Enter your Math score"]').type('650');
             cy.get('input[placeholder="Enter your Reading & Writing score"]').type('600');
             cy.contains('Next').click();
             cy.get('input[type="date"]').type('2025-12-01');
             cy.get('input[type="range"]').invoke('val', 10).trigger('change');
             cy.get('select').eq(1).select('5');
             cy.get('select').eq(2).select('Morning');
             cy.contains('Next').click();
             cy.contains('Start Test').click();
             cy.contains('Continue to Study Plan').click();
             cy.contains('Continue to Dashboard').click();
             cy.contains('Start Studying').click();
             cy.url().should('include', '/dashboard');
           });
         });
         ```

#### Step 3: Test Diagnostic Test

* **Feature Description**: Users take a diagnostic test to assess their SAT proficiency, which informs their study plan.
* **Files Involved**:
  * `frontend/pages/diagnostic.js` (web)
  * `SATSmartPrepApp/src/screens/DiagnosticScreen.js` (mobile)
  * `frontend/backend/src/diagnostic.py` (backend)
* **Test Cases**:
  1. **Start Diagnostic Test**:
     * **Test**: Start a diagnostic test and verify questions are returned.
     * **Steps**:
       * Navigate to the diagnostic test page.
       * Start the test.
     * **Expected Outcome**: 22 questions are returned, and a `Practice_Sessions` record is created.
     * **Tool**: Pytest (backend).
     *   **Test Code** (Pytest):

         ```python
         from fastapi.testclient import TestClient
         from backend.src.main import app

         client = TestClient(app)

         def test_start_diagnostic():
             response = client.post("/diagnostic/start/user1", params={"num_questions": 22, "test_type": "SAT"})
             assert response.status_code == 200
             data = response.json()
             assert "session_id" in data
             assert len(data["questions"]) == 22
         ```
  2. **Submit Diagnostic Test**:
     * **Test**: Submit answers and verify results.
     * **Steps**:
       * Start a diagnostic test.
       * Submit mock answers for 22 questions.
     * **Expected Outcome**: `theta` is calculated, `Proficiencies` are updated, and results are returned.
     * **Tool**: Pytest (backend).
     *   **Test Code** (Pytest):

         ```python
         def test_submit_diagnostic():
             start_response = client.post("/diagnostic/start/user1", params={"num_questions": 22, "test_type": "SAT"})
             session_id = start_response.json()["session_id"]
             responses = [{"question_id": "q1", "answer": "A", "time_spent": 60, "domain": "Math", "theta": 0.8}]
             response = client.post(f"/diagnostic/submit/{session_id}", json=responses)
             assert response.status_code == 200
             assert "theta" in response.json()
         ```

#### Step 4: Test Personalized Study Plans

* **Feature Description**: The app generates a study plan based on diagnostic results and user preferences.
* **Files Involved**:
  * `frontend/pages/study-plan.js` (web)
  * `SATSmartPrepApp/src/screens/StudyPlanScreen.js` (mobile)
  * `frontend/backend/src/study_plan.py` (backend)
* **Test Cases**:
  1. **Create Study Plan**:
     * **Test**: Create a study plan after a diagnostic test.
     * **Steps**:
       * Complete a diagnostic test.
       * Provide study preferences (test date: "2025-12-01", hours: 10, days: 5).
       * Create a study plan.
     * **Expected Outcome**: A `Study_Plans` record is created with associated `Study_Plan_Actions`.
     * **Tool**: Pytest (backend).
     *   **Test Code** (Pytest):

         ```python
         def test_create_study_plan():
             response = client.post("/study_plan/create/user1", json={
                 "test_date": "2025-12-01T00:00:00Z",
                 "study_hours": 10,
                 "study_days": 5
             })
             assert response.status_code == 200
             data = response.json()
             assert "plan_id" in data
             assert "milestones" in data
         ```
  2. **View Study Plan**:
     * **Test**: Retrieve and display the study plan.
     * **Steps**:
       * Create a study plan.
       * Navigate to the study plan page.
     * **Expected Outcome**: Study plan tasks are displayed with due dates and points.
     * **Tool**: Cypress (web).
     *   **Test Code** (Cypress):

         ```javascript
         describe('Study Plan', () => {
           it('should display study plan', () => {
             cy.visit('/study-plan');
             cy.contains('Your Study Plan').should('be.visible');
             cy.contains('Complete 5 sessions in Algebra').should('be.visible');
           });
         });
         ```

#### Step 5: Test AI-Driven Practice Sessions

* **Feature Description**: Users can take adaptive practice sessions with a Bluebook-like interface (web) and a nudge for full-length tests (mobile).
* **Files Involved**:
  * `frontend/pages/practice-bluebook.js` (web)
  * `SATSmartPrepApp/src/screens/PracticeScreen.js` (mobile)
  * `frontend/backend/src/practice.py` (backend)
* **Test Cases**:
  1. **Start Practice Session**:
     * **Test**: Start a practice session and verify questions.
     * **Steps**:
       * Navigate to the practice page.
       * Select domain (Math), test type (SAT), and number of questions (10).
       * Start the session.
     * **Expected Outcome**: 10 questions are returned, and a `Practice_Sessions` record is created with `device` set correctly.
     * **Tool**: Pytest (backend).
     *   **Test Code** (Pytest):

         ```python
         def test_start_practice():
             response = client.post("/practice/start/user1", json={
                 "domain": "Math",
                 "num_questions": 10,
                 "test_type": "SAT",
                 "device": "web"
             })
             assert response.status_code == 200
             data = response.json()
             assert "session_id" in data
             assert len(data["questions"]) == 10
         ```
  2. **Bluebook-Like Interface (Web)**:
     * **Test**: Verify the Bluebook-like UI on the web app.
     * **Steps**:
       * Start a full-length test (44 questions).
       * Check for timer, navigation buttons, and accessibility tools (zoom, line reader).
     * **Expected Outcome**: UI elements are present and functional.
     * **Tool**: Cypress (web).
     *   **Test Code** (Cypress):

         ```javascript
         describe('Bluebook-Like Practice', () => {
           it('should display Bluebook UI', () => {
             cy.visit('/practice-bluebook');
             cy.get('select').eq(1).select('44');
             cy.contains('Start').click();
             cy.contains('SAT Smart Prep App - Full-Length Test').should('be.visible');
             cy.contains('Time Remaining: 15:00').should('be.visible');
             cy.contains('Zoom In').click();
             cy.get('.passage').should('have.css', 'transform', 'scale(1.1)');
           });
         });
         ```
  3. **Nudge Modal (Mobile)**:
     * **Test**: Verify the nudge modal for full-length tests on mobile.
     * **Steps**:
       * Start a full-length test (44 questions) on mobile.
       * Check for the nudge modal.
       * Choose "Continue on Mobile".
     * **Expected Outcome**: Nudge modal appears, and user can proceed on mobile.
     * **Tool**: Detox (mobile).
     *   **Test Code** (Detox):

         ```javascript
         describe('Practice Nudge Modal', () => {
           it('should show nudge modal for full-length test', async () => {
             await element(by.text('Practice')).tap();
             await element(by.text('Short Practice (10 questions)')).tap();
             await element(by.text('Full-Length Test (44-98 questions)')).tap();
             await element(by.text('Start')).tap();
             await expect(element(by.text('Take Full-Length Tests on the Web App'))).toBeVisible();
             await element(by.text('Continue on Mobile')).tap();
             await expect(element(by.text('Take Full-Length Tests on the Web App'))).not.toBeVisible();
           });
         });
         ```

#### Step 6: Test Gamification

* **Feature Description**: Users earn coins, unlock rewards, complete challenges, and compete on leaderboards.
* **Files Involved**:
  * `frontend/pages/rewards-store.js`, `frontend/pages/leaderboard.js` (web)
  * `SATSmartPrepApp/src/screens/RewardsStoreScreen.js`, `SATSmartPrepApp/src/screens/LeaderboardScreen.js` (mobile)
  * `frontend/backend/src/gamification.py` (backend)
* **Test Cases**:
  1. **Earn Coins**:
     * **Test**: Earn coins after completing a practice session.
     * **Steps**:
       * Start and complete a practice session.
       * Check user’s coin balance.
     * **Expected Outcome**: User earns 10 coins, and `Users.coins` is updated.
     * **Tool**: Pytest (backend).
     *   **Test Code** (Pytest):

         ```python
         def test_earn_coins():
             start_response = client.post("/practice/start/user1", json={
                 "domain": "Math",
                 "num_questions": 1,
                 "test_type": "SAT",
                 "device": "web"
             })
             session_id = start_response.json()["session_id"]
             responses = [{"question_id": "q1", "answer": "A", "time_spent": 60, "session_id": session_id, "domain": "Math", "theta": 0.8}]
             response = client.post(f"/practice/submit/{session_id}", json=responses)
             assert response.status_code == 200
             assert response.json()["points_earned"] == 10

             coins_response = client.get("/gamification/coins/user1")
             assert coins_response.json()["coins"] == 10
         ```
  2. **Unlock Reward**:
     * **Test**: Unlock a reward using coins.
     * **Steps**:
       * Earn 100 coins.
       * Navigate to the rewards store.
       * Unlock the "Math Prodigy Badge" (cost: 100 coins).
     * **Expected Outcome**: Reward is unlocked, and coins are deducted.
     * **Tool**: Cypress (web).
     *   **Test Code** (Cypress):

         ```javascript
         describe('Unlock Reward', () => {
           it('should unlock a reward', () => {
             cy.visit('/rewards-store');
             cy.contains('Math Prodigy Badge').should('be.visible');
             cy.contains('100 Coins').should('be.visible');
             cy.contains('Unlock').click();
             cy.contains('Unlocked math_prodigy_badge!').should('be.visible');
           });
         });
         ```
  3. **Leaderboard**:
     * **Test**: View the leaderboard for a skill.
     * **Steps**:
       * Navigate to the leaderboard page.
       * Select a skill (e.g., Algebra).
       * View the global leaderboard.
     * **Expected Outcome**: Leaderboard displays top users with scores.
     * **Tool**: Detox (mobile).
     *   **Test Code** (Detox):

         ```javascript
         describe('Leaderboard', () => {
           it('should display global leaderboard', async () => {
             await element(by.text('Leaderboard')).tap();
             await expect(element(by.text('Algebra Leaderboard (Global)'))).toBeVisible();
             await expect(element(by.text('user1@example.com'))).toBeVisible();
           });
         });
         ```

#### Step 7: Test Social Features

* **Feature Description**: Users can create posts, add friends, and form teams.
* **Files Involved**:
  * `frontend/pages/community.js` (web)
  * `SATSmartPrepApp/src/screens/CommunityScreen.js` (mobile)
  * `frontend/backend/src/social.py` (backend)
* **Test Cases**:
  1. **Create Post**:
     * **Test**: Create a new community post.
     * **Steps**:
       * Navigate to the community page.
       * Enter a post ("Hello, world!").
       * Submit the post.
     * **Expected Outcome**: Post is created and displayed in the community feed.
     * **Tool**: Pytest (backend).
     *   **Test Code** (Pytest):

         ```python
         def test_create_post():
             response = client.post("/social/posts", params={"user_id": "user1", "content": "Hello, world!"})
             assert response.status_code == 200
             assert response.json()["message"] == "Post created"

             posts_response = client.get("/social/posts")
             assert any(post["content"] == "Hello, world!" for post in posts_response.json())
         ```
  2. **Add Friend**:
     * **Test**: Add a friend and verify the friendship.
     * **Steps**:
       * Navigate to the community page.
       * Add a friend (mocked in testing).
     * **Expected Outcome**: Friendship is created with status "pending".
     * **Tool**: Manual (requires additional UI for friend requests).

#### Step 8: Test Progress Monitoring

* **Feature Description**: Users can track proficiencies, score history, and check the score improvement guarantee.
* **Files Involved**:
  * `frontend/pages/dashboard.js` (web)
  * `SATSmartPrepApp/src/screens/DashboardScreen.js` (mobile)
  * `frontend/backend/src/progress_monitoring.py` (backend)
* **Test Cases**:
  1. **View Proficiencies**:
     * **Test**: View proficiencies after a diagnostic test.
     * **Steps**:
       * Complete a diagnostic test.
       * Navigate to the dashboard.
     * **Expected Outcome**: Proficiencies are displayed (e.g., Math - Algebra: Theta 0.8).
     * **Tool**: Pytest (backend).
     *   **Test Code** (Pytest):

         ```python
         def test_get_proficiencies():
             response = client.get("/progress_monitoring/proficiencies/user1")
             assert response.status_code == 200
             assert isinstance(response.json(), list)
         ```
  2. **Check Score Guarantee**:
     * **Test**: Check eligibility for the score improvement guarantee.
     * **Steps**:
       * Complete two full-length tests (first score: 1200, second score: 1300).
       * Check the guarantee.
     * **Expected Outcome**: User is not eligible (improvement < 150 points).
     * **Tool**: Pytest (backend).
     *   **Test Code** (Pytest):

         ```python
         def test_check_score_guarantee():
             response = client.get("/progress_monitoring/guarantee/user1")
             assert response.status_code == 200
             assert response.json()["eligible"] == False
             assert response.json()["improvement"] == 100
         ```

#### Step 9: Test AI Tutor

* **Feature Description**: Users can interact with an AI tutor via text or voice input.
* **Files Involved**:
  * `frontend/pages/practice-bluebook.js` (web)
  * `SATSmartPrepApp/src/screens/PracticeScreen.js` (mobile)
  * `frontend/backend/src/ai_tutor.py` (backend)
* **Test Cases**:
  1. **Text Input**:
     * **Test**: Send a text query to the AI tutor.
     * **Steps**:
       * Start a practice session.
       * Enter a query ("How do I solve this math problem?").
       * Submit the query.
     * **Expected Outcome**: AI tutor responds, and interaction is logged.
     * **Tool**: Pytest (backend).
     *   **Test Code** (Pytest):

         ```python
         def test_ask_ai_tutor_text():
             response = client.post("/ai_tutor/ask", params={"user_id": "user1", "query": "How do I solve this math problem?"})
             assert response.status_code == 200
             assert "response" in response.json()
         ```
  2. **Voice Input (Mobile)**:
     * **Test**: Use voice input to query the AI tutor.
     * **Steps**:
       * Start a practice session on mobile.
       * Enable voice input and record a query.
     * **Expected Outcome**: Voice input is converted to text, and AI tutor responds.
     * **Tool**: Detox (mobile).
     *   **Test Code** (Detox):

         ```javascript
         describe('AI Tutor Voice Input', () => {
           it('should handle voice input', async () => {
             await element(by.text('Practice')).tap();
             await element(by.text('Start')).tap();
             await element(by.text('Voice Input')).tap();
             await expect(element(by.text('Recording...'))).toBeVisible();
           });
         });
         ```

#### Step 10: Test Push Notifications

* **Feature Description**: Users receive push notifications (e.g., reminders, gamification updates).
* **Files Involved**:
  * `frontend/pages/_app.js` (web)
  * `SATSmartPrepApp/App.js` (mobile)
  * `frontend/backend/src/notifications.py` (backend)
* **Test Cases**:
  1. **Send Push Notification**:
     * **Test**: Send a push notification to a user.
     * **Steps**:
       * Ensure the user has a device token.
       * Send a notification via the backend.
     * **Expected Outcome**: Notification is sent successfully.
     * **Tool**: Pytest (backend).
     *   **Test Code** (Pytest):

         ```python
         def test_send_notification():
             response = client.post("/send-notification", params={"user_id": "user1", "message": "Test notification"})
             assert response.status_code == 200
             assert response.json()["message"] == "Notification sent"
         ```

#### Step 11: Test Accessibility

* **Feature Description**: The app supports high-contrast mode and voice input fallback.
* **Files Involved**:
  * `frontend/styles/styles.js` (web)
  * `SATSmartPrepApp/src/styles/styles.js` (mobile)
  * `SATSmartPrepApp/src/components/VoiceInputFallback.js` (mobile)
* **Test Cases**:
  1. **High-Contrast Mode**:
     * **Test**: Enable high-contrast mode and verify UI changes.
     * **Steps**:
       * Enable high-contrast mode (mocked in testing).
       * Check UI elements for color changes.
     * **Expected Outcome**: Text and background colors switch to high-contrast values.
     * **Tool**: Jest (web).
     *   **Test Code** (Jest):

         ```javascript
         import { colors } from '../styles/styles';

         describe('High-Contrast Mode', () => {
           it('should use high-contrast colors', () => {
             expect(colors.highContrastText).toBe('#000');
             expect(colors.highContrastBackground).toBe('#fff');
           });
         });
         ```
  2. **Voice Input Fallback (Mobile)**:
     * **Test**: Use the voice input fallback when offline.
     * **Steps**:
       * Start a practice session on mobile.
       * Simulate offline mode.
       * Switch to text input.
     * **Expected Outcome**: Voice input is disabled, and text input is available.
     * **Tool**: Detox (mobile).
     *   **Test Code** (Detox):

         ```javascript
         describe('Voice Input Fallback', () => {
           it('should switch to text input when offline', async () => {
             await element(by.text('Practice')).tap();
             await element(by.text('Start')).tap();
             // Simulate offline mode (mocked)
             await element(by.text('Switch to Text Input')).tap();
             await expect(element(by.text('Ask the AI tutor...'))).toBeVisible();
           });
         });
         ```

#### Step 12: Test Offline Support

* **Feature Description**: Users can practice offline, with responses queued for later sync.
* **Files Involved**:
  * `frontend/pages/practice-bluebook.js` (web)
  * `SATSmartPrepApp/src/screens/PracticeScreen.js` (mobile)
  * `frontend/utils/api.js`, `SATSmartPrepApp/src/utils/api.js` (API client)
* **Test Cases**:
  1. **Offline Practice**:
     * **Test**: Start a practice session offline and submit answers.
     * **Steps**:
       * Start a practice session.
       * Simulate offline mode.
       * Submit answers.
     * **Expected Outcome**: Answers are queued, and session continues offline.
     * **Tool**: Cypress (web).
     *   **Test Code** (Cypress):

         ```javascript
         describe('Offline Practice', () => {
           it('should queue responses offline', () => {
             cy.visit('/practice-bluebook');
             cy.get('select').eq(1).select('10');
             cy.contains('Start').click();
             cy.window().then(win => {
               win.navigator.__defineGetter__('onLine', () => false);
             });
             cy.get('input[value="A"]').click();
             cy.contains('Next').click();
             cy.contains('Practice completed offline! Sync when online.').should('be.visible');
           });
         });
         ```

***

### Additional Notes for the Testing Team

* **Environment Setup**: Ensure the local environment is set up as per the `README.md` instructions, including Docker for the backend and database.
* **Test Data**: Use mock data for testing (e.g., pre-populated `Questions` table) to ensure consistent results.
* **Edge Cases**:
  * Test with invalid inputs (e.g., negative study hours, invalid email).
  * Test with network interruptions (e.g., during diagnostic test submission).
  * Test with different device types (e.g., iOS 13.4, Android 6.0).
* **Performance Testing**: After functional testing, conduct load testing to ensure the app can handle 1,000 concurrent users, focusing on API endpoints like `/practice/start`.

This plan ensures that each feature of the SAT Smart Prep App is thoroughly tested, covering functionality, user experience, and edge cases. Let me know if you’d like to add more test cases or adjust the plan!
