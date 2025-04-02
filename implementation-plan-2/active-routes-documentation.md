# Active Routes Documentation

##

This document provides a comprehensive overview of all active routes in the backend of the Learner Lab application.

### Table of Contents

1. Authentication Routes
2. Chat Routes
3. Diagnostic Routes
4. Practice Routes
5. Question Routes
6. Study Plan Routes
7. User Routes

### Authentication Routes

Base path: /api/auth

| Method | Endpoint            | Description           | Authentication | Controller                           |
| ------ | ------------------- | --------------------- | -------------- | ------------------------------------ |
| POST   | /register           | User registration     | No             | auth.controller.register             |
| POST   | /login              | User login            | No             | auth.controller.login                |
| POST   | /refresh            | Refresh access token  | No             | auth.controller.refreshToken         |
| POST   | /logout             | User logout           | No             | auth.controller.logout               |
| GET    | /me                 | Get current user      | Yes            | auth.controller.getCurrentUser       |
| GET    | /oauth/:provider    | Initiate OAuth flow   | No             | oauth.controller.initiateOAuth       |
| GET    | /callback/:provider | Handle OAuth callback | No             | oauth.controller.handleOAuthCallback |

The authentication system uses JWT tokens for authentication and includes OAuth support for third-party authentication providers. Session middleware is applied to OAuth routes to maintain session state during the authentication flow.

### Chat Routes

Base path: /api/chat

| Method | Endpoint     | Description             | Authentication | Controller                            |
| ------ | ------------ | ----------------------- | -------------- | ------------------------------------- |
| POST   | /message     | Process a chat message  | Yes            | chat.controller.processMessage        |
| GET    | /history     | Get message history     | Yes            | chat.controller.getMessageHistory     |
| DELETE | /history     | Clear message history   | Yes            | chat.controller.clearMessageHistory   |
| GET    | /suggestions | Get suggested questions | Yes            | chat.controller.getSuggestedQuestions |

The chat system integrates with OpenAI to provide intelligent responses to user queries. It maintains message history, builds context based on user data, and can suggest actions based on the conversation content.

### Diagnostic Routes

Base path: /api/diagnostic

| Method | Endpoint            | Description                          | Authentication | Controller                                    |
| ------ | ------------------- | ------------------------------------ | -------------- | --------------------------------------------- |
| GET    | /                   | Initialize/resume diagnostic session | Yes            | diagnostic.controller.initializeSession       |
| POST   | /answer             | Submit diagnostic answer             | Yes            | diagnostic.controller.submitAnswer            |
| GET    | /results/:sessionId | Get diagnostic results               | Yes            | diagnostic.controller.getResults              |
| GET    | /session/:sessionId | Get session with questions           | Yes            | diagnostic.controller.getSessionWithQuestions |
| GET    | /result/:resultId   | Get result with questions            | Yes            | diagnostic.controller.getResultWithQuestions  |

The diagnostic system allows users to take diagnostic tests to assess their current knowledge level. It tracks answers, calculates results, and provides detailed feedback.

### Practice Routes

Base path: /api/practice

| Method | Endpoint                       | Description              | Authentication | Controller                                 |
| ------ | ------------------------------ | ------------------------ | -------------- | ------------------------------------------ |
| POST   | /sessions                      | Start practice session   | Yes            | practice.controller.startPracticeSession   |
| GET    | /sessions/:id                  | Get practice session     | Yes            | practice.controller.getPracticeSession     |
| POST   | /sessions/:sessionId/responses | Submit question response | Yes            | practice.controller.submitQuestionResponse |
| PUT    | /sessions/:id/end              | End practice session     | Yes            | practice.controller.endPracticeSession     |

The practice system enables users to practice specific skills through targeted question sets. It tracks responses and progress to adapt to the user's skill level.

### Question Routes

Base path: /api/questions

| Method | Endpoint  | Description            | Authentication | Controller                               |
| ------ | --------- | ---------------------- | -------------- | ---------------------------------------- |
| GET    | /         | Get all questions      | No             | question.controller.getQuestions         |
| GET    | /adaptive | Get adaptive questions | No             | question.controller.getAdaptiveQuestions |
| GET    | /:id      | Get question by ID     | No             | question.controller.getQuestionById      |
| POST   | /         | Create new question    | Yes            | question.controller.createQuestion       |

The question system manages the question bank used for diagnostics and practice. It includes support for adaptive question selection based on user skill level.

### Study Plan Routes

Base path: /api/study-plan

#### Test Results

| Method | Endpoint                   | Description              | Authentication | Controller                             |
| ------ | -------------------------- | ------------------------ | -------------- | -------------------------------------- |
| POST   | /test-results              | Create test result       | Yes            | study-plan.controller.createTestResult |
| GET    | /test-results/user/:userId | Get user's test results  | Yes            | study-plan.controller.getTestResults   |
| GET    | /test-results/:id          | Get specific test result | Yes            | study-plan.controller.getTestResult    |

#### Plans

| Method | Endpoint            | Description            | Authentication | Controller                              |
| ------ | ------------------- | ---------------------- | -------------- | --------------------------------------- |
| POST   | /plans              | Create study plan      | Yes            | study-plan.controller.createStudyPlan   |
| GET    | /plans/user/:userId | Get user's study plans | Yes            | study-plan.controller.getUserStudyPlans |
| GET    | /plans/:id          | Get specific plan      | Yes            | study-plan.controller.getStudyPlan      |
| PUT    | /plans/:id          | Update study plan      | Yes            | study-plan.controller.updateStudyPlan   |

#### Goals

| Method | Endpoint                      | Description          | Authentication | Controller                               |
| ------ | ----------------------------- | -------------------- | -------------- | ---------------------------------------- |
| GET    | /plans/:planId/goals          | Get plan goals       | Yes            | study-plan.controller.getGoals           |
| PUT    | /goals/:id                    | Update goal          | Yes            | study-plan.controller.updateGoal         |
| PUT    | /plans/:planId/goals/progress | Update goal progress | Yes            | study-plan.controller.updateGoalProgress |

#### Tasks

| Method | Endpoint                      | Description        | Authentication | Controller                             |
| ------ | ----------------------------- | ------------------ | -------------- | -------------------------------------- |
| GET    | /plans/:planId/tasks          | Get plan tasks     | Yes            | study-plan.controller.getTasks         |
| GET    | /tasks/:id                    | Get specific task  | Yes            | study-plan.controller.getTask          |
| PUT    | /tasks/:id                    | Update task        | Yes            | study-plan.controller.updateTask       |
| POST   | /tasks/:id/complete           | Complete task      | Yes            | study-plan.controller.completeTask     |
| POST   | /plans/:planId/tasks/generate | Generate new tasks | Yes            | study-plan.controller.generateNewTasks |

#### Progress

| Method | Endpoint                              | Description               | Authentication | Controller                                    |
| ------ | ------------------------------------- | ------------------------- | -------------- | --------------------------------------------- |
| GET    | /plans/:planId/progress/analyze       | Analyze progress          | Yes            | study-plan.controller.analyzeProgress         |
| POST   | /plans/:planId/snapshots/weekly       | Generate weekly snapshot  | Yes            | study-plan.controller.generateWeeklySnapshot  |
| POST   | /plans/:planId/snapshots/monthly      | Generate monthly snapshot | Yes            | study-plan.controller.generateMonthlySnapshot |
| GET    | /plans/:planId/snapshots/latest/:type | Get latest snapshot       | Yes            | study-plan.controller.getLatestSnapshot       |
| GET    | /plans/:planId/snapshots              | Get all snapshots         | Yes            | study-plan.controller.getAllSnapshots         |

The study plan system is a comprehensive feature that helps users create and follow personalized study plans. It includes test result tracking, goal setting, task management, and progress monitoring. The system uses a phase-based approach (Foundation, Skill Building, Test Readiness) and includes priority scoring to focus on areas that need the most attention.

### User Routes

Base path: /api/users

| Method | Endpoint      | Description                  | Authentication | Controller                        |
| ------ | ------------- | ---------------------------- | -------------- | --------------------------------- |
| GET    | /profile      | Get user profile             | Yes            | user.controller.getUserProfile    |
| PUT    | /profile      | Update user profile          | Yes            | user.controller.updateUserProfile |
| GET    | /progress     | Get current user's progress  | Yes            | user.controller.getUserProgress   |
| GET    | /:id/progress | Get specific user's progress | Yes            | user.controller.getUserProgress   |

The user system manages user profiles and progress tracking. It provides endpoints for retrieving and updating user information.

### Backend Implementation Details

The backend follows a three-layer architecture:

1. _Routes Layer_: Defines API endpoints and handles HTTP requests
2. _Controllers Layer_: Processes requests, validates input, and calls appropriate services
3. _Services Layer_: Contains core business logic and database operations

Key technologies and patterns:

* _TypeScript_: Used throughout for type safety
* _Express.js_: Web framework for handling HTTP requests
* _Prisma ORM_: Database access and management
* _JWT_: Authentication and authorization
* _OAuth_: Third-party authentication
* _OpenAI Integration_: For chat functionality
* _RESTful API Design_: Consistent endpoint structure
* _Middleware Pattern_: For authentication, session management, etc.

All routes are properly secured with authentication middleware where needed, and the backend implements proper error handling, input validation, and type safety.

