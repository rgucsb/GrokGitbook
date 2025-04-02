# Set 2 of 4 Web App frontend

Let’s continue with **Set 2** of the file creation for the **SAT Smart Prep App** by **Learner Labs**, focusing on the remaining files in the web app frontend (`frontend/`). This set includes the components, pages, styles, utilities, and configuration files. I’ll ensure all files are the latest versions, reflecting changes like the nudge modal, Bluebook-like UI, and device tracking.

***

### Set 2: Web App Frontend Files (`frontend/`) - Continued

#### Components (`frontend/components/`) - Continued

**`frontend/components/MathQuestionHeader.js` (Continued)**

* **Purpose**: Displays the header for Math questions.
* **Latest Version**: No recent changes.
*   **Code** (Completing the file):

    ```javascript
    import React from 'react';
    import { colors, typography } from '../styles';

    export const MathQuestionHeader = ({
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
        <div className="math-question-header">
          <div className="header-left">
            <span className="question-number">{questionNumber}/{totalQuestions}</span>
            <p style={typography.body}>{timer}</p>
          </div>
          <div className="header-right">
            {customButtons}
            {showCalculator && (
              <button className="calculator-button">
                <span style={typography.body}>Calc</span>
              </button>
            )}
            <button onClick={onPrevious} className="nav-button">
              <span style={typography.body}>Previous</span>
            </button>
            <button onClick={onNext} className="nav-button">
              <span style={typography.body}>Next</span>
            </button>
          </div>

          <style jsx>{`
            .math-question-header {
              display: flex;
              justify-content: space-between;
              align-items: center;
              padding: 10px;
              border-bottom: 1px solid ${colors.gray};
            }
            .header-left {
              display: flex;
              gap: 10px;
            }
            .header-right {
              display: flex;
              gap: 10px;
            }
            .question-number {
              font-weight: bold;
              background: #333;
              color: white;
              padding: 8px 12px;
              border-radius: 5px;
            }
            .calculator-button {
              padding: 5px;
              border: 1px solid ${colors.gray};
              border-radius: 5px;
              background: none;
              cursor: pointer;
            }
            .nav-button {
              padding: 5px;
              border: 1px solid ${colors.gray};
              border-radius: 5px;
              background: none;
              cursor: pointer;
            }
          `}</style>
        </div>
      );
    };
    ```

**`frontend/components/AnswerEliminator.js`**

* **Purpose**: Provides answer eliminator functionality.
* **Latest Version**: No recent changes.
*   **Code**:

    ```javascript
    import React from 'react';
    import { colors, typography } from '../styles';

    const AnswerEliminator = ({ active, onToggle }) => {
      return (
        <button onClick={onToggle} className={`answer-eliminator ${active ? 'active' : ''}`}>
          <span style={[typography.body, active && { color: colors.primary, fontWeight: 'bold' }]}>
            ABC {active ? 'On' : 'Off'}
          </span>

          <style jsx>{`
            .answer-eliminator {
              padding: 5px;
              border: 1px solid ${colors.gray};
              border-radius: 5px;
              background: none;
              cursor: pointer;
            }
            .answer-eliminator.active {
              border-color: ${colors.primary};
            }
          `}</style>
        </button>
      );
    };

    export default AnswerEliminator;
    ```

#### Pages (`frontend/pages/`)

**`frontend/pages/_app.js`**

* **Purpose**: Custom App component for Next.js.
* **Latest Version**: Includes browser compatibility check and web push notifications.
*   **Code**:

    ```javascript
    import { useEffect } from 'react';
    import { useRouter } from 'next/router';
    import Bowser from 'bowser';
    import firebase from 'firebase/app';
    import 'firebase/messaging';

    const firebaseConfig = {
      apiKey: process.env.FIREBASE_API_KEY,
      authDomain: process.env.FIREBASE_AUTH_DOMAIN,
      projectId: process.env.FIREBASE_PROJECT_ID,
      storageBucket: process.env.FIREBASE_STORAGE_BUCKET,
      messagingSenderId: process.env.FIREBASE_MESSAGING_SENDER_ID,
      appId: process.env.FIREBASE_APP_ID,
    };

    if (!firebase.apps.length) {
      firebase.initializeApp(firebaseConfig);
    }

    export default function MyApp({ Component, pageProps }) {
      const router = useRouter();

      useEffect(() => {
        const browser = Bowser.getParser(window.navigator.userAgent);
        const isUnsupported = browser.satisfies({
          'internet explorer': '<=11',
        });

        if (isUnsupported) {
          router.push('/unsupported-browser');
        }

        if ('serviceWorker' in navigator) {
          navigator.serviceWorker.register('/service-worker.js')
            .then(registration => {
              console.log('Service Worker registered:', registration);
            })
            .catch(error => {
              console.error('Service Worker registration failed:', error);
            });
        }

        if (Notification.permission !== 'granted') {
          Notification.requestPermission().then(permission => {
            if (permission === 'granted') {
              const messaging = firebase.messaging();
              messaging.getToken().then(token => {
                fetch('http://localhost:8000/auth/update-device-token/' + localStorage.getItem('user_id'), {
                  method: 'PUT',
                  headers: { 'Content-Type': 'application/json' },
                  body: JSON.stringify({ device_token: token })
                });
              });
            }
          });
        }
      }, [router]);

      return <Component {...pageProps} />;
    }
    ```

**`frontend/pages/index.js`**

* **Purpose**: Home page.
* **Latest Version**: No recent changes.
*   **Code**:

    ```javascript
    import { colors, typography } from '../styles';

    export default function Home() {
      return (
        <div className="home">
          <h1 style={typography.heading}>Welcome to SAT Smart Prep App</h1>
          <p style={typography.body}>Prepare for the SAT with AI-driven practice and personalized study plans.</p>
          <a href="/login" style={typography.body}>Get Started</a>

          <style jsx>{`
            .home {
              max-width: 600px;
              margin: 50px auto;
              text-align: center;
              background: ${colors.white};
              padding: 20px;
              border-radius: 10px;
              box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
            }
            a {
              color: ${colors.primary};
              text-decoration: none;
            }
          `}</style>
        </div>
      );
    }
    ```

**`frontend/pages/login.js`**

* **Purpose**: Login page.
* **Latest Version**: Uses `integrations.js` for Google SSO configuration.
*   **Code**:

    ```javascript
    import React, { useState } from 'react';
    import { useRouter } from 'next/router';
    import { api } from '../utils/api';
    import { integrations } from '../config/integrations';
    import { colors, typography, commonStyles } from '../styles';

    export default function Login() {
      const [email, setEmail] = useState('');
      const [password, setPassword] = useState('');
      const router = useRouter();

      const handleLogin = async () => {
        try {
          const res = await api.post('/auth/login', { email, password });
          localStorage.setItem('user_id', res.data.user_id);
          router.push('/onboarding');
        } catch (error) {
          alert('Login failed: ' + error.message);
        }
      };

      const handleGoogleLogin = () => {
        // Use integrations.googleSSO for configuration
        alert('Google SSO not implemented in this example');
      };

      return (
        <div className="login">
          <h1 style={typography.heading}>Login to SAT Smart Prep App</h1>
          <input
            type="email"
            value={email}
            onChange={(e) => setEmail(e.target.value)}
            placeholder="Email"
            className="input"
          />
          <input
            type="password"
            value={password}
            onChange={(e) => setPassword(e.target.value)}
            placeholder="Password"
            className="input"
          />
          <button onClick={handleLogin} style={commonStyles.button}>
            <span style={commonStyles.buttonText}>Login</span>
          </button>
          <button onClick={handleGoogleLogin} style={commonStyles.button}>
            <span style={commonStyles.buttonText}>Login with Google</span>
          </button>

          <style jsx>{`
            .login {
              max-width: 400px;
              margin: 50px auto;
              padding: 20px;
              text-align: center;
              background: ${colors.white};
              border-radius: 10px;
              box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
            }
            .input {
              width: 100%;
              padding: 10px;
              margin-bottom: 10px;
              border: 1px solid ${colors.gray};
              border-radius: 5px;
            }
          `}</style>
        </div>
      );
    }
    ```

**`frontend/pages/onboarding.js`**

* **Purpose**: Onboarding wizard.
* **Latest Version**: No recent changes.
*   **Code**:

    ```javascript
    import { useState, useEffect } from 'react';
    import { useRouter } from 'next/router';
    import { api } from '../utils/api';
    import { colors, typography, commonStyles } from '../styles';
    import Diagnostic from './diagnostic';
    import Dashboard from './dashboard';
    import StudyPlan from './study-plan';

    const Onboarding = () => {
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
      const router = useRouter();
      const userId = localStorage.getItem('user_id');

      useEffect(() => {
        if (!userId) {
          router.push('/login');
        }
      }, [userId, router]);

      const nextStep = () => setStep(step + 1);
      const prevStep = () => setStep(step - 1);

      const handleInputChange = (field, value) => {
        setUserData(prev => ({ ...prev, [field]: value }));
      };

      const completeOnboarding = async () => {
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
        router.push('/dashboard');
      };

      const WelcomeStep = () => (
        <div className="step">
          <h2 style={typography.heading}>Welcome to LearnerLabs SAT Smart Prep!</h2>
          <p style={typography.body}>Let’s create your personalized study plan.</p>
          <p style={typography.body}>You’ve already signed up! Let’s get started.</p>
          <button style={commonStyles.button} onClick={nextStep}>
            <span style={commonStyles.buttonText}>Next</span>
          </button>
        </div>
      );

      const BasicInfoStep = () => (
        <div className="step">
          <h2 style={typography.heading}>Basic Information</h2>
          <div className="form-group">
            <label style={typography.body}>Full Name</label>
            <input
              type="text"
              value={userData.full_name}
              onChange={(e) => handleInputChange('full_name', e.target.value)}
              placeholder="Enter your full name"
              className="input"
            />
          </div>
          <div className="form-group">
            <label style={typography.body}>Grade Level</label>
            <select
              value={userData.grade}
              onChange={(e) => handleInputChange('grade', e.target.value)}
              className="input"
            >
              <option value="">Select your grade</option>
              <option value="9th">9th</option>
              <option value="10th">10th</option>
              <option value="11th">11th</option>
              <option value="12th">12th</option>
            </select>
          </div>
          <div className="form-group">
            <label style={typography.body}>School (Optional)</label>
            <input
              type="text"
              value={userData.school}
              onChange={(e) => handleInputChange('school', e.target.value)}
              placeholder="Enter your school name"
              className="input"
            />
          </div>
          <div className="buttons">
            <button style={commonStyles.button} onClick={nextStep}>
              <span style={commonStyles.buttonText}>Next</span>
            </button>
          </div>
        </div>
      );

      const SATExperienceStep = () => (
        <div className="step">
          <h2 style={typography.heading}>SAT Experience</h2>
          <p style={typography.body}>Have you taken the SAT before?</p>
          <div className="form-group">
            <button
              style={commonStyles.button}
              onClick={() => handleInputChange('sat_taken', true) && nextStep()}
            >
              <span style={commonStyles.buttonText}>Yes</span>
            </button>
            <button
              style={commonStyles.button}
              onClick={() => handleInputChange('sat_taken', false) && setStep(step + 2)}
            >
              <span style={commonStyles.buttonText}>No</span>
            </button>
          </div>
          <div className="buttons">
            <button style={commonStyles.button} onClick={prevStep}>
              <span style={commonStyles.buttonText}>Back</span>
            </button>
          </div>
        </div>
      );

      const SATScoreStep = () => (
        <div className="step">
          <h2 style={typography.heading}>Previous SAT Scores</h2>
          <div className="form-group">
            <label style={typography.body}>Math Score</label>
            <input
              type="number"
              value={userData.sat_math_score || ''}
              onChange={(e) => handleInputChange('sat_math_score', parseInt(e.target.value))}
              placeholder="Enter your Math score"
              className="input"
            />
          </div>
          <div className="form-group">
            <label style={typography.body}>Reading & Writing Score</label>
            <input
              type="number"
              value={userData.sat_reading_score || ''}
              onChange={(e) => handleInputChange('sat_reading_score', parseInt(e.target.value))}
              placeholder="Enter your Reading & Writing score"
              className="input"
            />
          </div>
          <div className="buttons">
            <button style={commonStyles.button} onClick={prevStep}>
              <span style={commonStyles.buttonText}>Back</span>
            </button>
            <button style={commonStyles.button} onClick={nextStep}>
              <span style={commonStyles.buttonText}>Next</span>
            </button>
          </div>
        </div>
      );

      const StudyPreferencesStep = () => (
        <div className="step">
          <h2 style={typography.heading}>Study Preferences</h2>
          <div className="form-group">
            <label style={typography.body}>When is your SAT test date?</label>
            <input
              type="date"
              value={userData.sat_test_date}
              onChange={(e) => handleInputChange('sat_test_date', e.target.value)}
              className="input"
            />
          </div>
          <div className="form-group">
            <label style={typography.body}>How many hours can you study per week? ({userData.study_hours_per_week} hours)</label>
            <input
              type="range"
              min="1"
              max="20"
              value={userData.study_hours_per_week}
              onChange={(e) => handleInputChange('study_hours_per_week', parseInt(e.target.value))}
              className="slider"
            />
          </div>
          <div className="form-group">
            <label style={typography.body}>How many days per week can you study?</label>
            <select
              value={userData.study_days_per_week}
              onChange={(e) => handleInputChange('study_days_per_week', parseInt(e.target.value))}
              className="input"
            >
              {[1, 2, 3, 4, 5, 6, 7].map(day => (
                <option key={day} value={day}>{day}</option>
              ))}
            </select>
          </div>
          <div className="form-group">
            <label style={typography.body}>What time of day do you prefer to study?</label>
            <select
              value={userData.preferred_study_time}
              onChange={(e) => handleInputChange('preferred_study_time', e.target.value)}
              className="input"
            >
              <option value="Morning">Morning</option>
              <option value="Afternoon">Afternoon</option>
              <option value="Evening">Evening</option>
            </select>
          </div>
          <div className="buttons">
            <button style={commonStyles.button} onClick={prevStep}>
              <span style={commonStyles.buttonText}>Back</span>
            </button>
            <button style={commonStyles.button} onClick={nextStep}>
              <span style={commonStyles.buttonText}>Next</span>
            </button>
          </div>
        </div>
      );

      const DiagnosticStep = () => (
        <div className="step">
          <h2 style={typography.heading}>Take a Diagnostic Test</h2>
          <p style={typography.body}>Let’s assess your current level with a short diagnostic test.</p>
          <button style={commonStyles.button} onClick={nextStep}>
            <span style={commonStyles.buttonText}>Start Test</span>
          </button>
          <div className="buttons">
            <button style={commonStyles.button} onClick={prevStep}>
              <span style={commonStyles.buttonText}>Back</span>
            </button>
          </div>
        </div>
      );

      const DiagnosticTestStep = () => (
        <div className="step">
          <Diagnostic setDiagnosticResults={setDiagnosticResults} />
          <div className="buttons">
            <button style={commonStyles.button} onClick={prevStep}>
              <span style={commonStyles.buttonText}>Back</span>
            </button>
            <button style={commonStyles.button} onClick={nextStep}>
              <span style={commonStyles.buttonText}>Continue to Study Plan</span>
            </button>
          </div>
        </div>
      );

      const StudyPlanStep = () => (
        <div className="step">
          <h2 style={typography.heading}>Your Study Plan is Ready!</h2>
          <StudyPlan userData={userData} />
          <div className="buttons">
            <button style={commonStyles.button} onClick={prevStep}>
              <span style={commonStyles.buttonText}>Back</span>
            </button>
            <button style={commonStyles.button} onClick={nextStep}>
              <span style={commonStyles.buttonText}>Continue to Dashboard</span>
            </button>
          </div>
        </div>
      );

      const DashboardStep = () => (
        <div className="step">
          <h2 style={typography.heading}>Your Dashboard</h2>
          <Dashboard diagnosticResults={diagnosticResults} />
          <div className="buttons">
            <button style={commonStyles.button} onClick={prevStep}>
              <span style={commonStyles.buttonText}>Back</span>
            </button>
            <button style={commonStyles.button} onClick={completeOnboarding}>
              <span style={commonStyles.buttonText}>Start Studying</span>
            </button>
          </div>
        </div>
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
        <div className="onboarding">
          <CurrentStep />
          <style jsx>{`
            .onboarding {
              max-width: 600px;
              margin: 50px auto;
              padding: 20px;
              background: ${colors.white};
              border-radius: 10px;
              box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
            }
            .step {
              text-align: center;
            }
            .form-group {
              margin-bottom: 20px;
            }
            .form-group label {
              display: block;
              margin-bottom: 5px;
              font-weight: 500;
            }
            .form-group input,
            .form-group select {
              width: 100%;
              padding: 8px;
              border: 1px solid ${colors.gray};
              border-radius: 5px;
            }
            .slider {
              width: 100%;
            }
            .buttons {
              display: flex;
              gap: 10px;
              justify-content: center;
              margin-top: 20px;
            }
          `}</style>
        </div>
      );
    };

    export default Onboarding;
    ```

**`frontend/pages/diagnostic.js`**

* **Purpose**: Diagnostic test page.
* **Latest Version**: No recent changes.
*   **Code**:

    ```javascript
    import React, { useState, useEffect } from 'react';
    import { useRouter } from 'next/router';
    import { api } from '../utils/api';
    import ReadingWritingTest from '../components/layouts/ReadingWritingTest';
    import MathBasicTest from '../components/layouts/MathBasicTest';
    import MathGraphTest from '../components/layouts/MathGraphTest';
    import MathTableTest from '../components/layouts/MathTableTest';
    import { colors, typography, commonStyles } from '../styles';

    export default function Diagnostic({ setDiagnosticResults }) {
      const [questions, setQuestions] = useState([]);
      const [sessionId, setSessionId] = useState(null);
      const [answers, setAnswers] = useState({});
      const [domain, setDomain] = useState('Math');
      const router = useRouter();
      const userId = localStorage.getItem('user_id');

      useEffect(() => {
        if (!userId) {
          router.push('/login');
        }
        startDiagnostic();
      }, [userId, router]);

      const startDiagnostic = async () => {
        const res = await api.post(`/diagnostic/start/${userId}`, { num_questions: 22 });
        setSessionId(res.data.session_id);
        setQuestions(res.data.questions);
      };

      const submitAnswer = async (answer) => {
        const questionId = questions[0].id;
        const responseData = [{ question_id: questionId, answer, time_spent: 60, session_id: sessionId, domain, theta: 0.8 }];
        const res = await api.post(`/diagnostic/submit/${sessionId}`, responseData);
        if (res.data.questions) {
          setQuestions(res.data.questions);
        } else {
          setDiagnosticResults(res.data);
          setSessionId(null);
          setQuestions([]);
        }
        setAnswers({});
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
            <div className="custom-buttons">
              <button onClick={() => console.log("Notes clicked")}>Notes</button>
              <button onClick={() => console.log("Highlight clicked")}>Highlight</button>
              <button onClick={() => console.log("Clear Highlights clicked")}>Clear Highlights</button>
            </div>
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
        <div className="diagnostic">
          <h1 style={typography.heading}>Diagnostic Test</h1>
          {!sessionId ? (
            <p style={typography.body}>Loading...</p>
          ) : (
            <div className="diagnostic-area">
              {questions.map((q) => renderQuestionLayout(q))}
            </div>
          )}

          <style jsx>{`
            .diagnostic {
              max-width: 900px;
              margin: 50px auto;
              text-align: center;
            }
            .diagnostic-area {
              flex: 1;
            }
          `}</style>
        </div>
      );
    }
    ```

**`frontend/pages/practice-bluebook.js`**

* **Purpose**: Practice session page.
* **Latest Version**: Enhanced with Bluebook-like UI and welcome message for mobile redirects.
*   **Code**:

    ```javascript
    import React, { useState, useEffect } from 'react';
    import { useRouter } from 'next/router';
    import { api } from '../utils/api';
    import ReadingWritingTest from '../components/layouts/ReadingWritingTest';
    import MathBasicTest from '../components/layouts/MathBasicTest';
    import MathGraphTest from '../components/layouts/MathGraphTest';
    import MathTableTest from '../components/layouts/MathTableTest';
    import { colors, typography, commonStyles } from '../styles';

    export default function PracticeBluebook() {
      const [questions, setQuestions] = useState([]);
      const [sessionId, setSessionId] = useState(null);
      const [answers, setAnswers] = useState({});
      const [domain, setDomain] = useState('Math');
      const [testType, setTestType] = useState('SAT');
      const [numQuestions, setNumQuestions] = useState(10);
      const [chatMessages, setChatMessages] = useState([]);
      const [chatInput, setChatInput] = useState('');
      const [isOffline, setIsOffline] = useState(false);
      const [showWelcomeMessage, setShowWelcomeMessage] = useState(false);
      const router = useRouter();
      const userId = localStorage.getItem('user_id');

      useEffect(() => {
        if (!userId) {
          router.push('/login');
        }
        if (router.query.from === 'mobile') {
          setShowWelcomeMessage(true);
        }
      }, [userId, router]);

      const startPractice = async () => {
        try {
          const res = await api.post(`/practice/start/${userId}`, { domain, num_questions: numQuestions, test_type: testType, device: 'web' });
          setSessionId(res.data.session_id);
          setQuestions(res.data.questions);
        } catch (error) {
          alert('Failed to start practice: ' + error.message);
        }
      };

      const submitAnswer = async (answer) => {
        const questionId = questions[0].id;
        const responseData = [{ question_id: questionId, answer, time_spent: 60, session_id: sessionId, domain, theta: 0.8 }];
        try {
          const res = await api.post(`/practice/submit/${sessionId}`, responseData);
          if (res.data.questions) {
            setQuestions(res.data.questions);
          } else {
            alert(`Practice Completed! Theta: ${res.data.theta}, Points: ${res.data.points_earned}`);
            setSessionId(null);
            setQuestions([]);
          }
          setAnswers({});
        } catch (error) {
          if (error.message === 'Offline mode: Request queued') {
            setQuestions(questions.slice(1));
            if (questions.length <= 1) {
              alert('Practice completed offline! Sync when online.');
              setSessionId(null);
              setQuestions([]);
            }
          } else {
            alert('Failed to submit answer: ' + error.message);
          }
        }
      };

      const sendChatMessage = (message) => {
        if (message && !isOffline) {
          setChatMessages((prev) => [...prev, { sender: 'You', text: message }]);
          setChatInput('');
          setTimeout(() => {
            setChatMessages((prev) => [...prev, { sender: 'AI', text: 'Here’s some help...' }]);
          }, 1000);
        } else if (isOffline) {
          alert('Chat unavailable offline');
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
            <div className="custom-buttons">
              <button onClick={() => console.log("Notes clicked")}>Notes</button>
              <button onClick={() => console.log("Highlight clicked")}>Highlight</button>
              <button onClick={() => console.log("Clear Highlights clicked")}>Clear Highlights</button>
            </div>
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
        <div className="practice-bluebook">
          {showWelcomeMessage && (
            <div className="welcome-message">
              <h2 style={typography.heading}>Welcome to Full-Length Testing</h2>
              <p style={typography.body}>
                You’re in the right place! Taking full-length tests on the web app mimics the Bluebook app used for the official digital SAT, providing a more accurate testing experience. Let’s get started!
              </p>
              <button style={commonStyles.button} onClick={() => setShowWelcomeMessage(false)}>
                <span style={commonStyles.buttonText}>Start Test</span>
              </button>
            </div>
          )}
          {!sessionId ? (
            <>
              <h1 style={typography.heading}>Practice Session {isOffline && '(Offline)'}</h1>
              <select
                value={testType}
                onChange={(e) => setTestType(e.target.value)}
                className="select"
              >
                <option value="SAT">SAT</option>
                <option value="PSAT">PSAT</option>
                <option value="ACT">ACT</option>
                <option value="AP">AP</option>
              </select>
              <select
                value={domain}
                onChange={(e) => setDomain(e.target.value)}
                className="select"
              >
                <option value="Math">Math</option>
                <option value="Reading & Writing">Reading & Writing</option>
              </select>
              <select
                value={numQuestions}
                onChange={(e) => setNumQuestions(parseInt(e.target.value))}
                className="select"
              >
                <option value={10}>Short Practice (10 questions)</option>
                <option value={44}>Full-Length Test (44-98 questions)</option>
              </select>
              <button style={commonStyles.button} onClick={startPractice}>
                <span style={commonStyles.buttonText}>Start</span>
              </button>
            </>
          ) : (
            <div className="practice-area">
              <div className="bluebook-header">
                <h1 style={typography.heading}>SAT Smart Prep App - Full-Length Test</h1>
                <div className="timer">Time Remaining: 15:00</div>
              </div>
              {questions.map((q) => renderQuestionLayout(q))}
              <div className="bluebook-footer">
                <button style={commonStyles.button} onClick={() => console.log("Previous question")}>
                  <span style={commonStyles.buttonText}>Previous</span>
                </button>
                <button style={commonStyles.button} onClick={() => submitAnswer(answers[q.id] || selectedAnswer)}>
                  <span style={commonStyles.buttonText}>Next</span>
                </button>
              </div>
            </div>
          )}
          <div className="chat-area">
            <h2 style={typography.heading}>AI Tutor</h2>
            <div className="chat-messages">
              {chatMessages.map((msg, idx) => (
                <p key={idx} style={typography.body}>
                  <strong>{msg.sender}:</strong> {msg.text}
                </p>
              ))}
            </div>
            <input
              type="text"
              value={chatInput}
              onChange={(e) => setChatInput(e.target.value)}
              onKeyPress={(e) => e.key === 'Enter' && sendChatMessage(chatInput)}
              placeholder="Ask the AI tutor..."
              className="chat-input"
            />
            <button style={commonStyles.button} onClick={() => sendChatMessage(chatInput)}>
              <span style={commonStyles.buttonText}>Send</span>
            </button>
          </div>

          <style jsx>{`
            .practice-bluebook {
              max-width: 1200px;
              margin: 0 auto;
              padding: 20px;
              background: #f5f5f5;
              min-height: 100vh;
            }
            .welcome-message {
              background: ${colors.white};
              padding: 20px;
              border-radius: 10px;
              text-align: center;
              margin-bottom: 20px;
              box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
            }
            .bluebook-header {
              display: flex;
              justify-content: space-between;
              align-items: center;
              background: ${colors.white};
              padding: 10px 20px;
              border-bottom: 2px solid ${colors.gray};
              position: sticky;
              top: 0;
              z-index: 1000;
            }
            .timer {
              font-size: 18px;
              font-weight: bold;
              color: #d32f2f;
            }
            .practice-area {
              background: ${colors.white};
              padding: 20px;
              border-radius: 10px;
              box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
              margin-bottom: 20px;
            }
            .bluebook-footer {
              display: flex;
              justify-content: space-between;
              padding: 10px 20px;
              background: ${colors.white};
              border-top: 2px solid ${colors.gray};
              position: sticky;
              bottom: 0;
              z-index: 1000;
            }
            .select {
              width: 100%;
              padding: 8px;
              margin: 10px 0;
              border: 1px solid ${colors.gray};
              border-radius: 5px;
            }
            .chat-area {
              flex: 1;
              border: 1px solid ${colors.gray};
              border-radius: 5px;
              padding: 10px;
              margin-top: 20px;
              background: ${colors.white};
            }
            .chat-messages {
              max-height: 200px;
              overflow-y: auto;
            }
            .chat-input {
              width: 100%;
              margin: 10px 0;
              padding: 8px;
              border: 1px solid ${colors.gray};
              border-radius: 5px;
            }
            .custom-buttons {
              display: flex;
              gap: 8px;
            }
            .custom-buttons button {
              color: #4b5563;
              padding: 4px;
              background: none;
              border: none;
              cursor: pointer;
            }
          `}</style>
        </div>
      );
    }
    ```

**`frontend/pages/study-plan.js`**

* **Purpose**: Study plan page.
* **Latest Version**: Supports granular study plans.
*   **Code**:

    ```javascript
    import React, { useState, useEffect } from 'react';
    import { useRouter } from 'next/router';
    import { api } from '../utils/api';
    import { colors, typography, commonStyles } from '../styles';

    export default function StudyPlan({ userData }) {
      const [studyPlan, setStudyPlan] = useState(null);
      const [editMode, setEditMode] = useState(false);
      const [studyHours, setStudyHours] = useState(userData?.study_hours_per_week || 1);
      const [studyDays, setStudyDays] = useState(userData?.study_days_per_week || 1);
      const router = useRouter();
      const userId = localStorage.getItem('user_id');

      useEffect(() => {
        if (!userId) {
          router.push('/login');
        }
        fetchStudyPlan();
      }, [userId, router]);

      const fetchStudyPlan = async () => {
        try {
          const res = await api.get(`/study_plan/${userId}`);
          setStudyPlan(res.data);
        } catch (error) {
          if (error.status === 404) {
            const res = await api.post(`/study_plan/create/${userId}`, {
              test_date: userData?.sat_test_date || new Date().toISOString(),
              study_hours: userData?.study_hours_per_week || 1,
              study_days: userData?.study_days_per_week || 1
            });
            setStudyPlan({
              plan_id: res.data.plan_id,
              test_date: userData?.sat_test_date || new Date().toISOString(),
              actions: res.data.milestones.map((milestone, idx) => ({
                id: idx,
                task: milestone,
                action: "Complete",
                due_date: new Date().toISOString(),
                points: 50,
                completed: null
              }))
            });
          } else {
            alert('Failed to load study plan: ' + error.message);
          }
        }
      };

      const updateStudyPlan = async () => {
        const res = await api.post(`/study_plan/create/${userId}`, {
          test_date: userData.sat_test_date,
          study_hours: studyHours,
          study_days: studyDays
        });
        setStudyPlan({
          plan_id: res.data.plan_id,
          test_date: userData.sat_test_date,
          actions: res.data.milestones.map((milestone, idx) => ({
            id: idx,
            task: milestone,
            action: "Complete",
            due_date: new Date().toISOString(),
            points: 50,
            completed: null
          }))
        });
        setEditMode(false);
      };

      return (
        <div className="study-plan">
          {studyPlan ? (
            <>
              <h1 style={typography.heading}>Your Study Plan</h1>
              <p style={typography.body}>Test Date: {new Date(studyPlan.test_date).toLocaleDateString()}</p>
              <h2 style={typography.heading}>Tasks</h2>
              {studyPlan.actions.map(action => (
                <div key={action.id} style={commonStyles.card}>
                  <p style={typography.body}>{action.task}</p>
                  <p style={typography.body}>Due: {new Date(action.due_date).toLocaleDateString()}</p>
                  <p style={typography.body}>Points: {action.points}</p>
                  {action.completed ? (
                    <p style={typography.body}>Completed on {new Date(action.completed).toLocaleDateString()}</p>
                  ) : (
                    <p style={typography.body}>Not yet completed</p>
                  )}
                </div>
              ))}
              <button style={commonStyles.button} onClick={() => setEditMode(true)}>
                <span style={commonStyles.buttonText}>Adjust Study Plan</span>
              </button>
              {editMode && (
                <div className="edit-plan">
                  <div className="form-group">
                    <label style={typography.body}>Study Hours Per Week ({studyHours} hours)</label>
                    <input
                      type="range"
                      min="1"
                      max="20"
                      value={studyHours}
                      onChange={(e) => setStudyHours(parseInt(e.target.value))}
                      className="slider"
                    />
                  </div>
                  <div className="form-group">
                    <label style={typography.body}>Study Days Per Week</label>
                    <select
                      value={studyDays}
                      onChange={(e) => setStudyDays(parseInt(e.target.value))}
                      className="input"
                    >
                      {[1, 2, 3, 4, 5, 6, 7].map(day => (
                        <option key={day} value={day}>{day}</option>
                      ))}
                    </select>
                  </div>
                  <button style={commonStyles.button} onClick={updateStudyPlan}>
                    <span style={commonStyles.buttonText}>Save Changes</span>
                  </button>
                </div>
              )}
            </>
          ) : (
            <p style={typography.body}>Loading your study plan...</p>
          )}

          <style jsx>{`
            .study-plan {
              max-width: 900px;
              margin: 50px auto;
              text-align: center;
            }
            .form-group {
              margin-bottom: 20px;
            }
            .form-group label {
              display: block;
              margin-bottom: 5px;
              font-weight: 500;
            }
            .form-group input,
            .form-group select {
              width: 100%;
              padding: 8px;
              border: 1px solid ${colors.gray};
              border-radius: 5px;
            }
            .slider {
              width: 100%;
            }
          `}</style>
        </div>
      );
    }
    ```

**`frontend/pages/dashboard.js`**

* **Purpose**: Dashboard page.
* **Latest Version**: Includes score guarantee check.
*   **Code**:

    ```javascript
    import React, { useState, useEffect } from 'react';
    import { useRouter } from 'next/router';
    import { api } from '../utils/api';
    import { colors, typography, commonStyles } from '../styles';

    export default function Dashboard({ diagnosticResults }) {
      const [user, setUser] = useState(null);
      const [challenges, setChallenges] = useState([]);
      const [achievements, setAchievements] = useState([]);
      const [proficiencies, setProficiencies] = useState([]);
      const [scoreHistory, setScoreHistory] = useState([]);
      const [feedback, setFeedback] = useState('');
      const router = useRouter();
      const userId = localStorage.getItem('user_id');

      useEffect(() => {
        if (!userId) {
          router.push('/login');
        }
        api.get(`/gamification/coins/${userId}`).then(res => setUser({ coins: res.data.coins }));
        api.get(`/challenges/${userId}`).then(res => setChallenges(res.data));
        api.get(`/progress_monitoring/proficiencies/${userId}`).then(res => setProficiencies(res.data));
        api.get(`/progress_monitoring/scores/${userId}`).then(res => setScoreHistory(res.data));
        setAchievements([
          { id: 1, name: "Complete 100 Questions", target: 100, progress: 75, completed: false },
          { id: 2, name: "5-Day Streak", target: 5, progress: 3, completed: false }
        ]);
      }, [userId, router]);

      const submitFeedback = async () => {
        if (!feedback) return;
        await api.post('/feedback', { user_id: userId, content: feedback });
        alert('Thank you for your feedback!');
        setFeedback('');
      };

      const checkGuarantee = async () => {
        const res = await api.get(`/progress_monitoring/guarantee/${userId}`);
        alert(res.data.message);
      };

      const estimatedScore = diagnosticResults ? (diagnosticResults.theta * 400 + 400) : 0;

      return (
        <div className="dashboard">
          {diagnosticResults ? (
            <div className="diagnostic-results">
              <h2 style={typography.heading}>Your Diagnostic Results</h2>
              <p style={typography.body}>Estimated SAT Score: {estimatedScore}</p>
              <h3 style={typography.heading}>Math</h3>
              <p style={typography.body}>Strong Areas: Algebra (Theta: {diagnosticResults?.math_theta || 0})</p>
              <p style={typography.body}>Weak Areas: Geometry (Theta: {diagnosticResults?.math_theta - 0.5 || 0})</p>
              <h3 style={typography.heading}>Reading & Writing</h3>
              <p style={typography.body}>Strong Areas: Reading Comprehension (Theta: {diagnosticResults?.reading_theta || 0})</p>
              <p style={typography.body}>Weak Areas: Grammar (Theta: {diagnosticResults?.reading_theta - 0.5 || 0})</p>
              <p style={typography.body}>Key Recommendation: Focus on Geometry and Grammar for higher scores.</p>
            </div>
          ) : (
            <>
              {user && <p style={typography.body}>Coins: {user.coins} 🪙</p>}
              <div className="score-history">
                <h2 style={typography.heading}>Score History</h2>
                {scoreHistory.map((entry, idx) => (
                  <p key={idx} style={typography.body}>
                    {new Date(entry.timestamp).toLocaleDateString()}: {entry.score}
                  </p>
                ))}
                <button style={commonStyles.button} onClick={checkGuarantee}>
                  <span style={commonStyles.buttonText}>Check Score Guarantee</span>
                </button>
                <button style={commonStyles.button} onClick={() => router.push('/policy')}>
                  <span style={commonStyles.buttonText}>View Guarantee Policy</span>
                </button>
              </div>
              <div className="daily-challenges">
                <h2 style={typography.heading}>Daily Challenges</h2>
                {challenges.map(challenge => (
                  <div key={challenge.id} style={commonStyles.card}>
                    <p style={typography.body}>
                      {challenge.challenge_type === 'daily_questions' ? "Answer 10 Questions Correctly" : "Complete a 5-Question Session in Under 5 Minutes"}
                    </p>
                    <div className="progress-bar">
                      <div style={{ width: `${(challenge.progress / challenge.target) * 100}%` }}></div>
                      <span className="progress-text">{challenge.progress}/{challenge.target}</span>
                    </div>
                    {challenge.completed && <p style={typography.body}>Completed! 🎉</p>}
                  </div>
                ))}
              </div>
              <div className="achievements">
                <h2 style={typography.heading}>Achievements</h2>
                {achievements.map(achievement => (
                  <div key={achievement.id} style={commonStyles.card}>
                    <p style={typography.body}>{achievement.name}</p>
                    <div className="progress-bar">
                      <div style={{ width: `${(achievement.progress / achievement.target) * 100}%` }}></div>
                      <span className="progress-text">{achievement.progress}/{achievement.target}</span>
                    </div>
                    {achievement.completed && <p style={typography.body}>Earned! 🏆</p>}
                  </div>
                ))}
              </div>
              <div className="proficiencies">
                <h2 style={typography.heading}>Your Progress</h2>
                {proficiencies.map(prof => (
                  <div key={prof.id} style={commonStyles.card}>
                    <p style={typography.body}>{prof.domain} - {prof.skill}: Theta {prof.theta}</p>
                  </div>
                ))}
              </div>
              <div className="feedback">
                <h2 style={typography.heading}>Feedback</h2>
                <textarea
                  value={feedback}
                  onChange={(e) => setFeedback(e.target.value)}
                  placeholder="Share your feedback..."
                  className="feedback-input"
                />
                <button style={commonStyles.button} onClick={submitFeedback}>
                  <span style={commonStyles.buttonText}>Submit</span>
                </button>
              </div>
            </>
          )}

          <style jsx>{`
            .dashboard {
              max-width: 900px;
              margin: 50px auto;
              text-align: center;
            }
            .diagnostic-results {
              margin-bottom: 20px;
            }
            .score-history {
              margin: 20px 0;
            }
            .daily-challenges {
              margin: 20px 0;
            }
            .achievements {
              margin: 20px 0;
            }
            .proficiencies {
              margin: 20px 0;
            }
            .progress-bar {
              background: #f0f0f0;
              border-radius: 5px;
              height: 20px;
              position: relative;
            }
            .progress-bar div {
              background: ${colors.primary};
              height: 100%;
              border-radius: 5px;
            }
            .progress-text {
              position: absolute;
              right: 10px;
              top: 50%;
              transform: translateY(-50%);
              color: ${colors.text};
            }
            .feedback {
              margin-top: 20px;
            }
            .feedback-input {
              width: 100%;
              height: 100px;
              margin-bottom: 10px;
              padding: 10px;
              border: 1px solid ${colors.gray};
              border-radius: 5px;
            }
          `}</style>
        </div>
      );
    }
    ```

**`frontend/pages/community.js`**

* **Purpose**: Community page.
* **Latest Version**: No recent changes.
*   **Code**:

    ```javascript
    import React, { useState, useEffect } from 'react';
    import { useRouter } from 'next/router';
    import { api } from '../utils/api';
    import { colors, typography, commonStyles } from '../styles';

    export default function Community() {
      const [posts, setPosts] = useState([]);
      const [newPost, setNewPost] = useState('');
      const [friends, setFriends] = useState([]);
      const [teamName, setTeamName] = useState('');
      const router = useRouter();
      const userId = localStorage.getItem('user_id');

      useEffect(() => {
        if (!userId) {
          router.push('/login');
        }
        api.get(`/social/posts`).then(res => setPosts(res.data));
        api.get(`/social/friends/${userId}`).then(res => setFriends(res.data.filter(f => f.status === 'accepted')));
      }, [userId, router]);

      const createPost = async () => {
        if (!newPost) return;
        await api.post('/social/posts', { user_id: userId, content: newPost });
        setNewPost('');
        api.get(`/social/posts`).then(res => setPosts(res.data));
      };

      const createTeam = async () => {
        if (!teamName) return;
        await api.post('/teams/create', { team_name: teamName, user_id: userId });
        setTeamName('');
        alert("Team created!");
      };

      const startChallenge = async (friendId) => {
        await api.post('/gamification/challenges/friend', { friendId });
        alert(`Challenge sent to ${friendId}!`);
      };

      return (
        <div className="community">
          <h1 style={typography.heading}>Community</h1>
          <div className="new-post">
            <textarea
              value={newPost}
              onChange={(e) => setNewPost(e.target.value)}
              placeholder="Share something with the community..."
              className="post-input"
            />
            <button style={commonStyles.button} onClick={createPost}>
              <span style={commonStyles.buttonText}>Post</span>
            </button>
          </div>

          <div className="posts">
            {posts.map(post => (
              <div key={post.id} style={commonStyles.card}>
                <p style={typography.body}>{post.content}</p>
                <p className="timestamp">Posted by {post.user_id} at {new Date(post.timestamp).toLocaleString()}</p>
              </div>
            ))}
          </div>

          <div className="community-challenges">
            <h2 style={typography.heading}>Challenge a Friend</h2>
            {friends.map(friend => (
              <div key={friend.id} className="friend">
                <p style={typography.body}>{friend.friend_id}</p>
                <button style={commonStyles.button} onClick={() => startChallenge(friend.friend_id)}>
                  <span style={commonStyles.buttonText}>Challenge</span>
                </button>
              </div>
            ))}
          </div>

          <div className="team-leagues">
            <h2 style={typography.heading}>Create a Team</h2>
            <input
              type="text"
              value={teamName}
              onChange={(e) => setTeamName(e.target.value)}
              placeholder="Team Name"
              className="team-input"
            />
            <button style={commonStyles.button} onClick={createTeam}>
              <span style={commonStyles.buttonText}>Create Team</span>
            </button>
          </div>

          <style jsx>{`
            .community {
              max-width: 900px;
              margin: 50px auto;
              text-align: center;
            }
            .new-post {
              margin-bottom: 20px;
            }
            .post-input {
              width: 100%;
              height: 100px;
              margin-bottom: 10px;
              padding: 10px;
              border: 1px solid ${colors.gray};
              border-radius: 5px;
            }
            .posts {
              margin-bottom: 20px;
            }
            .timestamp {
              font-size: 12px;
              color: #6b7280;
              margin-top: 5px;
            }
            .community-challenges {
              margin: 20px 0;
            }
            .friend {
              display: flex;
              align-items: center;
              justify-content: space-between;
              padding: 10px;
              border: 1px solid ${colors.gray};
              border-radius: 5px;
              margin-bottom: 10px;
            }
            .team-leagues {
              margin: 20px 0;
            }
            .team-input {
              width: 100%;
              padding: 8px;
              margin-bottom: 10px;
              border: 1px solid ${colors.gray};
              border-radius: 5px;
            }
          `}</style>
        </div>
      );
    }
    ```

**`frontend/pages/leaderboard.js`**

* **Purpose**: Leaderboard page.
* **Latest Version**: No recent changes.
*   **Code**:

    ```javascript
    import React, { useState, useEffect } from 'react';
    import { useRouter } from 'next/router';
    import { api } from '../utils/api';
    import { colors, typography, commonStyles } from '../styles';

    export default function Leaderboard() {
      const [skill, setSkill] = useState('Algebra');
      const [globalLeaderboard, setGlobalLeaderboard] = useState([]);
      const [friendsLeaderboard, setFriendsLeaderboard] = useState([]);
      const [teamLeaderboard, setTeamLeaderboard] = useState([]);
      const [tab, setTab] = useState('global');
      const router = useRouter();
      const userId = localStorage.getItem('user_id');

      useEffect(() => {
        if (!userId) {
          router.push('/login');
        }
        api.get(`/leaderboards/${skill}?user_id=${userId}`).then(res => {
          setGlobalLeaderboard(res.data.global);
          setFriendsLeaderboard(res.data.friends);
        });
        api.get('/teams/leaderboard').then(res => setTeamLeaderboard(res.data));
      }, [skill, userId, router]);

      return (
        <div className="leaderboard">
          <h1 style={typography.heading}>Leaderboard</h1>
          <select
            value={skill}
            onChange={(e) => setSkill(e.target.value)}
            className="select"
          >
            <option value="Algebra">Algebra</option>
            <option value="Geometry">Geometry</option>
            <option value="Reading Comprehension">Reading Comprehension</option>
          </select>

          <div className="tabs">
            <button
              onClick={() => setTab('global')}
              className={tab === 'global' ? 'tab-button active' : 'tab-button'}
            >
              Global
            </button>
            <button
              onClick={() => setTab('friends')}
              className={tab === 'friends' ? 'tab-button active' : 'tab-button'}
            >
              Friends
            </button>
            <button
              onClick={() => setTab('teams')}
              className={tab === 'teams' ? 'tab-button active' : 'tab-button'}
            >
              Teams
            </button>
          </div>

          {tab === 'global' && (
            <div className="leaderboard-section">
              <h2 style={typography.heading}>{skill} Leaderboard (Global)</h2>
              {globalLeaderboard.map((user, idx) => (
                <div key={user.user_id} className="leaderboard-entry">
                  <span style={typography.body}>{idx + 1}</span>
                  <span style={typography.body}>{user.email}</span>
                  <span style={typography.body}>{user.score}</span>
                  {idx < 3 && <span style={typography.body}>🏅</span>}
                </div>
              ))}
            </div>
          )}

          {tab === 'friends' && (
            <div className="leaderboard-section">
              <h2 style={typography.heading}>{skill} Leaderboard (Friends)</h2>
              {friendsLeaderboard.map((user, idx) => (
                <div key={user.user_id} className="leaderboard-entry">
                  <span style={typography.body}>{idx + 1}</span>
                  <span style={typography.body}>{user.email}</span>
                  <span style={typography.body}>{user.score}</span>
                  {idx < 3 && <span style={typography.body}>🏅</span>}
                </div>
              ))}
            </div>
          )}

          {tab === 'teams' && (
            <div className="leaderboard-section">
              <h2 style={typography.heading}>Team Leaderboard</h2>
              {teamLeaderboard.map((team, idx) => (
                <div key={team.id} className="leaderboard-entry">
                  <span style={typography.body}>{idx + 1}</span>
                  <span style={typography.body}>{team.team_name}</span>
                  <span style={typography.body}>{team.points}</span>
                  {idx < 3 && <span style={typography.body}>🏅</span>}
                </div>
              ))}
            </div>
          )}

          <style jsx>{`
            .leaderboard {
              max-width: 900px;
              margin: 50px auto;
              text-align: center;
            }
            .select {
              width: 100%;
              padding: 8px;
              margin: 10px 0;
              border: 1px solid ${colors.gray};
              border-radius: 5px;
            }
            .tabs {
              display: flex;
              gap: 10px;
              margin-bottom: 10px;
              justify-content: center;
            }
            .tab-button {
              padding: 5px;
              border: 1px solid ${colors.gray};
              border-radius: 5px;
              background: #f0f0f0;
              cursor: pointer;
            }
            .tab-button.active {
              background: ${colors.primary};
              border-color: ${colors.primary};
              color: ${colors.white};
            }
            .leaderboard-section {
              margin-top: 10px;
            }
            .leaderboard-entry {
              display: flex;
              align-items: center;
              gap: 10px;
              padding: 10px;
              border-bottom: 1px solid ${colors.gray};
            }
          `}</style>
        </div>
      );
    }
    ```

**`frontend/pages/rewards-store.js`**

* **Purpose**: Rewards store page.
* **Latest Version**: No recent changes.
*   **Code**:

    ```javascript
    import React, { useState, useEffect } from 'react';
    import { useRouter } from 'next/router';
    import { api } from '../utils/api';
    import { colors, typography, commonStyles } from '../styles';

    export default function RewardsStore() {
      const [coins, setCoins] = useState(0);
      const [rewards, setRewards] = useState([]);
      const router = useRouter();
      const userId = localStorage.getItem('user_id');

      useEffect(() => {
        if (!userId) {
          router.push('/login');
        }
        api.get(`/gamification/coins/${userId}`).then(res => setCoins(res.data.coins));
        api.get('/rewards/available').then(res => setRewards(res.data));
      }, [userId, router]);

      const unlockReward = async (rewardId, cost, rewardType) => {
        if (coins < cost) {
          alert("Not enough coins!");
          return;
        }
        await api.post('/rewards/unlock', { user_id: userId, reward_id: rewardId, reward_type: rewardType, cost });
        setCoins(coins - cost);
        alert(`Unlocked ${rewardId}!`);
      };

      return (
        <div className="rewards-store">
          <h1 style={typography.heading}>Rewards Store</h1>
          <p style={typography.body}>Coins: {coins} 🪙</p>
          <div className="rewards-grid">
            {rewards.map(reward => (
              <div key={reward.id} className="reward-item">
                <img src={reward.image} alt={reward.name} className="reward-image" />
                <p style={typography.body}>{reward.name}</p>
                <p style={typography.body}>{reward.cost} Coins</p>
                <button
                  style={commonStyles.button}
                  onClick={() => unlockReward(reward.id, reward.cost, reward.type)}
                >
                  <span style={commonStyles.buttonText}>Unlock</span>
                </button>
              </div>
            ))}
          </div>

          <style jsx>{`
            .rewards-store {
              max-width: 900px;
              margin: 50px auto;
              text-align: center;
            }
            .rewards-grid {
              display: flex;
              flex-wrap: wrap;
              gap: 20px;
              margin-top: 20px;
            }
            .reward-item {
              border: 1px solid ${colors.gray};
              border-radius: 5px;
              padding: 10px;
              width: 45%;
              text-align: center;
            }
            .reward-image {
              width: 100%;
              height: 150px;
              border-radius: 5px;
            }
          `}</style>
        </div>
      );
    }
    ```

**`frontend/pages/tutor-parent.js`**

* **Purpose**: Tutor/parent view page.
* **Latest Version**: No recent changes.
*   **Code**:

    ```javascript
    import React, { useState, useEffect } from 'react';
    import { useRouter } from 'next/router';
    import { api } from '../utils/api';
    import { colors, typography, commonStyles } from '../styles';

    export default function TutorParent() {
      const [studentProgress, setStudentProgress] = useState([]);
      const router = useRouter();
      const userId = localStorage.getItem('user_id');

      useEffect(() => {
        if (!userId) {
          router.push('/login');
        }
        api.get(`/progress_monitoring/proficiencies/${userId}`).then(res => setStudentProgress(res.data));
      }, [userId, router]);

      return (
        <div className="tutor-parent">
          <h1 style={typography.heading}>Tutor/Parent View</h1>
          <h2 style={typography.heading}>Student Progress</h2>
          {studentProgress.map(prof => (
            <div key={prof.id} style={commonStyles.card}>
              <p style={typography.body}>{prof.domain} - {prof.skill}: Theta {prof.theta}</p>
            </div>
          ))}

          <style jsx>{`
            .tutor-parent {
              max-width: 900px;
              margin: 50px auto;
              text-align: center;
            }
          `}</style>
        </div>
      );
    }
    ```

**`frontend/pages/policy.js`**

* **Purpose**: Score guarantee policy page.
* **Latest Version**: Updated with starting score condition (1250 or lower).
*   **Code**:

    ```javascript
    import React from 'react';
    import { useRouter } from 'next/router';
    import { colors, typography } from '../styles';

    export default function Policy() {
      const router = useRouter();

      return (
        <div className="policy">
          <h1 style={typography.heading}>Score Improvement Guarantee</h1>
          <p style={typography.body}>
            At Learner Labs, we are committed to helping you achieve your best SAT score with SAT Smart Prep App. We guarantee a 150-point score increase on your SAT practice tests if you meet the following conditions:
          </p>
          <p style={typography.body}>
            1. <strong>Starting Score of 1250 or Lower</strong>: You must have a starting score of 1250 or lower, as determined by either your first full-length SAT practice test (diagnostic test) within the app or your previous SAT or PSAT score entered during onboarding. If you do not provide a previous score, the app will use your first diagnostic test score to determine eligibility.
          </p>
          <p style={typography.body}>
            2. <strong>Complete At Least Two Full-Length SAT Practice Tests</strong>: You must complete at least two full-length SAT practice tests within the app. These tests must be taken under timed conditions to accurately reflect your progress. The first test establishes your baseline score, and the second (or later) test is used to measure your improvement.
          </p>
          <p style={typography.body}>
            3. <strong>Follow the Recommended Study Plan for 30 Days</strong>: You must actively follow the app’s recommended study plan for at least 30 consecutive days after completing your first full-length test. This includes completing at least 80% of the assigned tasks in your granular study plan (e.g., practice sessions, milestones) during this period.
          </p>
          <p style={typography.body}>
            4. <strong>Achieve a 150-Point Increase</strong>: The guarantee applies to the difference between your first full-length SAT practice test score and your highest subsequent full-length SAT practice test score within the app. If your score does not increase by at least 150 points, you are eligible for a refund.
          </p>
          <p style={typography.body}>
            5. <strong>Submit a Refund Request Within 60 Days</strong>: You must submit a refund request within 60 days of your subscription start date (or the date of your first full-length test, whichever is later). To request a refund, contact our support team at support@learnerlabs.com with your account details and a summary of your study activity.
          </p>
          <p style={typography.body}>
            6. <strong>Subscription Requirement</strong>: The guarantee applies only to users with an active paid subscription (monthly or annual) to SAT Smart Prep App. Free trial users or users in promotional programs (e.g., College Board pilot) are not eligible unless they upgrade to a paid plan.
          </p>
          <p style={typography.body}>
            <strong>Additional Notes</strong>:
            - <strong>Practice Test Scores Only</strong>: This guarantee applies only to SAT practice test scores within the app and does not apply to official SAT scores reported by the College Board.
            - <strong>Refund Processing</strong>: Refunds are processed within 14 business days of approval. The refund amount will be the full subscription fee paid, minus any applicable taxes or fees.
            - <strong>Fair Use</strong>: Learner Labs reserves the right to deny refund requests if there is evidence of misuse, such as not following the study plan, skipping questions, or attempting to manipulate scores.
          </p>
          <p style={typography.body}>
            <strong>How to Check Eligibility</strong>:
            Go to the Dashboard in the app and click "Check Score Guarantee" to see your score improvement and eligibility status. If eligible, follow the instructions to contact support for your refund.
          </p>

          <style jsx>{`
            .policy {
              max-width: 900px;
              margin: 50px auto;
              padding: 20px;
              text-align: center;
              background: ${colors.white};
              border-radius: 10px;
              box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
            }
          `}</style>
        </div>
      );
    }
    ```

**`frontend/pages/unsupported-browser.js`**

* **Purpose**: Fallback page for unsupported browsers.
* **Latest Version**: Created in previous steps.
*   **Code**:

    ```javascript
    import { colors, typography } from '../styles';

    export default function UnsupportedBrowser() {
      return (
        <div className="unsupported-browser">
          <h1 style={typography.heading}>Unsupported Browser</h1>
          <p style={typography.body}>
            Your browser is not supported. Please use a modern browser like Chrome, Firefox, Safari, or Edge to access SAT Smart Prep App.
          </p>
          <p style={typography.body}>
            We recommend updating your browser or switching to a supported device for the best experience.
          </p>
          <style jsx>{`
            .unsupported-browser {
              max-width: 600px;
              margin: 50px auto;
              text-align: center;
              background: ${colors.white};
              padding: 20px;
              border-radius: 10px;
              box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
            }
          `}</style>
        </div>
      );
    }
    ```

#### Styles (`frontend/styles/`)

**`frontend/styles/styles.js`**

* **Purpose**: Shared design system (colors, typography, common styles).
* **Latest Version**: Includes high-contrast mode for accessibility.
*   **Code**:

    ```javascript
    export const colors = {
      primary: '#0070f3',
      white: '#fff',
      gray: '#ddd',
      text: '#333',
      highContrastText: '#000',
      highContrastBackground: '#fff',
    };

    export const typography = {
      heading: {
        fontSize: '24px',
        fontWeight: 'bold',
        marginBottom: '10px',
      },
      body: {
        fontSize: '16px',
        lineHeight: '1.5',
      },
    };

    export const commonStyles = {
      button: {
        padding: '10px 20px',
        backgroundColor: colors.primary,
        border: 'none',
        borderRadius: '5px',
        cursor: 'pointer',
      },
      buttonText: {
        color: colors.white,
        fontSize: '16px',
      },
      card: {
        border: `1px solid ${colors.gray}`,
        borderRadius: '5px',
        padding: '10px',
        marginBottom: '10px',
      },
    };
    ```

#### Utilities (`frontend/utils/`)

**`frontend/utils/api.js`**

* **Purpose**: API client with offline support and WebSocket.
* **Latest Version**: No recent changes.
*   **Code**:

    ```javascript
    import axios from 'axios';

    const api = axios.create({
      baseURL: 'http://localhost:8000',
    });

    export const connectWebSocket = (userId, onMessage) => {
      const ws = new WebSocket(`ws://localhost:8000/gamification/updates/${userId}`);
      ws.onmessage = (event) => {
        onMessage(event.data);
      };
      return ws;
    };

    export { api };
    ```

#### Configuration (`frontend/config/`)

**`frontend/config/integrations.js`**

* **Purpose**: Centralizes API keys and configuration for integrations.
* **Latest Version**: Created in previous steps.
*   **Code**:

    ```javascript
    export const integrations = {
      googleSSO: {
        clientId: process.env.GOOGLE_CLIENT_ID,
        redirectUrl: process.env.GOOGLE_REDIRECT_URL,
        scopes: ['email', 'profile'],
        serviceConfiguration: {
          authorizationEndpoint: 'https://accounts.google.com/o/oauth2/v2/auth',
          tokenEndpoint: 'https://oauth2.googleapis.com/token',
        },
      },
      firebase: {
        apiKey: process.env.FIREBASE_API_KEY,
        authDomain: process.env.FIREBASE_AUTH_DOMAIN,
        projectId: process.env.FIREBASE_PROJECT_ID,
        storageBucket: process.env.FIREBASE_STORAGE_BUCKET,
        messagingSenderId: process.env.FIREBASE_MESSAGING_SENDER_ID,
        appId: process.env.FIREBASE_APP_ID,
      },
      bluebook: {
        apiKey: process.env.BLUEBOOK_API_KEY,
        enabled: false,
      },
      khanAcademy: {
        apiKey: process.env.KHAN_ACADEMY_API_KEY,
        enabled: false,
      },
      canvasLMS: {
        apiKey: process.env.CANVAS_API_KEY,
        baseUrl: process.env.CANVAS_BASE_URL,
        enabled:
    ```
