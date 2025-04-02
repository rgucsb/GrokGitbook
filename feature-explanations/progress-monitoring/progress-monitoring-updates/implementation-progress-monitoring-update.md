# Implementation progress monitoring update

Yes, the code for the **Progress Monitoring** feature in the **SAT Smart Prep App** by **Learner Labs** needs to be updated to reflect the change in the UI, where the `theta` scores (ranging from -3 to 3) are transformed into a more intuitive **0 to 100% mastery scale** for display purposes, while the backend continues to use `theta` for all calculations. Below, I’ll provide the updated code for the relevant components, specifically focusing on the dashboard where progress monitoring is primarily displayed. The updates will ensure that the UI shows mastery percentages, while the backend logic remains unchanged, preserving the use of `theta` for calculations such as Study Plan updates, practice session adjustments, and analytics.

***

### Updated Code for Progress Monitoring Changes

#### 1. Update the Dashboard (Web) - `frontend/pages/dashboard.js`

The dashboard is the primary place where progress monitoring is displayed. We’ll update this file to transform `theta` scores into mastery percentages for display in the UI, while keeping `theta` for backend calculations.

```javascript
import React, { useState, useEffect } from 'react';
import { useRouter } from 'next/router';
import { api } from '../utils/api';
import { colors, typography, commonStyles } from '../styles';

// Helper function to convert theta (-3 to 3) to mastery percentage (0 to 100%)
const thetaToMastery = (theta) => {
    return Math.round(((theta + 3) / 6) * 100); // Linear transformation
};

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

    const updateStudyPlan = async () => {
        try {
            const res = await api.post(`/study_plan/update/${userId}`, {
                test_date: userData.sat_test_date,
                study_hours: userData.study_hours_per_week,
                study_days: userData.study_days_per_week
            });
            alert('Study Plan updated successfully!');
            router.push('/study-plan');
        } catch (error) {
            alert('Failed to update Study Plan: ' + error.message);
        }
    };

    // Calculate overall mastery from diagnostic results
    const overallMastery = diagnosticResults ? thetaToMastery(diagnosticResults.theta) : 0;
    const estimatedScore = diagnosticResults ? (diagnosticResults.theta * 400 + 400) : 0;

    // Calculate domain-level mastery
    const readingMastery = diagnosticResults ? thetaToMastery(diagnosticResults.reading_theta || 0) : 0;
    const mathMastery = diagnosticResults ? thetaToMastery(diagnosticResults.math_theta || 0) : 0;

    return (
        <div className="dashboard">
            {diagnosticResults ? (
                <div className="diagnostic-results">
                    <h2 style={typography.heading}>Your Diagnostic Results</h2>
                    <p style={typography.body}>Overall Mastery: {overallMastery}%</p>
                    <p style={typography.body}>Estimated SAT Score: {estimatedScore}</p>
                    <h3 style={typography.heading}>Math</h3>
                    <p style={typography.body}>Math Mastery: {mathMastery}%</p>
                    <p style={typography.body}>Strong Areas: Algebra (Mastery: {thetaToMastery(diagnosticResults.math_theta || 0)}%)</p>
                    <p style={typography.body}>Weak Areas: Geometry (Mastery: {thetaToMastery((diagnosticResults.math_theta || 0) - 0.5)}%)</p>
                    <h3 style={typography.heading}>Reading & Writing</h3>
                    <p style={typography.body}>Reading & Writing Mastery: {readingMastery}%</p>
                    <p style={typography.body}>Strong Areas: Reading Comprehension (Mastery: {thetaToMastery(diagnosticResults.reading_theta || 0)}%)</p>
                    <p style={typography.body}>Weak Areas: Grammar (Mastery: {thetaToMastery((diagnosticResults.reading_theta || 0) - 0.5)}%)</p>
                    <p style={typography.body}>Key Recommendation: Focus on Geometry and Grammar to improve your mastery.</p>
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
                                {new Date(entry.timestamp).toLocaleDateString()}: {entry.score} (Mastery: {thetaToMastery(entry.theta)}%)
                            </p>
                        ))}
                        <button style={commonStyles.button} onClick={checkGuarantee}>
                            <span style={commonStyles.buttonText}>Check Score Guarantee</span>
                        </button>
                        <button style={commonStyles.button} onClick={() => router.push('/policy')}>
                            <span style={commonStyles.buttonText}>View Guarantee Policy</span>
                        </button>
                        <button style={commonStyles.button} onClick={updateStudyPlan}>
                            <span style={commonStyles.buttonText}>Update Study Plan</span>
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
                                <p style={typography.body}>{prof.domain} - {prof.skill}: Mastery {thetaToMastery(prof.theta)}%</p>
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

#### 2. Update the Dashboard (Mobile) - `SATSmartPrepApp/src/screens/DashboardScreen.js`

The mobile app’s dashboard also needs to be updated to display mastery percentages instead of `theta` scores.

```javascript
import React, { useState, useEffect } from 'react';
import { View, Text, TextInput, TouchableOpacity, StyleSheet, Alert, ScrollView } from 'react-native';
import Animated, { FadeIn, FadeOut } from 'react-native-reanimated';
import { api } from '../utils/api';
import { colors, typography, commonStyles } from '../styles';

// Helper function to convert theta (-3 to 3) to mastery percentage (0 to 100%)
const thetaToMastery = (theta) => {
    return Math.round(((theta + 3) / 6) * 100); // Linear transformation
};

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

    const updateStudyPlan = async () => {
        try {
            const res = await api.post(`/study_plan/update/${userId}`, {
                test_date: userData.sat_test_date,
                study_hours: userData.study_hours_per_week,
                study_days: userData.study_days_per_week
            });
            Alert.alert('Success', 'Study Plan updated successfully!');
            navigation.navigate('StudyPlan');
        } catch (error) {
            Alert.alert('Error', 'Failed to update Study Plan: ' + error.message);
        }
    };

    // Calculate overall mastery from diagnostic results
    const overallMastery = diagnosticResults ? thetaToMastery(diagnosticResults.theta) : 0;
    const estimatedScore = diagnosticResults ? (diagnosticResults.theta * 400 + 400) : 0;

    // Calculate domain-level mastery
    const readingMastery = diagnosticResults ? thetaToMastery(diagnosticResults.reading_theta || 0) : 0;
    const mathMastery = diagnosticResults ? thetaToMastery(diagnosticResults.math_theta || 0) : 0;

    return (
        <Animated.View entering={FadeIn} exiting={FadeOut} style={styles.container}>
            <ScrollView>
                {diagnosticResults ? (
                    <View style={styles.diagnosticResults}>
                        <Text style={typography.heading}>Your Diagnostic Results</Text>
                        <Text style={typography.body}>Overall Mastery: {overallMastery}%</Text>
                        <Text style={typography.body}>Estimated SAT Score: {estimatedScore}</Text>
                        <Text style={typography.heading}>Math</Text>
                        <Text style={typography.body}>Math Mastery: {mathMastery}%</Text>
                        <Text style={typography.body}>Strong Areas: Algebra (Mastery: {thetaToMastery(diagnosticResults.math_theta || 0)}%)</Text>
                        <Text style={typography.body}>Weak Areas: Geometry (Mastery: {thetaToMastery((diagnosticResults.math_theta || 0) - 0.5)}%)</Text>
                        <Text style={typography.heading}>Reading & Writing</Text>
                        <Text style={typography.body}>Reading & Writing Mastery: {readingMastery}%</Text>
                        <Text style={typography.body}>Strong Areas: Reading Comprehension (Mastery: {thetaToMastery(diagnosticResults.reading_theta || 0)}%)</Text>
                        <Text style={typography.body}>Weak Areas: Grammar (Mastery: {thetaToMastery((diagnosticResults.reading_theta || 0) - 0.5)}%)</Text>
                        <Text style={typography.body}>Key Recommendation: Focus on Geometry and Grammar to improve your mastery.</Text>
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
                                    {new Date(entry.timestamp).toLocaleDateString()}: {entry.score} (Mastery: {thetaToMastery(entry.theta)}%)
                                </Text>
                            ))}
                            <TouchableOpacity style={commonStyles.button} onPress={checkGuarantee}>
                                <Text style={commonStyles.buttonText}>Check Score Guarantee</Text>
                            </TouchableOpacity>
                            <TouchableOpacity style={commonStyles.button} onPress={() => navigation.navigate('Policy')}>
                                <Text style={commonStyles.buttonText}>View Guarantee Policy</Text>
                            </TouchableOpacity>
                            <TouchableOpacity style={commonStyles.button} onPress={updateStudyPlan}>
                                <Text style={commonStyles.buttonText}>Update Study Plan</Text>
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
                                    <Text style={typography.body}>{prof.domain} - {prof.skill}: Mastery {thetaToMastery(prof.theta)}%</Text>
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
```

#### 3. Update the Review Screen (Web) - `frontend/pages/review.js`

The review screen also displays performance metrics, so we’ll update it to show mastery percentages instead of `theta` scores.

```javascript
import React, { useState, useEffect } from 'react';
import { useRouter } from 'next/router';
import { api } from '../utils/api';
import { colors, typography, commonStyles } from '../styles';

// Helper function to convert theta (-3 to 3) to mastery percentage (0 to 100%)
const thetaToMastery = (theta) => {
    return Math.round(((theta + 3) / 6) * 100); // Linear transformation
};

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
                            Mastery: {thetaToMastery(reviewData.summary[activeTab === 'Reading & Writing' ? 'reading_writing' : 'math'].theta)}%
                        </p>
                        <p style={typography.body}>
                            Weakest Subskill: {reviewData.summary[activeTab === 'Reading & Writing' ? 'reading_writing' : 'math'].weakest_subskill || 'N/A'}
                        </p>
                        {reviewData.summary[activeTab === 'Reading & Writing' ? 'reading_writing' : 'math'].weakest_subskill && (
                            <p style={typography.body}>
                                Recommendation: Focus on {reviewData.summary[activeTab === 'Reading & Writing' ? 'reading_writing' : 'math'].weakest_subskill} to improve your {activeTab} mastery.
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

#### 4. Update the Review Screen (Mobile) - `SATSmartPrepApp/src/screens/ReviewScreen.js`

The mobile app’s review screen also needs to display mastery percentages instead of `theta` scores.

```javascript
import React, { useState, useEffect } from 'react';
import { View, Text, ScrollView, TouchableOpacity, Picker, StyleSheet } from 'react-native';
import Animated, { FadeIn, FadeOut } from 'react-native-reanimated';
import { api } from '../utils/api';
import { colors, typography, commonStyles } from '../styles';

// Helper function to convert theta (-3 to 3) to mastery percentage (0 to 100%)
const thetaToMastery = (theta) => {
    return Math.round(((theta + 3) / 6) * 100); // Linear transformation
};

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
                            Mastery: {thetaToMastery(reviewData.summary[activeTab === 'Reading & Writing' ? 'reading_writing' : 'math'].theta)}%
                        </Text>
                        <Text style={typography.body}>
                            Weakest Subskill: {reviewData.summary[activeTab === 'Reading & Writing' ? 'reading_writing' : 'math'].weakest_subskill || 'N/A'}
                        </Text>
                        {reviewData.summary[activeTab === 'Reading & Writing' ? 'reading_writing' : 'math'].weakest_subskill && (
                            <Text style={typography.body}>
                                Recommendation: Focus on {reviewData.summary[activeTab === 'Reading & Writing' ? 'reading_writing' : 'math'].weakest_subskill} to improve your {activeTab} mastery.
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

#### 5. Update the Study Plan Screen (Web) - `frontend/pages/study-plan.js`

The Study Plan screen may also display progress-related metrics, so we’ll update it to show mastery percentages.

```javascript
import React, { useState, useEffect } from 'react';
import { useRouter } from 'next/router';
import { api } from '../utils/api';
import { colors, typography, commonStyles } from '../styles';

// Helper function to convert theta (-3 to 3) to mastery percentage (0 to 100%)
const thetaToMastery = (theta) => {
    return Math.round(((theta + 3) / 6) * 100); // Linear transformation
};

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
                        phase: milestone.phase,
                        task: milestone.task,
                        action: milestone.action,
                        due_date: milestone.due_date,
                        points: milestone.points,
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
                phase: milestone.phase,
                task: milestone.task,
                action: milestone.action,
                due_date: milestone.due_date,
                points: milestone.points,
                completed: null
            }))
        });
        setEditMode(false);
    };

    const groupByPhase = (actions) => {
        const phases = {
            "Phase 1: Foundation Building": [],
            "Phase 2: Practice and Strategy": [],
            "Phase 3: Final Review and Refinement": []
        };
        actions.forEach(action => {
            phases[action.phase].push(action);
        });
        return phases;
    };

    return (
        <div className="study-plan">
            {studyPlan ? (
                <>
                    <h1 style={typography.heading}>Your Study Plan</h1>
                    <p style={typography.body}>Test Date: {new Date(studyPlan.test_date).toLocaleDateString()}</p>
                    <h2 style={typography.heading}>Tasks</h2>
                    {Object.entries(groupByPhase(studyPlan.actions)).map(([phase, actions]) => (
                        <div key={phase} className="phase">
                            <h3 style={typography.heading}>{phase}</h3>
                            {actions.map(action => (
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
                .phase {
                    margin-bottom: 30px;
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

#### 5. Update the Study Plan Screen (Mobile) - `SATSmartPrepApp/src/screens/StudyPlanScreen.js`

The mobile app’s Study Plan screen also needs to reflect the mastery scale if it displays progress metrics.

```javascript
import React, { useState, useEffect } from 'react';
import { View, Text, TextInput, Picker, TouchableOpacity, StyleSheet, Alert, ScrollView } from 'react-native';
import Animated, { FadeIn, FadeOut } from 'react-native-reanimated';
import { api } from '../utils/api';
import { colors, typography, commonStyles } from '../styles';

// Helper function to convert theta (-3 to 3) to mastery percentage (0 to 100%)
const thetaToMastery = (theta) => {
    return Math.round(((theta + 3) / 6) * 100); // Linear transformation
};

const StudyPlanScreen = ({ userData }) => {
    const [studyPlan, setStudyPlan] = useState(null);
    const [editMode, setEditMode] = useState(false);
    const [studyHours, setStudyHours] = useState(userData?.study_hours_per_week || 1);
    const [studyDays, setStudyDays] = useState(userData?.study_days_per_week || 1);
    const userId = localStorage.getItem('user_id');

    useEffect(() => {
        fetchStudyPlan();
    }, []);

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
                        phase: milestone.phase,
                        task: milestone.task,
                        action: milestone.action,
                        due_date: milestone.due_date,
                        points: milestone.points,
                        completed: null
                    }))
                });
            } else {
                Alert.alert('Error', 'Failed to load study plan: ' + error.message);
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
                phase: milestone.phase,
                task: milestone.task,
                action: milestone.action,
                due_date: milestone.due_date,
                points: milestone.points,
                completed: null
            }))
        });
        setEditMode(false);
    };

    const groupByPhase = (actions) => {
        const phases = {
            "Phase 1: Foundation Building": [],
            "Phase 2: Practice and Strategy": [],
            "Phase 3: Final Review and Refinement": []
        };
        actions.forEach(action => {
            phases[action.phase].push(action);
        });
        return phases;
    };

    return (
        <Animated.View entering={FadeIn} exiting={FadeOut} style={styles.container}>
            <ScrollView>
                {studyPlan ? (
                    <>
                        <Text style={typography.heading}>Your Study Plan</Text>
                        <Text style={typography.body}>Test Date: {new Date(studyPlan.test_date).toLocaleDateString()}</Text>
                        <Text style={typography.heading}>Tasks</Text>
                        {Object.entries(groupByPhase(studyPlan.actions)).map(([phase, actions]) => (
                            <View key={phase} style={styles.phase}>
                                <Text style={typography.heading}>{phase}</Text>
                                {actions.map(action => (
                                    <View key={action.id} style={commonStyles.card}>
                                        <Text style={typography.body}>{action.task}</Text>
                                        <Text style={typography.body}>Due: {new Date(action.due_date).toLocaleDateString()}</Text>
                                        <Text style={typography.body}>Points: {action.points}</Text>
                                        {action.completed ? (
                                            <Text style={typography.body}>Completed on {new Date(action.completed).toLocaleDateString()}</Text>
                                        ) : (
                                            <Text style={typography.body}>Not yet completed</Text>
                                        )}
                                    </View>
                                ))}
                            </View>
                        ))}
                        <TouchableOpacity style={commonStyles.button} onPress={() => setEditMode(true)}>
                            <Text style={commonStyles.buttonText}>Adjust Study Plan</Text>
                        </TouchableOpacity>
                        {editMode && (
                            <View style={styles.editPlan}>
                                <View style={styles.formGroup}>
                                    <Text style={typography.body}>Study Hours Per Week ({studyHours} hours)</Text>
                                    <TextInput
                                        style={styles.input}
                                        value={studyHours.toString()}
                                        onChangeText={(value) => setStudyHours(parseInt(value))}
                                        keyboardType="numeric"
                                    />
                                </View>
                                <View style={styles.formGroup}>
                                    <Text style={typography.body}>Study Days Per Week</Text>
                                    <Picker
                                        selectedValue={studyDays}
                                        onValueChange={(value) => setStudyDays(value)}
                                        style={styles.input}
                                    >
                                        {[1, 2, 3, 4, 5, 6, 7].map(day => (
                                            <Picker.Item key={day} label={day.toString()} value={day} />
                                        ))}
                                    </Picker>
                                </View>
                                <TouchableOpacity style={commonStyles.button} onPress={updateStudyPlan}>
                                    <Text style={commonStyles.buttonText}>Save Changes</Text>
                                </TouchableOpacity>
                            </View>
                        )}
                    </>
                ) : (
                    <Text style={typography.body}>Loading your study plan...</Text>
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
        alignItems: 'center',
    },
    phase: {
        marginBottom: 30,
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
    editPlan: {
        marginTop: 20,
        width: '100%',
    },
});
```

***

### Summary of Changes

The code for the **Progress Monitoring** feature has been updated to reflect the new UI requirement of displaying performance metrics as **0 to 100% mastery** instead of `theta` scores. The updates include:

1. **Added a Helper Function**: A `thetaToMastery` function was added to both web and mobile components to transform `theta` scores (-3 to 3) into mastery percentages (0 to 100%) using the formula `((theta + 3) / 6) * 100`.
2. **Updated Dashboard (Web and Mobile)**:
   * The dashboard now displays overall mastery, domain-level mastery (Reading & Writing, Math), and subskill mastery as percentages.
   * Score history includes mastery percentages alongside SAT scores.
   * Proficiency metrics for subskills are shown as mastery percentages.
3. **Updated Review Screen (Web and Mobile)**:
   * The review screen now shows domain-level mastery percentages in the summary section (e.g., "Reading & Writing Mastery: 60%").
4. **Updated Study Plan Screen (Web and Mobile)**:
   * The Study Plan screen was updated to ensure consistency, although it primarily displays tasks and does not currently show progress metrics. If progress metrics are added in the future, they will use the mastery scale.
5. **Backend Unchanged**: The backend continues to use `theta` scores for all calculations, ensuring that analytics, Study Plan updates, and practice session adjustments remain accurate and unaffected by the UI change.

These updates make the Progress Monitoring feature more intuitive for users by presenting performance metrics in a familiar percentage format, while maintaining the precision of `theta`-based calculations in the backend. Let me know if you’d like to further refine these changes or add additional features!
