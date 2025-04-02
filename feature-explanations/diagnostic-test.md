# Diagnostic Test

Let’s modify the **Diagnostic Test** in the **SAT Smart Prep App** by **Learner Labs** to mimic the structure of the first modules of the digital SAT test, as administered through the College Board’s Bluebook app. The digital SAT consists of two sections: **Reading and Writing** and **Math**, each divided into two modules. For the Diagnostic Test, we’ll focus on mimicking the **first modules** of each section, adjust the number of questions to match the real SAT, and implement a 5-minute break between the two domains.

***

### Digital SAT Structure (First Modules)

The digital SAT’s first modules for each section are as follows:

* **Reading and Writing Section**:
  * **Module 1**: 27 questions, 32 minutes.
  * Questions cover skills like Reading Comprehension, Grammar, and Vocabulary.
* **Math Section**:
  * **Module 1**: 22 questions, 35 minutes.
  * Questions cover skills like Algebra, Geometry, and Data Analysis.
* **Break**: There is a 10-minute break between the Reading and Writing and Math sections in the real SAT. For the Diagnostic Test, we’ll implement a 5-minute break as requested.

#### Adjustments for the Diagnostic Test

* **Number of Questions**:
  * Reading and Writing: 27 questions (to match Module 1).
  * Math: 22 questions (to match Module 1).
  * Total: 49 questions (27 + 22).
* **Timing**:
  * Reading and Writing: 32 minutes.
  * Break: 5 minutes.
  * Math: 35 minutes.
  * Total: 72 minutes (32 + 5 + 35).
* **Flow**:
  * Start with the Reading and Writing domain (27 questions).
  * After completion, enforce a 5-minute break.
  * Proceed to the Math domain (22 questions).
* **Adaptivity**: Retain the IRT-based adaptivity from the previous update, ensuring questions are selected dynamically based on the user’s performance.

***

### Updated Diagnostic Test Implementation

#### Flow and Logic

**Step 1: Start the Diagnostic Test (Reading and Writing)**

* **Trigger**: The user initiates the Diagnostic Test.
* **Logic**:
  * Start with the Reading and Writing domain.
  * Select the first question (medium difficulty, `difficulty=3`, domain="Reading & Writing").
  * Set a timer for 32 minutes for the Reading and Writing section.
* **Frontend**:
  * Display the first question and start the timer.
* **Backend**:
  * Create a `Practice_Sessions` record with an initial `theta` of 0.
  * Track the current domain ("Reading & Writing") and the number of questions answered in that domain.

**Step 2: Answer Reading and Writing Questions (27 Questions)**

* **Logic**:
  * Use the adaptive IRT logic to select and answer 27 questions in the Reading and Writing domain.
  * After each answer, calculate the user’s `theta` and select the next question with a difficulty matching the updated `theta`.
  * Stop after 27 questions or if the timer (32 minutes) expires.
* **Frontend**:
  * Display each question, allow the user to answer, and submit the response.
  * Update the timer and question count.
* **Backend**:
  * Log each response in the `Responses` table.
  * Update `theta` in the `Practice_Sessions` table.
  * Create `Proficiencies` records for Reading and Writing skills.

**Step 3: 5-Minute Break**

* **Logic**:
  * After completing the 27 Reading and Writing questions, enforce a 5-minute break.
  * Display a countdown timer and a message (e.g., "Take a 5-minute break before the Math section").
  * Prevent the user from proceeding until the break is over.
* **Frontend**:
  * Show a break screen with a countdown timer.
  * Automatically proceed to the Math section after 5 minutes.

**Step 4: Start the Math Section**

* **Logic**:
  * Begin the Math domain with a new timer (35 minutes).
  * Select the first Math question (medium difficulty, `difficulty=3`, domain="Math").
* **Frontend**:
  * Display the first Math question and start the timer.
* **Backend**:
  * Continue using the same `Practice_Sessions` record, but track the new domain ("Math").

**Step 5: Answer Math Questions (22 Questions)**

* **Logic**:
  * Use the adaptive IRT logic to select and answer 22 questions in the Math domain.
  * Stop after 22 questions or if the timer (35 minutes) expires.
* **Frontend**:
  * Display each question, allow the user to answer, and submit the response.
* **Backend**:
  * Log responses and update `theta` and `Proficiencies` for Math skills.

**Step 6: Complete the Diagnostic Test**

* **Logic**:
  * After completing the Math section, calculate the final `theta` for the entire test.
  * Display the results to the user.
* **Frontend**:
  * Show the final `theta` score and estimated SAT score.
* **Backend**:
  * Update the `Practice_Sessions` record with the final `theta`.

***

### Updated Code Implementation

#### Backend (`frontend/backend/src/diagnostic.py`)

The backend needs to track the current domain, enforce the break, and ensure the correct number of questions per domain.

```python
import math
from fastapi import APIRouter, HTTPException
from pydantic import BaseModel
from psycopg2.extras import RealDictCursor
import psycopg2
import uuid
import time

router = APIRouter()

def get_db_connection():
    return psycopg2.connect(
        dbname="satprep",
        user="user",
        password="password",
        host="localhost",
        port="5432"
    )

class DiagnosticResponse(BaseModel):
    question_id: str
    answer: str
    time_spent: int
    session_id: str
    domain: str

@router.post("/diagnostic/start/{user_id}")
async def start_diagnostic(user_id: str, test_type: str = 'SAT'):
    conn = get_db_connection()
    cursor = conn.cursor(cursor_factory=RealDictCursor)
    try:
        session_id = str(uuid.uuid4())
        cursor.execute("""
            INSERT INTO practice_sessions (session_id, user_id, test_type, theta)
            VALUES (%s, %s, %s, %s);
        """, (session_id, user_id, test_type, 0.0))  # Initial theta = 0

        # Start with Reading & Writing domain
        cursor.execute("""
            SELECT * FROM questions
            WHERE test_type = %s AND domain = 'Reading & Writing' AND difficulty = 3
            ORDER BY RANDOM()
            LIMIT 1;
        """, (test_type,))
        first_question = cursor.fetchone()

        conn.commit()
        return {
            "session_id": session_id,
            "question": first_question,
            "current_domain": "Reading & Writing",
            "questions_remaining_in_domain": 27,
            "time_limit": 32 * 60  # 32 minutes in seconds
        }
    except Exception as e:
        conn.rollback()
        raise HTTPException(status_code=500, detail=str(e))
    finally:
        cursor.close()
        conn.close()

@router.post("/diagnostic/submit_answer/{session_id}")
async def submit_answer(session_id: str, response: DiagnosticResponse):
    conn = get_db_connection()
    cursor = conn.cursor(cursor_factory=RealDictCursor)
    try:
        # Fetch the question to get its difficulty and correct answer
        cursor.execute("SELECT difficulty, correct_answer FROM questions WHERE id = %s", (response.question_id,))
        question = cursor.fetchone()
        if not question:
            raise HTTPException(status_code=404, detail="Question not found")

        # Log the response
        response_id = str(uuid.uuid4())
        cursor.execute("""
            INSERT INTO responses (response_id, session_id, question_id, answer, time_spent, domain)
            VALUES (%s, %s, %s, %s, %s, %s);
        """, (response_id, session_id, response.question_id, response.answer, response.time_spent, response.domain))

        # Fetch all responses for this session to calculate theta
        cursor.execute("SELECT * FROM responses WHERE session_id = %s", (session_id,))
        responses = cursor.fetchall()

        # Fetch questions for difficulty and correct answers
        question_ids = [r['question_id'] for r in responses]
        cursor.execute("SELECT id, difficulty, correct_answer FROM questions WHERE id IN %s", (tuple(question_ids),))
        questions = {q['id']: q for q in cursor.fetchall()}

        # Simplified IRT: Calculate theta using 1PL model
        theta = 0.0  # Initial guess
        for _ in range(10):  # Iterate to converge (simplified MLE)
            gradient = 0.0
            hessian = 0.0
            for resp in responses:
                q = questions[resp['question_id']]
                b = q['difficulty']  # Difficulty parameter
                correct = 1 if resp['answer'] == q['correct_answer'] else 0
                p = 1 / (1 + math.exp(-(theta - b)))  # Probability of correct response
                gradient += (correct - p)
                hessian += -p * (1 - p)
            if hessian == 0:
                break
            theta += gradient / hessian

        # Update theta in the session
        cursor.execute("UPDATE practice_sessions SET theta = %s WHERE session_id = %s", (theta, session_id))

        # Create proficiency record
        cursor.execute("""
            INSERT INTO proficiencies (id, user_id, domain, skill, theta)
            VALUES (%s, %s, %s, %s, %s);
        """, (str(uuid.uuid4()), user_id, response.domain, response.domain, theta))

        # Determine the current domain and questions answered
        cursor.execute("SELECT domain, COUNT(*) as count FROM responses WHERE session_id = %s GROUP BY domain", (session_id,))
        domain_counts = {row['domain']: row['count'] for row in cursor.fetchall()}

        reading_writing_count = domain_counts.get("Reading & Writing", 0)
        math_count = domain_counts.get("Math", 0)

        # Check if Reading & Writing section is complete (27 questions)
        if reading_writing_count >= 27 and math_count == 0:
            # Enforce a 5-minute break
            return {
                "completed": False,
                "theta": theta,
                "break": True,
                "break_duration": 5 * 60,  # 5 minutes in seconds
                "next_domain": "Math",
                "questions_remaining_in_domain": 22,
                "time_limit": 35 * 60  # 35 minutes in seconds
            }

        # Check if Math section is complete (22 questions)
        if math_count >= 22:
            conn.commit()
            return {"completed": True, "theta": theta}

        # Select the next question based on theta and domain
        current_domain = "Math" if reading_writing_count >= 27 else "Reading & Writing"
        target_difficulty = max(1, min(5, int(round(theta + 3))))  # Map theta (-3 to 3) to difficulty (1 to 5)
        cursor.execute("""
            SELECT * FROM questions
            WHERE test_type = %s AND domain = %s AND difficulty = %s AND id NOT IN %s
            ORDER BY RANDOM()
            LIMIT 1;
        """, (test_type, current_domain, target_difficulty, tuple(r['question_id'] for r in responses)))
        next_question = cursor.fetchone()

        if not next_question:
            cursor.execute("""
                SELECT * FROM questions
                WHERE test_type = %s AND domain = %s AND id NOT IN %s
                ORDER BY RANDOM()
                LIMIT 1;
            """, (test_type, current_domain, tuple(r['question_id'] for r in responses)))
            next_question = cursor.fetchone()

        conn.commit()
        return {
            "completed": False,
            "theta": theta,
            "break": False,
            "next_question": next_question,
            "current_domain": current_domain,
            "questions_remaining_in_domain": (22 if current_domain == "Math" else 27) - (math_count if current_domain == "Math" else reading_writing_count),
            "time_limit": (35 * 60 if current_domain == "Math" else 32 * 60)
        }
    except Exception as e:
        conn.rollback()
        raise HTTPException(status_code=500, detail=str(e))
    finally:
        cursor.close()
        conn.close()
```

#### Frontend (`frontend/pages/diagnostic.js`)

The frontend needs to handle the new flow, including the break between sections and the updated timers.

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
    const [currentQuestion, setCurrentQuestion] = useState(null);
    const [sessionId, setSessionId] = useState(null);
    const [answers, setAnswers] = useState({});
    const [currentDomain, setCurrentDomain] = useState('Reading & Writing');
    const [questionsRemaining, setQuestionsRemaining] = useState(27);
    const [timeRemaining, setTimeRemaining] = useState(32 * 60); // 32 minutes in seconds
    const [completed, setCompleted] = useState(false);
    const [theta, setTheta] = useState(0);
    const [isBreak, setIsBreak] = useState(false);
    const [breakTimeRemaining, setBreakTimeRemaining] = useState(0);
    const router = useRouter();
    const userId = localStorage.getItem('user_id');

    useEffect(() => {
        if (!userId) {
            router.push('/login');
        }
        startDiagnostic();
    }, [userId, router]);

    useEffect(() => {
        if (timeRemaining > 0 && !completed && !isBreak) {
            const timer = setInterval(() => {
                setTimeRemaining(prev => prev - 1);
            }, 1000);
            return () => clearInterval(timer);
        } else if (timeRemaining <= 0 && !completed && !isBreak) {
            submitAnswer(answers[currentQuestion.id] || null); // Auto-submit if time runs out
        }
    }, [timeRemaining, completed, isBreak]);

    useEffect(() => {
        if (isBreak && breakTimeRemaining > 0) {
            const breakTimer = setInterval(() => {
                setBreakTimeRemaining(prev => prev - 1);
            }, 1000);
            return () => clearInterval(breakTimer);
        } else if (isBreak && breakTimeRemaining <= 0) {
            setIsBreak(false);
            submitAnswer(null); // Trigger the next section
        }
    }, [isBreak, breakTimeRemaining]);

    const startDiagnostic = async () => {
        const res = await api.post(`/diagnostic/start/${userId}`, { test_type: 'SAT' });
        setSessionId(res.data.session_id);
        setCurrentQuestion(res.data.question);
        setCurrentDomain(res.data.current_domain);
        setQuestionsRemaining(res.data.questions_remaining_in_domain);
        setTimeRemaining(res.data.time_limit);
    };

    const submitAnswer = async (answer) => {
        if (!currentQuestion && !isBreak) return;

        const responseData = {
            question_id: currentQuestion?.id,
            answer,
            time_spent: 60,
            session_id: sessionId,
            domain: currentQuestion?.domain
        };
        try {
            const res = await api.post(`/diagnostic/submit_answer/${sessionId}`, responseData);
            if (res.data.completed) {
                setCompleted(true);
                setTheta(res.data.theta);
                setDiagnosticResults({ theta: res.data.theta });
                setCurrentQuestion(null);
            } else if (res.data.break) {
                setIsBreak(true);
                setBreakTimeRemaining(res.data.break_duration);
                setCurrentDomain(res.data.next_domain);
                setQuestionsRemaining(res.data.questions_remaining_in_domain);
                setTimeRemaining(res.data.time_limit);
            } else {
                setTheta(res.data.theta);
                setCurrentQuestion(res.data.next_question);
                setCurrentDomain(res.data.current_domain);
                setQuestionsRemaining(res.data.questions_remaining_in_domain);
                setTimeRemaining(res.data.time_limit);
            }
            setAnswers({});
        } catch (error) {
            console.error('Failed to submit answer:', error);
        }
    };

    const renderQuestionLayout = (q) => {
        const props = {
            questionNumber: (currentDomain === 'Reading & Writing' ? 27 : 22) - questionsRemaining + 1,
            totalQuestions: currentDomain === 'Reading & Writing' ? 27 : 22,
            timer: `${Math.floor(timeRemaining / 60)}:${(timeRemaining % 60).toString().padStart(2, '0')}`,
            onNext: () => submitAnswer(answers[q.id] || selectedAnswer),
            onPrevious: () => console.log("Previous question"),
            showCalculator: currentDomain === 'Math',
            userName: userId || "Student",
            onTimeEnd: () => submitAnswer(answers[q.id] || null),
            customButtons: currentDomain === 'Reading & Writing' ? (
                <div className="custom-buttons">
                    <button onClick={() => console.log("Notes clicked")}>Notes</button>
                    <button onClick={() => console.log("Highlight clicked")}>Highlight</button>
                    <button onClick={() => console.log("Clear Highlights clicked")}>Clear Highlights</button>
                </div>
            ) : null
        };

        if (currentDomain === 'Reading & Writing') {
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
            ) : completed ? (
                <div>
                    <p style={typography.body}>Diagnostic Test Completed!</p>
                    <p style={typography.body}>Your Theta Score: {theta}</p>
                    <p style={typography.body}>Estimated SAT Score: {Math.round(theta * 400 + 400)}</p>
                </div>
            ) : isBreak ? (
                <div>
                    <p style={typography.body}>Take a 5-minute break before the {currentDomain} section.</p>
                    <p style={typography.body}>Time Remaining: {Math.floor(breakTimeRemaining / 60)}:{(breakTimeRemaining % 60).toString().padStart(2, '0')}</p>
                </div>
            ) : (
                <div className="diagnostic-area">
                    <p style={typography.body}>Section: {currentDomain}</p>
                    <p style={typography.body}>Questions Remaining: {questionsRemaining}</p>
                    {currentQuestion && renderQuestionLayout(currentQuestion)}
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

#### Mobile App (`SATSmartPrepApp/src/screens/DiagnosticScreen.js`)

The mobile app implementation is similar, adapted for React Native.

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
    const [currentQuestion, setCurrentQuestion] = useState(null);
    const [sessionId, setSessionId] = useState(null);
    const [answers, setAnswers] = useState({});
    const [currentDomain, setCurrentDomain] = useState('Reading & Writing');
    const [questionsRemaining, setQuestionsRemaining] = useState(27);
    const [timeRemaining, setTimeRemaining] = useState(32 * 60); // 32 minutes in seconds
    const [completed, setCompleted] = useState(false);
    const [theta, setTheta] = useState(0);
    const [isBreak, setIsBreak] = useState(false);
    const [breakTimeRemaining, setBreakTimeRemaining] = useState(0);

    useEffect(() => {
        const loadUserData = async () => {
            const storedUserId = await AsyncStorage.getItem('user_id');
            if (!storedUserId) {
                navigation.navigate('Login');
                return;
            }
            startDiagnostic();
        };
        loadUserData();
    }, []);

    useEffect(() => {
        if (timeRemaining > 0 && !completed && !isBreak) {
            const timer = setInterval(() => {
                setTimeRemaining(prev => prev - 1);
            }, 1000);
            return () => clearInterval(timer);
        } else if (timeRemaining <= 0 && !completed && !isBreak) {
            submitAnswer(answers[currentQuestion.id] || null);
        }
    }, [timeRemaining, completed, isBreak]);

    useEffect(() => {
        if (isBreak && breakTimeRemaining > 0) {
            const breakTimer = setInterval(() => {
                setBreakTimeRemaining(prev => prev - 1);
            }, 1000);
            return () => clearInterval(breakTimer);
        } else if (isBreak && breakTimeRemaining <= 0) {
            setIsBreak(false);
            submitAnswer(null);
        }
    }, [isBreak, breakTimeRemaining]);

    const startDiagnostic = async () => {
        try {
            const res = await api.post(`/diagnostic/start/${userId}`, { test_type: 'SAT' });
            setSessionId(res.data.session_id);
            setCurrentQuestion(res.data.question);
            setCurrentDomain(res.data.current_domain);
            setQuestionsRemaining(res.data.questions_remaining_in_domain);
            setTimeRemaining(res.data.time_limit);
        } catch (error) {
            Alert.alert('Error', 'Failed to start diagnostic: ' + error.message);
        }
    };

    const submitAnswer = async (answer) => {
        if (!currentQuestion && !isBreak) return;

        const responseData = {
            question_id: currentQuestion?.id,
            answer,
            time_spent: 60,
            session_id: sessionId,
            domain: currentQuestion?.domain
        };
        try {
            const res = await api.post(`/diagnostic/submit_answer/${sessionId}`, responseData);
            if (res.data.completed) {
                setCompleted(true);
                setTheta(res.data.theta);
                setDiagnosticResults({ theta: res.data.theta });
                setCurrentQuestion(null);
            } else if (res.data.break) {
                setIsBreak(true);
                setBreakTimeRemaining(res.data.break_duration);
                setCurrentDomain(res.data.next_domain);
                setQuestionsRemaining(res.data.questions_remaining_in_domain);
                setTimeRemaining(res.data.time_limit);
            } else {
                setTheta(res.data.theta);
                setCurrentQuestion(res.data.next_question);
                setCurrentDomain(res.data.current_domain);
                setQuestionsRemaining(res.data.questions_remaining_in_domain);
                setTimeRemaining(res.data.time_limit);
            }
            setAnswers({});
        } catch (error) {
            Alert.alert('Error', 'Failed to submit answer: ' + error.message);
        }
    };

    const renderQuestionLayout = (q) => {
        const props = {
            questionNumber: (currentDomain === 'Reading & Writing' ? 27 : 22) - questionsRemaining + 1,
            totalQuestions: currentDomain === 'Reading & Writing' ? 27 : 22,
            timer: `${Math.floor(timeRemaining / 60)}:${(timeRemaining % 60).toString().padStart(2, '0')}`,
            onNext: () => submitAnswer(answers[q.id] || selectedAnswer),
            onPrevious: () => console.log("Previous question"),
            showCalculator: currentDomain === 'Math',
            userName: userId || "Student",
            onTimeEnd: () => submitAnswer(answers[q.id] || null),
            customButtons: currentDomain === 'Reading & Writing' ? (
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

        if (currentDomain === 'Reading & Writing') {
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
            ) : completed ? (
                <View>
                    <Text style={typography.body}>Diagnostic Test Completed!</Text>
                    <Text style={typography.body}>Your Theta Score: {theta}</Text>
                    <Text style={typography.body}>Estimated SAT Score: {Math.round(theta * 400 + 400)}</Text>
                </View>
            ) : isBreak ? (
                <View>
                    <Text style={typography.body}>Take a 5-minute break before the {currentDomain} section.</Text>
                    <Text style={typography.body}>Time Remaining: {Math.floor(breakTimeRemaining / 60)}:{(breakTimeRemaining % 60).toString().padStart(2, '0')}</Text>
                </View>
            ) : (
                <View style={styles.diagnosticArea}>
                    <Text style={typography.body}>Section: {currentDomain}</Text>
                    <Text style={typography.body}>Questions Remaining: {questionsRemaining}</Text>
                    {currentQuestion && renderQuestionLayout(currentQuestion)}
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

***

### Summary of Changes

1. **Number of Questions**:
   * Increased to 49 questions (27 for Reading and Writing, 22 for Math) to match the first modules of the digital SAT.
2. **Domain Order and Break**:
   * The test starts with Reading and Writing (27 questions, 32 minutes).
   * After completion, a 5-minute break is enforced.
   * The test then proceeds to Math (22 questions, 35 minutes).
3. **Adaptive Testing**:
   * Retained the IRT-based adaptivity, ensuring questions are selected dynamically within each domain.
4. **Timers**:
   * Implemented separate timers for each section (32 minutes for Reading and Writing, 35 minutes for Math).
   * Added a 5-minute break timer between sections.

These changes ensure the Diagnostic Test mimics the structure of the first modules of the digital SAT, providing a realistic assessment experience for users. Let me know if you’d like to further refine the implementation or add additional features!
