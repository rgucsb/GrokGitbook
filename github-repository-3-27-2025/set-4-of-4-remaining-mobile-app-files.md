# Set 4 of 4 Remaining mobile app files

Let’s continue with **Set 4** of the file creation for the **SAT Smart Prep App** by **Learner Labs**, focusing on the remaining mobile app files in `SATSmartPrepApp/`. I’ll complete the `PracticeScreen.js` file and include the rest of the screens, tests, and configuration files. All files will be the latest versions as of March 27, 2025, reflecting changes like the nudge modal, Bluebook-like UI, and device tracking.

***

### Set 4: Mobile App Files (`SATSmartPrepApp/`) - Continued

#### Screens (`SATSmartPrepApp/src/screens/`) - Continued

**`SATSmartPrepApp/src/screens/PracticeScreen.js` (Continued)**

* **Purpose**: Practice session screen.
* **Latest Version**: Includes nudge modal for full-length tests.
*   **Code** (Completing the file):

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
                    onPress={() => setShowNudgeModal(false)}
                  >
                    <Text style={commonStyles.buttonText}>I’ll Use the Web App</Text>
                  </TouchableOpacity>
                  <TouchableOpacity
                    style={[commonStyles.button, styles.mobileButton]}
                    onPress={proceedWithPractice}
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

**`SATSmartPrepApp/src/screens/StudyPlanScreen.js`**

* **Purpose**: Study plan screen.
* **Latest Version**: Supports granular study plans.
*   **Code**:

    ```javascript
    import React, { useState, useEffect } from 'react';
    import { View, Text, TextInput, Picker, TouchableOpacity, StyleSheet, Alert } from 'react-native';
    import Animated, { FadeIn, FadeOut } from 'react-native-reanimated';
    import { api } from '../utils/api';
    import { colors, typography, commonStyles } from '../styles';

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

    export default StudyPlanScreen;
    ```

**`SATSmartPrepApp/src/screens/DashboardScreen.js`**

* **Purpose**: Dashboard screen.
* **Latest Version**: Includes score guarantee check.
*   **Code**:

    ```javascript
    import React, { useState, useEffect } from 'react';
    import { View, Text, TextInput, TouchableOpacity, StyleSheet, Alert, ScrollView } from 'react-native';
    import Animated, { FadeIn, FadeOut } from 'react-native-reanimated';
    import { api } from '../utils/api';
    import { colors, typography, commonStyles } from '../styles';

    const DashboardScreen = ({ diagnosticResults }) => {
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

**`SATSmartPrepApp/src/screens/CommunityScreen.js`**

* **Purpose**: Community screen.
* **Latest Version**: No recent changes.
*   **Code**:

    ```javascript
    import React, { useState, useEffect } from 'react';
    import { View, Text, TextInput, TouchableOpacity, StyleSheet, Alert, ScrollView } from 'react-native';
    import Animated, { FadeIn, FadeOut } from 'react-native-reanimated';
    import { api } from '../utils/api';
    import { colors, typography, commonStyles } from '../styles';

    const CommunityScreen = () => {
      const [posts, setPosts] = useState([]);
      const [newPost, setNewPost] = useState('');
      const [friends, setFriends] = useState([]);
      const [teamName, setTeamName] = useState('');
      const userId = localStorage.getItem('user_id');

      useEffect(() => {
        api.get(`/social/posts`).then(res => setPosts(res.data));
        api.get(`/social/friends/${userId}`).then(res => setFriends(res.data.filter(f => f.status === 'accepted')));
      }, []);

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
        Alert.alert("Team created!");
      };

      const startChallenge = async (friendId) => {
        await api.post('/gamification/challenges/friend', { friendId });
        Alert.alert(`Challenge sent to ${friendId}!`);
      };

      return (
        <Animated.View entering={FadeIn} exiting={FadeOut} style={styles.container}>
          <ScrollView>
            <Text style={typography.heading}>Community</Text>
            <View style={styles.newPost}>
              <TextInput
                style={styles.postInput}
                value={newPost}
                onChangeText={setNewPost}
                placeholder="Share something with the community..."
                multiline
              />
              <TouchableOpacity style={commonStyles.button} onPress={createPost}>
                <Text style={commonStyles.buttonText}>Post</Text>
              </TouchableOpacity>
            </View>

            <View style={styles.posts}>
              {posts.map(post => (
                <View key={post.id} style={commonStyles.card}>
                  <Text style={typography.body}>{post.content}</Text>
                  <Text style={styles.timestamp}>Posted by {post.user_id} at {new Date(post.timestamp).toLocaleString()}</Text>
                </View>
              ))}
            </View>

            <View style={styles.communityChallenges}>
              <Text style={typography.heading}>Challenge a Friend</Text>
              {friends.map(friend => (
                <View key={friend.id} style={styles.friend}>
                  <Text style={typography.body}>{friend.friend_id}</Text>
                  <TouchableOpacity style={commonStyles.button} onPress={() => startChallenge(friend.friend_id)}>
                    <Text style={commonStyles.buttonText}>Challenge</Text>
                  </TouchableOpacity>
                </View>
              ))}
            </View>

            <View style={styles.teamLeagues}>
              <Text style={typography.heading}>Create a Team</Text>
              <TextInput
                style={styles.teamInput}
                value={teamName}
                onChangeText={setTeamName}
                placeholder="Team Name"
              />
              <TouchableOpacity style={commonStyles.button} onPress={createTeam}>
                <Text style={commonStyles.buttonText}>Create Team</Text>
              </TouchableOpacity>
            </View>
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
      newPost: {
        marginBottom: 20,
      },
      postInput: {
        width: '100%',
        height: 100,
        marginBottom: 10,
        padding: 10,
        borderWidth: 1,
        borderColor: colors.gray,
        borderRadius: 5,
      },
      posts: {
        marginBottom: 20,
      },
      timestamp: {
        fontSize: 12,
        color: '#6b7280',
        marginTop: 5,
      },
      communityChallenges: {
        marginVertical: 20,
      },
      friend: {
        flexDirection: 'row',
        alignItems: 'center',
        justifyContent: 'space-between',
        padding: 10,
        borderWidth: 1,
        borderColor: colors.gray,
        borderRadius: 5,
        marginBottom: 10,
      },
      teamLeagues: {
        marginVertical: 20,
      },
      teamInput: {
        width: '100%',
        padding: 8,
        marginBottom: 10,
        borderWidth: 1,
        borderColor: colors.gray,
        borderRadius: 5,
      },
    });

    export default CommunityScreen;
    ```

**`SATSmartPrepApp/src/screens/LeaderboardScreen.js`**

* **Purpose**: Leaderboard screen.
* **Latest Version**: No recent changes.
*   **Code**:

    ```javascript
    import React, { useState, useEffect } from 'react';
    import { View, Text, Picker, TouchableOpacity, StyleSheet, ScrollView } from 'react-native';
    import Animated, { FadeIn, FadeOut } from 'react-native-reanimated';
    import { api } from '../utils/api';
    import { colors, typography, commonStyles } from '../styles';

    const LeaderboardScreen = () => {
      const [skill, setSkill] = useState('Algebra');
      const [globalLeaderboard, setGlobalLeaderboard] = useState([]);
      const [friendsLeaderboard, setFriendsLeaderboard] = useState([]);
      const [teamLeaderboard, setTeamLeaderboard] = useState([]);
      const [tab, setTab] = useState('global');
      const userId = localStorage.getItem('user_id');

      useEffect(() => {
        api.get(`/leaderboards/${skill}?user_id=${userId}`).then(res => {
          setGlobalLeaderboard(res.data.global);
          setFriendsLeaderboard(res.data.friends);
        });
        api.get('/teams/leaderboard').then(res => setTeamLeaderboard(res.data));
      }, [skill]);

      return (
        <Animated.View entering={FadeIn} exiting={FadeOut} style={styles.container}>
          <ScrollView>
            <Text style={typography.heading}>Leaderboard</Text>
            <Picker
              selectedValue={skill}
              onValueChange={(value) => setSkill(value)}
              style={styles.select}
            >
              <Picker.Item label="Algebra" value="Algebra" />
              <Picker.Item label="Geometry" value="Geometry" />
              <Picker.Item label="Reading Comprehension" value="Reading Comprehension" />
            </Picker>

            <View style={styles.tabs}>
              <TouchableOpacity
                onPress={() => setTab('global')}
                style={[styles.tabButton, tab === 'global' && styles.tabButtonActive]}
              >
                <Text style={tab === 'global' ? styles.tabTextActive : styles.tabText}>Global</Text>
              </TouchableOpacity>
              <TouchableOpacity
                onPress={() => setTab('friends')}
                style={[styles.tabButton, tab === 'friends' && styles.tabButtonActive]}
              >
                <Text style={tab === 'friends' ? styles.tabTextActive : styles.tabText}>Friends</Text>
              </TouchableOpacity>
              <TouchableOpacity
                onPress={() => setTab('teams')}
                style={[styles.tabButton, tab === 'teams' && styles.tabButtonActive]}
              >
                <Text style={tab === 'teams' ? styles.tabTextActive : styles.tabText}>Teams</Text>
              </TouchableOpacity>
            </View>

            {tab === 'global' && (
              <View style={styles.leaderboardSection}>
                <Text style={typography.heading}>{skill} Leaderboard (Global)</Text>
                {globalLeaderboard.map((user, idx) => (
                  <View key={user.user_id} style={styles.leaderboardEntry}>
                    <Text style={typography.body}>{idx + 1}</Text>
                    <Text style={typography.body}>{user.email}</Text>
                    <Text style={typography.body}>{user.score}</Text>
                    {idx < 3 && <Text style={typography.body}>🏅</Text>}
                  </View>
                ))}
              </View>
            )}

            {tab === 'friends' && (
              <View style={styles.leaderboardSection}>
                <Text style={typography.heading}>{skill} Leaderboard (Friends)</Text>
                {friendsLeaderboard.map((user, idx) => (
                  <View key={user.user_id} style={styles.leaderboardEntry}>
                    <Text style={typography.body}>{idx + 1}</Text>
                    <Text style={typography.body}>{user.email}</Text>
                    <Text style={typography.body}>{user.score}</Text>
                    {idx < 3 && <Text style={typography.body}>🏅</Text>}
                  </View>
                ))}
              </View>
            )}

            {tab === 'teams' && (
              <View style={styles.leaderboardSection}>
                <Text style={typography.heading}>Team Leaderboard</Text>
                {teamLeaderboard.map((team, idx) => (
                  <View key={team.id} style={styles.leaderboardEntry}>
                    <Text style={typography.body}>{idx + 1}</Text>
                    <Text style={typography.body}>{team.team_name}</Text>
                    <Text style={typography.body}>{team.points}</Text>
                    {idx < 3 && <Text style={typography.body}>🏅</Text>}
                  </View>
                ))}
              </View>
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
      select: {
        width: '100%',
        padding: 8,
        marginVertical: 10,
        borderWidth: 1,
        borderColor: colors.gray,
        borderRadius: 5,
      },
      tabs: {
        flexDirection: 'row',
        gap: 10,
        marginBottom: 10,
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
      leaderboardSection: {
        marginTop: 10,
      },
      leaderboardEntry: {
        flexDirection: 'row',
        alignItems: 'center',
        gap: 10,
        padding: 10,
        borderBottomWidth: 1,
        borderBottomColor: colors.gray,
      },
    });

    export default LeaderboardScreen;
    ```

**`SATSmartPrepApp/src/screens/RewardsStoreScreen.js`**

* **Purpose**: Rewards store screen.
* **Latest Version**: No recent changes.
*   **Code**:

    ```javascript
    import React, { useState, useEffect } from 'react';
    import { View, Text, Image, TouchableOpacity, StyleSheet, Alert, ScrollView } from 'react-native';
    import Animated, { FadeIn, FadeOut } from 'react-native-reanimated';
    import { api } from '../utils/api';
    import { colors, typography, commonStyles } from '../styles';

    const RewardsStoreScreen = () => {
      const [coins, setCoins] = useState(0);
      const [rewards, setRewards] = useState([]);
      const userId = localStorage.getItem('user_id');

      useEffect(() => {
        api.get(`/gamification/coins/${userId}`).then(res => setCoins(res.data.coins));
        api.get('/rewards/available').then(res => setRewards(res.data));
      }, []);

      const unlockReward = async (rewardId, cost, rewardType) => {
        if (coins < cost) {
          Alert.alert("Not enough coins!");
          return;
        }
        await api.post('/rewards/unlock', { user_id: userId, reward_id: rewardId, reward_type: rewardType, cost });
        setCoins(coins - cost);
        Alert.alert(`Unlocked ${rewardId}!`);
      };

      return (
        <Animated.View entering={FadeIn} exiting={FadeOut} style={styles.container}>
          <ScrollView>
            <Text style={typography.heading}>Rewards Store</Text>
            <Text style={typography.body}>Coins: {coins} 🪙</Text>
            <View style={styles.rewardsGrid}>
              {rewards.map(reward => (
                <View key={reward.id} style={styles.rewardItem}>
                  <Image source={{ uri: reward.image }} style={styles.rewardImage} />
                  <Text style={typography.body}>{reward.name}</Text>
                  <Text style={typography.body}>{reward.cost} Coins</Text>
                  <TouchableOpacity
                    style={commonStyles.button}
                    onPress={() => unlockReward(reward.id, reward.cost, reward.type)}
                  >
                    <Text style={commonStyles.buttonText}>Unlock</Text>
                  </TouchableOpacity>
                </View>
              ))}
            </View>
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
      rewardsGrid: {
        flexDirection: 'row',
        flexWrap: 'wrap',
        gap: 20,
        marginTop: 20,
      },
      rewardItem: {
        borderWidth: 1,
        borderColor: colors.gray,
        borderRadius: 5,
        padding: 10,
        width: '45%',
        alignItems: 'center',
      },
      rewardImage: {
        width: '100%',
        height: 150,
        borderRadius: 5,
      },
    });

    export default RewardsStoreScreen;
    ```

**`SATSmartPrepApp/src/screens/PolicyScreen.js`**

* **Purpose**: Score guarantee policy screen.
* **Latest Version**: Updated with starting score condition (1250 or lower).
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
              At Learner Labs, we are committed to helping you achieve your best SAT score with SAT Smart Prep App. We guarantee a 150-point score increase on your SAT practice tests if you meet the following conditions:
            </Text>
            <Text style={typography.body}>
              1. **Starting Score of 1250 or Lower**: You must have a starting score of 1250 or lower, as determined by either your first full-length SAT practice test (diagnostic test) within the app or your previous SAT or PSAT score entered during onboarding. If you do not provide a previous score, the app will use your first diagnostic test score to determine eligibility.
            </Text>
            <Text style={typography.body}>
              2. **Complete At Least Two Full-Length SAT Practice Tests**: You must complete at least two full-length SAT practice tests within the app. These tests must be taken under timed conditions to accurately reflect your progress. The first test establishes your baseline score, and the second (or later) test is used to measure your improvement.
            </Text>
            <Text style={typography.body}>
              3. **Follow the Recommended Study Plan for 30 Days**: You must actively follow the app’s recommended study plan for at least 30 consecutive days after completing your first full-length test. This includes completing at least 80% of the assigned tasks in your granular study plan (e.g., practice sessions, milestones) during this period.
            </Text>
            <Text style={typography.body}>
              4. **Achieve a 150-Point Increase**: The guarantee applies to the difference between your first full-length SAT practice test score and your highest subsequent full-length SAT practice test score within the app. If your score does not increase by at least 150 points, you are eligible for a refund.
            </Text>
            <Text style={typography.body}>
              5. **Submit a Refund Request Within 60 Days**: You must submit a refund request within 60 days of your subscription start date (or the date of your first full-length test, whichever is later). To request a refund, contact our support team at support@learnerlabs.com with your account details and a summary of your study activity.
            </Text>
            <Text style={typography.body}>
              6. **Subscription Requirement**: The guarantee applies only to users with an active paid subscription (monthly or annual) to SAT Smart Prep App. Free trial users or users in promotional programs (e.g., College Board pilot) are not eligible unless they upgrade to a paid plan.
            </Text>
            <Text style={typography.body}>
              **Additional Notes**:
              - **Practice Test Scores Only**: This guarantee applies only to SAT practice test scores within the app and does not apply to official SAT scores reported by the College Board.
              - **Refund Processing**: Refunds are processed within 14 business days of approval. The refund amount will be the full subscription fee paid, minus any applicable taxes or fees.
              - **Fair Use**: Learner Labs reserves the right to deny refund requests if there is evidence of misuse, such as not following the study plan, skipping questions, or attempting to manipulate scores.
            </Text>
            <Text style={typography.body}>
              **How to Check Eligibility**:
              Go to the Dashboard in the app and click "Check Score Guarantee" to see your score improvement and eligibility status. If eligible, follow the instructions to contact support for your refund.
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

#### Utilities (`SATSmartPrepApp/src/utils/`)

**`SATSmartPrepApp/src/utils/api.js`**

* **Purpose**: API client with offline support and WebSocket.
* **Latest Version**: No recent changes.
*   **Code**:

    ```javascript
    import axios from 'axios';
    import AsyncStorage from '@react-native-async-storage/async-storage';

    const api = axios.create({
      baseURL: 'http://localhost:8000',
    });

    api.interceptors.request.use(async (config) => {
      const token = await AsyncStorage.getItem('token');
      if (token) {
        config.headers.Authorization = `Bearer ${token}`;
      }
      return config;
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

#### Context (`SATSmartPrepApp/src/context/`)

**`SATSmartPrepApp/src/context/AppContext.js`**

* **Purpose**: App-wide context for state management.
*   **Code**:

    ```javascript
    import React, { createContext, useState } from 'react';

    export const AppContext = createContext();

    export const AppProvider = ({ children }) => {
      const [theme, setTheme] = useState('light');

      return (
        <AppContext.Provider value={{ theme, setTheme }}>
          {children}
        </AppContext.Provider>
      );
    };
    ```

#### Assets (`SATSmartPrepApp/src/assets/`)

**`SATSmartPrepApp/src/assets/images/`, `SATSmartPrepApp/src/assets/icons/`, `SATSmartPrepApp/src/assets/splash/`**

* **Purpose**: Placeholder directories for images, icons, and splash screens.
* **Note**: These are placeholders. In a real project, you would include the actual assets (e.g., app icon, splash screen, reward images).

#### Navigation (`SATSmartPrepApp/src/navigation/`)

**`SATSmartPrepApp/src/navigation/index.js`**

* **Purpose**: Navigation setup using React Navigation.
*   **Code**:

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
          </Stack.Navigator>
        </NavigationContainer>
      );
    };

    export default AppNavigator;
    ```

#### Styles (`SATSmartPrepApp/src/styles/`)

**`SATSmartPrepApp/src/styles/styles.js`**

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
        fontSize: 24,
        fontWeight: 'bold',
        marginBottom: 10,
      },
      body: {
        fontSize: 16,
        lineHeight: 24,
      },
    };

    export const commonStyles = {
      button: {
        paddingVertical: 10,
        paddingHorizontal: 20,
        backgroundColor: colors.primary,
        borderRadius: 5,
      },
      buttonText: {
        color: colors.white,
        fontSize: 16,
      },
      card: {
        borderWidth: 1,
        borderColor: colors.gray,
        borderRadius: 5,
        padding: 10,
        marginBottom: 10,
      },
    };
    ```

#### Configuration (`SATSmartPrepApp/src/config/`)

**`SATSmartPrepApp/src/config/integrations.js`**

* **Purpose**: Centralizes API keys and configuration for mobile integrations.
* **Latest Version**: Created in previous steps.
*   **Code**:

    ```javascript
    export const integrations = {
      firebase: {
        apiKey: process.env.FIREBASE_API_KEY,
        authDomain: process.env.FIREBASE_AUTH_DOMAIN,
        projectId: process.env.FIREBASE_PROJECT_ID,
        storageBucket: process.env.FIREBASE_STORAGE_BUCKET,
        messagingSenderId: process.env.FIREBASE_MESSAGING_SENDER_ID,
        appId: process.env.FIREBASE_APP_ID,
      },
      googleSSO: {
        clientId: process.env.GOOGLE_CLIENT_ID,
        redirectUrl: 'com.satsmartprepapp:/oauthredirect',
        scopes: ['email', 'profile'],
        serviceConfiguration: {
          authorizationEndpoint: 'https://accounts.google.com/o/oauth2/v2/auth',
          tokenEndpoint: 'https://oauth2.googleapis.com/token',
        },
      },
      bluebook: {
        enabled: false,
      },
      khanAcademy: {
        enabled: false,
      },
      canvasLMS: {
        enabled: false,
      },
    };
    ```

#### Unit Tests (`SATSmartPrepApp/__tests__/`)

**`SATSmartPrepApp/__tests__/QuestionDisplay.test.js`**

* **Purpose**: Tests the QuestionDisplay component.
*   **Code**:

    ```javascript
    import { render, fireEvent } from '@testing-library/react-native';
    import QuestionDisplay from '../src/components/QuestionDisplay';

    describe('QuestionDisplay', () => {
      test('renders question and options', () => {
        const { getByText } = render(<QuestionDisplay questionNumber={1} eliminatorActive={false} toggleEliminator={() => {}} />);
        expect(getByText('The student wants to emphasize a similarity between the two works. Which choice most effectively uses relevant information from the notes to accomplish this goal?')).toBeTruthy();
        expect(getByText('A.')).toBeTruthy();
        expect(getByText('B.')).toBeTruthy();
        expect(getByText('C.')).toBeTruthy();
        expect(getByText('D.')).toBeTruthy();
      });

      test('selects an answer when eliminator is off', () => {
        const { getByText } = render(<QuestionDisplay questionNumber={1} eliminatorActive={false} toggleEliminator={() => {}} />);
        fireEvent.press(getByText('A.'));
        expect(getByText('A.').parent).toHaveStyle({ backgroundColor: 'rgba(37, 99, 235, 0.1)' });
      });
    });
    ```

**`SATSmartPrepApp/__tests__/PassageDisplay.test.js`**

* **Purpose**: Tests the PassageDisplay component.
*   **Code**:

    ```javascript
    import { render, fireEvent } from '@testing-library/react-native';
    import PassageDisplay from '../src/components/PassageDisplay';

    describe('PassageDisplay', () => {
      test('toggles highlight on passage', () => {
        const { getByText } = render(<PassageDisplay passageText="Test passage" />);
        const passage = getByText('Test passage');
        expect(passage).not.toHaveStyle({ backgroundColor: 'yellow' });
        fireEvent.press(getByText('Highlight'));
        expect(passage).toHaveStyle({ backgroundColor: 'yellow' });
        fireEvent.press(getByText('Remove Highlight'));
        expect(passage).not.toHaveStyle({ backgroundColor: 'yellow' });
      });
    });
    ```

**`SATSmartPrepApp/__tests__/AnswerEliminator.test.js`**

* **Purpose**: Tests the AnswerEliminator component.
*   **Code**:

    ```javascript
    import { render, fireEvent } from '@testing-library/react-native';
    import AnswerEliminator from '../src/components/AnswerEliminator';

    describe('AnswerEliminator', () => {
      test('toggles eliminator state', () => {
        const toggleEliminator = jest.fn();
        const { getByText } = render(<AnswerEliminator active={false} onToggle={toggleEliminator} />);
        fireEvent.press(getByText('ABC Off'));
        expect(toggleEliminator).toHaveBeenCalled();
      });
    });
    ```

**`SATSmartPrepApp/__tests__/MathQuestionHeader.test.js`**

* **Purpose**: Tests the MathQuestionHeader component.
*   **Code**:

    ```javascript
    import { render, fireEvent } from '@testing-library/react-native';
    import MathQuestionHeader from '../src/components/MathQuestionHeader';

    describe('MathQuestionHeader', () => {
      test('displays question number and timer', () => {
        const { getByText } = render(<MathQuestionHeader questionNumber={1} totalQuestions={10} timer="15:00" />);
        expect(getByText('1/10')).toBeTruthy();
        expect(getByText('15:00')).toBeTruthy();
      });

      test('shows calculator button when enabled', () => {
        const { getByText } = render(<MathQuestionHeader questionNumber={1} totalQuestions={10} timer="15:00" showCalculator={true} />);
        expect(getByText('Calc')).toBeTruthy();
      });
    });
    ```

**`SATSmartPrepApp/__tests__/OnboardingScreen.test.js`**

* **Purpose**: Tests the OnboardingScreen.
*   **Code**:

    ```javascript
    import { render, fireEvent, waitFor } from '@testing-library/react-native';
    import OnboardingScreen from '../src/screens/OnboardingScreen';
    import { api } from '../utils/api';
    import AsyncStorage from '@react-native-async-storage/async-storage';

    jest.mock('../utils/api');
    jest.mock('@react-native-async-storage/async-storage');

    describe('OnboardingScreen', () => {
      beforeEach(() => {
        localStorage.setItem('user_id', 'user1');
        AsyncStorage.getItem.mockResolvedValue('user1');
        api.put.mockImplementation(() => Promise.resolve({}));
        api.post.mockImplementation(() => Promise.resolve({}));
      });

      test('progresses through onboarding steps', async () => {
        const navigation = { navigate: jest.fn() };
        const { getByText, getByPlaceholderText } = render(<OnboardingScreen navigation={navigation} />);

        // Welcome Step
        fireEvent.press(getByText('Next'));

        // Basic Info Step
        expect(getByText('Basic Information')).toBeTruthy();
        fireEvent.changeText(getByPlaceholderText('Enter your full name'), 'Alex Smith');
        fireEvent.press(getByText('Next'));

        // SAT Experience Step
        expect(getByText('SAT Experience')).toBeTruthy();
        fireEvent.press(getByText('Yes'));

        // SAT Score Step
        expect(getByText('Previous SAT Scores')).toBeTruthy();
        fireEvent.changeText(getByPlaceholderText('Enter your Math score'), '650');
        fireEvent.changeText(getByPlaceholderText('Enter your Reading & Writing score'), '600');
        fireEvent.press(getByText('Next'));

        // Study Preferences Step
        expect(getByText('Study Preferences')).toBeTruthy();
        fireEvent.press(getByText('Next'));

        // Diagnostic Step
        expect(getByText('Take a Diagnostic Test')).toBeTruthy();
      });
    });
    ```

**`SATSmartPrepApp/__tests__/PracticeScreen.test.js`**

* **Purpose**: Tests the PracticeScreen.
* **Latest Version**: Includes tests for nudge modal.
*   **Code**:

    ```javascript
    import { render, fireEvent, waitFor } from '@testing-library/react-native';
    import PracticeScreen from '../src/screens/PracticeScreen';
    import { api, connectWebSocket } from '../utils/api';
    import AsyncStorage from '@react-native-async-storage/async-storage';

    jest.mock('../utils/api');
    jest.mock('@react-native-async-storage/async-storage');
    jest.mock('@react-native-voice/voice', () => ({
      onSpeechResults: jest.fn(),
      start: jest.fn(),
      stop: jest.fn(),
      destroy: jest.fn().mockResolvedValue(),
      removeAllListeners: jest.fn(),
    }));

    describe('PracticeScreen', () => {
      beforeEach(() => {
        localStorage.setItem('user_id', 'user1');
        AsyncStorage.getItem.mockResolvedValue('user1');
        connectWebSocket.mockImplementation(() => ({ close: jest.fn() }));
        api.post.mockImplementation(() => Promise.resolve({
          data: {
            session_id: 'session1',
            questions: [{ id: 'q1', domain: 'Math', test_type: 'SAT' }],
          },
        }));
      });

      test('displays nudge modal for full-length test', async () => {
        const navigation = { navigate: jest.fn() };
        const { getByText } = render(<PracticeScreen navigation={navigation} />);
        
        fireEvent.changeText(getByText('Short Practice (10 questions)'), 44);
        fireEvent.press(getByText('Start'));

        await waitFor(() => {
          expect(getByText('Take Full-Length Tests on the Web App')).toBeTruthy();
          expect(getByText('I’ll Use the Web App')).toBeTruthy();
          expect(getByText('Continue on Mobile')).toBeTruthy();
        });
      });

      test('closes nudge modal and does not start test when choosing web app', async () => {
        const navigation = { navigate: jest.fn() };
        const { getByText, queryByText } = render(<PracticeScreen navigation={navigation} />);
        
        fireEvent.changeText(getByText('Short Practice (10 questions)'), 44);
        fireEvent.press(getByText('Start'));

        await waitFor(() => {
          fireEvent.press(getByText('I’ll Use the Web App'));
        });

        expect(queryByText('Take Full-Length Tests on the Web App')).toBeNull();
        expect(api.post).not.toHaveBeenCalled();
      });

      test('proceeds with test on mobile when user insists', async () => {
        const navigation = { navigate: jest.fn() };
        const { getByText, queryByText } = render(<PracticeScreen navigation={navigation} />);
        
        fireEvent.changeText(getByText('Short Practice (10 questions)'), 44);
        fireEvent.press(getByText('Start'));

        await waitFor(() => {
          fireEvent.press(getByText('Continue on Mobile'));
        });

        expect(queryByText('Take Full-Length Tests on the Web App')).toBeNull();
        expect(api.post).toHaveBeenCalledWith('/practice/start/user1', {
          domain: 'Math',
          num_questions: 44,
          test_type: 'SAT',
          device: 'mobile',
        });
      });

      test('does not show nudge modal for short practice', async () => {
        const navigation = { navigate: jest.fn() };
        const { getByText, queryByText } = render(<PracticeScreen navigation={navigation} />);
        
        fireEvent.changeText(getByText('Short Practice (10 questions)'), 10);
        fireEvent.press(getByText('Start'));

        await waitFor(() => {
          expect(queryByText('Take Full-Length Tests on the Web App')).toBeNull();
          expect(api.post).toHaveBeenCalledWith('/practice/start/user1', {
            domain: 'Math',
            num_questions: 10,
            test_type: 'SAT',
            device: 'mobile',
          });
        });
      });
    });
    ```

**`SATSmartPrepApp/__tests__/DashboardScreen.test.js`**

* **Purpose**: Tests the DashboardScreen.
* **Latest Version**: Includes score guarantee test.
*   **Code**:

    ```javascript
    import { render, fireEvent, waitFor } from '@testing-library/react-native';
    import DashboardScreen from '../src/screens/DashboardScreen';
    import { api } from '../utils/api';

    jest.mock('../utils/api');

    describe('DashboardScreen', () => {
      beforeEach(() => {
        localStorage.setItem('user_id', 'user1');
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
        const { getByText } = render(<DashboardScreen />);
        await waitFor(() => {
          expect(getByText('Coins: 100 🪙')).toBeTruthy();
        });
      });

      test('checks score guarantee', async () => {
        const navigation = { navigate: jest.fn() };
        const { getByText } = render(<DashboardScreen navigation={navigation} />);
        fireEvent.press(getByText('Check Score Guarantee'));
        await waitFor(() => {
          expect(api.get).toHaveBeenCalledWith('/progress_monitoring/guarantee/user1');
        });
      });
    });
    ```

**`SATSmartPrepApp/__tests__/CommunityScreen.test.js`**

* **Purpose**: Tests the CommunityScreen.
*   **Code**:

    ```javascript
    import { render, fireEvent, waitFor } from '@testing-library/react-native';
    import CommunityScreen from '../src/screens/CommunityScreen';
    import { api } from '../utils/api';

    jest.mock('../utils/api');

    describe('CommunityScreen', () => {
      beforeEach(() => {
        localStorage.setItem('user_id', 'user1');
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
        const { getByPlaceholderText, getByText } = render(<CommunityScreen />);
        fireEvent.changeText(getByPlaceholderText('Share something with the community...'), 'Hello!');
        fireEvent.press(getByText('Post'));
        await waitFor(() => {
          expect(api.post).toHaveBeenCalledWith('/social/posts', { user_id: 'user1', content: 'Hello!' });
        });
      });
    });
    ```

**`SATSmartPrepApp/__tests__/LeaderboardScreen.test.js`**

* **Purpose**: Tests the LeaderboardScreen.
*   **Code**:

    ```javascript
    import { render, fireEvent, waitFor } from '@testing-library/react-native';
    import LeaderboardScreen from '../src/screens/LeaderboardScreen';
    import { api } from '../utils/api';

    jest.mock('../utils/api');

    describe('LeaderboardScreen', () => {
      beforeEach(() => {
        localStorage.setItem('user_id', 'user1');
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
        const { getByText } = render(<LeaderboardScreen />);
        await waitFor(() => {
          expect(getByText('Algebra Leaderboard (Global)')).toBeTruthy();
          expect(getByText('user1@example.com')).toBeTruthy();
        });
      });

      test('switches to friends tab', async () => {
        const { getByText } = render(<LeaderboardScreen />);
        fireEvent.press(getByText('Friends'));
        await waitFor(() => {
          expect(getByText('Algebra Leaderboard (Friends)')).toBeTruthy();
        });
      });
    });
    ```

**`SATSmartPrepApp/__tests__/RewardsStoreScreen.test.js`**

* **Purpose**: Tests the RewardsStoreScreen.
*   **Code**:

    ```javascript
    import { render, fireEvent, waitFor } from '@testing-library/react-native';
    import RewardsStoreScreen from '../src/screens/RewardsStoreScreen';
    import { api } from '../utils/api';

    jest.mock('../utils/api');

    describe('RewardsStoreScreen', () => {
      beforeEach(() => {
        localStorage.setItem('user_id', 'user1');
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
        const { getByText } = render(<RewardsStoreScreen />);
        await waitFor(() => {
          expect(getByText('Math Prodigy Badge')).toBeTruthy();
          expect(getByText('100 Coins')).toBeTruthy();
        });

        fireEvent.press(getByText('Unlock'));
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

**`SATSmartPrepApp/__tests__/StudyPlanScreen.test.js`**

* **Purpose**: Tests the StudyPlanScreen.
*   **Code**:

    ```javascript
    import { render, fireEvent, waitFor } from '@testing-library/react-native';
    import StudyPlanScreen from '../src/screens/StudyPlanScreen';
    import { api } from '../utils/api';

    jest.mock('../utils/api');

    describe('StudyPlanScreen', () => {
      beforeEach(() => {
        localStorage.setItem('user_id', 'user1');
        api.get.mockImplementation(() => Promise.resolve({
          data: {
            plan_id: 'plan1',
            test_date: new Date().toISOString(),
            actions: [],
          },
        }));
        api.post.mockImplementation(() => Promise.resolve({
          data: {
            plan_id: 'plan1',
            milestones: [],
          },
        }));
      });

      test('displays study plan', async () => {
        const userData = { sat_test_date: new Date().toISOString(), study_hours_per_week: 10, study_days_per_week: 5 };
        const { getByText } = render(<StudyPlanScreen userData={userData} />);
        await waitFor(() => {
          expect(getByText('Your Study Plan')).toBeTruthy();
        });
      });
    });
    ```

#### E2E Tests (`SATSmartPrepApp/e2e/`)

**`SATSmartPrepApp/e2e/onboarding.e2e.js`**

* **Purpose**: E2E tests for the onboarding flow.
*   **Code**:

    ```javascript
    describe('Onboarding Flow', () => {
      beforeEach(async () => {
        await device.reloadReactNative();
      });

      it('should progress through onboarding steps', async () => {
        await element(by.text('Next')).tap();
        await element(by.text('Basic Information')).toBeVisible();
        await element(by.text('Next')).tap();
        await element(by.text('SAT Experience')).toBeVisible();
        await element(by.text('Yes')).tap();
        await element(by.text('Previous SAT Scores')).toBeVisible();
        await element(by.text('Next')).tap();
        await element(by.text('Study Preferences')).toBeVisible();
        await element(by.text('Next')).tap();
        await element(by.text('Take a Diagnostic Test')).toBeVisible();
      });
    });
    ```

**`SATSmartPrepApp/e2e/practice.e2e.js`**

* **Purpose**: E2E tests for the practice flow.
* **Latest Version**: Includes tests for nudge modal.
*   **Code**:

    ```javascript
    describe('Practice Flow', () => {
      beforeEach(async () => {
        await device.reloadReactNative();
      });

      it('should show nudge modal for full-length test and allow switching to web app', async () => {
        await element(by.text('Practice')).tap();
        await element(by.text('Short Practice (10 questions)')).tap();
        await element(by.text('Full-Length Test (44-98 questions)')).tap();
        await element(by.text('Start')).tap();
        await expect(element(by.text('Take Full-Length Tests on the Web App'))).toBeVisible();
        await expect(element(by.text('I’ll Use the Web App'))).toBeVisible();
        await expect(element(by.text('Continue on Mobile'))).toBeVisible();
        await element(by.text('I’ll Use the Web App')).tap();
        await expect(element(by.text('Take Full-Length Tests on the Web App'))).not.toBeVisible();
        await expect(element(by.text('SAT Smart Prep App - Full-Length Test'))).not.toBeVisible();
      });

      it('should allow continuing full-length test on mobile after nudge', async () => {
        await element(by.text('Practice')).tap();
        await element(by.text('Short Practice (10 questions)')).tap();
        await element(by.text('Full-Length Test (44-98 questions)')).tap();
        await element(by.text('Start')).tap();
        await expect(element(by.text('Take Full-Length Tests on the Web App'))).toBeVisible();
        await element(by.text('Continue on Mobile')).tap();
        await expect(element(by.text('Take Full-Length Tests on the Web App'))).not.toBeVisible();
        await expect(element(by.text('SAT Smart Prep App - Full-Length Test'))).toBeVisible();
      });
    });
    ```

**`SATSmartPrepApp/e2e/community.e2e.js`**

* **Purpose**: E2E tests for the community flow.
*   **Code**:

    ```javascript
    describe('Community Flow', () => {
      beforeEach(async () => {
        await device.reloadReactNative();
      });

      it('should create a new post', async () => {
        await element(by.text('Community')).tap();
        await element(by.text('Share something with the community...')).typeText('Hello, world!');
        await element(by.text('Post')).tap();
        await expect(element(by.text('Hello, world!'))).toBeVisible();
      });
    });
    ```

#### Android Configuration (`SATSmartPrepApp/android/`)

**`SATSmartPrepApp/android/app/build.gradle`**

* **Purpose**: Android app build configuration.
*   **Code**:

    ```gradle
    apply plugin: "com.android.application"
    apply plugin: "com.google.gms.google-services"

    android {
        compileSdkVersion 31
        defaultConfig {
            applicationId "com.satsmartprepapp"
            minSdkVersion 23
            targetSdkVersion 31
            versionCode 1
            versionName "1.0"
        }
        buildTypes {
            release {
                minifyEnabled false
                proguardFiles getDefaultProguardFile("proguard-android-optimize.txt"), "proguard-rules.pro"
            }
        }
    }

    dependencies {
        implementation fileTree(dir: "libs", include: ["*.jar"])
        implementation "androidx.appcompat:appcompat:1.3.0"
        implementation "com.google.firebase:firebase-messaging:23.0.0"
        implementation project(':react-native-voice')
        implementation project(':react-native-background-fetch')
        implementation project(':react-native-gesture-handler')
        implementation project(':react-native-reanimated')
    }
    ```

**`SATSmartPrepApp/android/build.gradle`**

* **Purpose**: Project-level build configuration.
*   **Code**:

    ```gradle
    buildscript {
        repositories {
            google()
            mavenCentral()
        }
        dependencies {
            classpath("com.android.tools.build:gradle:7.0.4")
            classpath("com.google.gms:google-services:4.3.10")
        }
    }

    allprojects {
        repositories {
            google()
            mavenCentral()
        }
    }
    ```

**`SATSmartPrepApp/android/gradlew`**

* **Purpose**: Gradle wrapper script.
* **Note**: This is a placeholder. In a real project, you would include the actual `gradlew` script.

#### iOS Configuration (`SATSmartPrepApp/ios/`)

**`SATSmartPrepApp/ios/SATSmartPrepApp/Info.plist`**

* **Purpose**: iOS app configuration.
*   **Code**:

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0
    ```
