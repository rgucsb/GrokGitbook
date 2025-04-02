# User Journey

## SAT Prep Suite User Journey: Alex's Path to the SAT

### Overview

This document follows **Alex**, a high school junior, as they use the **SAT Prep Suite** from signup to their final SAT on **June 1, 2025**. Starting on **March 26, 2025**, Alex aims for a **1400+ score**, leveraging the app’s adaptive testing, AI-driven insights, gamification, and parental support.

***

### Step 1: Signup and Onboarding (March 26, 2025)

* **Action**: Alex downloads the mobile app (React Native) or visits the web version (Next.js) and signs up.
  * **Feature**: User-friendly interface with onboarding (Web: `login.js`, Mobile: `Login.js`).
  * **Experience**: Alex enters their name, email, and sets a password. A guided tour highlights key features (practice, analytics, AI tutor) with tooltips and animations (`framer-motion`).
  * **Outcome**: User ID `alex123` created, role set to "student" in PostgreSQL (`users` table).
* **Parent/Tutor Link**: Alex links their account to their parent (`parent123`) and tutor (`tutor123`) via settings.
  * **Feature**: Tutor/parent integration (`tutor_parent.py`).
  * **Outcome**: Parent and tutor receive access to Alex’s progress.

***

### Step 2: Diagnostic Test (March 27, 2025)

* **Action**: Alex takes a **Math Diagnostic Test** to establish a baseline.
  * **Feature**: Adaptive diagnostic test (`diagnostic.py`).
  * **Experience**: Alex starts via `/diagnostic/start/alex123` with "Math" section. The app serves 22 questions (35 minutes), adapting difficulty using IRT 3PL based on initial (\theta = 0). Alex answers questions (e.g., "Solve 2x + 3 = 7"), submitting via mobile (`Diagnostic.js`).
  * **Outcome**: Score: **620** (Math), (\theta = 0.8). AI review (`question_review.py`) generates feedback: "You missed quadratics—review factoring techniques."
* **Parent Notification**: Post-test, a notification is sent to `parent123`.
  * **Feature**: `tutor_parent.py`.
  * **Outcome**: "Alex completed Math Diagnostic: 620. Focus on quadratics."

***

### Step 3: Study Plan Creation (March 28, 2025)

* **Action**: Alex generates a study plan for June 1, 2025.
  * **Feature**: Personalized study plan (`study_plan.py`).
  * **Experience**: Alex inputs their goal (1400) and weekly hours (15). The app analyzes diagnostic data and creates a 9-week plan, prioritizing weaknesses (e.g., Geometry, Vocabulary).
  * **Sample Plan**:
    * Week 1 (March 28 - April 3):
      * "Practice Geometry (Math)": 10 questions, 50 pts, due March 31.
      * "Practice Vocabulary (R\&W)": 10 questions, 50 pts, due March 31.
      * "Review Math": 5 questions, 25 pts, due April 2.
    * ...continues weekly...
  * **Outcome**: Plan cached offline (`api.js`), viewable in `StudyPlan.js`.

***

### Step 4: Daily Practice and Gamification (March 29 - April 15, 2025)

* **Action**: Alex completes daily adaptive practice sessions.
  * **Feature**: Adaptive practice (`practice_module.py`), gamification (`gamification.py`).
  * **Experience**: Alex tackles 10 Geometry questions (starting (\theta = -0.5)), earning 50 points for completion. Correct answers increase (\theta), unlocking a "Geometry Novice" badge at 90% accuracy. Streaks build daily (3-day streak → motivational tip).
  * **Outcome**: Points: 250, (\theta\_{\text{Geometry\}} = 0.2), streak: 5 days.
* **Offline Mode**: On a bus ride, Alex practices offline.
  * **Feature**: Offline caching (`api.js`).
  * **Outcome**: Responses queued, synced later, points updated.

***

### Step 5: AI Tutor Interaction (April 5, 2025)

* **Action**: Alex seeks help on a tough question.
  * **Feature**: Advanced AI tutor (`ai_tutor.py`).
  * **Experience**: Alex asks via voice (Web Speech API/`react-native-voice`), “How do I solve x^2 - 4 = 0?” The AI responds: “Factor it as (x-2)(x+2)=0, so x=2 or x=-2.” Chat history cached in Redis.
  * **Outcome**: Alex masters quadratics, (\theta\_{\text{Algebra\}} = 1.2).

***

### Step 6: Progress Monitoring and Insights (April 20, 2025)

* **Action**: Alex reviews their progress.
  * **Feature**: Detailed analytics (`progress_monitoring.py`).
  * **Experience**: Dashboard (`analytics.js`) shows:
    * Trends: (\theta\_{\text{Geometry\}} = 0.5) (up from -0.5).
    * Pacing: Algebra avg 80s (improved from 100s).
    * Accuracy: Math 85%, R\&W 70%.
    * Predicted Score: 1250 (Math: 650, R\&W: 600).
    * Insights: "Focus on Vocabulary pacing (avg 95s)."
  * **Outcome**: Alex adjusts study focus.
* **Tutor Review**: Tutor (`tutor123`) views analytics via `/tutor_parent/student_analytics/alex123`.
  * **Outcome**: Tutor suggests extra Vocabulary drills.

***

### Step 7: Full-Length Practice Test (May 10, 2025)

* **Action**: Alex takes a full-length SAT simulation.
  * **Feature**: Full-length test (`full_length_test.py`).
  * **Experience**: Starts with `/full_test/start/alex123` ("Both"). Completes:
    * R\&W Module 1: 27 questions, 32 min.
    * R\&W Module 2: 27 questions, 32 min (harder, (\theta\_{\text{R\&W\}} = 0.9)).
    * 10-min break.
    * Math Module 1: 22 questions, 35 min.
    * Math Module 2: 22 questions, 35 min.
    * Total: 98 questions, 134 min + break.
  * **Outcome**: Score: 1320 (Math: 680, R\&W: 640). AI review: "Missed R\&W inference—practice context clues."
* **Parent Notification**: "Alex scored 1320 on full test. Strong in Math, work on R\&W."

***

### Step 8: Final Prep and Review (May 20 - May 31, 2025)

* **Action**: Alex intensifies practice and reviews.
  * **Feature**: Study plan tasks, AI review (`question_review.py`).
  * **Experience**: Completes remaining tasks (e.g., "Review Writing"), reviews past mistakes (e.g., “Your answer ‘A’ missed the idiom—‘B’ fits context”). Predicted score rises to 1380.
  * **Outcome**: (\theta\_{\text{Math\}} = 1.5), (\theta\_{\text{R\&W\}} = 1.2).

***

### Step 9: SAT Day (June 1, 2025)

* **Action**: Alex takes the official SAT.
  * **Experience**: Confident from adaptive practice, Alex navigates the digital SAT’s 98 questions. Familiarity with pacing (e.g., 1.5 min Math, 1 min R\&W) and strategies from AI feedback shines through.
  * **Outcome**: Official score (received later): **1410**—exceeds goal!
* **Post-SAT**: Parent and tutor receive final analytics update: "Alex’s prep paid off—1410 achieved!"

***

### Key Features in Alex’s Journey

* **Signup**: Seamless onboarding (`login.js`).
* **Diagnostic**: Baseline set (35 min Math, `diagnostic.py`).
* **Study Plan**: 9-week roadmap (`study_plan.py`).
* **Practice**: Adaptive, gamified sessions (`practice_module.py`, `gamification.py`).
* **AI Tutor**: Real-time help (`ai_tutor.py`).
* **Analytics**: Detailed insights (`progress_monitoring.py`).
* **Full Test**: SAT simulation (134 min, `full_length_test.py`).
* **Review**: AI-driven feedback (`question_review.py`).
* **Tutor/Parent**: Oversight and support (`tutor_parent.py`).
* **Offline**: Flexibility (`api.js`).
* **Scalability**: Handles Alex and thousands more (PostgreSQL, Docker).

***

### Conclusion

Alex’s journey showcases how the **SAT Prep Suite** transforms SAT prep into a personalized, engaging, and effective process. From a 620 Math baseline to a 1410 final score, Alex leverages every feature to succeed. Ready to deploy and scale—let’s get more students like Alex to their dream scores!
