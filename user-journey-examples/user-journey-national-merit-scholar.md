# User Journey National Merit Scholar

Let’s create a user journey for a high-achieving student who scored 1350 on the PSAT10, is now a sophomore/junior, and is preparing for both the PSAT/NMSQT (to qualify for the National Merit Scholarship) and the SAT. This user will leverage the **SAT Smart Prep App** by **Learner Labs**, focusing on the granular study plan to target specific weaknesses and achieve a top percentile score. The journey will highlight how the app’s features (diagnostics, granular study plans, AI tutor, gamification, and social features) help the user reach their goal.

***

### User Journey: Priya, a Sophomore/Junior Aiming for National Merit Scholarship

#### User Profile

* **Name**: Priya Sharma
* **Age**: 16
* **Background**: Priya is a sophomore (soon to be junior) at a competitive high school in an urban area. She scored 1350 on the PSAT10 (Math: 680, Reading/Writing: 670) as a sophomore, placing her in the 98th percentile. She wants to qualify for the National Merit Scholarship by scoring in the top 1% on the PSAT/NMSQT in October 2025 (her junior year) and aims for a 1500+ SAT score in spring 2026 to apply to Ivy League schools like Harvard.
* **Study Habits**: Priya can study 15 hours per week, 5 days a week, and prefers afternoon study sessions. She’s highly motivated and enjoys gamified learning.
* **Goals**:
  * PSAT/NMSQT: Score 1450+ (top 1% for National Merit, typically 1440-1480 depending on the state, per College Board 2023 data).
  * SAT: Score 1500+ to be competitive for Ivy League admissions.
* **Timeline**: Priya starts using the app in April 2025, giving her 6 months to prepare for the PSAT/NMSQT (October 2025) and 12 months for the SAT (April 2026).

#### Journey

**Step 1: Onboarding and Initial Assessment (April 2025, Day 1)**

* Priya hears about **SAT Smart Prep App** through a school counselor and downloads it from the Google Play Store. She signs up using her Google account (SSO) and begins the onboarding process.
* **Onboarding Details**:
  * **Basic Info**: Priya enters her name, grade (10th, soon to be 11th), and school.
  * **SAT History**: She inputs her PSAT10 scores (Math: 680, Reading/Writing: 670, total: 1350).
  * **Study Preferences**: She sets her PSAT/NMSQT test date (October 1, 2025) and SAT test date (April 1, 2026), study hours (15 hours/week), study days (5 days/week), and preferred time (afternoon).
  * **Diagnostic Test**: Priya completes a 22-question diagnostic test to establish a baseline for SAT prep. Results show:
    * Math Theta: 0.8 (estimated score: 720; strong in Algebra, weak in Advanced Math topics like trigonometry).
    * Reading/Writing Theta: 0.75 (estimated score: 700; strong in Reading Comprehension, weak in Command of Evidence).
  * **Granular Study Plan Creation**:
    * **Backend Logic** (from `study_plan.py`):
      * Proficiencies: Advanced Math (theta: 0.6), Command of Evidence (theta: 0.65) (theta < 0.7, relative weaknesses for a high achiever).
      * Focus Areas: Advanced Math, Command of Evidence.
      * Sessions per Week: 5 days \* (15 hours // 5 days) = 15 sessions.
      * Total Sessions: 2 focus areas \* 15 sessions = 30 sessions.
      * Weeks Until PSAT: (October 1, 2025 - April 1, 2025) ≈ 26 weeks.
      * Sessions per Skill: 30 sessions // 2 skills = 15 sessions per skill.
      * Milestones:
        * Complete 15 sessions in Advanced Math by July 1, 2025 (50 points).
        * Complete 15 sessions in Command of Evidence by October 1, 2025 (50 points).
    * **Granular Study Plan**:
      * **Test Date (PSAT/NMSQT)**: October 1, 2025
      * **Test Date (SAT)**: April 1, 2026
      * **Sessions per Week**: 15 (3 hours/day, 5 days/week)
      * **Focus Areas**: Advanced Math, Command of Evidence
      * **Milestones**:
        * Complete 15 sessions in Advanced Math by July 1, 2025.
        * Complete 15 sessions in Command of Evidence by October 1, 2025.
* Priya earns 50 coins for completing onboarding and unlocks a “Learner Star” avatar as a bonus. She’s excited about the gamified elements and sets a goal to reach the “Gold” league.

**Step 2: Early Preparation Phase (April-May 2025, Days 2-60)**

* Priya starts her study plan, focusing on Advanced Math and Command of Evidence to boost her PSAT/NMSQT score.
* **Day 3**: She completes a 5-question Advanced Math session (afternoon, 45 minutes), focusing on trigonometry. The AI tutor helps her with a sine-cosine problem using real voice input (“How do I solve for sin(x) in a right triangle?”). She earns 25 coins.
* **Day 7**: Priya completes a Command of Evidence session, practicing questions that require identifying supporting evidence in passages. Her theta improves to 0.7.
* **Day 30**: After 4 weeks, Priya has completed 8 Advanced Math sessions and 7 Command of Evidence sessions.
  * **Granular Study Plan Progress**:
    * Advanced Math: 8/15 sessions completed (theta: 0.65, estimated score: 740).
    * Command of Evidence: 7/15 sessions completed (theta: 0.75, estimated score: 720).
  * **Dynamic Adjustment**: The app notices Priya’s improvement in Command of Evidence (theta: 0.75) and adds a new focus area, Vocabulary (theta: 0.7, a relative weakness), with a milestone: "Complete 5 sessions in Vocabulary by June 1, 2025."
* **Gamification**: Priya earns a “Math Prodigy” badge for completing 5 Advanced Math sessions and reaches the “Silver” league (500 points). She checks the leaderboard and sees she’s in the top 10 for Advanced Math among her friends, motivating her to keep going.
* **Push Notification**: “Priya, you’re on a 5-day streak! Complete today’s challenge to earn 50 coins.”

**Step 3: Mid-Term Progress Check and PSAT Prep (July 2025, Day 90)**

* Priya has completed her Advanced Math milestone and is close to finishing Command of Evidence.
* **Granular Study Plan Update**:
  * Advanced Math: 15/15 sessions completed (theta: 0.9, estimated score: 760).
  * Command of Evidence: 12/15 sessions completed (theta: 0.85, estimated score: 740).
  * Vocabulary: 3/5 sessions completed (theta: 0.75, estimated score: 720).
  * **Dynamic Adjustment**: The app adds a new milestone to prepare for the PSAT/NMSQT: "Complete 2 full-length PSAT practice tests by September 1, 2025."
* Priya takes a full-length PSAT practice test, scoring 1400 (Math: 720, Reading/Writing: 680). She’s on track for National Merit but wants to improve her Reading/Writing score.
* **AI Tutor**: Priya uses the AI tutor to review a Command of Evidence question she missed, asking, “Why is option B the best evidence for the author’s claim?” The tutor explains the reasoning, helping her understand the question type better.
* **Social Features**: Priya joins a study team with her friends and challenges a friend to a Command of Evidence competition, earning 50 coins for her team’s leaderboard ranking.

**Step 4: Final PSAT Preparation (September 2025, Day 150)**

* Priya completes her remaining milestones and the full-length PSAT tests.
* **Granular Study Plan Progress**:
  * Command of Evidence: 15/15 sessions completed (theta: 0.9, estimated score: 760).
  * Vocabulary: 5/5 sessions completed (theta: 0.8, estimated score: 740).
  * Full-Length PSAT Tests: 2/2 completed.
* She takes another PSAT practice test, scoring 1460 (Math: 740, Reading/Writing: 720), exceeding her goal of 1450 for National Merit qualification.
* **Gamification**: Priya reaches the “Gold” league (1000 points) and earns a “PSAT Master” badge for her high practice score.

**Step 5: Transition to SAT Prep (October 2025 - March 2026, Days 180-360)**

* After the PSAT/NMSQT in October 2025, Priya shifts focus to SAT prep. She scores 1455 on the PSAT/NMSQT, qualifying for National Merit Semifinalist status in her state.
* **Granular Study Plan Update for SAT**:
  * **Backend Logic**:
    * New Focus Areas: Data Analysis (theta: 0.85), Expression of Ideas (theta: 0.8) (relative weaknesses after PSAT).
    * Sessions per Week: 15 (unchanged).
    * Weeks Until SAT: (April 1, 2026 - October 1, 2025) ≈ 26 weeks.
    * Milestones:
      * Complete 10 sessions in Data Analysis by January 1, 2026 (50 points).
      * Complete 10 sessions in Expression of Ideas by April 1, 2026 (50 points).
      * Complete 3 full-length SAT practice tests by March 1, 2026 (100 points).
* **Day 200**: Priya completes Data Analysis sessions, improving her theta to 0.9. She uses the AI tutor to clarify a statistical inference question.
* **Day 300**: She completes all milestones, including full-length SAT tests, scoring 1490 (Math: 760, Reading/Writing: 730).
* **Score Guarantee Check**: Improvement: 1490 - 1350 (PSAT10) = 140 points. Message: “You’re almost there! Keep going to meet the 150-point guarantee.”
* **Final Push**: Priya focuses on Expression of Ideas, completing additional sessions. She takes a final SAT practice test, scoring 1510 (Math: 770, Reading/Writing: 740).

**Step 6: SAT and Outcome (April 2026, Day 365)**

* Priya takes the SAT in April 2026, scoring 1505 (Math: 765, Reading/Writing: 740), achieving her goal of 1500+.
* **National Merit**: She advances to Finalist status and wins a National Merit Scholarship, boosting her college applications.
* **Satisfaction**: Priya posts on the Community screen: “Thanks to SAT Smart Prep App, I got a 1505 and a National Merit Scholarship!” She earns 100 coins for her post and reaches the top 5 on the global leaderboard.

***

### Summary of Priya’s Granular Study Plan

* **PSAT/NMSQT Prep (April-September 2025)**:
  * **Focus Areas**: Advanced Math, Command of Evidence, Vocabulary (added dynamically), full-length PSAT tests.
  * **Milestones**:
    * Advanced Math: 15 sessions by July 1, 2025.
    * Command of Evidence: 15 sessions by October 1, 2025.
    * Vocabulary: 5 sessions by June 1, 2025.
    * Full-Length PSAT Tests: 2 tests by September 1, 2025.
  * **Impact**: Helped Priya improve her PSAT score to 1460, qualifying for National Merit.
* **SAT Prep (October 2025-March 2026)**:
  * **Focus Areas**: Data Analysis, Expression of Ideas, full-length SAT tests.
  * **Milestones**:
    * Data Analysis: 10 sessions by January 1, 2026.
    * Expression of Ideas: 10 sessions by April 1, 2026.
    * Full-Length SAT Tests: 3 tests by March 1, 2026.
  * **Impact**: Guided Priya to a 1510 SAT score, exceeding her goal of 1500.

***

### Key Takeaways

* **Granular Study Plan**: Priya’s study plan dynamically adjusted to her needs, focusing on weaker areas (Advanced Math, Command of Evidence, Vocabulary for PSAT; Data Analysis, Expression of Ideas for SAT) and ensuring comprehensive preparation with full-length tests.
* **AI Tutor and Gamification**: The AI tutor provided personalized support, while badges, coins, and leaderboards kept Priya motivated.
* **Outcome**: Priya achieved both her PSAT/NMSQT and SAT goals, securing a National Merit Scholarship and a competitive score for Ivy League applications.

This journey demonstrates how **SAT Smart Prep App** supports high-achieving students in reaching ambitious goals through personalized, skill-specific study plans and engaging features. Let me know if you’d like to explore more user journeys or proceed with additional enhancements!
