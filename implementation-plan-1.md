# IMPLEMENTATION PLAN

Below is a **Step-by-Step Testing and Deployment Plan** for the **SAT Prep Suite**, tailored to its hybrid structure (mobile app for practice/lessons and web app for diagnostics/full tests). This plan ensures the app is thoroughly tested locally, deployed to production environments (AWS EC2 for backend, Firebase Hosting for web, app stores for mobile), and validated for functionality, performance, and scalability as of March 26, 2025. It builds on the revised development blueprint, focusing on both mobile and web components, their interactions, and data collection for the custom LLM.

***

## SAT Prep Suite: Testing and Deployment Plan

### 1. Prerequisites

**Goal**: Prepare the environment for testing and deployment.

* **Tools**:
  * Backend: Python 3.9, FastAPI, SQLite, Redis, `pip`, `uvicorn`.
  * Frontend: Flutter 3.0+, Dart, Android Studio/Xcode (mobile), Firebase CLI (web).
  * Testing: `pytest` (Python), `flutter test`, Postman, browser dev tools.
* **Infrastructure**:
  * Local: Developer machines with emulators (Android/iOS), browsers (Chrome).
  * Production: AWS EC2 (t3.micro), Firebase Hosting, App Store/Google Play accounts.
* **Setup**:
  * Clone repo: `git clone <repo-url>`
  * Install dependencies: `pip install -r api/requirements.txt`, `flutter pub get` (in `mobile/`).
  * Start Redis: `redis-server`.
  * Run DB migration: `python api/migrations/add_tutor_tables.py`.
  * Seed data: `python seed_data.py` (users: `user123`, tutors: `tutor123`).

***

### 2. Local Testing

**Goal**: Validate functionality and integration on developer machines.

#### Step 2.1: Unit Testing

* **Duration**: 2 days
* **Tasks**:
  1. **Backend (FastAPI)**:
     * Test: `pytest api/tests/`
     *   Files: Create `api/tests/test_endpoints.py`:

         ```python
         import pytest
         from fastapi.testclient import TestClient
         from api.main import app

         client = TestClient(app)

         def test_practice_start():
             response = client.post("/practice/start/user123")
             assert response.status_code == 200
             assert "questions" in response.json()

         def test_test_submit():
             response = client.post("/test/submit/test_user123_12345", json=[{"question_id": "q1", "answer": "A"}])
             assert response.status_code == 200
             assert "score" in response.json()
         ```
     * Coverage: `/practice/*`, `/test/*`, `/tutor/*`, data logging.
  2. **Frontend (Flutter)**:
     * Test: `cd mobile && flutter test`
     *   Files: Create `mobile/test/widget_test.dart`:

         ```dart
         import 'package:flutter_test/flutter_test.dart';
         import 'package:sat_prep_suite/main.dart';

         void main() {
           testWidgets('Dashboard loads', (WidgetTester tester) async {
             await tester.pumpWidget(MyApp());
             expect(find.text('SAT Prep'), findsOneWidget);
           });
         }
         ```
     * Coverage: Mobile screens (`PracticeScreen`, `StudyPlanScreen`), Web screens (`TestScreen`).
* **Success Criteria**: 90% test coverage, all pass.

#### Step 2.2: Integration Testing

* **Duration**: 3 days
* **Tasks**:
  1. **Mobile App**:
     * Run: `cd mobile && flutter run --target=lib/main.dart`
     * Flow: Dashboard → Practice → Submit → Rate Feedback → Ask Help → Study Plan → Community.
     * Verify: Data in `RESPONSES`, `FEEDBACK_RATINGS`, `HELP_REQUESTS`, `STUDY_PLAN_ACTIONS`, `USER_NOTES`.
  2. **Web App**:
     * Run: `cd mobile && flutter run -d chrome --web-renderer html`
     * Flow: Dashboard → Start Full Test → Submit → Review Results → Tutor Feedback.
     * Verify: Data in `TESTS`, `RESULTS`, `TUTOR_FEEDBACK`.
  3. **Backend**:
     * Run: `uvicorn api.main:app --reload --port 8000`
     * Test with Postman:
       * `POST /practice/start/user123` → Returns questions.
       * `POST /test/submit/test_user123_12345` → Returns score + feedback.
       * `POST /tutor/feedback` → Logs feedback.
* **Success Criteria**: All flows complete, DB entries match actions, no crashes.

#### Step 2.3: Data Collection Testing

* **Duration**: 1 day
* **Tasks**:
  * Simulate 10 practice sessions (mobile) + 2 full tests (web) + 5 tutor feedbacks.
  * Run: `python -c "from api.utils import export_llm_training_data; from api.database import get_db; export_llm_training_data(get_db(), 'real_sat_data.jsonl')"`
  * Verify: `real_sat_data.jsonl` has \~20-30 pairs (e.g., practice feedback, test results).
* **Success Criteria**: Data exported correctly, matches user actions.

#### Step 2.4: Bug Fixing

* **Duration**: 2 days
* **Tasks**:
  * Log errors (e.g., console, `tail -f api.log` if logging added).
  * Fix: 404s, DB connection issues, UI glitches (e.g., feedback not displaying).
* **Success Criteria**: No critical bugs, flows work seamlessly.

***

### 3. Staging Deployment

**Goal**: Test in a production-like environment.

#### Step 3.1: Backend Deployment (AWS EC2)

* **Duration**: 1 day
* **Tasks**:
  1. Launch EC2 (t3.micro, Ubuntu 20.04):
     * SSH: `ssh -i <key.pem> ec2-user@<ec2-ip>`
     * Install: `sudo apt update && sudo apt install python3-pip redis-server nginx`
  2. Copy backend: `scp -r api/ ec2-user@<ec2-ip>:~/`.
  3. Setup:
     * `pip install -r api/requirements.txt`
     * Run DB migration, seed data on EC2.
     * Start Redis: `redis-server &`
     * Run FastAPI: `uvicorn api.main:app --host 0.0.0.0 --port 8000 &`
  4. Configure Nginx:
     *   Edit `/etc/nginx/sites-available/default`:

         ```
         server {
             listen 80;
             server_name <ec2-public-ip>;
             location / {
                 proxy_pass http://localhost:8000;
                 proxy_set_header Host $host;
             }
         }
         ```
     * Restart: `sudo systemctl restart nginx`
  5. **Verify**: `http://<ec2-ip>/docs` shows Swagger UI.

#### Step 3.2: Web App Deployment (Firebase Hosting)

* **Duration**: 1 day
* **Tasks**:
  1. Build: `cd mobile && flutter build web --web-renderer html`
  2. Setup Firebase:
     * Install CLI: `npm install -g firebase-tools`
     * Login: `firebase login`
     * Init: `firebase init hosting` (in `mobile/`), select `build/web`.
  3. Update `api_service.dart`: `const String baseUrl = 'http://<ec2-ip>';`
  4. Deploy: `firebase deploy`
  5. **Verify**: Access `https://<project-id>.web.app`, see Dashboard.

#### Step 3.3: Mobile App Testing (Local Emulators)

* **Duration**: 1 day
* **Tasks**:
  * Update `api_service.dart`: Use EC2 URL.
  * Run: `flutter run --target=lib/main.dart` (Android/iOS emulator).
  * **Verify**: Practice flow works with backend on EC2.

#### Step 3.4: Staging Tests

* **Duration**: 2 days
* **Tasks**:
  1. **Mobile**:
     * Flow: Practice → Feedback → Study Plan → Community.
     * Check: DB tables populated via EC2.
  2. **Web**:
     * Flow: Diagnostic Test → Submit → Results → Tutor Feedback.
     * Check: `TESTS`, `TUTOR_FEEDBACK` updated.
  3. **Performance**:
     * Simulate 10 concurrent users (e.g., Postman scripts).
     * Measure: Response time <2s.
* **Success Criteria**: All features work, no downtime, data collected.

***

### 4. Production Deployment

**Goal**: Launch publicly for real users.

#### Step 4.1: Mobile App Deployment (App Stores)

* **Duration**: 3 days
* **Tasks**:
  1. Build:
     * Android: `flutter build apk --release`
     * iOS: `flutter build ios --release` (requires Xcode, Apple Developer account).
  2. Submit:
     * Google Play: Upload `.apk` to Play Console, configure listing.
     * App Store: Archive via Xcode, submit via App Store Connect.
  3. **Verify**: Download from stores, confirm functionality.

#### Step 4.2: Web App Deployment (Production Firebase)

* **Duration**: 1 day
* **Tasks**:
  * Re-deploy: `firebase deploy` (ensure EC2 URL is final).
  * **Verify**: Public URL (e.g., `https://sat-prep-suite.web.app`) accessible.

#### Step 4.3: Backend Deployment (Production EC2)

* **Duration**: 1 day
* **Tasks**:
  * Same as staging, but:
    * Use domain (e.g., `satprep.com`) via Route 53 + Nginx.
    * Secure: Add SSL (`certbot` for Let’s Encrypt).
  * **Verify**: `https://api.satprep.com/docs` works.

***

### 5. Production Testing

**Goal**: Validate with real-world usage.

#### Step 5.1: Beta Testing

* **Duration**: 1 week
* **Tasks**:
  * Onboard 100 beta users (students, tutors).
  * Flows:
    * Mobile: 10 practice sessions/user.
    * Web: 1 diagnostic + 1 full test/user.
    * Tutor: 5 feedbacks/user.
  * Collect: Feedback via app (e.g., ratings), email surveys.
  * Export: Check `real_sat_data.jsonl` (\~500 pairs expected).
* **Success Criteria**: 80% user satisfaction, data collected.

#### Step 5.2: Performance Testing

* **Duration**: 2 days
* **Tasks**:
  * Simulate 100 concurrent users (e.g., JMeter).
  * Measure: Latency (<2s), uptime (99.9%).
  * **Verify**: No crashes, DB handles load.

#### Step 5.3: Bug Fixing

* **Duration**: 3 days
* **Tasks**:
  * Monitor: EC2 logs (`/var/log/nginx/`), Firebase Crashlytics.
  * Fix: Critical issues (e.g., test submission failures).

***

### 6. Post-Deployment

**Goal**: Prepare for scale and LLM integration.

#### Step 6.1: Monitoring

* **Duration**: Ongoing
* **Tasks**:
  * Add logging: Update `main.py` with `logging` module.
  * Check: DB growth (`SELECT COUNT(*) FROM TESTS;` weekly).
  * Tools: AWS CloudWatch, Firebase Analytics.

#### Step 6.2: Initial LLM Training

* **Duration**: 1 week
* **Tasks**:
  * Combine synthetic (50K pairs) + beta data (\~500 pairs).
  * Train: Use `train_llm.py` on EC2 g4dn.xlarge (\~$10).
  * Integrate: Update `utils.py` → Deploy to EC2.
* **Verify**: Feedback quality matches GPT-4o (manual review).

#### Step 6.3: Scale Plan

* **Tasks**:
  * 1K users: Retrain LLM, monitor latency.
  * 10K users: Upgrade EC2 (t3.medium), add load balancer.

***

### Timeline

* **Week 1**: Local Testing (Unit + Integration)
* **Week 2**: Local Testing (Data + Fixes), Staging Deployment
* **Week 3**: Staging Tests, Production Deployment (Web + Backend)
* **Week 4**: Production Deployment (Mobile), Beta Testing
* **Week 5**: Performance Testing, Fixes, Initial LLM Training

***

### Success Criteria

* **Local**: All tests pass, data flows correctly.
* **Staging**: Mobile + web work with EC2 backend, 10 users handled.
* **Production**: 100 beta users onboarded, 500+ data pairs, <2s latency.
* **Post-Deployment**: LLM deployed, ready for 1K users.

***

#### Conclusion

This plan ensures the **SAT Prep Suite** is tested and deployed across mobile (practice) and web (tests) platforms, with robust data collection for the LLM. Start with local testing—any tweaks before we proceed?
