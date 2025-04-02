# User Journey Examples

Let’s explore two user journeys for the **SAT Smart Prep App** by **Learner Labs**, focusing on how the granular study plan enhances the experience for different types of users. These journeys will illustrate how the app identifies weak areas, creates skill-specific study plans, and guides users toward their SAT goals. Each journey will include details of the granular study plan, showing how it adapts to the user’s progress and needs. The examples will cover a typical high school student preparing for the SAT and an underserved student participating in the College Board pilot program, reflecting the app’s target audience.

***

### User Journey 1: Alex, a High School Junior Preparing for the SAT

#### User Profile

* **Name**: Alex Johnson
* **Age**: 16
* **Background**: Junior at a suburban high school, aiming for a 1400+ SAT score to apply to competitive colleges like UCLA. Alex has taken the PSAT (scored 1100: 550 Math, 550 Reading/Writing) and has 6 months until the SAT in May 2026.
* **Study Habits**: Can study 10 hours per week, 5 days a week, prefers evening study sessions.
* **Goals**: Improve Math score by 150 points (to 700) and Reading/Writing by 100 points (to 650).

#### Journey

**Step 1: Onboarding (Day 1)**

* Alex downloads **SAT Smart Prep App** from the App Store after seeing a TikTok ad. He signs up using his Google account (SSO) and starts the onboarding process.
* **Onboarding Details**:
  * **Basic Info**: Alex enters his name, grade (11th), and school.
  * **SAT History**: He inputs his PSAT scores (Math: 550, Reading/Writing: 550).
  * **Study Preferences**: He sets his SAT test date (May 1, 2026), study hours (10 hours/week), study days (5 days/week), and preferred time (evening).
  * **Diagnostic Test**: Alex completes a 22-question diagnostic test. Results show:
    * Math Theta: 0.2 (estimated score: 480; weak in Geometry, strong in Algebra).
    * Reading/Writing Theta: 0.3 (estimated score: 520; weak in Grammar, strong in Reading Comprehension).
  * **Study Plan Creation**: The app generates a granular study plan based on Alex’s weak areas (Geometry, Grammar):
    * **Backend Logic** (from `study_plan.py`):
      * Proficiencies: Geometry (theta: 0.1), Grammar (theta: 0.2).
      * Focus Areas: Geometry, Grammar (theta < 0.5).
      * Sessions per Week: 5 days \* (10 hours // 5 days) = 10 sessions.
      * Total Sessions: 2 focus areas \* 10 sessions = 20 sessions.
      * Weeks Until Test: (May 1, 2026 - November 1, 2025) ≈ 26 weeks.
      * Sessions per Skill: 20 sessions // 2 skills = 10 sessions per skill.
      * Milestones: "Complete 10 sessions in Geometry", "Complete 10 sessions in Grammar".
      * Due Dates: Spread over 26 weeks (e.g., Geometry due by week 13, Grammar by week 26).
    * **Granular Study Plan**:
      * **Test Date**: May 1, 2026
      * **Sessions per Week**: 10 (2 hours/day, 5 days/week)
      * **Focus Areas**: Geometry, Grammar
      * **Milestones**:
        * Complete 10 sessions in Geometry by February 1, 2026 (50 points).
        * Complete 10 sessions in Grammar by May 1, 2026 (50 points).
* Alex earns 50 coins for completing onboarding and unlocks a “Learner Star” avatar as a bonus.

**Step 2: First Week of Study (Days 2-7)**

* Alex starts his study plan, focusing on Geometry and Grammar.
* **Day 2**: He completes a 5-question Geometry practice session (evening, 30 minutes). The AI tutor helps him with a tricky angle problem using real voice input (“How do I find the angle of a triangle?”). He earns 25 coins.
* **Day 3**: He completes a 5-question Grammar session, focusing on sentence structure. The app highlights his improvement (Grammar theta increases to 0.25).
* **Day 5**: Alex completes another Geometry session, improving his theta to 0.15. He earns a “Geometry Novice” badge for completing 3 Geometry sessions.
* **Granular Study Plan Progress**:
  * Geometry: 2/10 sessions completed.
  * Grammar: 1/10 sessions completed.
* **Push Notification**: On Day 6, Alex receives a notification: “Alex, you’re on a 3-day streak! Keep going to earn 50 coins.”

**Step 3: Mid-Term Progress Check (Day 30)**

* After 4 weeks, Alex has completed 8 Geometry sessions and 6 Grammar sessions.
* **Granular Study Plan Update**:
  * Geometry: 8/10 sessions completed (theta: 0.3, estimated score: 520).
  * Grammar: 6/10 sessions completed (theta: 0.35, estimated score: 540).
  * **Dynamic Adjustment**: The app notices Alex’s Geometry improvement (theta > 0.25) and shifts focus to Algebra (theta: 0.4, still a relative weakness). A new milestone is added: "Complete 5 sessions in Algebra by March 1, 2026."
* Alex takes a full-length practice test, scoring 1150 (Math: 570, Reading/Writing: 580). He checks the score guarantee on the Dashboard:
  * Improvement: 1150 - 1100 (PSAT) = 50 points.
  * Message: “You’re making progress! Keep going to meet the 150-point guarantee.”

**Step 4: Final Preparation (Day 150)**

* By April 2026, Alex has completed all milestones:
  * Geometry: 10/10 sessions (theta: 0.5, estimated score: 600).
  * Grammar: 10/10 sessions (theta: 0.6, estimated score: 640).
  * Algebra: 5/5 sessions (theta: 0.55, estimated score: 620).
* He takes another full-length test, scoring 1300 (Math: 650, Reading/Writing: 650).
* **Score Guarantee Check**:
  * Improvement: 1300 - 1100 = 200 points.
  * Message: “Great job! You’ve improved by 150+ points.”
* Alex feels confident for his SAT and shares his success on the Community screen, earning 50 coins for his post.

**Outcome**

* **Score Improvement**: Alex improves by 200 points, exceeding the guarantee.
* **Engagement**: He completes 90% of his practice tests and averages 5 challenges/week.
* **Satisfaction**: Alex rates the app 5 stars on the App Store, citing the granular study plan and AI tutor as key to his success.

***

### User Journey 2: Maria, an Underserved Student in the College Board Pilot

#### User Profile

* **Name**: Maria Gonzalez
* **Age**: 17
* **Background**: Senior at a rural high school, first-generation college applicant, part of the College Board pilot program. Maria has limited internet access and has never taken the SAT or PSAT. She aims for a 1200 SAT score to apply to state universities.
* **Study Habits**: Can study 8 hours per week, 4 days a week, prefers morning study sessions.
* **Goals**: Achieve a baseline SAT score of 1200 in 3 months (SAT in February 2026).

#### Journey

**Step 1: Onboarding via College Board Pilot (Day 1)**

* Maria receives free access to **SAT Smart Prep App** through the College Board’s Opportunity Scholarships program. She signs up using her email and starts onboarding.
* **Onboarding Details**:
  * **Basic Info**: Maria enters her name, grade (12th), and school.
  * **SAT History**: She hasn’t taken the SAT or PSAT, so she skips this step.
  * **Study Preferences**: She sets her SAT test date (February 1, 2026), study hours (8 hours/week), study days (4 days/week), and preferred time (morning).
  * **Diagnostic Test**: Maria completes a 22-question diagnostic test in offline mode (due to limited internet). Results show:
    * Math Theta: 0.0 (estimated score: 400; weak in Algebra, Geometry).
    * Reading/Writing Theta: 0.1 (estimated score: 440; weak in Vocabulary, Grammar).
  * **Granular Study Plan Creation**:
    * **Backend Logic**:
      * Proficiencies: Algebra (theta: 0.0), Geometry (theta: 0.05), Vocabulary (theta: 0.1), Grammar (theta: 0.1).
      * Focus Areas: Algebra, Geometry, Vocabulary, Grammar (all theta < 0.5).
      * Sessions per Week: 4 days \* (8 hours // 4 days) = 8 sessions.
      * Total Sessions: 4 focus areas \* 8 sessions = 32 sessions.
      * Weeks Until Test: (February 1, 2026 - November 1, 2025) ≈ 13 weeks.
      * Sessions per Skill: 32 sessions // 4 skills = 8 sessions per skill.
      * Milestones:
        * Complete 8 sessions in Algebra by December 1, 2025 (50 points).
        * Complete 8 sessions in Geometry by December 15, 2025 (50 points).
        * Complete 8 sessions in Vocabulary by January 1, 2026 (50 points).
        * Complete 8 sessions in Grammar by February 1, 2026 (50 points).
    * **Granular Study Plan**:
      * **Test Date**: February 1, 2026
      * **Sessions per Week**: 8 (2 hours/day, 4 days/week)
      * **Focus Areas**: Algebra, Geometry, Vocabulary, Grammar
      * **Milestones**:
        * Complete 8 sessions in Algebra by December 1, 2025.
        * Complete 8 sessions in Geometry by December 15, 2025.
        * Complete 8 sessions in Vocabulary by January 1, 2026.
        * Complete 8 sessions in Grammar by February 1, 2026.
* Maria earns 50 coins for completing onboarding and unlocks a “Learner Star” avatar.

**Step 2: First Month of Study (Days 2-30)**

* Maria studies in offline mode, syncing her progress when she visits the school library weekly.
* **Day 2**: She completes an Algebra session (morning, 1 hour). The app caches her responses.
* **Day 5**: She completes a Grammar session, focusing on sentence structure. She syncs her progress at the library, earning 25 coins.
* **Day 15**: Maria completes 4 Algebra sessions and 3 Grammar sessions. Her Algebra theta improves to 0.1, Grammar to 0.15.
* **Granular Study Plan Progress**:
  * Algebra: 4/8 sessions completed.
  * Grammar: 3/8 sessions completed.
  * Geometry: 0/8 sessions completed.
  * Vocabulary: 0/8 sessions completed.
* **Push Notification**: “Maria, you’ve completed 7 sessions! Keep going to earn a badge.”

**Step 3: Mid-Term Progress Check (Day 45)**

* Maria has completed 6 Algebra sessions, 5 Grammar sessions, 2 Geometry sessions, and 1 Vocabulary session.
* **Granular Study Plan Update**:
  * Algebra: 6/8 sessions (theta: 0.2, estimated score: 480).
  * Grammar: 5/8 sessions (theta: 0.25, estimated score: 500).
  * Geometry: 2/8 sessions (theta: 0.1, estimated score: 440).
  * Vocabulary: 1/8 sessions (theta: 0.15, estimated score: 460).
  * **Dynamic Adjustment**: The app prioritizes Geometry and Vocabulary (lowest thetas) by adding new milestones: "Complete 3 more Geometry sessions by January 15, 2026" and "Complete 3 more Vocabulary sessions by January 15, 2026."
* Maria takes a full-length practice test, scoring 950 (Math: 470, Reading/Writing: 480).

**Step 4: Final Preparation (Day 90)**

* By January 2026, Maria completes all milestones:
  * Algebra: 8/8 sessions (theta: 0.3, estimated score: 520).
  * Grammar: 8/8 sessions (theta: 0.4, estimated score: 560).
  * Geometry: 5/5 sessions (theta: 0.25, estimated score: 500).
  * Vocabulary: 4/4 sessions (theta: 0.3, estimated score: 520).
* She takes another full-length test, scoring 1100 (Math: 540, Reading/Writing: 560).
* **Score Guarantee Check**:
  * Improvement: 1100 - 880 (initial diagnostic) = 220 points.
  * Message: “Great job! You’ve improved by 150+ points.”
* Maria takes the SAT in February 2026, scoring 1150, exceeding her goal of 1200.

**Outcome**

* **Score Improvement**: Maria improves by 220 points on practice tests, exceeding the guarantee.
* **Engagement**: She completes 85% of her practice tests and averages 4 challenges/week, despite limited internet access.
* **Satisfaction**: Maria shares her success in the College Board pilot survey, noting the granular study plan’s focus on her weak areas as a key factor.

***

### Summary of Granular Study Plans in User Journeys

* **Alex’s Study Plan**:
  * **Focus Areas**: Geometry, Grammar (later adjusted to include Algebra).
  * **Milestones**: 10 sessions per skill, dynamically adjusted based on progress (e.g., added Algebra when Geometry improved).
  * **Impact**: Helped Alex target his weakest areas, leading to a 200-point improvement.
* **Maria’s Study Plan**:
  * **Focus Areas**: Algebra, Geometry, Vocabulary, Grammar (later adjusted to prioritize Geometry and Vocabulary).
  * **Milestones**: 8 sessions per skill initially, with additional sessions added for lagging areas.
  * **Impact**: Supported Maria’s limited study time and internet access, resulting in a 220-point improvement.

These journeys demonstrate how the granular study plan personalizes the learning experience, dynamically adjusts to user progress, and supports diverse user needs, ultimately driving significant score improvements. Let me know if you’d like to explore more user journeys or proceed with additional features!
