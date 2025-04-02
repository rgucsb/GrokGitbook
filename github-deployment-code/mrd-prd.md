# MRD/PRD

Below is an updated **Market Requirements Document (MRD)** and **Product Requirements Document (PRD)** for the **SAT Prep Suite** project, reflecting the current state of the application as of March 27, 2025. The MRD outlines the market needs, target audience, and competitive landscape, while the PRD details the product features, technical requirements, and implementation specifics based on the development progress, including the recent updates to the Bluebook-inspired layouts and option eliminator functionality.

***

### Market Requirements Document (MRD)

#### 1. Executive Summary

The **SAT Prep Suite** is a comprehensive, AI-driven platform designed to help high school students prepare for the SAT exam. It addresses the growing demand for personalized, adaptive, and engaging test preparation tools by offering features like adaptive diagnostics, practice sessions, full-length tests, AI tutoring, gamification, social features, and offline capabilities. The platform targets students, tutors, and parents, aiming to capture a significant share of the $1.2 billion test prep market by providing a modern, technology-driven alternative to traditional prep methods.

#### 2. Market Overview

**2.1 Market Size and Growth**

* **Total Addressable Market (TAM)**: The global test prep market is valued at $1.2 billion in 2025, with a CAGR of 5.2% (source: industry reports).
* **Serviceable Addressable Market (SAM)**: The SAT prep segment in the U.S. is approximately $400 million, targeting 1.7 million annual SAT test-takers.
* **Growth Drivers**:
  * Increasing competition for college admissions.
  * Shift to digital SAT (Bluebook app) since 2024, requiring digital prep tools.
  * Demand for personalized learning and AI-driven education solutions.

**2.2 Target Audience**

* **Primary Users**: High school students (ages 15-18) preparing for the SAT, primarily in the U.S.
* **Secondary Users**:
  * Tutors: Professionals supporting students in SAT prep.
  * Parents: Monitoring student progress and performance.
* **Demographics**:
  * Students: 1.7 million annual SAT test-takers, 60% female, 40% male, diverse socioeconomic backgrounds.
  * Tutors: 50,000+ SAT tutors in the U.S., often working independently or with test prep companies.
  * Parents: Middle to upper-middle-class families, tech-savvy, invested in education.

**2.3 Market Needs**

* **Personalized Learning**: Students need adaptive tools that adjust to their skill levels (e.g., IRT-based question difficulty).
* **Digital SAT Alignment**: Prep tools must mimic the Bluebook app’s interface and functionality (e.g., option eliminator, highlighter).
* **Engagement**: Gamification (points, badges, leagues) to motivate consistent study habits.
* **Accessibility**: Offline mode for students with limited internet access.
* **Support for Tutors/Parents**: Analytics and notifications to track student progress.
* **Social Features**: Community engagement (posts, comments, friends) to reduce isolation during prep.

**2.4 Competitive Landscape**

* **Khan Academy**: Free SAT prep with official College Board partnership, lacks gamification and social features.
* **The Princeton Review**: Comprehensive but expensive ($500-$2,000), limited personalization.
* **Kaplan**: Offers adaptive learning but lacks offline mode and social features.
* **UWorld**: Strong question bank, but UI is less intuitive than Bluebook.
* **Competitive Advantage**:
  * Bluebook-inspired UI for Reading/Writing and Math layouts.
  * AI-driven tutoring with voice input.
  * Offline mode with sync.
  * Gamification and social features for engagement.

#### 3. Market Opportunity

* **Gap**: Existing tools lack a combination of Bluebook-aligned UI, AI tutoring, offline capabilities, and social engagement.
* **Opportunity**: Capture 5% of the U.S. SAT prep market ($20 million) within 2 years by targeting 85,000 students (1,000 active users initially, scaling to 2,000+).
* **Revenue Model**:
  * Freemium: Free diagnostics and limited practice, premium subscription ($10/month) for full access.
  * Tutor/Parent Analytics: $5/month add-on for detailed reports.

#### 4. Business Objectives

* **Year 1**: Onboard 1,000 active users, achieve 80% user satisfaction (NPS > 50).
* **Year 2**: Scale to 2,000+ users, generate $120,000 in annual recurring revenue (ARR).
* **Long-Term**: Expand to other exams (ACT, GRE), targeting $1 million ARR by Year 5.

***

### Product Requirements Document (PRD)

#### 1. Product Overview

The **SAT Prep Suite** is a web-based platform that provides a personalized, engaging, and accessible SAT preparation experience. It aligns with the digital SAT (Bluebook app) by offering adaptive testing, AI tutoring, gamification, social features, and offline capabilities, catering to students, tutors, and parents.

#### 2. Goals and Objectives

* **User Goal**: Improve SAT scores by 100+ points through personalized prep.
* **Product Goal**: Deliver a Bluebook-aligned, adaptive, and engaging SAT prep experience.
* **Business Goal**: Achieve 1,000 active users in Year 1 with 80% satisfaction.

#### 3. Features and Requirements

**3.1 User Management**

* **Signup/Login**:
  * Users can sign up with email, password, and weekly study hours.
  * Roles: Student, Tutor, Parent (linked to student accounts).
* **Implementation**: Backend (`auth.py`), Frontend (`login.js`).

**3.2 Diagnostics**

* **Adaptive Diagnostic Test**:
  * 22-27 questions, adaptive based on IRT (Item Response Theory).
  * Sections: Math, Reading & Writing.
* **Implementation**: Backend (`diagnostic.py`), Frontend (`diagnostic.js`).

**3.3 Practice Sessions**

* **Adaptive Practice**:
  * 5-10 questions per session, IRT-based difficulty adjustment.
  * Bluebook-inspired layouts:
    * **Reading & Writing**: Passage (25-150 words) on the left, MCQ on the right, highlighter with notes.
    * **Math (Basic)**: Simple problem, MCQ.
    * **Math (Graph)**: Problem with image (e.g., scatterplot), MCQ.
    * **Math (Table)**: Table instructions on the left, SRQ on the right.
* **Option Eliminator**:
  * Toggle in header ("ABC" button), persists across questions.
  * When active, each MCQ option has a circular button (e.g., "A" with a horizontal line).
  * Clicking the button crosses out the option (strikethrough, reduced opacity) and changes to "Undo".
  * New questions reset eliminator buttons to unselected state.
* **Implementation**: Backend (`practice_module.py`), Frontend (`practice-bluebook.js`, `QuestionDisplay.js`).

**3.4 Full-Length Tests**

* **Simulated SAT**:
  * 44-98 questions, modular structure (Math, Reading & Writing).
  * Bluebook layouts as above.
* **Implementation**: Backend (`full_length_test.py`), Frontend (`full-test.js`).

**3.5 Study Plan**

* **Dynamic Plan**:
  * 3 phases: Foundation, Skill Building, Test Readiness.
  * 6-8 full tests, weekly tasks with points.
* **Implementation**: Backend (`study_plan.py`), Frontend (`study-plan.js`).

**3.6 Gamification**

* **Engagement Features**:
  * Points: Earned via practice/tests (e.g., 10 points per question).
  * Badges: "Beginner", "Streak Master", etc.
  * Streaks: Daily study streaks.
  * Leagues: Bronze, Silver, Gold, Platinum (based on points).
* **Implementation**: Backend (`gamification.py`), Frontend (`dashboard.js`, `leaderboard.js`).

**3.7 AI Tutor**

* **Real-Time Assistance**:
  * Chat with voice input (simulated).
  * AI-generated question reviews.
* **Implementation**: Backend (`ai_tutor.py`), Frontend (`practice-bluebook.js`).

**3.8 Social Features**

* **Community**:
  * Posts, comments, friend requests.
* **Implementation**: Backend (`social.py`), Frontend (`community.js`).

**3.9 Offline Mode**

* **Local Caching**:
  * Cache practice/test questions locally.
  * Sync responses when online.
* **Implementation**: Backend (`sync.py`), Frontend (`api.js`).

**3.10 Analytics**

* **Dashboard**:
  * Proficiency trends, predicted scores, insights.
* **Tutor/Parent View**:
  * Student analytics, notifications.
* **Implementation**: Backend (`progress_monitoring.py`, `tutor_parent.py`), Frontend (`dashboard.js`, `tutor-parent.js`).

#### 4. Technical Requirements

**4.1 Backend**

* **Framework**: FastAPI (Python 3.9).
* **Database**: PostgreSQL 14 (14 tables: `users`, `questions`, `responses`, etc.).
* **Caching**: Redis 6 for session data.
* **AI**: WebSocket for real-time chat (`ai_tutor.py`).
* **Deployment**: Docker, AWS ECS (Fargate), 1 vCPU, 2 GB RAM.

**4.2 Frontend**

* **Framework**: Next.js 13 (React 18).
* **UI**: Bluebook-inspired layouts (`ReadingWritingTest.js`, `MathBasicTest.js`, etc.).
* **Libraries**: Framer Motion (animations), Chart.js (analytics).
* **Deployment**: Docker, AWS ECS (Fargate), 0.5 vCPU, 1 GB RAM.

**4.3 Infrastructure**

* **Database**: AWS RDS (PostgreSQL), 20 GB storage.
* **Caching**: AWS ElastiCache (Redis).
* **CI/CD**: GitHub Actions (`deploy.yml`).

#### 5. User Experience (UX)

**5.1 Reading & Writing Layout**

* **Passage**: Left side, 25-150 words, highlightable (yellow).
* **Question**: Right side, MCQ with 4 options.
* **Tools**: Highlights & Notes (yellow solid/dashed/dotted, delete, notes), option eliminator.

**5.2 Math Layouts**

* **Basic**: Centered problem, MCQ, option eliminator.
* **Graph**: Problem with image (e.g., scatterplot), MCQ, option eliminator.
* **Table**: Instructions on left, SRQ on right, numeric keypad.

**5.3 Navigation**

* **Header**: Timer, Directions, Hide, Highlights & Notes, More (Zoom, Line Reader, Answer Eliminator).
* **Footer**: User name, question navigation, Back/Next buttons.

#### 6. Performance Requirements

* **Latency**: < 500ms for API responses.
* **Scalability**: Support 1,000 concurrent users initially, scale to 2,000+.
* **Cost**: \~$365.86/month for 1,000 users (ECS: $36.04, RDS: $12.41, Redis: $12.41, AI: $180).

#### 7. Testing Requirements

* **Unit Tests**:
  * Backend: `pytest` (14 tests: `test_auth.py`, `test_practice_module.py`, etc.).
  * Frontend: Jest (3 tests: `login.test.js`, `practice.test.js`, `full-test.test.js`).
* **Integration Tests**: Verify API endpoints (`auth`, `practice`, `ai_tutor`, etc.).
* **User Testing**: 80% satisfaction (NPS > 50).

#### 8. Timeline and Milestones

* **MVP (Month 1-3)**:
  * User management, diagnostics, practice, study plan.
  * Deployment: Local dev, AWS ECS.
* **Phase 1 (Month 4-6)**:
  * Full-length tests, gamification, AI tutor, social features.
  * Bluebook layouts, option eliminator.
* **Phase 2 (Month 7-9)**:
  * Offline mode, analytics, tutor/parent features.
  * Scale to 1,000 users.
* **Phase 3 (Month 10-12)**:
  * Performance optimization, user feedback integration.
  * Scale to 2,000+ users.

#### 9. Risks and Mitigation

* **Risk**: Low user adoption.
  * **Mitigation**: Gamification, social features, free tier.
* **Risk**: High AWS costs.
  * **Mitigation**: Optimize ECS instances, use spot instances.
* **Risk**: Bluebook UI changes.
  * **Mitigation**: Modular layout components, easy to update.

#### 10. Success Metrics

* **User Metrics**:
  * 1,000 active users in Year 1.
  * 80% user satisfaction (NPS > 50).
* **Business Metrics**:
  * $120,000 ARR in Year 2.
  * 5% market share ($20 million) in 2 years.
* **Product Metrics**:
  * 90% test completion rate.
  * < 500ms API latency.

***

### Summary of Updates

* **MRD**:
  * Updated market size and growth projections for 2025.
  * Added competitive analysis with Bluebook alignment as a key differentiator.
  * Refined target audience and revenue model.
* **PRD**:
  * Added Bluebook-inspired layouts for Reading/Writing and Math (Basic, Graph, Table).
  * Updated option eliminator functionality to persist across questions, with per-option cross-out/undo buttons.
  * Included performance and cost estimates for 1,000 users.
  * Adjusted timeline to reflect current development progress.

This MRD/PRD provides a comprehensive roadmap for the **SAT Prep Suite**, aligning with market needs and the latest product updates. Let me know if you’d like to adjust any sections or add more details!
