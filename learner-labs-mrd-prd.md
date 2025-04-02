# Learner Labs MRD/PRD

Below is a **Market Requirements Document (MRD)** and **Product Requirements Document (PRD)** combined into a single cohesive document for the **SAT Prep Suite**. This hybrid approach is tailored for clarity and brevity, suitable for pitching to stakeholders (e.g., investors, developers) while detailing the app’s features, market context, and technical requirements as of March 26, 2025. I’ll explain the features within the PRD section, aligning them with the market needs identified in the MRD.

***

## SAT Prep Suite: MRD/PRD

### 1. Market Requirements Document (MRD)

#### 1.1 Market Overview

* **Target Market**: High school students (juniors/seniors) preparing for the SAT, parents, tutors, and educational institutions in the U.S.
* **Market Size**: 1.5M+ SAT test-takers annually; $4B global test prep market, growing at 5% CAGR.
* **Competitors**:
  * **Free**: Khan Academy (generic, limited personalization).
  * **Premium**: Princeton Review, Kaplan ($1500-$7,500/course, tutor-based).
  * **Mid-Tier**: Varsity Tutors ($20 (group)-$100/hour, semi-personalized).
* **Trends**: Rising demand for affordable, tech-driven education solutions; AI in edtech projected to hit $20B by 2030.

#### 1.2 Customer Pain Points

* **Cost**: Traditional tutoring ($50-$150/hr) is unaffordable for many.
* **Personalization**: Generic platforms fail to adapt to individual strengths/weaknesses.
* **Engagement**: Dry content and lack of motivation lead to drop-off.
* **Access**: Limited options for underserved or rural students.
* **Progress Tracking**: Parents/tutors lack real-time insights into student performance.

#### 1.3 Market Opportunity

* **Gap**: A scalable, affordable, AI-powered SAT prep solution with personalized learning and stakeholder integration.
* **Value Proposition**: "Prep smarter, score higher" with adaptive AI, gamification, and tutor/parent tools at $49-$99/month.
* **Target Penetration**: Capture 1% of SAT takers (15K users) in Year 1, 10% (150K) in Year 5.
* **Future Expansion:** Expand to other tests (ACT, PSAT, MCAT, JEE etc.), Provide a smart platform for corporate training&#x20;

#### 1.4 Business Objectives

* **Revenue**: $5-7M ARR by Year 2 (50K users).
* **User Growth**: 1K beta users in 3 months, 10K in 12 months.
* **Cost Efficiency**: Transition from external AI ($10K/month) to custom LLM ($360/month) by Q3 2025.
* **Impact**: Improve average SAT scores by 100+ points for 80% of users.

***

### 2. Product Requirements Document (PRD)

#### 2.1 Product Overview

* **Name**: SAT Prep Suite
* **Type**: Mobile app (Flutter) + web backend (FastAPI)
* **Purpose**: Deliver personalized SAT prep through AI-driven insights, engaging features, and stakeholder collaboration, while collecting data to train a custom LLM.
* **Platforms**: iOS, Android, Web (initially mobile-focused).
* **Target Release**: Q2 2025 (Beta), Q4 2025 (Full Launch).

#### 2.2 User Personas

1. **Student (Primary)**:
   * Age: 15-18
   * Needs: Affordable prep, score improvement, motivation.
   * Example: "Sara, 16, wants a 1300+ SAT score for college but can’t afford tutors."
2. **Parent**:
   * Age: 35-50
   * Needs: Monitor progress, ensure value.
   * Example: "Mike, 42, wants to track his son’s prep."
3. **Tutor**:
   * Age: 25-50
   * Needs: Student insights, feedback tools.
   * Example: "Jane, 30, tutors 10 students and needs efficiency."

#### 2.3 Features and Requirements

**Feature 1: AI-Driven Insights**

* **Description**: Adaptive learning powered by AI to personalize practice, feedback, and study plans.
* **Sub-Features**:
  1. **Skill Prediction**:
     * Tracks proficiency (1-7 scale) per SAT skill (e.g., Linear Functions).
     * Uses response patterns (accuracy, time) and predictive models.
     * Example: “Your Algebra predicted proficiency is 5.2—on track!”
  2. **Custom Feedback**:
     * Generates real-time, conversational feedback based on performance.
     * Example: “You’re at 70% on Triangles—try breaking proofs into steps.”
  3. **Study Plan Optimization**:
     * Adjusts daily tasks based on skill trends and test date.
     * Example: “Shift day 3 to Geometry—your accuracy dipped to 40%.”
* **Technical Requirements**:
  * Backend: FastAPI endpoint (`/review/next`) with initial external AI (e.g., GPT-4o, Watson).
  * Transition: Custom LLM (300M params) trained on synthetic + user data by Q3 2025.
  * Data: Stored in `RESPONSES`, `RESULTS`, exported to `sat_data.jsonl`.
* **User Benefit**: Tailored prep boosts efficiency and scores.

**Feature 2: Gamification**

* **Description**: Motivational elements to keep students engaged.
* **Sub-Features**:
  1. **Points System**:
     * Earn points for correct answers, streaks (e.g., 10 pts/question, 50 pts/5-day streak).
  2. **Leaderboards**:
     * Weekly rankings among friends or global users.
  3. **Achievements**:
     * Badges for milestones (e.g., “Algebra Ace” for 90% proficiency).
* **Technical Requirements**:
  * Backend: `GAMIFICATION` table (user\_id, points, streak).
  * Frontend: Flutter widgets for progress bars, leaderboard display.
* **User Benefit**: Fun, competitive prep reduces dropout rates.

**Feature 3: Tutor/Parent Integration**

* **Description**: Tools for tutors and parents to monitor and support students.
* **Sub-Features**:
  1. **Progress Tracking**:
     * Real-time view of student proficiencies and trends.
     * Example: “Bob’s Reading Comprehension rose from 4 to 5.5.”
  2. **Tutor Feedback**:
     * Tutors submit contextual feedback (practice, questions, plans).
     * Example: “Great effort, but check your pacing on timed sections.”
  3. **Parent Notifications**:
     * Push alerts for milestones or inactivity (via Firebase).
* **Technical Requirements**:
  * Backend: `TUTORS`, `TUTOR_FEEDBACK` tables; `/tutor/progress`, `/tutor/feedback` endpoints.
  * Frontend: Tutor Dashboard screen in Flutter.
* **User Benefit**: Collaboration enhances accountability and guidance.

**Feature 4: Content Variety**

* **Description**: Diverse learning materials to suit all styles.
* **Sub-Features**:
  1. **Practice Questions**:
     * Adaptive selection (difficulty adjusts via Knewton initially).
  2. **Lessons**:
     * Text + audio explanations (Amazon Polly).
     * Example: “Linear functions: y = mx + b…” (spoken).
  3. **Speech Input**:
     * Voice-based Q\&A (Google Speech-to-Text).
     * Example: “How do I solve quadratics?” → AI response.
* **Technical Requirements**:
  * Backend: `QUESTIONS`, `LESSONS` tables; `/practice/start`, `/lessons/audio` endpoints.
  * Frontend: Audio playback/recording in Flutter.
* **User Benefit**: Flexible, accessible learning for all.

**Feature 5: Community**

* **Description**: Peer support and knowledge sharing.
* **Sub-Features**:
  1. **Notes/Questions**:
     * Users post notes or queries (e.g., “I don’t get proofs”).
  2. **AI Responses**:
     * Initial GPT-4o, later custom LLM answers.
* **Technical Requirements**:
  * Backend: `USER_NOTES` table; `/community/note` endpoint.
  * Frontend: Community screen with text input.
* **User Benefit**: Social learning boosts retention.

**Feature 6: Data Collection for LLM**

* **Description**: Capture user and tutor interactions to train a custom LLM.
* **Sub-Features**:
  1. **Student Feedback Ratings**:
     * Rate AI feedback (1-5 stars).
  2. **Help Requests**:
     * Log user queries and AI responses.
  3. **Study Plan Actions**:
     * Track completions/skips with performance updates.
  4. **Tutor Feedback**:
     * Expert input on student progress.
* **Technical Requirements**:
  * Backend: `FEEDBACK_RATINGS`, `HELP_REQUESTS`, `STUDY_PLAN_ACTIONS`, `TUTOR_FEEDBACK` tables.
  * Export: `export_llm_training_data` to `real_sat_data.jsonl`.
* **User Benefit**: Drives cost-efficient, high-quality AI over time.

***

#### 2.4 Non-Functional Requirements

* **Performance**: <2s response time for AI feedback; handle 10K concurrent users by Year 2.
* **Scalability**: AWS EC2 (t3.micro → t3.medium), Redis caching.
* **Security**: Encrypt user data (AES-256), OAuth for API access.
* **Reliability**: 99.9% uptime, fallback to external AI if LLM fails.
* **Accessibility**: Offline mode for practice, WCAG 2.1 compliance.

***

#### 2.5 Success Metrics

* **User Engagement**: 70% monthly active users (MAU), 5 sessions/week average.
* **Learning Outcomes**: 80% of users improve SAT scores by 100+ points.
* **Data Collection**: 50K training pairs by 10K users (Q4 2025).
* **Revenue**: $150K/month at 10K subscribers.
* **Customer Satisfaction**: 4.5/5 avg. feedback rating.

***

#### 2.6 Development Plan

* **Phase 1 (Q2 2025)**: Local deployment, beta with 100 users, synthetic data LLM.
  * Deliverables: Core app, initial AI integrations.
* **Phase 2 (Q3 2025)**: Production on AWS/Firebase, 1K users, early real data.
  * Deliverables: Custom LLM v1, tutor features.
* **Phase 3 (Q4 2025)**: 10K users, full LLM transition.
  * Deliverables: Scalability upgrades, analytics dashboard.

***

#### 2.7 Risks and Mitigation

* **Risk**: Slow user adoption.
  * **Mitigation**: Freemium tier, school partnerships.
* **Risk**: LLM training delays.
  * **Mitigation**: Start with synthetic data, fallback to GPT-4o.
* **Risk**: High server costs.
  * **Mitigation**: Optimize with caching, transition to custom LLM.

***

#### Conclusion

The **SAT Prep Suite** addresses a critical gap in the $4B test prep market with an AI-powered, affordable, and engaging solution. Its features—AI insights, gamification, tutor/parent tools, diverse content, community, and data collection—meet student needs while building a cost-efficient, scalable platform. By Q4 2025, we aim to serve 10K users, generate $1M+ ARR, and redefine SAT prep with our custom LLM. This is an investment in education’s future—ready to build and scale.

***

#### Explanation of Features

1. **AI-Driven Insights**: Solves personalization pain by adapting to each student’s unique skill profile, leveraging external AI initially and our LLM later for cost savings.
2. **Gamification**: Tackles engagement issues with proven motivational tactics, keeping students consistent.
3. **Tutor/Parent Integration**: Meets oversight needs, adding value for secondary users and enriching LLM data with expert feedback.
4. **Content Variety**: Addresses access and learning style diversity, making prep flexible and inclusive.
5. **Community**: Boosts retention through peer interaction, while collecting additional data.
6. **Data Collection**: The backbone of our long-term strategy, enabling a custom LLM to slash costs and enhance quality as we scale.

This MRD/PRD serves as both a market justification and a technical blueprint—perfect for aligning teams and convincing VCs. Ready to refine or proceed with implementation? Let me know!
