# Full length tests on Web App

Let’s implement a feature in the **SAT Smart Prep App** by **Learner Labs** to encourage users to take full-length tests on the web app (via laptop or desktop) to mimic the Bluebook experience, while still allowing them to take the test on the mobile app if they insist. This will involve adding a nudge mechanism in the mobile app’s `PracticeScreen.js` when a user attempts to start a full-length test, along with a modal or dialog to guide them to the web app. We’ll also update the web app to ensure a seamless Bluebook-like experience for full-length tests. The implementation will include:

1. **Mobile App**: Add a nudge modal in `PracticeScreen.js` to encourage users to take full-length tests on the web app, with an option to proceed on mobile if they insist.
2. **Web App**: Enhance the full-length test experience in `practice-bluebook.js` to closely mimic Bluebook’s UI and functionality.
3. **Backend**: Ensure the backend supports tracking where tests are taken (web vs. mobile) for analytics purposes.

***

### Implementation Plan

#### 1. Mobile App: Add Nudge Modal for Full-Length Tests

We’ll modify the `PracticeScreen.js` in the mobile app to detect when a user attempts to start a full-length test (44-98 questions, as defined earlier) and display a modal encouraging them to use the web app on a laptop or desktop. The modal will include a “Continue on Mobile” option to allow users to proceed if they insist.

**Update `PracticeScreen.js`**

* **File**: `SATSmartPrepApp/src/screens/PracticeScreen.js`
* **Action**: Add a modal using React Native’s `Modal` component to display the nudge message. Track the user’s choice and proceed accordingly.
*   **Updated Code**:

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
      const [numQuestions, setNumQuestions] = useState(10); // Default to short practice
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
        // Check if the user is starting a full-length test (44-98 questions)
        if (numQuestions >= 44) {
          setShowNudgeModal(true); // Show nudge modal for full-length tests
        } else {
          proceedWithPractice();
        }
      };

      const proceedWithPractice = async () => {
        try {
          const res = await api.post(`/practice/start/${userId}`, { domain, num_questions: numQuestions, test_type: testType, device: 'mobile' });
          setSessionId(res.data.session_id);
          setQuestions(res.data.questions);
          setShowNudgeModal(false); // Close modal after proceeding
        } catch (error) {
          Alert.alert('Error', 'Failed to start practice: ' + error.message);
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
            Alert.alert('Practice Completed', `Theta: ${res.data.theta}, Points: ${res.data.points_earned}`);
            setSessionId(null);
            setQuestions([]);
          }
          setAnswers({});
        } catch (error) {
          if (error.message === 'Offline mode: Request queued') {
            setQuestions(questions.slice(1));
            if (questions.length <= 1) {
              Alert.alert('Practice completed offline! Sync when online.');
              setSessionId(null);
              setQuestions([]);
            }
          } else {
            Alert.alert('Error', 'Failed to submit answer: ' + error.message);
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
          Alert.alert('Chat unavailable offline');
        }
      };

      const startVoiceRecording = async () => {
        setIsVoiceRecording(true);
        await Voice.start('en-US');
      };

      const stopVoiceRecording = async () => {
        setIsVoiceRecording(false);
        await Voice.stop();
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
          <Text style={typography.heading}>Practice Session {isOffline && '(Offline)'}</Text>
          {!sessionId ? (
            <>
              <Picker
                selectedValue={testType}
                onValueChange={(value) => setTestType(value)}
                style={styles.picker}
              >
                <Picker.Item label="SAT" value="SAT" />
                <Picker.Item label="PSAT" value="PSAT" />
                <Picker.Item label="ACT" value="ACT" />
                <Picker.Item label="AP" value="AP" />
              </Picker>
              <Picker
                selectedValue={domain}
                onValueChange={(value) => setDomain(value)}
                style={styles.picker}
              >
                <Picker.Item label="Math" value="Math" />
                <Picker.Item label="Reading & Writing" value="Reading & Writing" />
              </Picker>
              <Picker
                selectedValue={numQuestions}
                onValueChange={(value) => setNumQuestions(value)}
                style={styles.picker}
              >
                <Picker.Item label="Short Practice (10 questions)" value={10} />
                <Picker.Item label="Full-Length Test (44-98 questions)" value={44} />
              </Picker>
              <TouchableOpacity style={commonStyles.button} onPress={startPractice}>
                <Text style={commonStyles.buttonText}>Start</Text>
              </TouchableOpacity>
            </>
          ) : (
            <View style={styles.practiceArea}>
              {questions.map((q) => renderQuestionLayout(q))}
            </View>
          )}
          <View style={styles.chatArea}>
            <Text style={typography.heading}>AI Tutor</Text>
            <View style={styles.chatMessages}>
              {chatMessages.map((msg, idx) => (
                <Text key={idx} style={typography.body}>
                  <Text style={{ fontWeight: 'bold' }}>{msg.sender}:</Text> {msg.text}
                </Text>
              ))}
            </View>
            <TouchableOpacity onPress={() => setUseVoice(!useVoice)}>
              <Text style={typography.body}>{useVoice ? 'Switch to Text Input' : 'Switch to Voice Input'}</Text>
            </TouchableOpacity>
            {useVoice ? (
              <TouchableOpacity
                style={commonStyles.button}
                onPress={isVoiceRecording ? stopVoiceRecording : startVoiceRecording}
                disabled={isVoiceRecording}
              >
                <Text style={commonStyles.buttonText}>
                  {isVoiceRecording ? 'Recording...' : 'Voice Input'}
                </Text>
              </TouchableOpacity>
            ) : (
              <VoiceInputFallback
                chatInput={chatInput}
                setChatInput={setChatInput}
                sendChatMessage={sendChatMessage}
                isOffline={isOffline}
              />
            )}
          </View>

          {/* Nudge Modal for Full-Length Tests */}
          <Modal
            animationType="slide"
            transparent={true}
            visible={showNudgeModal}
            onRequestClose={() => setShowNudgeModal(false)}
          >
            <View style={styles.modalOverlay}>
              <View style={styles.modalContent}>
                <Text style={typography.heading}>Take Full-Length Tests on the Web App</Text>
                <Text style={typography.body}>
                  For the best experience, we recommend taking full-length tests on the SAT Smart Prep App web version using a laptop or desktop. This mimics the Bluebook app used for the official digital SAT, providing a more accurate testing environment.
                </Text>
                <Text style={typography.body}>
                  Visit www.satsmartprepapp.com on your laptop or desktop to start your test.
                </Text>
                <View style={styles.modalButtons}>
                  <TouchableOpacity
                    style={[commonStyles.button, styles.webButton]}
                    onPress={() => setShowNudgeModal(false)} // Close modal, user will switch to web
                  >
                    <Text style={commonStyles.buttonText}>I’ll Use the Web App</Text>
                  </TouchableOpacity>
                  <TouchableOpacity
                    style={[commonStyles.button, styles.mobileButton]}
                    onPress={proceedWithPractice} // Proceed on mobile
                  >
                    <Text style={commonStyles.buttonText}>Continue on Mobile</Text>
                  </TouchableOpacity>
                </View>
              </View>
            </View>
          </Modal>
        </Animated.View>
      );
    };

    const styles = StyleSheet.create({
      container: {
        flex: 1,
        padding: 20,
        backgroundColor: colors.white,
      },
      picker: {
        width: '100%',
        marginVertical: 10,
        padding: 8,
      },
      practiceArea: {
        flex: 1,
      },
      chatArea: {
        flex: 1,
        borderWidth: 1,
        borderColor: colors.gray,
        borderRadius: 5,
        padding: 10,
      },
      chatMessages: {
        maxHeight: 200,
        overflow: 'scroll',
      },
      chatInput: {
        width: '100%',
        marginVertical: 10,
        padding: 8,
        borderWidth: 1,
        borderColor: colors.gray,
        borderRadius: 5,
      },
      customButtons: {
        flexDirection: 'row',
        gap: 8,
      },
      customButton: {
        color: '#4b5563',
        padding: 4,
      },
      modalOverlay: {
        flex: 1,
        justifyContent: 'center',
        alignItems: 'center',
        backgroundColor: 'rgba(0, 0, 0, 0.5)',
      },
      modalContent: {
        width: '80%',
        backgroundColor: colors.white,
        padding: 20,
        borderRadius: 10,
        alignItems: 'center',
      },
      modalButtons: {
        flexDirection: 'row',
        justifyContent: 'space-between',
        marginTop: 20,
        width: '100%',
      },
      webButton: {
        backgroundColor: colors.primary,
      },
      mobileButton: {
        backgroundColor: '#6b7280',
      },
    });

    export default PracticeScreen;
    ```

**Explanation**

* **Nudge Modal**: When the user selects a full-length test (`numQuestions >= 44`), the `showNudgeModal` state is set to `true`, displaying a modal with a message encouraging them to use the web app on a laptop or desktop.
* **User Choice**:
  * **I’ll Use the Web App**: Closes the modal, expecting the user to switch to the web app.
  * **Continue on Mobile**: Calls `proceedWithPractice()` to start the test on the mobile app.
* **Device Tracking**: The `device: 'mobile'` parameter is added to the `/practice/start` API call to track where the test is taken (for analytics).

***

#### 2. Web App: Enhance Full-Length Test Experience to Mimic Bluebook

We’ll update the `practice-bluebook.js` page in the web app to enhance the full-length test experience, ensuring it closely mimics the Bluebook app’s UI and functionality (e.g., timer, navigation, layout, as per College Board’s Bluebook app design). We’ll also add a welcome message to reinforce that the web app is the recommended platform for full-length tests.

**Update `practice-bluebook.js`**

* **File**: `frontend/pages/practice-bluebook.js`
* **Action**: Enhance the UI to mimic Bluebook (e.g., full-screen layout, prominent timer, navigation buttons) and add a welcome message for users coming from the mobile nudge.
*   **Updated Code**:

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
        // Check if user was redirected from mobile app (e.g., via query parameter)
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

**Explanation**

* **Welcome Message**: If the user is redirected from the mobile app (via a query parameter `from=mobile`), a welcome message is displayed to confirm they’re in the right place for full-length tests.
* **Bluebook-Like UI**:
  * **Header**: A sticky header with the app title and a prominent timer (mimicking Bluebook’s timer display).
  * **Layout**: Full-screen layout with a white background, rounded corners, and shadows to match Bluebook’s clean design.
  * **Footer**: Sticky navigation buttons (Previous/Next) for easy question navigation, similar to Bluebook’s interface.
* **Device Tracking**: The `device: 'web'` parameter is added to the `/practice/start` API call to track where the test is taken.

***

#### 3. Backend: Track Test Device for Analytics

We’ll update the backend to track whether tests are taken on the web or mobile app, allowing Learner Labs to analyze user behavior and the effectiveness of the nudge.

**Update `practice.py`**

* **File**: `frontend/backend/src/practice.py`
* **Action**: Add a `device` field to the `practice_sessions` table and include it in the `/practice/start` endpoint.
*   **Updated Code**:

    ```python
    from fastapi import APIRouter, HTTPException
    from pydantic import BaseModel
    from psycopg2.extras import RealDictCursor
    import psycopg2
    import uuid

    router = APIRouter()

    def get_db_connection():
        return psycopg2.connect(
            dbname="satprep",
            user="user",
            password="password",
            host="localhost",
            port="5432"
        )

    class PracticeStart(BaseModel):
        domain: str
        num_questions: int
        test_type: str
        device: str  # New field to track device (web or mobile)

    @router.post("/practice/start/{user_id}")
    async def start_practice(user_id: str, practice: PracticeStart):
        conn = get_db_connection()
        cursor = conn.cursor(cursor_factory=RealDictCursor)
        try:
            session_id = str(uuid.uuid4())
            cursor.execute("""
                INSERT INTO practice_sessions (session_id, user_id, test_type, section, device)
                VALUES (%s, %s, %s, %s, %s);
            """, (session_id, user_id, practice.test_type, practice.domain, practice.device))
            cursor.execute("""
                SELECT * FROM questions
                WHERE domain = %s AND test_type = %s
                ORDER BY RANDOM()
                LIMIT %s;
            """, (practice.domain, practice.test_type, practice.num_questions))
            questions = cursor.fetchall()
            conn.commit()
            return {"session_id": session_id, "questions": questions}
        except Exception as e:
            conn.rollback()
            raise HTTPException(status_code=500, detail=str(e))
        finally:
            cursor.close()
            conn.close()
    ```

**Update Database Schema**

* **File**: `frontend/backend/migrations/init_db.py`
* **Action**: Add a `device` column to the `practice_sessions` table.
*   **Updated Code** (Add to `init_db.py`):

    ```python
    cursor.execute("""
        ALTER TABLE practice_sessions
        ADD COLUMN IF NOT EXISTS device VARCHAR(50);
    """)
    ```

**Analytics Endpoint (Optional)**

To analyze where users take tests, add an endpoint to retrieve device usage statistics:

* **File**: `frontend/backend/src/practice.py`
*   **Code** (Add to the file):

    ```python
    @router.get("/practice/device-stats")
    async def get_device_stats():
        conn = get_db_connection()
        cursor = conn.cursor(cursor_factory=RealDictCursor)
        try:
            cursor.execute("""
                SELECT device, COUNT(*) as count
                FROM practice_sessions
                WHERE device IS NOT NULL
                GROUP BY device;
            """)
            stats = cursor.fetchall()
            return stats
        except Exception as e:
            raise HTTPException(status_code=500, detail=str(e))
        finally:
            cursor.close()
            conn.close()
    ```

***

### Testing and Verification

#### Test Case 1: User Attempts Full-Length Test on Mobile

* **User**: Maria (from a previous journey) tries to start a full-length SAT test on her mobile app.
* **Expected Behavior**:
  1. Maria selects “Full-Length Test (44-98 questions)” in the `PracticeScreen`.
  2. A modal appears with the message: “For the best experience, we recommend taking full-length tests on the SAT Smart Prep App web version using a laptop or desktop.”
  3. Maria chooses “I’ll Use the Web App” and closes the app to switch to her laptop.
* **Verification**:
  * The modal appears correctly when `numQuestions >= 44`.
  * The “I’ll Use the Web App” button closes the modal without starting the test.

#### Test Case 2: User Insists on Taking Full-Length Test on Mobile

* **User**: Maria insists on taking the test on her mobile app.
* **Expected Behavior**:
  1. Maria selects “Full-Length Test (44-98 questions)” in the `PracticeScreen`.
  2. The modal appears with the nudge message.
  3. Maria chooses “Continue on Mobile”.
  4. The test starts on her mobile app, and the backend records `device: 'mobile'`.
* **Verification**:
  * The “Continue on Mobile” button calls `proceedWithPractice()`, starting the test.
  * The `/practice/start` API call includes `device: 'mobile'`, verifiable in the `practice_sessions` table.

#### Test Case 3: User Takes Full-Length Test on Web App

* **User**: Priya (from a previous journey) takes a full-length test on the web app after being nudged from her mobile.
* **Expected Behavior**:
  1. Priya visits `www.satsmartprepapp.com` on her laptop with the query parameter `?from=mobile`.
  2. A welcome message appears: “You’re in the right place! Taking full-length tests on the web app mimics the Bluebook app used for the official digital SAT.”
  3. Priya starts a full-length test, experiencing a Bluebook-like UI with a sticky header (timer) and footer (navigation).
  4. The backend records `device: 'web'`.
* **Verification**:
  * The welcome message appears when `from=mobile` is in the URL.
  * The UI includes a sticky header with a timer and a footer with navigation buttons.
  * The `/practice/start` API call includes `device: 'web'`, verifiable in the `practice_sessions` table.

***

### Summary of Changes

* **Mobile App**:
  * Added a nudge modal in `PracticeScreen.js` to encourage users to take full-length tests on the web app, with an option to continue on mobile.
  * The modal provides a clear message and two buttons: “I’ll Use the Web App” (closes modal) and “Continue on Mobile” (proceeds with the test).
* **Web App**:
  * Enhanced `practice-bluebook.js` to mimic Bluebook’s UI with a sticky header (timer), full-screen layout, and sticky footer (navigation).
  * Added a welcome message for users redirected from the mobile app.
* **Backend**:
  * Updated `practice.py` to track the `device` (web or mobile) in the `practice_sessions` table.
  * Added a `device` column to the `practice_sessions` table in `init_db.py`.
  * Added an analytics endpoint to retrieve device usage statistics.

These changes ensure that users are encouraged to take full-length tests on the web app for a Bluebook-like experience, while still providing flexibility to use the mobile app if needed. The device tracking enables Learner Labs to analyze user behavior and optimize the nudge strategy. Let me know if you’d like to test these changes further or proceed with additional features!
