# Review of Diagnostic test

I apologize for the delay! I’m not stuck, but I was processing your previous requests and ensuring the responses were thorough and accurate. I’ve been working on the detailed documents, code updates, and plans you requested for the **SAT Smart Prep App** by **Learner Labs**, including the diagnostic test, review process, and SAT structure. Let me summarize where we are and address your latest request about the **Diagnostic Review Process**.

#### Where We Are

* **Previous Requests**:
  * I’ve provided detailed documents, code updates, and plans for various aspects of the app, including:
    * File structure and implementation for the app.
    * Onboarding process, diagnostic test, and review process.
    * Testing and deployment plans.
    * SAT structure, domains, and subskills.
  * The most recent request was to explain the **Diagnostic Review Process**, which I started but didn’t complete due to the complexity of the implementation and the need to ensure accuracy.

#### Current Request: Diagnostic Review Process

Let’s complete the **Diagnostic Review Process** document, which explains how users can review their performance after completing the Diagnostic Test in the SAT Smart Prep App. This process builds on the updated Diagnostic Test (mimicking the first modules of the digital SAT: 27 Reading and Writing questions, 22 Math questions, with a 5-minute break between sections). Since the app’s codebase does not currently include a dedicated review page, I’ll propose a detailed implementation, including the logic, flow, backend API, frontend rendering, and relevant files.

***

### Diagnostic Review Process

#### Purpose

The **Diagnostic Review Process** allows users to analyze their performance after completing the Diagnostic Test, which consists of 49 questions (27 Reading and Writing, 22 Math) across two sections with a 5-minute break in between. The review process provides detailed feedback on each question, helping users understand their mistakes, identify strengths and weaknesses, and receive actionable recommendations to improve their SAT skills.

#### Flow and Logic

**Step 1: Access the Review**

* **Trigger**: After completing the Diagnostic Test, the user navigates to the review section from the dashboard.
* **Frontend**:
  * The review is accessible via the dashboard’s diagnostic results section (`frontend/pages/dashboard.js` for web, `SATSmartPrepApp/src/screens/DashboardScreen.js` for mobile).
  * The user clicks a "Review Diagnostic Test" button to view detailed feedback.
  * Since the app does not currently have a dedicated review page, I’ll propose a new page/component: `frontend/pages/review.js` (web) and `SATSmartPrepApp/src/screens/ReviewScreen.js` (mobile).
* **Expected Outcome**:
  * The user is directed to a review page showing their Diagnostic Test performance.

**Step 2: Fetch Review Data**

* **Frontend**:
  * The frontend calls a new endpoint `/practice/review/{session_id}` to fetch detailed review data for the Diagnostic Test session.
  * The `session_id` is obtained from the `diagnosticResults` object passed from the Diagnostic Test (stored during onboarding).
* **Backend** (`frontend/backend/src/practice.py`):
  * **New Endpoint**: `GET /practice/review/{session_id}`
  * **Logic**:
    1.  Query the `Responses` table for all responses associated with the given `session_id`:

        ```sql
        SELECT * FROM responses WHERE session_id = %s;
        ```
    2.  For each response, join with the `Questions` table to fetch additional details (`question_text`, `options`, `correct_answer`, `explanation`, `domain`, `skill`):

        ```sql
        SELECT q.* FROM questions q
        JOIN responses r ON q.id = r.question_id
        WHERE r.session_id = %s;
        ```
    3. Construct a list of review items, each containing:
       * `question_text`: The question text.
       * `user_answer`: The user’s answer (from `Responses`).
       * `correct_answer`: The correct answer (from `Questions`).
       * `is_correct`: Boolean indicating if the user’s answer was correct.
       * `explanation`: Explanation of the correct answer (from `Questions`).
       * `time_spent`: Time spent on the question (from `Responses`).
       * `domain`: Domain of the question (e.g., "Reading & Writing").
       * `skill`: Subskill of the question (e.g., "Words in Context").
    4. Group the review items by domain (Reading and Writing, Math) for easier navigation.
    5. Calculate summary statistics:
       * Number of correct answers per domain.
       * Average `theta` per domain (from `Responses`).
       * Weakest subskills (subskills with the most incorrect answers).
    6.  Return the review data to the frontend:

        ```json
        {
          "review_items": [
            {
              "question_text": "What is the meaning of 'ephemeral' in the passage?",
              "user_answer": "A",
              "correct_answer": "B",
              "is_correct": false,
              "explanation": "'Ephemeral' means short-lived. Option B, 'temporary,' is the correct answer.",
              "time_spent": 60,
              "domain": "Reading & Writing",
              "skill": "Words in Context"
            },
            ...
          ],
          "summary": {
            "reading_writing": {
              "correct": 20,
              "total": 27,
              "theta": 0.7,
              "weakest_subskill": "Grammar"
            },
            "math": {
              "correct": 15,
              "total": 22,
              "theta": 0.5,
              "weakest_subskill": "Geometry"
            }
          }
        }
        ```
* **Expected Outcome**:
  * The frontend receives a structured dataset containing all review items and summary statistics.

**Step 3: Display Review Data**

* **Frontend**:
  * **New Page**: `frontend/pages/review.js` (web), `SATSmartPrepApp/src/screens/ReviewScreen.js` (mobile).
  * **Logic**:
    * Display the review data in a tabbed interface with two tabs: "Reading & Writing" and "Math".
    * For each tab, list the review items in a scrollable list:
      * Show the `question_text`, `domain`, and `skill`.
      * Display the user’s answer (`user_answer`) with a visual indicator (e.g., green for correct, red for incorrect).
      * Show the `correct_answer` and `explanation`.
      * Display `time_spent` to help users understand their pacing.
    * Provide filters to view:
      * All questions.
      * Incorrect answers only.
      * Specific subskills (e.g., "Words in Context", "Algebra").
    * At the top of each tab, show summary statistics:
      * Number of correct answers (e.g., "20/27 correct").
      * Average `theta` for the domain.
      * Weakest subskill with a recommendation (e.g., "Focus on Grammar to improve your Reading & Writing score").
* **Expected Outcome**:
  * Users can review each question, understand their mistakes, and see a summary of their performance by domain.

**Step 4: Provide Recommendations**

* **Frontend**:
  * Use the summary statistics to provide actionable recommendations:
    * Highlight the weakest subskill in each domain (e.g., "You missed 4 Grammar questions in Reading & Writing").
    * Suggest focus areas (e.g., "Practice more Geometry problems to improve your Math score").
    * Link to relevant practice sessions (e.g., "Start a Grammar practice session").
* **Backend**:
  * The `/practice/review/{session_id}` endpoint already includes the weakest subskill in the summary data, calculated by counting incorrect answers per subskill.
* **Expected Outcome**:
  * Users receive personalized recommendations to guide their SAT preparation.

***

### Implementation Details

#### Backend (`frontend/backend/src/practice.py`)

Add a new endpoint to fetch review data for the Diagnostic Test.

```python
from fastapi import APIRouter, HTTPException
from psycopg2.extras import RealDictCursor
import psycopg2

router = APIRouter()

def get_db_connection():
    return psycopg2.connect(
        dbname="satprep",
        user="user",
        password="password",
        host="localhost",
        port="5432"
    )

@router.get("/practice/review/{session_id}")
async def review_session(session_id: str):
    conn = get_db_connection()
    cursor = conn.cursor(cursor_factory=RealDictCursor)
    try:
        # Fetch all responses for the session
        cursor.execute("SELECT * FROM responses WHERE session_id = %s", (session_id,))
        responses = cursor.fetchall()

        if not responses:
            raise HTTPException(status_code=404, detail="Session not found")

        # Fetch question details for each response
        question_ids = [r['question_id'] for r in responses]
        cursor.execute("SELECT * FROM questions WHERE id IN %s", (tuple(question_ids),))
        questions = {q['id']: q for q in cursor.fetchall()}

        # Construct review items
        review_items = []
        for resp in responses:
            q = questions[resp['question_id']]
            review_items.append({
                "question_text": q['question_text'],
                "user_answer": resp['answer'],
                "correct_answer": q['correct_answer'],
                "is_correct": resp['answer'] == q['correct_answer'],
                "explanation": q['explanation'],
                "time_spent": resp['time_spent'],
                "domain": resp['domain'],
                "skill": q['skill']
            })

        # Calculate summary statistics
        summary = {
            "reading_writing": {"correct": 0, "total": 0, "theta": 0, "weakest_subskill": None},
            "math": {"correct": 0, "total": 0, "theta": 0, "weakest_subskill": None}
        }
        subskill_incorrect = {"Reading & Writing": {}, "Math": {}}

        for item in review_items:
            domain = item['domain']
            key = "reading_writing" if domain == "Reading & Writing" else "math"
            summary[key]["total"] += 1
            if item['is_correct']:
                summary[key]["correct"] += 1
            else:
                subskill = item['skill']
                subskill_incorrect[domain][subskill] = subskill_incorrect[domain].get(subskill, 0) + 1

        # Calculate average theta per domain
        for domain in ["Reading & Writing", "Math"]:
            domain_responses = [r for r in responses if r['domain'] == domain]
            if domain_responses:
                key = "reading_writing" if domain == "Reading & Writing" else "math"
                summary[key]["theta"] = sum(r['theta'] for r in domain_responses) / len(domain_responses)

        # Identify weakest subskill (most incorrect answers)
        for domain in ["Reading & Writing", "Math"]:
            key = "reading_writing" if domain == "Reading & Writing" else "math"
            if subskill_incorrect[domain]:
                weakest = max(subskill_incorrect[domain], key=subskill_incorrect[domain].get)
                summary[key]["weakest_subskill"] = weakest

        return {"review_items": review_items, "summary": summary}
    except Exception as e:
        conn.rollback()
        raise HTTPException(status_code=500, detail=str(e))
    finally:
        cursor.close()
        conn.close()
```

#### Frontend (`frontend/pages/review.js`)

Create a new page to display the review data for the Diagnostic Test.

```javascript
import React, { useState, useEffect } from 'react';
import { useRouter } from 'next/router';
import { api } from '../utils/api';
import { colors, typography, commonStyles } from '../styles';

export default function Review() {
    const [reviewData, setReviewData] = useState(null);
    const [activeTab, setActiveTab] = useState('Reading & Writing');
    const [filter, setFilter] = useState('all');
    const router = useRouter();
    const { sessionId } = router.query;

    useEffect(() => {
        if (sessionId) {
            fetchReviewData();
        }
    }, [sessionId]);

    const fetchReviewData = async () => {
        try {
            const res = await api.get(`/practice/review/${sessionId}`);
            setReviewData(res.data);
        } catch (error) {
            console.error('Failed to fetch review data:', error);
        }
    };

    const filteredItems = reviewData?.review_items.filter(item => {
        if (activeTab !== item.domain) return false;
        if (filter === 'incorrect') return !item.is_correct;
        if (filter !== 'all' && filter !== 'incorrect') return item.skill === filter;
        return true;
    });

    return (
        <div className="review">
            <h1 style={typography.heading}>Diagnostic Test Review</h1>
            {reviewData ? (
                <>
                    <div className="tabs">
                        <button
                            onClick={() => setActiveTab('Reading & Writing')}
                            className={activeTab === 'Reading & Writing' ? 'tab-button active' : 'tab-button'}
                        >
                            Reading & Writing
                        </button>
                        <button
                            onClick={() => setActiveTab('Math')}
                            className={activeTab === 'Math' ? 'tab-button active' : 'tab-button'}
                        >
                            Math
                        </button>
                    </div>

                    <div className="summary">
                        <h2 style={typography.heading}>
                            {activeTab === 'Reading & Writing' ? 'Reading & Writing Summary' : 'Math Summary'}
                        </h2>
                        <p style={typography.body}>
                            Correct: {reviewData.summary[activeTab === 'Reading & Writing' ? 'reading_writing' : 'math'].correct} / {reviewData.summary[activeTab === 'Reading & Writing' ? 'reading_writing' : 'math'].total}
                        </p>
                        <p style={typography.body}>
                            Average Theta: {reviewData.summary[activeTab === 'Reading & Writing' ? 'reading_writing' : 'math'].theta.toFixed(2)}
                        </p>
                        <p style={typography.body}>
                            Weakest Subskill: {reviewData.summary[activeTab === 'Reading & Writing' ? 'reading_writing' : 'math'].weakest_subskill || 'N/A'}
                        </p>
                        {reviewData.summary[activeTab === 'Reading & Writing' ? 'reading_writing' : 'math'].weakest_subskill && (
                            <p style={typography.body}>
                                Recommendation: Focus on {reviewData.summary[activeTab === 'Reading & Writing' ? 'reading_writing' : 'math'].weakest_subskill} to improve your {activeTab} score.
                            </p>
                        )}
                    </div>

                    <div className="filters">
                        <select onChange={(e) => setFilter(e.target.value)} className="select">
                            <option value="all">All Questions</option>
                            <option value="incorrect">Incorrect Answers</option>
                            {[...new Set(reviewData.review_items.filter(item => item.domain === activeTab).map(item => item.skill))].map(skill => (
                                <option key={skill} value={skill}>{skill}</option>
                            ))}
                        </select>
                    </div>

                    <div className="review-items">
                        {filteredItems?.map((item, idx) => (
                            <div key={idx} style={commonStyles.card}>
                                <p style={typography.body}><strong>Question:</strong> {item.question_text}</p>
                                <p style={[typography.body, { color: item.is_correct ? 'green' : 'red' }]}>
                                    <strong>Your Answer:</strong> {item.user_answer} {item.is_correct ? '(Correct)' : '(Incorrect)'}
                                </p>
                                <p style={typography.body}><strong>Correct Answer:</strong> {item.correct_answer}</p>
                                <p style={typography.body}><strong>Explanation:</strong> {item.explanation}</p>
                                <p style={typography.body}><strong>Time Spent:</strong> {item.time_spent} seconds</p>
                                <p style={typography.body}><strong>Domain:</strong> {item.domain}</p>
                                <p style={typography.body}><strong>Subskill:</strong> {item.skill}</p>
                            </div>
                        ))}
                    </div>
                </>
            ) : (
                <p style={typography.body}>Loading review data...</p>
            )}

            <style jsx>{`
                .review {
                    max-width: 900px;
                    margin: 50px auto;
                    text-align: center;
                }
                .tabs {
                    display: flex;
                    gap: 10px;
                    margin-bottom: 20px;
                    justify-content: center;
                }
                .tab-button {
                    padding: 5px 10px;
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
                .summary {
                    margin-bottom: 20px;
                }
                .filters {
                    margin-bottom: 20px;
                }
                .select {
                    padding: 5px;
                    border: 1px solid ${colors.gray};
                    border-radius: 5px;
                }
                .review-items {
                    text-align: left;
                }
            `}</style>
        </div>
    );
}
```

#### Mobile App (`SATSmartPrepApp/src/screens/ReviewScreen.js`)

The mobile app implementation is similar, adapted for React Native.

```javascript
import React, { useState, useEffect } from 'react';
import { View, Text, ScrollView, TouchableOpacity, Picker, StyleSheet } from 'react-native';
import Animated, { FadeIn, FadeOut } from 'react-native-reanimated';
import { api } from '../utils/api';
import { colors, typography, commonStyles } from '../styles';

const ReviewScreen = ({ route }) => {
    const { sessionId } = route.params;
    const [reviewData, setReviewData] = useState(null);
    const [activeTab, setActiveTab] = useState('Reading & Writing');
    const [filter, setFilter] = useState('all');

    useEffect(() => {
        if (sessionId) {
            fetchReviewData();
        }
    }, [sessionId]);

    const fetchReviewData = async () => {
        try {
            const res = await api.get(`/practice/review/${sessionId}`);
            setReviewData(res.data);
        } catch (error) {
            console.error('Failed to fetch review data:', error);
        }
    };

    const filteredItems = reviewData?.review_items.filter(item => {
        if (activeTab !== item.domain) return false;
        if (filter === 'incorrect') return !item.is_correct;
        if (filter !== 'all' && filter !== 'incorrect') return item.skill === filter;
        return true;
    });

    return (
        <Animated.View entering={FadeIn} exiting={FadeOut} style={styles.container}>
            <Text style={typography.heading}>Diagnostic Test Review</Text>
            {reviewData ? (
                <ScrollView>
                    <View style={styles.tabs}>
                        <TouchableOpacity
                            onPress={() => setActiveTab('Reading & Writing')}
                            style={[styles.tabButton, activeTab === 'Reading & Writing' && styles.tabButtonActive]}
                        >
                            <Text style={activeTab === 'Reading & Writing' ? styles.tabTextActive : styles.tabText}>
                                Reading & Writing
                            </Text>
                        </TouchableOpacity>
                        <TouchableOpacity
                            onPress={() => setActiveTab('Math')}
                            style={[styles.tabButton, activeTab === 'Math' && styles.tabButtonActive]}
                        >
                            <Text style={activeTab === 'Math' ? styles.tabTextActive : styles.tabText}>
                                Math
                            </Text>
                        </TouchableOpacity>
                    </View>

                    <View style={styles.summary}>
                        <Text style={typography.heading}>
                            {activeTab === 'Reading & Writing' ? 'Reading & Writing Summary' : 'Math Summary'}
                        </Text>
                        <Text style={typography.body}>
                            Correct: {reviewData.summary[activeTab === 'Reading & Writing' ? 'reading_writing' : 'math'].correct} / {reviewData.summary[activeTab === 'Reading & Writing' ? 'reading_writing' : 'math'].total}
                        </Text>
                        <Text style={typography.body}>
                            Average Theta: {reviewData.summary[activeTab === 'Reading & Writing' ? 'reading_writing' : 'math'].theta.toFixed(2)}
                        </Text>
                        <Text style={typography.body}>
                            Weakest Subskill: {reviewData.summary[activeTab === 'Reading & Writing' ? 'reading_writing' : 'math'].weakest_subskill || 'N/A'}
                        </Text>
                        {reviewData.summary[activeTab === 'Reading & Writing' ? 'reading_writing' : 'math'].weakest_subskill && (
                            <Text style={typography.body}>
                                Recommendation: Focus on {reviewData.summary[activeTab === 'Reading & Writing' ? 'reading_writing' : 'math'].weakest_subskill} to improve your {activeTab} score.
                            </Text>
                        )}
                    </View>

                    <View style={styles.filters}>
                        <Picker
                            selectedValue={filter}
                            onValueChange={(value) => setFilter(value)}
                            style={styles.select}
                        >
                            <Picker.Item label="All Questions" value="all" />
                            <Picker.Item label="Incorrect Answers" value="incorrect" />
                            {[...new Set(reviewData.review_items.filter(item => item.domain === activeTab).map(item => item.skill))].map(skill => (
                                <Picker.Item key={skill} label={skill} value={skill} />
                            ))}
                        </Picker>
                    </View>

                    <View style={styles.reviewItems}>
                        {filteredItems?.map((item, idx) => (
                            <View key={idx} style={commonStyles.card}>
                                <Text style={typography.body}><Text style={{ fontWeight: 'bold' }}>Question:</Text> {item.question_text}</Text>
                                <Text style={[typography.body, { color: item.is_correct ? 'green' : 'red' }]}>
                                    <Text style={{ fontWeight: 'bold' }}>Your Answer:</Text> {item.user_answer} {item.is_correct ? '(Correct)' : '(Incorrect)'}
                                </Text>
                                <Text style={typography.body}><Text style={{ fontWeight: 'bold' }}>Correct Answer:</Text> {item.correct_answer}</Text>
                                <Text style={typography.body}><Text style={{ fontWeight: 'bold' }}>Explanation:</Text> {item.explanation}</Text>
                                <Text style={typography.body}><Text style={{ fontWeight: 'bold' }}>Time Spent:</Text> {item.time_spent} seconds</Text>
                                <Text style={typography.body}><Text style={{ fontWeight: 'bold' }}>Domain:</Text> {item.domain}</Text>
                                <Text style={typography.body}><Text style={{ fontWeight: 'bold' }}>Subskill:</Text> {item.skill}</Text>
                            </View>
                        ))}
                    </View>
                </ScrollView>
            ) : (
                <Text style={typography.body}>Loading review data...</Text>
            )}
        </Animated.View>
    );
};

const styles = StyleSheet.create({
    container: {
        flex: 1,
        padding: 20,
        backgroundColor: colors.white,
    },
    tabs: {
        flexDirection: 'row',
        gap: 10,
        marginBottom: 20,
        justifyContent: 'center',
    },
    tabButton: {
        padding: 5,
        borderWidth: 1,
        borderColor: colors.gray,
        borderRadius: 5,
        backgroundColor: '#f0f0f0',
    },
    tabButtonActive: {
        backgroundColor: colors.primary,
        borderColor: colors.primary,
    },
    tabText: {
        color: colors.text,
    },
    tabTextActive: {
        color: colors.white,
    },
    summary: {
        marginBottom: 20,
    },
    filters: {
        marginBottom: 20,
    },
    select: {
        width: '100%',
        padding: 5,
        borderWidth: 1,
        borderColor: colors.gray,
        borderRadius: 5,
    },
    reviewItems: {
        marginBottom: 20,
    },
});

export default ReviewScreen;
```

#### Update Dashboard to Link to Review (`frontend/pages/dashboard.js`)

Modify the dashboard to include a link to the review page.

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
                    <button style={commonStyles.button} onClick={() => router.push(`/review?sessionId=${diagnosticResults.session_id}`)}>
                        <span style={commonStyles.buttonText}>Review Diagnostic Test</span>
                    </button>
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

#### Update Dashboard to Link to Review (`SATSmartPrepApp/src/screens/DashboardScreen.js`)

The mobile app dashboard also needs to link to the review screen.

```javascript
import React, { useState, useEffect } from 'react';
import { View, Text, TextInput, TouchableOpacity, StyleSheet, Alert, ScrollView } from 'react-native';
import Animated, { FadeIn, FadeOut } from 'react-native-reanimated';
import { api } from '../utils/api';
import { colors, typography, commonStyles } from '../styles';

export default function DashboardScreen({ diagnosticResults, navigation }) {
    const [user, setUser] = useState(null);
    const [challenges, setChallenges] = useState([]);
    const [achievements, setAchievements] = useState([]);
    const [proficiencies, setProficiencies] = useState([]);
    const [scoreHistory, setScoreHistory] = useState([]);
    const [feedback, setFeedback] = useState('');
    const userId = localStorage.getItem('user_id');

    useEffect(() => {
        api.get(`/gamification/coins/${userId}`).then(res => setUser({ coins: res.data.coins }));
        api.get(`/challenges/${userId}`).then(res => setChallenges(res.data));
        api.get(`/progress_monitoring/proficiencies/${userId}`).then(res => setProficiencies(res.data));
        api.get(`/progress_monitoring/scores/${userId}`).then(res => setScoreHistory(res.data));
        setAchievements([
            { id: 1, name: "Complete 100 Questions", target: 100, progress: 75, completed: false },
            { id: 2, name: "5-Day Streak", target: 5, progress: 3, completed: false }
        ]);
    }, []);

    const submitFeedback = async () => {
        if (!feedback) return;
        await api.post('/feedback', { user_id: userId, content: feedback });
        Alert.alert('Thank you for your feedback!');
        setFeedback('');
    };

    const checkGuarantee = async () => {
        const res = await api.get(`/progress_monitoring/guarantee/${userId}`);
        Alert.alert('Score Guarantee', res.data.message);
    };

    const estimatedScore = diagnosticResults ? (diagnosticResults.theta * 400 + 400) : 0;

    return (
        <Animated.View entering={FadeIn} exiting={FadeOut} style={styles.container}>
            <ScrollView>
                {diagnosticResults ? (
                    <View style={styles.diagnosticResults}>
                        <Text style={typography.heading}>Your Diagnostic Results</Text>
                        <Text style={typography.body}>Estimated SAT Score: {estimatedScore}</Text>
                        <Text style={typography.heading}>Math</Text>
                        <Text style={typography.body}>Strong Areas: Algebra (Theta: {diagnosticResults?.math_theta || 0})</Text>
                        <Text style={typography.body}>Weak Areas: Geometry (Theta: {diagnosticResults?.math_theta - 0.5 || 0})</Text>
                        <Text style={typography.heading}>Reading & Writing</Text>
                        <Text style={typography.body}>Strong Areas: Reading Comprehension (Theta: {diagnosticResults?.reading_theta || 0})</Text>
                        <Text style={typography.body}>Weak Areas: Grammar (Theta: {diagnosticResults?.reading_theta - 0.5 || 0})</Text>
                        <Text style={typography.body}>Key Recommendation: Focus on Geometry and Grammar for higher scores.</Text>
                        <TouchableOpacity
                            style={commonStyles.button}
                            onPress={() => navigation.navigate('Review', { sessionId: diagnosticResults.session_id })}
                        >
                            <Text style={commonStyles.buttonText}>Review Diagnostic Test</Text>
                        </TouchableOpacity>
                    </View>
                ) : (
                    <>
                        {user && <Text style={typography.body}>Coins: {user.coins} 🪙</Text>}
                        <View style={styles.scoreHistory}>
                            <Text style={typography.heading}>Score History</Text>
                            {scoreHistory.map((entry, idx) => (
                                <Text key={idx} style={typography.body}>
                                    {new Date(entry.timestamp).toLocaleDateString()}: {entry.score}
                                </Text>
                            ))}
                            <TouchableOpacity style={commonStyles.button} onPress={checkGuarantee}>
                                <Text style={commonStyles.buttonText}>Check Score Guarantee</Text>
                            </TouchableOpacity>
                            <TouchableOpacity style={commonStyles.button} onPress={() => navigation.navigate('Policy')}>
                                <Text style={commonStyles.buttonText}>View Guarantee Policy</Text>
                            </TouchableOpacity>
                        </View>
                        <View style={styles.dailyChallenges}>
                            <Text style={typography.heading}>Daily Challenges</Text>
                            {challenges.map(challenge => (
                                <View key={challenge.id} style={commonStyles.card}>
                                    <Text style={typography.body}>
                                        {challenge.challenge_type === 'daily_questions' ? "Answer 10 Questions Correctly" : "Complete a 5-Question Session in Under 5 Minutes"}
                                    </Text>
                                    <View style={styles.progressBar}>
                                        <View style={[styles.progressFill, { width: `${(challenge.progress / challenge.target) * 100}%` }]} />
                                        <Text style={styles.progressText}>{challenge.progress}/{challenge.target}</Text>
                                    </View>
                                    {challenge.completed && <Text style={typography.body}>Completed! 🎉</Text>}
                                </View>
                            ))}
                        </View>
                        <View style={styles.achievements}>
                            <Text style={typography.heading}>Achievements</Text>
                            {achievements.map(achievement => (
                                <View key={achievement.id} style={commonStyles.card}>
                                    <Text style={typography.body}>{achievement.name}</Text>
                                    <View style={styles.progressBar}>
                                        <View style={[styles.progressFill, { width: `${(achievement.progress / achievement.target) * 100}%` }]} />
                                        <Text style={styles.progressText}>{achievement.progress}/{achievement.target}</Text>
                                    </View>
                                    {achievement.completed && <Text style={typography.body}>Earned! 🏆</Text>}
                                </View>
                            ))}
                        </View>
                        <View style={styles.proficiencies}>
                            <Text style={typography.heading}>Your Progress</Text>
                            {proficiencies.map(prof => (
                                <View key={prof.id} style={commonStyles.card}>
                                    <Text style={typography.body}>{prof.domain} - {prof.skill}: Theta {prof.theta}</Text>
                                </View>
                            ))}
                        </View>
                        <View style={styles.feedback}>
                            <Text style={typography.heading}>Feedback</Text>
                            <TextInput
                                style={styles.feedbackInput}
                                value={feedback}
                                onChangeText={setFeedback}
                                placeholder="Share your feedback..."
                                multiline
                            />
                            <TouchableOpacity style={commonStyles.button} onPress={submitFeedback}>
                                <Text style={commonStyles.buttonText}>Submit</Text>
                            </TouchableOpacity>
                        </View>
                    </>
                )}
            </ScrollView>
        </Animated.View>
    );
};

const styles = StyleSheet.create({
    container: {
        flex: 1,
        padding: 20,
        backgroundColor: colors.white,
    },
    diagnosticResults: {
        marginBottom: 20,
    },
    scoreHistory: {
        marginVertical: 20,
    },
    dailyChallenges: {
        marginVertical: 20,
    },
    achievements: {
        marginVertical: 20,
    },
    proficiencies: {
        marginVertical: 20,
    },
    progressBar: {
        backgroundColor: '#f0f0f0',
        borderRadius: 5,
        height: 20,
        position: 'relative',
    },
    progressFill: {
        backgroundColor: colors.primary,
        height: '100%',
        borderRadius: 5,
    },
    progressText: {
        position: 'absolute',
        right: 10,
        top: '50%',
        transform: [{ translateY: -10 }],
        color: colors.text,
    },
    feedback: {
        marginTop: 20,
    },
    feedbackInput: {
        width: '100%',
        height: 100,
        marginBottom: 10,
        padding: 10,
        borderWidth: 1,
        borderColor: colors.gray,
        borderRadius: 5,
    },
});

export default DashboardScreen;
```

#### Update Navigation (`SATSmartPrepApp/src/navigation/index.js`)

Add the `ReviewScreen` to the navigation stack.

```javascript
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import LoginScreen from '../screens/LoginScreen';
import OnboardingScreen from '../screens/OnboardingScreen';
import DiagnosticScreen from '../screens/DiagnosticScreen';
import PracticeScreen from '../screens/PracticeScreen';
import StudyPlanScreen from '../screens/StudyPlanScreen';
import DashboardScreen from '../screens/DashboardScreen';
import CommunityScreen from '../screens/CommunityScreen';
import LeaderboardScreen from '../screens/LeaderboardScreen';
import RewardsStoreScreen from '../screens/RewardsStoreScreen';
import PolicyScreen from '../screens/PolicyScreen';
import ReviewScreen from '../screens/ReviewScreen';

const Stack = createStackNavigator();

const AppNavigator = () => {
    return (
        <NavigationContainer>
            <Stack.Navigator initialRouteName="Login">
                <Stack.Screen name="Login" component={LoginScreen} />
                <Stack.Screen name="Onboarding" component={OnboardingScreen} />
                <Stack.Screen name="Diagnostic" component={DiagnosticScreen} />
                <Stack.Screen name="Practice" component={PracticeScreen} />
                <Stack.Screen name="StudyPlan" component={StudyPlanScreen} />
                <Stack.Screen name="Dashboard" component={DashboardScreen} />
                <Stack.Screen name="Community" component={CommunityScreen} />
                <Stack.Screen name="Leaderboard" component={LeaderboardScreen} />
                <Stack.Screen name="RewardsStore" component={RewardsStoreScreen} />
                <Stack.Screen name="Policy" component={PolicyScreen} />
                <Stack.Screen name="Review" component={ReviewScreen} />
            </Stack.Navigator>
        </NavigationContainer>
    );
};

export default AppNavigator;
```

***

### Summary of the Diagnostic Review Process

The **Diagnostic Review Process** in the SAT Smart Prep App allows users to analyze their performance after completing the Diagnostic Test. The process involves:

* Accessing the review from the dashboard, where a "Review Diagnostic Test" button links to a dedicated review page.
* Fetching review data via the `/practice/review/{session_id}` endpoint, which retrieves all responses, question details, and summary statistics.
* Displaying the review data in a tabbed interface (Reading & Writing, Math), with filters for incorrect answers and subskills, and summary statistics (e.g., correct answers, average theta, weakest subskill).
* Providing recommendations based on the weakest subskills, encouraging users to focus on specific areas for improvement.

This implementation ensures users can learn from their mistakes, understand their strengths and weaknesses, and take actionable steps to improve their SAT performance. Let me know if you’d like to further refine this process or add additional features!
