# Implement Score Guarantee and Granular Study Plan

Let’s implement the **Score Improvement Guarantee** and **Granular Study Plans** for the **SAT Smart Prep App** by **Learner Labs**, as outlined in the next steps. These features will enhance user trust and provide more personalized study experiences, aligning with the app’s goal of becoming the best AI-based SAT test prep app by the end of 2025. I’ll provide the implementation details for both the web app and mobile app, ensuring tight integration and consistency across platforms. The timeline for these implementations will follow the previously proposed schedule: **Score Improvement Guarantee** in November 2025 and **Granular Study Plans** in February 2026.

***

### Implementation Plan

#### 1. Implement Score Improvement Guarantee (November 2025)

**Objective**

Offer a 150-point score increase guarantee or refund for users of **SAT Smart Prep App**, building trust and encouraging sign-ups. This feature will track user progress and allow users to check if they qualify for a refund based on their score improvement.

**Sub-Steps**

1. **Backend: Add Score Tracking and Guarantee Logic**
   * **File**: `frontend/backend/src/progress_monitoring.py`
   * **Action**: Add an endpoint to track score history and another to check the guarantee eligibility.
   *   **Code**:

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

       @router.get("/progress_monitoring/scores/{user_id}")
       async def get_score_history(user_id: str):
           conn = get_db_connection()
           cursor = conn.cursor(cursor_factory=RealDictCursor)
           try:
               cursor.execute("""
                   SELECT theta, timestamp
                   FROM practice_sessions
                   WHERE user_id = %s AND test_type = 'SAT'
                   ORDER BY timestamp;
               """, (user_id,))
               sessions = cursor.fetchall()
               return [{"score": session['theta'] * 400 + 400, "timestamp": session['timestamp']} for session in sessions]
           except Exception as e:
               raise HTTPException(status_code=500, detail=str(e))
           finally:
               cursor.close()
               conn.close()

       @router.get("/progress_monitoring/guarantee/{user_id}")
       async def check_score_guarantee(user_id: str):
           conn = get_db_connection()
           cursor = conn.cursor(cursor_factory=RealDictCursor)
           try:
               cursor.execute("""
                   SELECT theta, timestamp
                   FROM practice_sessions
                   WHERE user_id = %s AND test_type = 'SAT'
                   ORDER BY timestamp;
               """, (user_id,))
               sessions = cursor.fetchall()
               if len(sessions) < 2:
                   return {"eligible": False, "message": "Not enough test data to evaluate guarantee. Complete at least two full-length tests."}
               initial_score = sessions[0]['theta'] * 400 + 400
               latest_score = sessions[-1]['theta'] * 400 + 400
               improvement = latest_score - initial_score
               if improvement < 150:
                   return {
                       "eligible": True,
                       "improvement": improvement,
                       "message": "You qualify for a refund! Contact support at support@learnerlabs.com."
                   }
               return {
                   "eligible": False,
                   "improvement": improvement,
                   "message": "Great job! You’ve improved by 150+ points."
               }
           except Exception as e:
               raise HTTPException(status_code=500, detail=str(e))
           finally:
               cursor.close()
               conn.close()
       ```
2. **Mobile App: Add Score Guarantee Check in Dashboard**
   * **File**: `SATSmartPrepApp/src/screens/DashboardScreen.js`
   * **Action**: Update the `DashboardScreen` to include a button to check the score guarantee, displaying the user’s score history and eligibility for a refund.
   *   **Code**:

       ```javascript
       import React, { useState, useEffect } from 'react';
       import { View, Text, TouchableOpacity, StyleSheet, Alert } from 'react-native';
       import Animated, { FadeIn, FadeOut } from 'react-native-reanimated';
       import { api, connectWebSocket } from '../utils/api';
       import AsyncStorage from '@react-native-async-storage/async-storage';
       import { colors, typography, commonStyles } from '../styles';

       const DashboardScreen = ({ diagnosticResults, navigation }) => {
         const [user, setUser] = useState(null);
         const [challenges, setChallenges] = useState([]);
         const [achievements, setAchievements] = useState([]);
         const [proficiencies, setProficiencies] = useState([]);
         const [scoreHistory, setScoreHistory] = useState([]);
         const [feedback, setFeedback] = useState('');
         const userId = localStorage.getItem('user_id');

         useEffect(() => {
           const loadData = async () => {
             const storedUserId = await AsyncStorage.getItem('user_id');
             if (!storedUserId) {
               navigation.navigate('Login');
               return;
             }
             try {
               const coinsRes = await api.get(`/gamification/coins/${userId}`);
               setUser({ coins: coinsRes.data.coins });
               const challengesRes = await api.get(`/challenges/${userId}`);
               setChallenges(challengesRes.data);
               const proficienciesRes = await api.get(`/progress_monitoring/proficiencies/${userId}`);
               setProficiencies(proficienciesRes.data);
               const scoreHistoryRes = await api.get(`/progress_monitoring/scores/${userId}`);
               setScoreHistory(scoreHistoryRes.data);
               setAchievements([
                 { id: 1, name: "Complete 100 Questions", target: 100, progress: 75, completed: false },
                 { id: 2, name: "5-Day Streak", target: 5, progress: 3, completed: false }
               ]);

               const ws = connectWebSocket(userId, (message) => {
                 Alert.alert('Update', message);
                 api.get(`/gamification/coins/${userId}`).then(res => setUser({ coins: res.data.coins }));
               });
               return () => ws.close();
             } catch (error) {
               Alert.alert('Error', 'Failed to load dashboard: ' + error.message);
             }
           };
           loadData();
         }, []);

         const submitFeedback = async () => {
           if (!feedback) return;
           try {
             await api.post('/feedback', { user_id: userId, content: feedback });
             Alert.alert('Thank you for your feedback!');
             setFeedback('');
           } catch (error) {
             Alert.alert('Error', 'Failed to submit feedback: ' + error.message);
           }
         };

         const checkGuarantee = async () => {
           try {
             const res = await api.get(`/progress_monitoring/guarantee/${userId}`);
             Alert.alert('Score Guarantee', res.data.message);
           } catch (error) {
             Alert.alert('Error', 'Failed to check guarantee: ' + error.message);
           }
         };

         const estimatedScore = diagnosticResults ? (diagnosticResults.theta * 400 + 400) : 0;

         return (
           <Animated.View entering={FadeIn} exiting={FadeOut} style={styles.container}>
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
                 </View>
                 <View style={styles.dailyChallenges}>
                   <Text style={typography.heading}>Daily Challenges</Text>
                   {challenges.map(challenge => (
                     <View key={challenge.id} style={commonStyles.card}>
                       <Text style={typography.body}>
                         {challenge.challenge_type === 'daily_questions' ? "Answer 10 Questions Correctly" : "Complete a 5-Question Session in Under 5 Minutes"}
                       </Text>
                       <View style={styles.progressBar}>
                         <View style={[styles.progress, { width: `${(challenge.progress / challenge.target) * 100}%` }]} />
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
                         <View style={[styles.progress, { width: `${(achievement.progress / achievement.target) * 100}%` }]} />
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
         progress: {
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
3. **Web App: Add Score Guarantee Check in Dashboard**
   * **File**: `frontend/pages/dashboard.js`
   * **Action**: Update the `Dashboard` page to include a button to check the score guarantee, displaying the user’s score history and eligibility for a refund.
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
4. **Add Policy Page for Score Guarantee**
   * **Mobile App**: Create a new screen for the policy.
     * **File**: `SATSmartPrepApp/src/screens/PolicyScreen.js`
     *   **Code**:

         ```javascript
         import React from 'react';
         import { View, Text, StyleSheet, ScrollView } from 'react-native';
         import Animated, { FadeIn, FadeOut } from 'react-native-reanimated';
         import { colors, typography } from '../styles';

         const PolicyScreen = () => {
           return (
             <Animated.View entering={FadeIn} exiting={FadeOut} style={styles.container}>
               <ScrollView>
                 <Text style={typography.heading}>Score Improvement Guarantee</Text>
                 <Text style={typography.body}>
                   At Learner Labs, we are committed to helping you achieve your best SAT score with SAT Smart Prep App. We guarantee a 150-point score increase on your SAT practice tests after completing at least two full-length tests and following our recommended study plan for 30 days.
                 </Text>
                 <Text style={typography.body}>
                   If you do not achieve a 150-point increase, you are eligible for a full refund. To qualify, you must:
                   - Complete at least two full-length SAT practice tests within the app.
                   - Follow the study plan for at least 30 days.
                   - Submit a refund request within 60 days of your subscription start date.
                 </Text>
                 <Text style={typography.body}>
                   To check your eligibility, go to the Dashboard and click "Check Score Guarantee." If eligible, contact our support team at support@learnerlabs.com to process your refund.
                 </Text>
                 <Text style={typography.body}>
                   Note: This guarantee applies only to SAT practice tests within the app and does not apply to official SAT scores. Refunds are processed within 14 business days.
                 </Text>
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
         });

         export default PolicyScreen;
         ```
     *   **Update Navigation**: Add the Policy screen to the navigation stack in `SATSmartPrepApp/App.js`:

         ```javascript
         import PolicyScreen from './src/screens/PolicyScreen';

         const Stack = createStackNavigator();
         const Tab = createBottomTabNavigator();

         const MainTabs = () => (
           <Tab.Navigator>
             <Tab.Screen name="Dashboard" component={DashboardScreen} options={{ headerShown: false }} />
             <Tab.Screen name="Practice" component={PracticeScreen} options={{ headerShown: false }} />
             <Tab.Screen name="StudyPlan" component={StudyPlanScreen} options={{ headerShown: false }} />
             <Tab.Screen name="Community" component={CommunityScreen} options={{ headerShown: false }} />
             <Tab.Screen name="Leaderboard" component={LeaderboardScreen} options={{ headerShown: false }} />
           </Tab.Navigator>
         );

         export default function App() {
           const [theme, setTheme] = useState('light');

           useEffect(() => {
             const loadTheme = async () => {
               const userId = await AsyncStorage.getItem('user_id');
               if (userId) {
                 const res = await api.get(`/auth/user/${userId}`);
                 setTheme(res.data.theme || 'light');
               }
             };
             loadTheme();

             const requestPermission = async () => {
               const authStatus = await messaging().requestPermission();
               if (authStatus === messaging.AuthorizationStatus.AUTHORIZED) {
                 const token = await messaging().getToken();
                 const userId = await AsyncStorage.getItem('user_id');
                 if (userId) {
                   await api.put(`/auth/update-device-token/${userId}`, { device_token: token });
                 }
               }
             };
             requestPermission();

             messaging().setBackgroundMessageHandler(async remoteMessage => {
               console.log('Message handled in the background!', remoteMessage);
             });

             messaging().onMessage(async remoteMessage => {
               Alert.alert('New Notification', remoteMessage.notification.body);
             });
           }, []);

           return (
             <View style={theme === 'dark' ? styles.darkTheme : styles.lightTheme}>
               <NavigationContainer>
                 <Stack.Navigator initialRouteName="Login">
                   <Stack.Screen name="Login" component={LoginScreen} options={{ headerShown: false }} />
                   <Stack.Screen name="Onboarding" component={OnboardingScreen} options={{ headerShown: false }} />
                   <Stack.Screen name="Main" component={MainTabs} options={{ headerShown: false }} />
                   <Stack.Screen name="RewardsStore" component={RewardsStoreScreen} options={{ headerShown: false }} />
                   <Stack.Screen name="Policy" component={PolicyScreen} options={{ title: 'Score Guarantee Policy' }} />
                 </Stack.Navigator>
               </NavigationContainer>
             </View>
           );
         }

         const styles = StyleSheet.create({
           lightTheme: {
             flex: 1,
             backgroundColor: '#fff',
             color: '#333',
           },
           darkTheme: {
             flex: 1,
             backgroundColor: '#333',
             color: '#fff',
           },
         });
         ```
     *   **Add Navigation Link**: Update `DashboardScreen.js` to navigate to the Policy screen:

         ```javascript
         const DashboardScreen = ({ diagnosticResults, navigation }) => {
           return (
             <Animated.View entering={FadeIn} exiting={FadeOut} style={styles.container}>
               {/* Existing content */}
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
               {/* Existing content */}
             </Animated.View>
           );
         };
         ```
   * **Web App**: Create a new policy page.
     * **File**: `frontend/pages/policy.js`
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
                 At Learner Labs, we are committed to helping you achieve your best SAT score with SAT Smart Prep App. We guarantee a 150-point score increase on your SAT practice tests after completing at least two full-length tests and following our recommended study plan for 30 days.
               </p>
               <p style={typography.body}>
                 If you do not achieve a 150-point increase, you are eligible for a full refund. To qualify, you must:
                 - Complete at least two full-length SAT practice tests within the app.
                 - Follow the study plan for at least 30 days.
                 - Submit a refund request within 60 days of your subscription start date.
               </p>
               <p style={typography.body}>
                 To check your eligibility, go to the Dashboard and click "Check Score Guarantee." If eligible, contact our support team at support@learnerlabs.com to process your refund.
               </p>
               <p style={typography.body}>
                 Note: This guarantee applies only to SAT practice tests within the app and does not apply to official SAT scores. Refunds are processed within 14 business days.
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
     *   **Add Navigation Link**: Update `frontend/pages/dashboard.js` to link to the Policy page:

         ```javascript
         const Dashboard = ({ diagnosticResults }) => {
           return (
             <div className="dashboard">
               {/* Existing content */}
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
               {/* Existing content */}
             </div>
           );
         };
         ```

**Deliverables**

* Backend endpoints to track score history and check guarantee eligibility.
* Updated Dashboard screen on both mobile and web apps to display score history and check guarantee eligibility.
* New Policy screen/page on both mobile and web apps explaining the score guarantee.

***

#### 2. Add Granular Study Plans (February 2026)

**Objective**

Enhance study plans in **SAT Smart Prep App** with skill-specific granularity and dynamic adjustments based on user progress, improving personalization and user outcomes.

**Sub-Steps**

1. **Backend: Update Study Plan Generation**
   * **File**: `frontend/backend/src/study_plan.py`
   * **Action**: Modify the study plan creation endpoint to generate skill-specific plans based on user proficiencies, focusing on weak areas (theta < 0.5).
   *   **Code**:

       ```python
       from fastapi import APIRouter, HTTPException
       from pydantic import BaseModel
       from psycopg2.extras import RealDictCursor
       import psycopg2
       import uuid
       from datetime import datetime, timedelta

       router = APIRouter()

       def get_db_connection():
           return psycopg2.connect(
               dbname="satprep",
               user="user",
               password="password",
               host="localhost",
               port="5432"
           )

       class StudyPlanRequest(BaseModel):
           test_date: str
           study_hours: int
           study_days: int

       @router.post("/study_plan/create/{user_id}")
       async def create_study_plan(user_id: str, request: StudyPlanRequest):
           conn = get_db_connection()
           cursor = conn.cursor(cursor_factory=RealDictCursor)
           try:
               # Fetch user proficiencies to identify weak areas
               cursor.execute("SELECT * FROM proficiencies WHERE user_id = %s", (user_id,))
               proficiencies = cursor.fetchall()
               focus_areas = [prof['skill'] for prof in proficiencies if prof['theta'] < 0.5]
               if not focus_areas:
                   focus_areas = [prof['skill'] for prof in proficiencies]  # Default to all skills if no weak areas

               # Calculate sessions per week
               sessions_per_week = request.study_days * (request.study_hours // request.study_days)
               total_sessions = len(focus_areas) * sessions_per_week

               # Calculate due dates for milestones
               test_date = datetime.fromisoformat(request.test_date.replace('Z', '+00:00'))
               days_until_test = (test_date - datetime.now()).days
               weeks_until_test = max(1, days_until_test // 7)
               sessions_per_skill = total_sessions // len(focus_areas)

               # Generate milestones
               milestones = [f"Complete {sessions_per_skill} sessions in {skill}" for skill in focus_areas]

               # Create study plan
               plan_id = str(uuid.uuid4())
               cursor.execute("""
                   INSERT INTO study_plans (plan_id, user_id, test_date)
                   VALUES (%s, %s, %s);
               """, (plan_id, user_id, request.test_date))

               # Add skill-specific actions
               for i, milestone in enumerate(milestones):
                   due_date = (datetime.now() + timedelta(days=(i + 1) * (days_until_test // len(milestones)))).isoformat()
                   cursor.execute("""
                       INSERT INTO study_plan_actions (plan_id, task, action, due_date, points)
                       VALUES (%s, %s, %s, %s, 50);
                   """, (plan_id, milestone, "Complete", due_date))

               conn.commit()
               return {
                   "plan_id": plan_id,
                   "sessions_per_week": sessions_per_week,
                   "focus_areas": focus_areas,
                   "milestones": milestones
               }
           except Exception as e:
               conn.rollback()
               raise HTTPException(status_code=500, detail=str(e))
           finally:
               cursor.close()
               conn.close()

       @router.get("/study_plan/{user_id}")
       async def get_study_plan(user_id: str):
           conn = get_db_connection()
           cursor = conn.cursor(cursor_factory=RealDictCursor)
           try:
               cursor.execute("SELECT * FROM study_plans WHERE user_id = %s ORDER BY test_date DESC LIMIT 1", (user_id,))
               plan = cursor.fetchone()
               if not plan:
                   raise HTTPException(status_code=404, detail="Study plan not found")
               cursor.execute("SELECT * FROM study_plan_actions WHERE plan_id = %s", (plan['plan_id'],))
               actions = cursor.fetchall()
               return {
                   "plan_id": plan['plan_id'],
                   "test_date": plan['test_date'],
                   "actions": actions
               }
           except Exception as e:
               raise HTTPException(status_code=500, detail=str(e))
           finally:
               cursor.close()
               conn.close()
       ```
2. **Mobile App: Update Study Plan Screen**
   * **File**: `SATSmartPrepApp/src/screens/StudyPlanScreen.js`
   * **Action**: Update the `StudyPlanScreen` to display skill-specific study plans and allow dynamic adjustments.
   *   **Code**:

       ```javascript
       import React, { useState, useEffect } from 'react';
       import { View, Text, TouchableOpacity, Slider, Picker, StyleSheet, Alert } from 'react-native';
       import Animated, { FadeIn, FadeOut } from 'react-native-reanimated';
       import { api } from '../utils/api';
       import AsyncStorage from '@react-native-async-storage/async-storage';
       import { colors, typography, commonStyles } from '../styles';

       const StudyPlanScreen = ({ userData, navigation }) => {
         const [studyPlan, setStudyPlan] = useState(null);
         const [editMode, setEditMode] = useState(false);
         const [studyHours, setStudyHours] = useState(userData?.study_hours_per_week || 1);
         const [studyDays, setStudyDays] = useState(userData?.study_days_per_week || 1);
         const userId = localStorage.getItem('user_id');

         useEffect(() => {
           const loadStudyPlan = async () => {
             const storedUserId = await AsyncStorage.getItem('user_id');
             if (!storedUserId) {
               navigation.navigate('Login');
               return;
             }
             fetchStudyPlan();
           };
           loadStudyPlan();
         }, []);

         const fetchStudyPlan = async () => {
           try {
             const existingPlan = await api.get(`/study_plan/${userId}`);
             setStudyPlan(existingPlan.data);
           } catch (error) {
             if (error.status === 404) {
               // Create a new study plan if none exists
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
               Alert.alert('Error', 'Failed to load study plan: ' + error.message);
             }
           }
         };

         const updateStudyPlan = async () => {
           try {
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
           } catch (error) {
             Alert.alert('Error', 'Failed to update study plan: ' + error.message);
           }
         };

         return (
           <Animated.View entering={FadeIn} exiting={FadeOut} style={styles.container}>
             {studyPlan ? (
               <>
                 <Text style={typography.heading}>Your Study Plan</Text>
                 <Text style={typography.body}>Test Date: {new Date(studyPlan.test_date).toLocaleDateString()}</Text>
                 <Text style={typography.heading}>Tasks</Text>
                 {studyPlan.actions.map(action => (
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
                 <TouchableOpacity style={commonStyles.button} onPress={() => setEditMode(true)}>
                   <Text style={commonStyles.buttonText}>Adjust Study Plan</Text>
                 </TouchableOpacity>
                 {editMode && (
                   <View style={styles.editPlan}>
                     <View style={styles.formGroup}>
                       <Text style={styles.label}>Study Hours Per Week ({studyHours} hours)</Text>
                       <Slider
                         minimumValue={1}
                         maximumValue={20}
                         step={1}
                         value={studyHours}
                         onValueChange={setStudyHours}
                         style={styles.slider}
                       />
                     </View>
                     <View style={styles.formGroup}>
                       <Text style={styles.label}>Study Days Per Week</Text>
                       <Picker
                         selectedValue={studyDays}
                         onValueChange={setStudyDays}
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
           </Animated.View>
         );
       };

       const styles = StyleSheet.create({
         container: {
           flex: 1,
           padding: 20,
           backgroundColor: colors.white,
         },
         formGroup: {
           marginBottom: 20,
         },
         label: {
           marginBottom: 5,
           fontWeight: '500',
         },
         input: {
           borderWidth: 1,
           borderColor: colors.gray,
           borderRadius: 5,
           padding: 8,
         },
         slider: {
           width: '100%',
         },
       });

       export default StudyPlanScreen;
       ```
3. **Web App: Update Study Plan Page**
   * **File**: `frontend/pages/study-plan.js`
   * **Action**: Update the `StudyPlan` page to display skill-specific study plans and allow dynamic adjustments.
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
4. **Update Onboarding to Use Granular Study Plans**
   * **Mobile App**: Update `OnboardingScreen.js` to pass the updated study plan data.
     * **File**: `SATSmartPrepApp/src/screens/OnboardingScreen.js`
     * **Action**: Update the `StudyPlanStep` to use the new granular study plan.
     *   **Code**:

         ```javascript
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
         ```
     * **Verification**: The `StudyPlanScreen` already fetches the updated granular study plan, so no further changes are needed here.
   * **Web App**: Update `onboarding.js` similarly.
     * **File**: `frontend/pages/onboarding.js`
     *   **Code**:

         ```javascript
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
         ```

**Deliverables**

* Backend updated to generate skill-specific study plans based on user proficiencies.
* Updated Study Plan screen/page on both mobile and web apps to display granular tasks.
* Onboarding updated to use the new granular study plans.

***

### Testing and Verification

1. **Unit Tests for Backend**:
   * **File**: `frontend/backend/tests/test_study_plan.py`
   *   **Code**:

       ```python
       from fastapi.testclient import TestClient
       from backend.src.main import app

       client = TestClient(app)

       def test_create_study_plan():
           response = client.post("/study_plan/create/user1", json={
               "test_date": "2025-12-01T00:00:00Z",
               "study_hours": 10,
               "study_days": 5
           })
           assert response.status_code == 200
           data = response.json()
           assert "plan_id" in data
           assert "sessions_per_week" in data
           assert "focus_areas" in data
           assert "milestones" in data

       def test_get_study_plan():
           client.post("/study_plan/create/user1", json={
               "test_date": "2025-12-01T00:00:00Z",
               "study_hours": 10,
               "study_days": 5
           })
           response = client.get("/study_plan/user1")
           assert response.status_code == 200
           data = response.json()
           assert "plan_id" in data
           assert "actions" in data
       ```
2. **Unit Tests for Mobile App**:
   * **File**: `SATSmartPrepApp/__tests__/StudyPlanScreen.test.js`
   *   **Code**:

       ```javascript
       import { render, fireEvent } from '@testing-library/react-native';
       import StudyPlanScreen from '../src/screens/StudyPlanScreen';
       import { api } from '../src/utils/api';

       jest.mock('../src/utils/api');

       describe('StudyPlanScreen', () => {
         beforeEach(() => {
           localStorage.setItem('user_id', 'user1');
           api.get.mockImplementation(endpoint => {
             if (endpoint === '/study_plan/user1') {
               return Promise.resolve({
                 data: {
                   plan_id: 'plan1',
                   test_date: '2025-12-01T00:00:00Z',
                   actions: [
                     { id: 1, task: "Complete 5 sessions in Algebra", due_date: "2025-11-01T00:00:00Z", points: 50, completed: null }
                   ]
                 }
               });
             }
             return Promise.resolve({ data: {} });
           });
           api.post.mockImplementation(() => Promise.resolve({
             data: {
               plan_id: 'plan1',
               sessions_per_week: 5,
               focus_areas: ['Algebra'],
               milestones: ['Complete 5 sessions in Algebra']
             }
           }));
         });

         test('displays granular study plan', async () => {
           const { findByText } = render(<StudyPlanScreen userData={{ sat_test_date: '2025-12-01T00:00:00Z' }} />);
           expect(await findByText('Complete 5 sessions in Algebra')).toBeTruthy();
         });

         test('allows updating study plan', async () => {
           const { findByText, getByText } = render(<StudyPlanScreen userData={{ sat_test_date: '2025-12-01T00:00:00Z' }} />);
           fireEvent.press(await findByText('Adjust Study Plan'));
           fireEvent.press(getByText('Save Changes'));
           expect(api.post).toHaveBeenCalledWith('/study_plan/create/user1', expect.any(Object));
         });
       });
       ```
3. **End-to-End Tests**:
   * **File**: `SATSmartPrepApp/e2e/onboarding.e2e.js` (update)
   *   **Code**:

       ```javascript
       describe('Onboarding Flow', () => {
         beforeEach(async () => {
           await device.reloadReactNative();
         });

         it('should complete onboarding with granular study plan', async () => {
           await element(by.text('Next')).tap();
           await element(by.id('full_name')).typeText('Alex Smith');
           await element(by.id('grade')).tap();
           await element(by.text('11th')).tap();
           await element(by.text('Next')).tap();
           await element(by.text('Yes')).tap();
           await element(by.id('sat_math_score')).typeText('650');
           await element(by.id('sat_reading_score')).typeText('600');
           await element(by.text('Next')).tap();
           await element(by.id('sat_test_date')).typeText('2025-12-01');
           await element(by.text('Next')).tap();
           await element(by.text('Start Test')).tap();
           // Simulate diagnostic test completion
           await element(by.text('Continue to Study Plan')).tap();
           await expect(element(by.text('Complete 5 sessions in Algebra'))).toBeVisible();
           await element(by.text('Continue to Dashboard')).tap();
           await expect(element(by.text('Dashboard'))).toBeVisible();
         });
       });
       ```

**Impact**

* **Score Improvement Guarantee**: Builds user trust by offering a tangible commitment to results, encouraging sign-ups and retention.
* **Granular Study Plans**: Improves personalization by focusing on weak skills, leading to better score improvements and user satisfaction.

These implementations ensure **SAT Smart Prep App** by **Learner Labs** continues to deliver value to users while maintaining a competitive edge. Let me know if you’d like to proceed with testing, deployment, or further enhancements!
