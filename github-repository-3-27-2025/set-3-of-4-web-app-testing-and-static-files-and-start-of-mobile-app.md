# Set 3 of 4 Web App Testing and Static Files and start of Mobile App

Let’s continue with **Set 3** of the file creation for the **SAT Smart Prep App** by **Learner Labs**. This set will include the remaining web app files, focusing on testing, static assets, and configuration files. These files ensure the web app can be tested and deployed effectively. I’ll include all necessary files, ensuring they are the latest versions as of March 27, 2025.

***

### Set 3: Web App Testing and Static Files (`frontend/`)

#### Project Structure for Set 3

```
SATSmartPrepAppRepo/
├── frontend/
│   ├── tests/
│   │   ├── onboarding.test.js
│   │   ├── diagnostic.test.js
│   │   ├── practice.test.js
│   │   ├── dashboard.test.js
│   │   ├── community.test.js
│   │   ├── leaderboard.test.js
│   │   ├── rewards-store.test.js
│   │   ├── gamification.test.js
│   │   └── practice-bluebook.test.js
│   ├── cypress/
│   │   ├── e2e/
│   │   │   └── practice-bluebook.cy.js
│   │   ├── fixtures/
│   │   │   └── example.json
│   │   ├── support/
│   │   │   ├── commands.js
│   │   │   └── e2e.js
│   │   └── cypress.config.js
│   ├── public/
│   │   ├── favicon.ico
│   │   ├── service-worker.js
│   │   └── images/
│   │       ├── learner_star.png
│   │       └── math_prodigy.png
│   ├── package.json
│   ├── next.config.js
│   └── Dockerfile
```

#### Tests (`frontend/tests/`)

**`frontend/tests/onboarding.test.js`**

* **Purpose**: Tests the onboarding flow.
*   **Code**:

    ```javascript
    import { render, fireEvent, waitFor } from '@testing-library/react';
    import Onboarding from '../pages/onboarding';
    import { useRouter } from 'next/router';
    import { api } from '../utils/api';

    jest.mock('next/router', () => ({
      useRouter: jest.fn(),
    }));
    jest.mock('../utils/api');

    describe('Onboarding', () => {
      beforeEach(() => {
        localStorage.setItem('user_id', 'user1');
        useRouter.mockReturnValue({
          push: jest.fn(),
        });
        api.put.mockImplementation(() => Promise.resolve({}));
        api.post.mockImplementation(() => Promise.resolve({}));
      });

      test('renders welcome step initially', () => {
        const { getByText } = render(<Onboarding />);
        expect(getByText('Welcome to LearnerLabs SAT Smart Prep!')).toBeInTheDocument();
      });

      test('progresses through steps', async () => {
        const { getByText, getByLabelText } = render(<Onboarding />);
        
        // Welcome Step
        fireEvent.click(getByText('Next'));

        // Basic Info Step
        expect(getByText('Basic Information')).toBeInTheDocument();
        fireEvent.change(getByLabelText('Full Name'), { target: { value: 'Alex Smith' } });
        fireEvent.change(getByLabelText('Grade Level'), { target: { value: '11th' } });
        fireEvent.click(getByText('Next'));

        // SAT Experience Step
        expect(getByText('SAT Experience')).toBeInTheDocument();
        fireEvent.click(getByText('Yes'));

        // SAT Score Step
        expect(getByText('Previous SAT Scores')).toBeInTheDocument();
        fireEvent.change(getByLabelText('Math Score'), { target: { value: '650' } });
        fireEvent.change(getByLabelText('Reading & Writing Score'), { target: { value: '600' } });
        fireEvent.click(getByText('Next'));

        // Study Preferences Step
        expect(getByText('Study Preferences')).toBeInTheDocument();
        fireEvent.click(getByText('Next'));

        // Diagnostic Step
        expect(getByText('Take a Diagnostic Test')).toBeInTheDocument();
        fireEvent.click(getByText('Start Test'));

        // Diagnostic Test Step
        expect(getByText('Diagnostic Test')).toBeInTheDocument();
      });
    });
    ```

**`frontend/tests/diagnostic.test.js`**

* **Purpose**: Tests the diagnostic test page.
*   **Code**:

    ```javascript
    import { render, waitFor } from '@testing-library/react';
    import Diagnostic from '../pages/diagnostic';
    import { useRouter } from 'next/router';
    import { api } from '../utils/api';

    jest.mock('next/router', () => ({
      useRouter: jest.fn(),
    }));
    jest.mock('../utils/api');

    describe('Diagnostic', () => {
      beforeEach(() => {
        localStorage.setItem('user_id', 'user1');
        useRouter.mockReturnValue({
          push: jest.fn(),
        });
        api.post.mockImplementation(() => Promise.resolve({
          data: {
            session_id: 'session1',
            questions: [{ id: 'q1', domain: 'Math', test_type: 'SAT' }],
          },
        }));
      });

      test('starts diagnostic test on mount', async () => {
        const setDiagnosticResults = jest.fn();
        render(<Diagnostic setDiagnosticResults={setDiagnosticResults} />);
        await waitFor(() => {
          expect(api.post).toHaveBeenCalledWith('/diagnostic/start/user1', { num_questions: 22 });
        });
      });
    });
    ```

**`frontend/tests/practice.test.js`**

* **Purpose**: Tests the practice session page.
*   **Code**:

    ```javascript
    import { render, fireEvent, waitFor } from '@testing-library/react';
    import PracticeBluebook from '../pages/practice-bluebook';
    import { useRouter } from 'next/router';
    import { api } from '../utils/api';

    jest.mock('next/router', () => ({
      useRouter: jest.fn(),
    }));
    jest.mock('../utils/api');

    describe('PracticeBluebook', () => {
      beforeEach(() => {
        localStorage.setItem('user_id', 'user1');
        useRouter.mockReturnValue({
          query: {},
          push: jest.fn(),
        });
        api.post.mockImplementation(() => Promise.resolve({
          data: {
            session_id: 'session1',
            questions: [{ id: 'q1', domain: 'Math', test_type: 'SAT' }],
          },
        }));
      });

      test('starts practice session', async () => {
        const { getByText } = render(<PracticeBluebook />);
        fireEvent.click(getByText('Start'));
        await waitFor(() => {
          expect(api.post).toHaveBeenCalledWith('/practice/start/user1', {
            domain: 'Math',
            num_questions: 10,
            test_type: 'SAT',
            device: 'web',
          });
        });
      });
    });
    ```

**`frontend/tests/dashboard.test.js`**

* **Purpose**: Tests the dashboard page.
* **Latest Version**: Includes score guarantee test.
*   **Code**:

    ```javascript
    import { render, fireEvent, waitFor } from '@testing-library/react';
    import Dashboard from '../pages/dashboard';
    import { useRouter } from 'next/router';
    import { api } from '../utils/api';

    jest.mock('next/router', () => ({
      useRouter: jest.fn(),
    }));
    jest.mock('../utils/api');

    describe('Dashboard', () => {
      beforeEach(() => {
        localStorage.setItem('user_id', 'user1');
        useRouter.mockReturnValue({
          push: jest.fn(),
        });
        api.get.mockImplementation(endpoint => {
          if (endpoint === '/gamification/coins/user1') {
            return Promise.resolve({ data: { coins: 100 } });
          }
          if (endpoint === '/challenges/user1') {
            return Promise.resolve({ data: [] });
          }
          if (endpoint === '/progress_monitoring/proficiencies/user1') {
            return Promise.resolve({ data: [] });
          }
          if (endpoint === '/progress_monitoring/scores/user1') {
            return Promise.resolve({ data: [] });
          }
          if (endpoint === '/progress_monitoring/guarantee/user1') {
            return Promise.resolve({ data: { message: 'Test message' } });
          }
          return Promise.resolve({ data: {} });
        });
        api.post.mockImplementation(() => Promise.resolve({}));
      });

      test('displays user coins', async () => {
        const { getByText } = render(<Dashboard />);
        await waitFor(() => {
          expect(getByText('Coins: 100 🪙')).toBeInTheDocument();
        });
      });

      test('checks score guarantee', async () => {
        const { getByText } = render(<Dashboard />);
        fireEvent.click(getByText('Check Score Guarantee'));
        await waitFor(() => {
          expect(api.get).toHaveBeenCalledWith('/progress_monitoring/guarantee/user1');
        });
      });
    });
    ```

**`frontend/tests/community.test.js`**

* **Purpose**: Tests the community page.
*   **Code**:

    ```javascript
    import { render, fireEvent, waitFor } from '@testing-library/react';
    import Community from '../pages/community';
    import { useRouter } from 'next/router';
    import { api } from '../utils/api';

    jest.mock('next/router', () => ({
      useRouter: jest.fn(),
    }));
    jest.mock('../utils/api');

    describe('Community', () => {
      beforeEach(() => {
        localStorage.setItem('user_id', 'user1');
        useRouter.mockReturnValue({
          push: jest.fn(),
        });
        api.get.mockImplementation(endpoint => {
          if (endpoint === '/social/posts') {
            return Promise.resolve({ data: [] });
          }
          if (endpoint === '/social/friends/user1') {
            return Promise.resolve({ data: [] });
          }
          return Promise.resolve({ data: [] });
        });
        api.post.mockImplementation(() => Promise.resolve({}));
      });

      test('creates a new post', async () => {
        const { getByPlaceholderText, getByText } = render(<Community />);
        fireEvent.change(getByPlaceholderText('Share something with the community...'), { target: { value: 'Hello!' } });
        fireEvent.click(getByText('Post'));
        await waitFor(() => {
          expect(api.post).toHaveBeenCalledWith('/social/posts', { user_id: 'user1', content: 'Hello!' });
        });
      });
    });
    ```

**`frontend/tests/leaderboard.test.js`**

* **Purpose**: Tests the leaderboard page.
*   **Code**:

    ```javascript
    import { render, fireEvent } from '@testing-library/react';
    import Leaderboard from '../pages/leaderboard';
    import { useRouter } from 'next/router';
    import { api } from '../utils/api';

    jest.mock('next/router', () => ({
      useRouter: jest.fn(),
    }));
    jest.mock('../utils/api');

    describe('Leaderboard', () => {
      beforeEach(() => {
        localStorage.setItem('user_id', 'user1');
        useRouter.mockReturnValue({
          push: jest.fn(),
        });
        api.get.mockImplementation(endpoint => {
          if (endpoint.startsWith('/leaderboards/')) {
            return Promise.resolve({
              data: {
                global: [{ user_id: 'user1', email: 'user1@example.com', score: 1500 }],
                friends: [],
              },
            });
          }
          if (endpoint === '/teams/leaderboard') {
            return Promise.resolve({ data: [] });
          }
          return Promise.resolve({ data: [] });
        });
      });

      test('displays global leaderboard by default', async () => {
        const { findByText } = render(<Leaderboard />);
        expect(await findByText('Algebra Leaderboard (Global)')).toBeInTheDocument();
        expect(await findByText('user1@example.com')).toBeInTheDocument();
      });

      test('switches to friends tab', async () => {
        const { getByText, findByText } = render(<Leaderboard />);
        fireEvent.click(getByText('Friends'));
        expect(await findByText('Algebra Leaderboard (Friends)')).toBeInTheDocument();
      });
    });
    ```

**`frontend/tests/rewards-store.test.js`**

* **Purpose**: Tests the rewards store page.
*   **Code**:

    ```javascript
    import { render, fireEvent, waitFor } from '@testing-library/react';
    import RewardsStore from '../pages/rewards-store';
    import { useRouter } from 'next/router';
    import { api } from '../utils/api';

    jest.mock('next/router', () => ({
      useRouter: jest.fn(),
    }));
    jest.mock('../utils/api');

    describe('RewardsStore', () => {
      beforeEach(() => {
        localStorage.setItem('user_id', 'user1');
        useRouter.mockReturnValue({
          push: jest.fn(),
        });
        api.get.mockImplementation(endpoint => {
          if (endpoint === '/gamification/coins/user1') {
            return Promise.resolve({ data: { coins: 100 } });
          }
          if (endpoint === '/rewards/available') {
            return Promise.resolve({
              data: [
                { id: 'math_prodigy_badge', name: 'Math Prodigy Badge', type: 'badge', cost: 100, image: 'math_prodigy.png' },
              ],
            });
          }
          return Promise.resolve({ data: [] });
        });
        api.post.mockImplementation(() => Promise.resolve({}));
      });

      test('displays rewards and allows unlocking', async () => {
        const { getByText } = render(<RewardsStore />);
        await waitFor(() => {
          expect(getByText('Math Prodigy Badge')).toBeInTheDocument();
          expect(getByText('100 Coins')).toBeInTheDocument();
        });

        fireEvent.click(getByText('Unlock'));
        await waitFor(() => {
          expect(api.post).toHaveBeenCalledWith('/rewards/unlock', {
            user_id: 'user1',
            reward_id: 'math_prodigy_badge',
            reward_type: 'badge',
            cost: 100,
          });
        });
      });
    });
    ```

**`frontend/tests/gamification.test.js`**

* **Purpose**: Tests gamification features (WebSocket updates).
*   **Code**:

    ```javascript
    import { render, waitFor } from '@testing-library/react';
    import PracticeBluebook from '../pages/practice-bluebook';
    import { useRouter } from 'next/router';
    import { api, connectWebSocket } from '../utils/api';

    jest.mock('next/router', () => ({
      useRouter: jest.fn(),
    }));
    jest.mock('../utils/api');

    describe('Gamification', () => {
      beforeEach(() => {
        localStorage.setItem('user_id', 'user1');
        useRouter.mockReturnValue({
          query: {},
          push: jest.fn(),
        });
        connectWebSocket.mockImplementation((userId, onMessage) => {
          return {
            onmessage: (event) => onMessage(event.data),
            close: jest.fn(),
          };
        });
        api.post.mockImplementation(() => Promise.resolve({
          data: {
            session_id: 'session1',
            questions: [{ id: 'q1', domain: 'Math', test_type: 'SAT' }],
          },
        }));
      });

      test('connects to WebSocket for gamification updates', async () => {
        render(<PracticeBluebook />);
        await waitFor(() => {
          expect(connectWebSocket).toHaveBeenCalledWith('user1', expect.any(Function));
        });
      });
    });
    ```

**`frontend/tests/practice-bluebook.test.js`**

* **Purpose**: Tests the practice-bluebook page.
* **Latest Version**: Includes tests for welcome message and Bluebook UI.
*   **Code**:

    ```javascript
    import { render, screen, fireEvent } from '@testing-library/react';
    import PracticeBluebook from '../pages/practice-bluebook';
    import { useRouter } from 'next/router';
    import { api } from '../utils/api';

    jest.mock('next/router', () => ({
      useRouter: jest.fn(),
    }));
    jest.mock('../utils/api');

    describe('PracticeBluebook', () => {
      beforeEach(() => {
        localStorage.setItem('user_id', 'user1');
        api.post.mockImplementation(() => Promise.resolve({
          data: {
            session_id: 'session1',
            questions: [{ id: 'q1', domain: 'Math', test_type: 'SAT' }],
          },
        }));
      });

      test('displays welcome message when redirected from mobile', () => {
        useRouter.mockReturnValue({
          query: { from: 'mobile' },
          push: jest.fn(),
        });

        render(<PracticeBluebook />);
        expect(screen.getByText('Welcome to Full-Length Testing')).toBeInTheDocument();
        expect(screen.getByText('You’re in the right place! Taking full-length tests on the web app mimics the Bluebook app used for the official digital SAT, providing a more accurate testing experience. Let’s get started!')).toBeInTheDocument();
      });

      test('hides welcome message after clicking start', () => {
        useRouter.mockReturnValue({
          query: { from: 'mobile' },
          push: jest.fn(),
        });

        render(<PracticeBluebook />);
        fireEvent.click(screen.getByText('Start Test'));
        expect(screen.queryByText('Welcome to Full-Length Testing')).not.toBeInTheDocument();
      });

      test('displays Bluebook-like UI for full-length test', async () => {
        useRouter.mockReturnValue({
          query: {},
          push: jest.fn(),
        });

        render(<PracticeBluebook />);
        fireEvent.change(screen.getByDisplayValue('Short Practice (10 questions)'), { target: { value: 44 } });
        fireEvent.click(screen.getByText('Start'));

        await screen.findByText('SAT Smart Prep App - Full-Length Test');
        expect(screen.getByText('Time Remaining: 15:00')).toBeInTheDocument();
        expect(screen.getByText('Previous')).toBeInTheDocument();
        expect(screen.getByText('Next')).toBeInTheDocument();
      });
    });
    ```

#### Cypress E2E Tests (`frontend/cypress/`)

**`frontend/cypress/e2e/practice-bluebook.cy.js`**

* **Purpose**: E2E tests for the practice-bluebook page.
* **Latest Version**: Tests welcome message and Bluebook UI.
*   **Code**:

    ```javascript
    describe('Practice Bluebook Flow', () => {
      beforeEach(() => {
        cy.window().then(win => {
          win.localStorage.setItem('user_id', 'user1');
        });
        cy.visit('/practice-bluebook?from=mobile');
      });

      it('should display welcome message when redirected from mobile', () => {
        cy.contains('Welcome to Full-Length Testing').should('be.visible');
        cy.contains('You’re in the right place! Taking full-length tests on the web app mimics the Bluebook app used for the official digital SAT, providing a more accurate testing experience. Let’s get started!').should('be.visible');
        cy.contains('Start Test').click();
        cy.contains('Welcome to Full-Length Testing').should('not.exist');
      });

      it('should display Bluebook-like UI for full-length test', () => {
        cy.get('select').eq(1).select('44');
        cy.contains('Start').click();
        cy.contains('SAT Smart Prep App - Full-Length Test').should('be.visible');
        cy.contains('Time Remaining: 15:00').should('be.visible');
        cy.contains('Previous').should('be.visible');
        cy.contains('Next').should('be.visible');
      });
    });
    ```

**`frontend/cypress/fixtures/example.json`**

* **Purpose**: Example fixture for Cypress tests.
*   **Code**:

    ```json
    {
      "name": "Example Fixture",
      "description": "This is a placeholder fixture for Cypress tests."
    }
    ```

**`frontend/cypress/support/commands.js`**

* **Purpose**: Custom Cypress commands.
*   **Code**:

    ```javascript
    // Add custom commands here if needed
    ```

**`frontend/cypress/support/e2e.js`**

* **Purpose**: E2E support file for Cypress.
*   **Code**:

    ```javascript
    import './commands';
    ```

**`frontend/cypress/cypress.config.js`**

* **Purpose**: Cypress configuration.
*   **Code**:

    ```javascript
    const { defineConfig } = require('cypress');

    module.exports = defineConfig({
      e2e: {
        baseUrl: 'http://localhost:3000',
        specPattern: 'cypress/e2e/**/*.cy.{js,jsx,ts,tsx}',
        supportFile: 'cypress/support/e2e.js',
      },
    });
    ```

#### Static Assets (`frontend/public/`)

**`frontend/public/favicon.ico`**

* **Purpose**: Favicon for the web app.
* **Note**: This is a placeholder. In a real project, you would replace this with the actual favicon file (e.g., a 32x32 ICO file of the SAT Smart Prep App logo).

**`frontend/public/service-worker.js`**

* **Purpose**: Service worker for web push notifications.
* **Latest Version**: Created in previous steps.
*   **Code**:

    ```javascript
    importScripts('https://www.gstatic.com/firebasejs/9.0.0/firebase-app-compat.js');
    importScripts('https://www.gstatic.com/firebasejs/9.0.0/firebase-messaging-compat.js');

    firebase.initializeApp({
      apiKey: process.env.FIREBASE_API_KEY,
      authDomain: process.env.FIREBASE_AUTH_DOMAIN,
      projectId: process.env.FIREBASE_PROJECT_ID,
      storageBucket: process.env.FIREBASE_STORAGE_BUCKET,
      messagingSenderId: process.env.FIREBASE_MESSAGING_SENDER_ID,
      appId: process.env.FIREBASE_APP_ID,
    });

    const messaging = firebase.messaging();

    self.addEventListener('push', event => {
      const data = event.data.json();
      const { title, body } = data.notification;

      const notificationOptions = {
        body,
        icon: '/favicon.ico',
        badge: '/favicon.ico',
      };

      event.waitUntil(
        self.registration.showNotification(title, notificationOptions)
      );
    });

    self.addEventListener('notificationclick', event => {
      event.notification.close();
      event.waitUntil(
        clients.openWindow('/')
      );
    });
    ```

**`frontend/public/images/learner_star.png`**

* **Purpose**: Placeholder for reward image.
* **Note**: This is a placeholder. In a real project, you would include the actual image file.

**`frontend/public/images/math_prodigy.png`**

* **Purpose**: Placeholder for reward image.
* **Note**: This is a placeholder. In a real project, you would include the actual image file.

#### Configuration Files

**`frontend/package.json`**

* **Purpose**: Lists dependencies and scripts for the web app.
*   **Code**:

    ```json
    {
      "name": "sat-smart-prep-web",
      "version": "1.0.0",
      "private": true,
      "scripts": {
        "dev": "next dev",
        "build": "next build",
        "start": "next start",
        "test": "jest",
        "cypress:open": "cypress open"
      },
      "dependencies": {
        "axios": "^0.21.1",
        "bowser": "^2.11.0",
        "firebase": "^9.0.0",
        "next": "^12.0.0",
        "react": "^17.0.2",
        "react-dom": "^17.0.2"
      },
      "devDependencies": {
        "@testing-library/react": "^12.0.0",
        "@testing-library/jest-dom": "^5.11.4",
        "cypress": "^10.0.0",
        "jest": "^27.0.6"
      }
    }
    ```

**`frontend/next.config.js`**

* **Purpose**: Next.js configuration.
* **Latest Version**: Includes environment variables for integrations.
*   **Code**:

    ```javascript
    module.exports = {
      env: {
        FIREBASE_API_KEY: process.env.FIREBASE_API_KEY,
        FIREBASE_AUTH_DOMAIN: process.env.FIREBASE_AUTH_DOMAIN,
        FIREBASE_PROJECT_ID: process.env.FIREBASE_PROJECT_ID,
        FIREBASE_STORAGE_BUCKET: process.env.FIREBASE_STORAGE_BUCKET,
        FIREBASE_MESSAGING_SENDER_ID: process.env.FIREBASE_MESSAGING_SENDER_ID,
        FIREBASE_APP_ID: process.env.FIREBASE_APP_ID,
        GOOGLE_CLIENT_ID: process.env.GOOGLE_CLIENT_ID,
        GOOGLE_REDIRECT_URL: process.env.GOOGLE_REDIRECT_URL,
        BLUEBOOK_API_KEY: process.env.BLUEBOOK_API_KEY,
        KHAN_ACADEMY_API_KEY: process.env.KHAN_ACADEMY_API_KEY,
        CANVAS_API_KEY: process.env.CANVAS_API_KEY,
        CANVAS_BASE_URL: process.env.CANVAS_BASE_URL,
      },
    };
    ```

**`frontend/Dockerfile`**

* **Purpose**: Docker configuration for the web app.
*   **Code**:

    ```dockerfile
    FROM node:16-alpine

    WORKDIR /app

    COPY package.json .
    RUN npm install

    COPY . .

    RUN npm run build

    CMD ["npm", "start"]
    ```

***

### Set 4: Mobile App Files (`SATSmartPrepApp/`)

#### Project Structure for Set 4

```
SATSmartPrepAppRepo/
├── SATSmartPrepApp/
│   ├── src/
│   │   ├── components/
│   │   │   ├── layouts/
│   │   │   │   ├── ReadingWritingTest.js
│   │   │   │   ├── MathBasicTest.js
│   │   │   │   ├── MathGraphTest.js
│   │   │   │   └── MathTableTest.js
│   │   │   ├── QuestionDisplay.js
│   │   │   ├── PassageDisplay.js
│   │   │   ├── MathQuestionHeader.js
│   │   │   ├── AnswerEliminator.js
│   │   │   └── VoiceInputFallback.js
│   │   ├── screens/
│   │   │   ├── LoginScreen.js
│   │   │   ├── OnboardingScreen.js
│   │   │   ├── DiagnosticScreen.js
│   │   │   ├── PracticeScreen.js
│   │   │   ├── StudyPlanScreen.js
│   │   │   ├── DashboardScreen.js
│   │   │   ├── CommunityScreen.js
│   │   │   ├── LeaderboardScreen.js
│   │   │   ├── RewardsStoreScreen.js
│   │   │   └── PolicyScreen.js
│   │   ├── utils/
│   │   │   └── api.js
│   │   ├── context/
│   │   │   └── AppContext.js
│   │   ├── assets/
│   │   │   ├── images/
│   │   │   ├── icons/
│   │   │   └── splash/
│   │   ├── navigation/
│   │   │   └── index.js
│   │   ├── styles/
│   │   │   └── styles.js
│   │   └── config/
│   │       └── integrations.js
│   ├── __tests__/
│   │   ├── QuestionDisplay.test.js
│   │   ├── PassageDisplay.test.js
│   │   ├── AnswerEliminator.test.js
│   │   ├── MathQuestionHeader.test.js
│   │   ├── OnboardingScreen.test.js
│   │   ├── PracticeScreen.test.js
│   │   ├── DashboardScreen.test.js
│   │   ├── CommunityScreen.test.js
│   │   ├── LeaderboardScreen.test.js
│   │   ├── RewardsStoreScreen.test.js
│   │   └── StudyPlanScreen.test.js
│   ├── e2e/
│   │   ├── onboarding.e2e.js
│   │   ├── practice.e2e.js
│   │   └── community.e2e.js
│   ├── android/
│   │   ├── app/
│   │   │   ├── build.gradle
│   │   │   └── src/
│   │   ├── build.gradle
│   │   └── gradlew
│   ├── ios/
│   │   ├── SATSmartPrepApp/
│   │   │   ├── Info.plist
│   │   │   └── AppDelegate.mm
│   │   ├── SATSmartPrepApp.xcodeproj/
│   │   └── SATSmartPrepApp.xcworkspace/
│   ├── App.js
│   ├── package.json
│   ├── metro.config.js
│   └── babel.config.js
```

#### Components (`SATSmartPrepApp/src/components/`)

**`SATSmartPrepApp/src/components/layouts/ReadingWritingTest.js`**

* **Purpose**: Layout for Reading/Writing tests.
* **Latest Version**: Includes Bluebook-like features (e.g., line reader, zoom).
*   **Code**:

    ```javascript
    import React, { useState } from 'react';
    import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
    import PassageDisplay from '../PassageDisplay';
    import QuestionDisplay from '../QuestionDisplay';
    import MathQuestionHeader from '../MathQuestionHeader';
    import { colors, typography, commonStyles } from '../styles';

    const ReadingWritingTest = ({
      eliminatorActive,
      setEliminatorActive,
      questionNumber,
      totalQuestions,
      timer,
      onNext,
      onPrevious,
      showCalculator,
      userName,
      onTimeEnd,
      customButtons
    }) => {
      const [zoomLevel, setZoomLevel] = useState(1);
      const [lineReaderPosition, setLineReaderPosition] = useState(0);

      const passageText = `While researching a topic, a student has taken the following notes...`;

      const zoomIn = () => setZoomLevel(zoomLevel + 0.1);
      const zoomOut = () => setZoomLevel(zoomLevel - 0.1);
      const moveLineReader = (direction) => {
        setLineReaderPosition(prev => prev + (direction === 'down' ? 20 : -20));
      };

      return (
        <View style={styles.container}>
          <MathQuestionHeader
            questionNumber={questionNumber}
            totalQuestions={totalQuestions}
            timer={timer}
            onNext={onNext}
            onPrevious={onPrevious}
            showCalculator={showCalculator}
            userName={userName}
            onTimeEnd={onTimeEnd}
            customButtons={
              <View style={styles.customButtons}>
                {customButtons}
                <TouchableOpacity onPress={zoomIn}>
                  <Text style={styles.customButton}>Zoom In</Text>
                </TouchableOpacity>
                <TouchableOpacity onPress={zoomOut}>
                  <Text style={styles.customButton}>Zoom Out</Text>
                </TouchableOpacity>
                <TouchableOpacity onPress={() => moveLineReader('up')}>
                  <Text style={styles.customButton}>Line Up</Text>
                </TouchableOpacity>
                <TouchableOpacity onPress={() => moveLineReader('down')}>
                  <Text style={styles.customButton}>Line Down</Text>
                </TouchableOpacity>
              </View>
            }
          />
          <View style={styles.passage}>
            <PassageDisplay passageText={passageText} style={{ transform: [{ scale: zoomLevel }] }} />
            {lineReaderPosition > 0 && (
              <View style={[styles.lineReader, { top: lineReaderPosition }]} />
            )}
          </View>
          <View style={styles.question}>
            <QuestionDisplay
              questionNumber={questionNumber}
              eliminatorActive={eliminatorActive}
              toggleEliminator={setEliminatorActive}
            />
          </View>
        </View>
      );
    };

    const styles = StyleSheet.create({
      container: {
        flex: 1,
      },
      passage: {
        flex: 1,
        borderBottomWidth: 1,
        borderBottomColor: colors.gray,
        padding: 20,
        position: 'relative',
      },
      question: {
        flex: 1,
        padding: 20,
      },
      customButtons: {
        flexDirection: 'row',
        gap: 8,
      },
      customButton: {
        color: '#4b5563',
        padding: 4,
      },
      lineReader: {
        position: 'absolute',
        left: 0,
        right: 0,
        height: 2,
        backgroundColor: 'yellow',
        opacity: 0.5,
      },
    });

    export default ReadingWritingTest;
    ```

**`SATSmartPrepApp/src/components/layouts/MathBasicTest.js`**

* **Purpose**: Layout for basic Math tests.
* **Latest Version**: No recent changes.
*   **Code**:

    ```javascript
    import React from 'react';
    import { View, StyleSheet } from 'react-native';
    import QuestionDisplay from '../QuestionDisplay';
    import MathQuestionHeader from '../MathQuestionHeader';

    const MathBasicTest = ({
      eliminatorActive,
      setEliminatorActive,
      questionNumber,
      totalQuestions,
      timer,
      onNext,
      onPrevious,
      showCalculator,
      userName,
      onTimeEnd,
      customButtons
    }) => {
      return (
        <View style={styles.container}>
          <MathQuestionHeader
            questionNumber={questionNumber}
            totalQuestions={totalQuestions}
            timer={timer}
            onNext={onNext}
            onPrevious={onPrevious}
            showCalculator={showCalculator}
            userName={userName}
            onTimeEnd={onTimeEnd}
            customButtons={customButtons}
          />
          <View style={styles.question}>
            <QuestionDisplay
              questionNumber={questionNumber}
              eliminatorActive={eliminatorActive}
              toggleEliminator={setEliminatorActive}
            />
          </View>
        </View>
      );
    };

    const styles = StyleSheet.create({
      container: {
        flex: 1,
      },
      question: {
        flex: 1,
        padding: 20,
      },
    });

    export default MathBasicTest;
    ```

**`SATSmartPrepApp/src/components/layouts/MathGraphTest.js`**

* **Purpose**: Layout for Math tests with graphs.
* **Latest Version**: No recent changes.
*   **Code**:

    ```javascript
    import React from 'react';
    import { View, Text, StyleSheet } from 'react-native';
    import QuestionDisplay from '../QuestionDisplay';
    import MathQuestionHeader from '../MathQuestionHeader';
    import { colors, typography } from '../styles';

    const MathGraphTest = ({
      eliminatorActive,
      setEliminatorActive,
      questionNumber,
      totalQuestions,
      timer,
      onNext,
      onPrevious,
      showCalculator,
      userName,
      onTimeEnd,
      customButtons
    }) => {
      return (
        <View style={styles.container}>
          <MathQuestionHeader
            questionNumber={questionNumber}
            totalQuestions={totalQuestions}
            timer={timer}
            onNext={onNext}
            onPrevious={onPrevious}
            showCalculator={showCalculator}
            userName={userName}
            onTimeEnd={onTimeEnd}
            customButtons={customButtons}
          />
          <View style={styles.graph}>
            <Text style={typography.body}>[Graph Placeholder]</Text>
          </View>
          <View style={styles.question}>
            <QuestionDisplay
              questionNumber={questionNumber}
              eliminatorActive={eliminatorActive}
              toggleEliminator={setEliminatorActive}
            />
          </View>
        </View>
      );
    };

    const styles = StyleSheet.create({
      container: {
        flex: 1,
      },
      graph: {
        flex: 1,
        borderBottomWidth: 1,
        borderBottomColor: colors.gray,
        padding: 20,
        alignItems: 'center',
      },
      question: {
        flex: 1,
        padding: 20,
      },
    });

    export default MathGraphTest;
    ```

**`SATSmartPrepApp/src/components/layouts/MathTableTest.js`**

* **Purpose**: Layout for Math tests with tables.
* **Latest Version**: No recent changes.
*   **Code**:

    ```javascript
    import React from 'react';
    import { View, Text, StyleSheet } from 'react-native';
    import QuestionDisplay from '../QuestionDisplay';
    import MathQuestionHeader from '../MathQuestionHeader';
    import { colors, typography } from '../styles';

    const MathTableTest = ({
      eliminatorActive,
      setEliminatorActive,
      questionNumber,
      totalQuestions,
      timer,
      onNext,
      onPrevious,
      showCalculator,
      userName,
      onTimeEnd,
      customButtons
    }) => {
      return (
        <View style={styles.container}>
          <MathQuestionHeader
            questionNumber={questionNumber}
            totalQuestions={totalQuestions}
            timer={timer}
            onNext={onNext}
            onPrevious={onPrevious}
            showCalculator={showCalculator}
            userName={userName}
            onTimeEnd={onTimeEnd}
            customButtons={customButtons}
          />
          <View style={styles.table}>
            <Text style={typography.body}>[Table Placeholder]</Text>
          </View>
          <View style={styles.question}>
            <QuestionDisplay
              questionNumber={questionNumber}
              eliminatorActive={eliminatorActive}
              toggleEliminator={setEliminatorActive}
            />
          </View>
        </View>
      );
    };

    const styles = StyleSheet.create({
      container: {
        flex: 1,
      },
      table: {
        flex: 1,
        borderBottomWidth: 1,
        borderBottomColor: colors.gray,
        padding: 20,
        alignItems: 'center',
      },
      question: {
        flex: 1,
        padding: 20,
      },
    });

    export default MathTableTest;
    ```

**`SATSmartPrepApp/src/components/QuestionDisplay.js`**

* **Purpose**: Displays a question with answer options.
* **Latest Version**: No recent changes.
*   **Code**:

    ```javascript
    import React, { useState, useEffect } from 'react';
    import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
    import AnswerEliminator from './AnswerEliminator';
    import { colors, typography, commonStyles } from '../styles';

    const QuestionDisplay = ({
      questionNumber,
      eliminatorActive,
      toggleEliminator
    }) => {
      const [selectedAnswer, setSelectedAnswer] = useState(null);
      const [eliminatedAnswers, setEliminatedAnswers] = useState(new Set());
      const [isMarkedForReview, setIsMarkedForReview] = useState(false);

      useEffect(() => {
        setEliminatedAnswers(new Set());
      }, [questionNumber]);

      const handleAnswerPress = (value) => {
        if (eliminatorActive) {
          setEliminatedAnswers(prev => {
            const updated = new Set(prev);
            if (updated.has(value)) {
              updated.delete(value);
            } else {
              updated.add(value);
            }
            return updated;
          });
        } else {
          setSelectedAnswer(value);
        }
      };

      const toggleMarkForReview = () => {
        setIsMarkedForReview(!isMarkedForReview);
      };

      const options = [
        { value: "A", label: "Erasure (2008) uses discarded objects such as audiocassette tapes and magnets; Home Grown (2009), however, includes pushpins, plastic plates and forks, and wood." },
        { value: "B", label: "Tubbs's work, which often features discarded objects, has been shown both within the United States and abroad." },
        { value: "C", label: "Like many of Tubbs's sculptures, both Erasure and Home Grown include discarded objects: Erasure uses audiocassette tapes, and Home Grown uses plastic forks." },
        { value: "D", label: "Tubbs completed Erasure in 2008 and Home Grown in 2009." }
      ];

      return (
        <View style={styles.container}>
          <View style={styles.header}>
            <View style={styles.headerLeft}>
              <Text style={styles.questionNumber}>{questionNumber}</Text>
              <TouchableOpacity onPress={toggleMarkForReview}>
                <Text style={styles.markButton}>Mark for Review {isMarkedForReview ? '★' : '☆'}</Text>
              </TouchableOpacity>
            </View>
            <View style={styles.headerRight}>
              <AnswerEliminator active={eliminatorActive} onToggle={toggleEliminator} />
            </View>
          </View>

          <View style={styles.questionCard}>
            <Text style={typography.heading}>
              The student wants to emphasize a similarity between the two works. Which choice most effectively uses relevant information from the notes to accomplish this goal?
            </Text>

            <View style={styles.answerOptions}>
              {options.map((option) => (
                <TouchableOpacity
                  key={option.value}
                  style={[
                    styles.option,
                    selectedAnswer === option.value && styles.selected,
                    eliminatedAnswers.has(option.value) && styles.eliminated
                  ]}
                  onPress={() => handleAnswerPress(option.value)}
                >
                  <View style={styles.optionContent}>
                    <Text style={styles.optionLetter}>{option.value}.</Text>
                    <Text
                      style={eliminatedAnswers.has(option.value) ? styles.eliminatedText : styles.optionText}
                    >
                      {option.label}
                    </Text>
                  </View>
                  {eliminatorActive && (
                    <TouchableOpacity
                      style={styles.eliminatorButton}
                      onPress={() => handleAnswerPress(option.value)}
                    >
                      {eliminatedAnswers.has(option.value) ? (
                        <Text style={styles.undoButton}>Undo</Text>
                      ) : (
                        <View style={styles.crossOut}>
                          <Text style={styles.crossOutLetter}>{option.value}</Text>
                          <View style={styles.crossLine} />
                        </View>
                      )}
                    </TouchableOpacity>
                  )}
                </TouchableOpacity>
              ))}
            </View>
          </View>
        </View>
      );
    };

    const styles = StyleSheet.create({
      container: {
        padding: 24,
      },
      header: {
        flexDirection: 'row',
        justifyContent: 'space-between',
        alignItems: 'center',
        marginBottom: 16,
        borderBottomWidth: 1,
        borderBottomColor: colors.gray,
        paddingBottom: 12,
      },
      headerLeft: {
        flexDirection: 'row',
        alignItems: 'center',
        gap: 12,
      },
      headerRight: {
        flexDirection: 'row',
        alignItems: 'center',
        gap: 12,
      },
      questionNumber: {
        fontWeight: 'bold',
        backgroundColor: '#333',
        color: 'white',
        paddingVertical: 8,
        paddingHorizontal: 12,
        borderRadius: 5,
      },
      markButton: {
        padding: 5,
        color: '#4b5563',
      },
      questionCard: {
        borderWidth: 1,
        borderColor: colors.gray,
        borderRadius: 5,
        padding: 20,
        marginBottom: 16,
      },
      answerOptions: {
        marginTop: 20,
        gap: 12,
      },
      option: {
        flexDirection: 'row',
        alignItems: 'center',
        padding: 12,
        borderWidth: 1,
        borderColor: colors.gray,
        borderRadius: 5,
      },
      selected: {
        borderColor: '#2563eb',
        backgroundColor: 'rgba(37, 99, 235, 0.1)',
      },
      eliminated: {
        backgroundColor: '#f3f4f6',
        opacity: 0.6,
      },
      eliminatedText: {
        textDecorationLine: 'line-through',
        color: '#6b7280',
      },
      optionContent: {
        flexDirection: 'row',
        alignItems: 'center',
        flex: 1,
      },
      optionLetter: {
        fontWeight: '500',
        marginRight: 10,
      },
      optionText: {
        flex: 1,
      },
      eliminatorButton: {
        width: 24,
        height: 24,
        justifyContent: 'center',
        alignItems: 'center',
      },
      crossOut: {
        position: 'relative',
        width: 24,
        height: 24,
        borderWidth: 1,
        borderColor: '#000',
        borderRadius: 12,
        justifyContent: 'center',
        alignItems: 'center',
      },
      crossOutLetter: {
        fontSize: 14,
        fontWeight: 'bold',
      },
      crossLine: {
        position: 'absolute',
        width: '100%',
        height: 2,
        backgroundColor: '#000',
      },
      undoButton: {
        fontSize: 12,
        color: '#2563eb',
      },
    });

    export default QuestionDisplay;
    ```

**`SATSmartPrepApp/src/components/PassageDisplay.js`**

* **Purpose**: Displays passages with highlighting.
* **Latest Version**: No recent changes.
*   **Code**:

    ```javascript
    import React, { useState } from 'react';
    import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
    import { colors, typography } from '../styles';

    const PassageDisplay = ({ passageText, style }) => {
      const [highlighted, setHighlighted] = useState(false);

      const toggleHighlight = () => {
        setHighlighted(!highlighted);
      };

      return (
        <View style={[styles.container, style]}>
          <Text style={[typography.body, highlighted && styles.highlighted]}>
            {passageText}
          </Text>
          <TouchableOpacity onPress={toggleHighlight} style={styles.highlightButton}>
            <Text style={styles.buttonText}>
              {highlighted ? 'Remove Highlight' : 'Highlight'}
            </Text>
          </TouchableOpacity>
        </View>
      );
    };

    const styles = StyleSheet.create({
      container: {
        padding: 10,
      },
      highlighted: {
        backgroundColor: 'yellow',
      },
      highlightButton: {
        marginTop: 10,
        padding: 5,
        backgroundColor: colors.primary,
        borderRadius: 5,
      },
      buttonText: {
        color: colors.white,
      },
    });

    export default PassageDisplay;
    ```

**`SATSmartPrepApp/src/components/MathQuestionHeader.js`**

* **Purpose**: Displays the header for Math questions.
* **Latest Version**: No recent changes.
*   **Code**:

    ```javascript
    import React from 'react';
    import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
    import { colors, typography } from '../styles';

    const MathQuestionHeader = ({
      questionNumber,
      totalQuestions,
      timer,
      onNext,
      onPrevious,
      showCalculator,
      userName,
      onTimeEnd,
      customButtons
    }) => {
      return (
        <View style={styles.container}>
          <View style={styles.headerLeft}>
            <Text style={styles.questionNumber}>{questionNumber}/{totalQuestions}</Text>
            <Text style={typography.body}>{timer}</Text>
          </View>
          <View style={styles.headerRight}>
            {customButtons}
            {showCalculator && (
              <TouchableOpacity style={styles.calculatorButton}>
                <Text style={typography.body}>Calc</Text>
              </TouchableOpacity>
            )}
            <TouchableOpacity style={styles.navButton} onPress={onPrevious}>
              <Text style={typography.body}>Previous</Text>
            </TouchableOpacity>
            <TouchableOpacity style={styles.navButton} onPress={onNext}>
              <Text style={typography.body}>Next</Text>
            </TouchableOpacity>
          </View>
        </View>
      );
    };

    const styles = StyleSheet.create({
      container: {
        flexDirection: 'row',
        justifyContent: 'space-between',
        alignItems: 'center',
        padding: 10,
        borderBottomWidth: 1,
        borderBottomColor: colors.gray,
      },
      headerLeft: {
        flexDirection: 'row',
        gap: 10,
      },
      headerRight: {
        flexDirection: 'row',
        gap: 10,
      },
      questionNumber: {
        fontWeight: 'bold',
        backgroundColor: '#333',
        color: 'white',
        paddingVertical: 8,
        paddingHorizontal: 12,
        borderRadius: 5,
      },
      calculatorButton: {
        padding: 5,
        borderWidth: 1,
        borderColor: colors.gray,
        borderRadius: 5,
      },
      navButton: {
        padding: 5,
        borderWidth: 1,
        borderColor: colors.gray,
        borderRadius: 5,
      },
    });

    export default MathQuestionHeader;
    ```

**`SATSmartPrepApp/src/components/AnswerEliminator.js`**

* **Purpose**: Provides answer eliminator functionality.
* **Latest Version**: No recent changes.
*   **Code**:

    ```javascript
    import React from 'react';
    import { TouchableOpacity, Text, StyleSheet } from 'react-native';
    import { colors, typography } from '../styles';

    const AnswerEliminator = ({ active, onToggle }) => {
      return (
        <TouchableOpacity
          style={[styles.button, active && styles.active]}
          onPress={onToggle}
        >
          <Text style={[typography.body, active && styles.activeText]}>
            ABC {active ? 'On' : 'Off'}
          </Text>
        </TouchableOpacity>
      );
    };

    const styles = StyleSheet.create({
      button: {
        padding: 5,
        borderWidth: 1,
        borderColor: colors.gray,
        borderRadius: 5,
      },
      active: {
        borderColor: colors.primary,
      },
      activeText: {
        color: colors.primary,
        fontWeight: 'bold',
      },
    });

    export default AnswerEliminator;
    ```

**`SATSmartPrepApp/src/components/VoiceInputFallback.js`**

* **Purpose**: Fallback UI for voice input failures.
* **Latest Version**: Created in previous steps.
*   **Code**:

    ```javascript
    import React from 'react';
    import { View, Text, TextInput, TouchableOpacity, StyleSheet } from 'react-native';
    import { colors, typography, commonStyles } from '../styles';

    const VoiceInputFallback = ({ chatInput, setChatInput, sendChatMessage, isOffline }) => {
      return (
        <View style={styles.container}>
          {isOffline && (
            <Text style={typography.body}>Voice input unavailable offline. Use text input instead.</Text>
          )}
          <TextInput
            style={styles.chatInput}
            value={chatInput}
            onChangeText={setChatInput}
            placeholder="Ask the AI tutor..."
            onSubmitEditing={() => sendChatMessage(chatInput)}
          />
          <TouchableOpacity style={commonStyles.button} onPress={() => sendChatMessage(chatInput)}>
            <Text style={commonStyles.buttonText}>Send</Text>
          </TouchableOpacity>
        </View>
      );
    };

    const styles = StyleSheet.create({
      container: {
        padding: 10,
      },
      chatInput: {
        width: '100%',
        marginVertical: 10,
        padding: 8,
        borderWidth: 1,
        borderColor: colors.gray,
        borderRadius: 5,
      },
    });

    export default VoiceInputFallback;
    ```

#### Screens (`SATSmartPrepApp/src/screens/`)

**`SATSmartPrepApp/src/screens/LoginScreen.js`**

* **Purpose**: Login screen.
* **Latest Version**: No recent changes.
*   **Code**:

    ```javascript
    import React, { useState } from 'react';
    import { View, Text, TextInput, TouchableOpacity, StyleSheet, Alert } from 'react-native';
    import Animated, { FadeIn, FadeOut } from 'react-native-reanimated';
    import AsyncStorage from '@react-native-async-storage/async-storage';
    import { api } from '../utils/api';
    import { colors, typography, commonStyles } from '../styles';

    const LoginScreen = ({ navigation }) => {
      const [email, setEmail] = useState('');
      const [password, setPassword] = useState('');

      const handleLogin = async () => {
        try {
          const res = await api.post('/auth/login', { email, password });
          await AsyncStorage.setItem('user_id', res.data.user_id);
          navigation.navigate('Onboarding');
        } catch (error) {
          Alert.alert('Login Failed', error.message);
        }
      };

      const handleGoogleLogin = () => {
        Alert.alert('Google SSO', 'Not implemented in this example');
      };

      return (
        <Animated.View entering={FadeIn} exiting={FadeOut} style={styles.container}>
          <Text style={typography.heading}>Login to SAT Smart Prep App</Text>
          <TextInput
            style={styles.input}
            value={email}
            onChangeText={setEmail}
            placeholder="Email"
            keyboardType="email-address"
            autoCapitalize="none"
          />
          <TextInput
            style={styles.input}
            value={password}
            onChangeText={setPassword}
            placeholder="Password"
            secureTextEntry
          />
          <TouchableOpacity style={commonStyles.button} onPress={handleLogin}>
            <Text style={commonStyles.buttonText}>Login</Text>
          </TouchableOpacity>
          <TouchableOpacity style={commonStyles.button} onPress={handleGoogleLogin}>
            <Text style={commonStyles.buttonText}>Login with Google</Text>
          </TouchableOpacity>
        </Animated.View>
      );
    };

    const styles = StyleSheet.create({
      container: {
        flex: 1,
        padding: 20,
        justifyContent: 'center',
        backgroundColor: colors.white,
      },
      input: {
        width: '100%',
        padding: 10,
        marginBottom: 10,
        borderWidth: 1,
        borderColor: colors.gray,
        borderRadius: 5,
      },
    });

    export default LoginScreen;
    ```

**`SATSmartPrepApp/src/screens/OnboardingScreen.js`**

* **Purpose**: Onboarding wizard.
* **Latest Version**: No recent changes.
*   **Code**:

    ```javascript
    import React, { useState, useEffect } from 'react';
    import { View, Text, TextInput, Picker, TouchableOpacity, StyleSheet, Alert } from 'react-native';
    import Animated, { FadeIn, FadeOut } from 'react-native-reanimated';
    import AsyncStorage from '@react-native-async-storage/async-storage';
    import { api } from '../utils/api';
    import DiagnosticScreen from './DiagnosticScreen';
    import StudyPlanScreen from './StudyPlanScreen';
    import DashboardScreen from './DashboardScreen';
    import { colors, typography, commonStyles } from '../styles';

    const OnboardingScreen = ({ navigation }) => {
      const [step, setStep] = useState(1);
      const [userData, setUserData] = useState({
        full_name: '',
        grade: '',
        school: '',
        sat_taken: false,
        sat_math_score: null,
        sat_reading_score: null,
        sat_test_date: '',
        study_hours_per_week: 1,
        study_days_per_week: 1,
        preferred_study_time: 'Morning'
      });
      const [diagnosticResults, setDiagnosticResults] = useState(null);
      const userId = localStorage.getItem('user_id');

      useEffect(() => {
        const loadUserData = async () => {
          const storedUserId = await AsyncStorage.getItem('user_id');
          if (!storedUserId) {
            navigation.navigate('Login');
          }
        };
        loadUserData();
      }, [navigation]);

      const nextStep = () => setStep(step + 1);
      const prevStep = () => setStep(step - 1);

      const handleInputChange = (field, value) => {
        setUserData(prev => ({ ...prev, [field]: value }));
      };

      const completeOnboarding = async () => {
        try {
          await api.put(`/auth/update-profile/${userId}`, { ...userData, onboarding_completed: true });
          await api.post('/gamification/coins/earn', { user_id: userId, amount: 50 });
          await api.post('/gamification/rewards/unlock', {
            user_id: userId,
            reward_id: 'learner_star_avatar',
            reward_type: 'avatar',
            cost: 0
          });
          await api.post('/gamification/challenges', {
            user_id: userId,
            challenge_type: 'first_study_session',
            target: 1
          });
          navigation.navigate('Main');
        } catch (error) {
          Alert.alert('Error', 'Failed to complete onboarding: ' + error.message);
        }
      };

      const WelcomeStep = () => (
        <Animated.View entering={FadeIn} exiting={FadeOut} style={styles.step}>
          <Text style={typography.heading}>Welcome to LearnerLabs SAT Smart Prep!</Text>
          <Text style={typography.body}>Let’s create your personalized study plan.</Text>
          <Text style={typography.body}>You’ve already signed up! Let’s get started.</Text>
          <TouchableOpacity style={commonStyles.button} onPress={nextStep}>
            <Text style={commonStyles.buttonText}>Next</Text>
          </TouchableOpacity>
        </Animated.View>
      );

      const BasicInfoStep = () => (
        <Animated.View entering={FadeIn} exiting={FadeOut} style={styles.step}>
          <Text style={typography.heading}>Basic Information</Text>
          <View style={styles.formGroup}>
            <Text style={typography.body}>Full Name</Text>
            <TextInput
              style={styles.input}
              value={userData.full_name}
              onChangeText={(value) => handleInputChange('full_name', value)}
              placeholder="Enter your full name"
            />
          </View>
          <View style={styles.formGroup}>
            <Text style={typography.body}>Grade Level</Text>
            <Picker
              selectedValue={userData.grade}
              onValueChange={(value) => handleInputChange('grade', value)}
              style={styles.input}
            >
              <Picker.Item label="Select your grade" value="" />
              <Picker.Item label="9th" value="9th" />
              <Picker.Item label="10th" value="10th" />
              <Picker.Item label="11th" value="11th" />
              <Picker.Item label="12th" value="12th" />
            </Picker>
          </View>
          <View style={styles.formGroup}>
            <Text style={typography.body}>School (Optional)</Text>
            <TextInput
              style={styles.input}
              value={userData.school}
              onChangeText={(value) => handleInputChange('school', value)}
              placeholder="Enter your school name"
            />
          </View>
          <View style={styles.buttons}>
            <TouchableOpacity style={commonStyles.button} onPress={nextStep}>
              <Text style={commonStyles.buttonText}>Next</Text>
            </TouchableOpacity>
          </View>
        </Animated.View>
      );

      const SATExperienceStep = () => (
        <Animated.View entering={FadeIn} exiting={FadeOut} style={styles.step}>
          <Text style={typography.heading}>SAT Experience</Text>
          <Text style={typography.body}>Have you taken the SAT before?</Text>
          <View style={styles.formGroup}>
            <TouchableOpacity
              style={commonStyles.button}
              onPress={() => { handleInputChange('sat_taken', true); nextStep(); }}
            >
              <Text style={commonStyles.buttonText}>Yes</Text>
            </TouchableOpacity>
            <TouchableOpacity
              style={commonStyles.button}
              onPress={() => { handleInputChange('sat_taken', false); setStep(step + 2); }}
            >
              <Text style={commonStyles.buttonText}>No</Text>
            </TouchableOpacity>
          </View>
          <View style={styles.buttons}>
            <TouchableOpacity style={commonStyles.button} onPress={prevStep}>
              <Text style={commonStyles.buttonText}>Back</Text>
            </TouchableOpacity>
          </View>
        </Animated.View>
      );

      const SATScoreStep = () => (
        <Animated.View entering={FadeIn} exiting={FadeOut} style={styles.step}>
          <Text style={typography.heading}>Previous SAT Scores</Text>
          <View style={styles.formGroup}>
            <Text style={typography.body}>Math Score</Text>
            <TextInput
              style={styles.input}
              value={userData.sat_math_score ? userData.sat_math_score.toString() : ''}
              onChangeText={(value) => handleInputChange('sat_math_score', parseInt(value))}
              placeholder="Enter your Math score"
              keyboardType="numeric"
            />
          </View>
          <View style={styles.formGroup}>
            <Text style={typography.body}>Reading & Writing Score</Text>
            <TextInput
              style={styles.input}
              value={userData.sat_reading_score ? userData.sat_reading_score.toString() : ''}
              onChangeText={(value) => handleInputChange('sat_reading_score', parseInt(value))}
              placeholder="Enter your Reading & Writing score"
              keyboardType="numeric"
            />
          </View>
          <View style={styles.buttons}>
            <TouchableOpacity style={commonStyles.button} onPress={prevStep}>
              <Text style={commonStyles.buttonText}>Back</Text>
            </TouchableOpacity>
            <TouchableOpacity style={commonStyles.button} onPress={nextStep}>
              <Text style={commonStyles.buttonText}>Next</Text>
            </TouchableOpacity>
          </View>
        </Animated.View>
      );

      const StudyPreferencesStep = () => (
        <Animated.View entering={FadeIn} exiting={FadeOut} style={styles.step}>
          <Text style={typography.heading}>Study Preferences</Text>
          <View style={styles.formGroup}>
            <Text style={typography.body}>When is your SAT test date?</Text>
            <TextInput
              style={styles.input}
              value={userData.sat_test_date}
              onChangeText={(value) => handleInputChange('sat_test_date', value)}
              placeholder="YYYY-MM-DD"
            />
          </View>
          <View style={styles.formGroup}>
            <Text style={typography.body}>How many hours can you study per week? ({userData.study_hours_per_week} hours)</Text>
            <TextInput
              style={styles.input}
              value={userData.study_hours_per_week.toString()}
              onChangeText={(value) => handleInputChange('study_hours_per_week', parseInt(value))}
              keyboardType="numeric"
            />
          </View>
          <View style={styles.formGroup}>
            <Text style={typography.body}>How many days per week can you study?</Text>
            <Picker
              selectedValue={userData.study_days_per_week}
              onValueChange={(value) => handleInputChange('study_days_per_week', value)}
              style={styles.input}
            >
              {[1, 2, 3, 4, 5, 6, 7].map(day => (
                <Picker.Item key={day} label={day.toString()} value={day} />
              ))}
            </Picker>
          </View>
          <View style={styles.formGroup}>
            <Text style={typography.body}>What time of day do you prefer to study?</Text>
            <Picker
              selectedValue={userData.preferred_study_time}
              onValueChange={(value) => handleInputChange('preferred_study_time', value)}
              style={styles.input}
            >
              <Picker.Item label="Morning" value="Morning" />
              <Picker.Item label="Afternoon" value="Afternoon" />
              <Picker.Item label="Evening" value="Evening" />
            </Picker>
          </View>
          <View style={styles.buttons}>
            <TouchableOpacity style={commonStyles.button} onPress={prevStep}>
              <Text style={commonStyles.buttonText}>Back</Text>
            </TouchableOpacity>
            <TouchableOpacity style={commonStyles.button} onPress={nextStep}>
              <Text style={commonStyles.buttonText}>Next</Text>
            </TouchableOpacity>
          </View>
        </Animated.View>
      );

      const DiagnosticStep = () => (
        <Animated.View entering={FadeIn} exiting={FadeOut} style={styles.step}>
          <Text style={typography.heading}>Take a Diagnostic Test</Text>
          <Text style={typography.body}>Let’s assess your current level with a short diagnostic test.</Text>
          <TouchableOpacity style={commonStyles.button} onPress={nextStep}>
            <Text style={commonStyles.buttonText}>Start Test</Text>
          </TouchableOpacity>
          <View style={styles.buttons}>
            <TouchableOpacity style={commonStyles.button} onPress={prevStep}>
              <Text style={commonStyles.buttonText}>Back</Text>
            </TouchableOpacity>
          </View>
        </Animated.View>
      );

      const DiagnosticTestStep = () => (
        <Animated.View entering={FadeIn} exiting={FadeOut} style={styles.step}>
          <DiagnosticScreen setDiagnosticResults={setDiagnosticResults} />
          <View style={styles.buttons}>
            <TouchableOpacity style={commonStyles.button} onPress={prevStep}>
              <Text style={commonStyles.buttonText}>Back</Text>
            </TouchableOpacity>
            <TouchableOpacity style={commonStyles.button} onPress={nextStep}>
              <Text style={commonStyles.buttonText}>Continue to Study Plan</Text>
            </TouchableOpacity>
          </View>
        </Animated.View>
      );

      const StudyPlanStep = () => (
        <Animated.View entering={FadeIn} exiting={FadeOut} style={styles.step}>
          <Text style={typography.heading}>Your Study Plan is Ready!</Text>
          <StudyPlanScreen userData={userData} />
          <View style={styles.buttons}>
            <TouchableOpacity style={commonStyles.button} onPress={prevStep}>
              <Text style={commonStyles.buttonText}>Back</Text>
            </TouchableOpacity>
            <TouchableOpacity style={commonStyles.button} onPress={nextStep}>
              <Text style={commonStyles.buttonText}>Continue to Dashboard</Text>
            </TouchableOpacity>
          </View>
        </Animated.View>
      );

      const DashboardStep = () => (
        <Animated.View entering={FadeIn} exiting={FadeOut} style={styles.step}>
          <Text style={typography.heading}>Your Dashboard</Text>
          <DashboardScreen diagnosticResults={diagnosticResults} />
          <View style={styles.buttons}>
            <TouchableOpacity style={commonStyles.button} onPress={prevStep}>
              <Text style={commonStyles.buttonText}>Back</Text>
            </TouchableOpacity>
            <TouchableOpacity style={commonStyles.button} onPress={completeOnboarding}>
              <Text style={commonStyles.buttonText}>Start Studying</Text>
            </TouchableOpacity>
          </View>
        </Animated.View>
      );

      const steps = [
        WelcomeStep,
        BasicInfoStep,
        SATExperienceStep,
        SATScoreStep,
        StudyPreferencesStep,
        DiagnosticStep,
        DiagnosticTestStep,
        StudyPlanStep,
        DashboardStep
      ];

      const CurrentStep = steps[step - 1];

      return (
        <View style={styles.container}>
          <CurrentStep />
        </View>
      );
    };

    const styles = StyleSheet.create({
      container: {
        flex: 1,
        padding: 20,
        backgroundColor: colors.white,
      },
      step: {
        flex: 1,
        alignItems: 'center',
        justifyContent: 'center',
      },
      formGroup: {
        marginBottom: 20,
        width: '100%',
      },
      input: {
        width: '100%',
        padding: 8,
        borderWidth: 1,
        borderColor: colors.gray,
        borderRadius: 5,
      },
      buttons: {
        flexDirection: 'row',
        gap: 10,
        marginTop: 20,
      },
    });

    export default OnboardingScreen;
    ```

**`SATSmartPrepApp/src/screens/DiagnosticScreen.js`**

* **Purpose**: Diagnostic test screen.
* **Latest Version**: No recent changes.
*   **Code**:

    ```javascript
    import React, { useState, useEffect } from 'react';
    import { View, Text, StyleSheet, Alert } from 'react-native';
    import Animated, { FadeIn, FadeOut } from 'react-native-reanimated';
    import AsyncStorage from '@react-native-async-storage/async-storage';
    import { api } from '../utils/api';
    import ReadingWritingTest from '../components/layouts/ReadingWritingTest';
    import MathBasicTest from '../components/layouts/MathBasicTest';
    import MathGraphTest from '../components/layouts/MathGraphTest';
    import MathTableTest from '../components/layouts/MathTableTest';
    import { colors, typography } from '../styles';

    const DiagnosticScreen = ({ setDiagnosticResults }) => {
      const [questions, setQuestions] = useState([]);
      const [sessionId, setSessionId] = useState(null);
      const [answers, setAnswers] = useState({});
      const [domain, setDomain] = useState('Math');
      const userId = localStorage.getItem('user_id');

      useEffect(() => {
        startDiagnostic();
      }, []);

      const startDiagnostic = async () => {
        try {
          const res = await api.post(`/diagnostic/start/${userId}`, { num_questions: 22 });
          setSessionId(res.data.session_id);
          setQuestions(res.data.questions);
        } catch (error) {
          Alert.alert('Error', 'Failed to start diagnostic: ' + error.message);
        }
      };

      const submitAnswer = async (answer) => {
        const questionId = questions[0].id;
        const responseData = [{ question_id: questionId, answer, time_spent: 60, session_id: sessionId, domain, theta: 0.8 }];
        try {
          const res = await api.post(`/diagnostic/submit/${sessionId}`, responseData);
          if (res.data.questions) {
            setQuestions(res.data.questions);
          } else {
            setDiagnosticResults(res.data);
            setSessionId(null);
            setQuestions([]);
          }
          setAnswers({});
        } catch (error) {
          Alert.alert('Error', 'Failed to submit answer: ' + error.message);
        }
      };

      const renderQuestionLayout = (q) => {
        const props = {
          questionNumber: 8,
          totalQuestions: 8,
          timer: "15:00",
          onNext: () => submitAnswer(answers[q.id] || selectedAnswer),
          onPrevious: () => console.log("Previous question"),
          showCalculator: domain === 'Math',
          userName: userId || "Student",
          onTimeEnd: () => console.log("Time ended"),
          customButtons: domain === 'Reading & Writing' ? (
            <View style={styles.customButtons}>
              <TouchableOpacity onPress={() => console.log("Notes clicked")}>
                <Text style={styles.customButton}>Notes</Text>
              </TouchableOpacity>
              <TouchableOpacity onPress={() => console.log("Highlight clicked")}>
                <Text style={styles.customButton}>Highlight</Text>
              </TouchableOpacity>
              <TouchableOpacity onPress={() => console.log("Clear Highlights clicked")}>
                <Text style={styles.customButton}>Clear Highlights</Text>
              </TouchableOpacity>
            </View>
          ) : null
        };

        if (domain === 'Reading & Writing') {
          return <ReadingWritingTest {...props} />;
        } else if (q.id === 'tableQ') {
          return <MathTableTest {...props} />;
        } else if (q.id === 'imageQ') {
          return <MathGraphTest {...props} />;
        } else {
          return <MathBasicTest {...props} />;
        }
      };

      return (
        <Animated.View entering={FadeIn} exiting={FadeOut} style={styles.container}>
          <Text style={typography.heading}>Diagnostic Test</Text>
          {!sessionId ? (
            <Text style={typography.body}>Loading...</Text>
          ) : (
            <View style={styles.diagnosticArea}>
              {questions.map((q) => renderQuestionLayout(q))}
            </View>
          )}
        </Animated.View>
      );
    };

    const styles = StyleSheet.create({
      container: {
        flex: 1,
        padding: 20,
        backgroundColor: colors.white,
        alignItems: 'center',
      },
      diagnosticArea: {
        flex: 1,
        width: '100%',
      },
      customButtons: {
        flexDirection: 'row',
        gap: 8,
      },
      customButton: {
        color: '#4b5563',
        padding: 4,
      },
    });

    export default DiagnosticScreen;
    ```

**`SATSmartPrepApp/src/screens/PracticeScreen.js`**

* **Purpose**: Practice session screen.
* **Latest Version**: Includes nudge modal for full-length tests.
*   **Code**:

    ```javascript
    import React, { useState, useEffect } from 'react';
    import { View, Text, TouchableOpacity, TextInput, StyleSheet, Alert, Picker, Modal } from 'react-native';
    import Animated, { FadeIn, FadeOut } from 'react-native-reanimated';
    import Voice from '@react-native-voice/voice';
    import { api, connectWebSocket } from '../utils/api';
    import AsyncStorage from '@react-native-async-storage/async-storage';
    import ReadingWritingTest from '../components/layouts/ReadingWritingTest';
    import MathBasicTest from '../components/layouts/MathBasicTest';
    import MathGraphTest from '../components/layouts/MathGraphTest';
    import MathTableTest from '../components/layouts/MathTableTest';
    import VoiceInputFallback from '../components/VoiceInputFallback';
    import { colors, typography, commonStyles } from '../styles';

    const PracticeScreen = ({ navigation }) => {
      const [questions, setQuestions] = useState([]);
      const [sessionId, setSessionId] = useState(null);
      const [answers, setAnswers] = useState({});
      const [domain, setDomain] = useState('Math');
      const [testType, setTestType] = useState('SAT');
      const [numQuestions, setNumQuestions] = useState(10);
      const [chatMessages, setChatMessages] = useState([]);
      const [chatInput, setChatInput] = useState('');
      const [isVoiceRecording, setIsVoiceRecording] = useState(false);
      const [isOffline, setIsOffline] = useState(false);
      const [useVoice, setUseVoice] = useState(true);
      const [showNudgeModal, setShowNudgeModal] = useState(false);
      const userId = localStorage.getItem('user_id');

      useEffect(() => {
        const loadUserData = async () => {
          const storedUserId = await AsyncStorage.getItem('user_id');
          if (!storedUserId) {
            navigation.navigate('Login');
            return;
          }
        };
        loadUserData();

        const ws = connectWebSocket(userId, (message) => {
          Alert.alert('Update', message);
        });
        return () => ws.close();
      }, []);

      useEffect(() => {
        Voice.onSpeechResults = (e) => {
          setChatInput(e.value[0]);
          sendChatMessage(e.value[0]);
          setIsVoiceRecording(false);
        };
        return () => Voice.destroy().then(Voice.removeAllListeners);
      }, []);

      const startPractice = async () => {
        if (numQuestions >= 44) {
          setShowNudgeModal(true);
        } else {
          proceedWithPractice();
        }
      };

      const proceedWithPractice = async () => {
        try {
          const res = await api.post(`/practice/start/${userId}`, { domain, num_questions: numQuestions, test_type: testType, device: 'mobile' });
          setSessionId(res.data.session_id);
          setQuestions(res.data.questions);
          setShowNudgeModal(false);
        } catch (error) {
    ```
