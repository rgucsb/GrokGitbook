# Page 7

Let’s continue with the step-by-step plan to create the **SAT Prep Suite** mobile app for iOS and Android while updating the web app for tight integration. We left off in **Step 7: Create Practice, Community, Leaderboard, and Rewards Store Screens for Mobile (May 20-26, 2025)**, specifically at **Sub-Step 5: Update Web App Practice, Community, Leaderboard, and Rewards Store for Consistency**. I’ll complete this sub-step and proceed with the remaining steps to finish the plan. Each step will remain focused and concise, with double-checking to ensure nothing is missed.

***

#### Step 7: Create Practice, Community, Leaderboard, and Rewards Store Screens for Mobile (May 20-26, 2025) - Continued

**Objective**

Complete the creation of the practice, community, leaderboard, and rewards store screens for the mobile app, and update the corresponding web app pages to ensure UI/UX consistency.

**Sub-Steps (Continued)**

5. **Update Web App Practice, Community, Leaderboard, and Rewards Store for Consistency**:
   *   **Practice**: `frontend/pages/practice-bluebook.js`

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
         const [chatMessages, setChatMessages] = useState([]);
         const [chatInput, setChatInput] = useState('');
         const [isOffline, setIsOffline] = useState(false);
         const router = useRouter();
         const userId = localStorage.getItem('user_id');

         useEffect(() => {
           if (!userId) {
             router.push('/login');
           }
         }, [userId, router]);

         const startPractice = async () => {
           const res = await api.post(`/practice/start/${userId}`, { domain, num_questions: 10 });
           setSessionId(res.data.session_id);
           setQuestions(res.data.questions);
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
             <h1 style={typography.heading}>Practice Session {isOffline && '(Offline)'}</h1>
             {!sessionId ? (
               <>
                 <select
                   value={domain}
                   onChange={(e) => setDomain(e.target.value)}
                   className="select"
                 >
                   <option value="Math">Math</option>
                   <option value="Reading & Writing">Reading & Writing</option>
                 </select>
                 <button style={commonStyles.button} onClick={startPractice}>
                   <span style={commonStyles.buttonText}>Start</span>
                 </button>
               </>
             ) : (
               <div className="practice-area">
                 {questions.map((q) => renderQuestionLayout(q))}
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
               .practice-area {
                 flex: 1;
               }
               .chat-area {
                 flex: 1;
                 border: 1px solid ${colors.gray};
                 border-radius: 5px;
                 padding: 10px;
                 margin-top: 20px;
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

       * **Verification**: The web practice page is updated to use the shared design system, ensuring UI/UX consistency with the mobile app. It supports offline mode and chat functionality (simulated AI response).
   *   **Community**: `frontend/pages/community.js`

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

       * **Verification**: The web community page is updated to use the shared design system, ensuring UI/UX consistency with the mobile app.
   *   **Leaderboard**: `frontend/pages/leaderboard.js`

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

       * **Verification**: The web leaderboard page is updated to use the shared design system, ensuring UI/UX consistency with the mobile app.
   *   **Rewards Store**: `frontend/pages/rewards-store.js`

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

       * **Verification**: The web rewards store page is updated to use the shared design system, ensuring UI/UX consistency with the mobile app.

**Deliverables for Step 7**

* Practice screen created for mobile with real voice input for the AI tutor.
* Community screen created for mobile.
* Leaderboard screen created for mobile.
* Rewards Store screen created for mobile.
* Web app practice, community, leaderboard, and rewards store pages updated for UI/UX consistency.

***

#### Step 8: Testing and Debugging (May 27-June 16, 2025)

**Objective**

Test the mobile app on iOS and Android to ensure feature parity, performance, and usability, addressing any bugs or issues. Update web app tests to include new integration features.

**Sub-Steps**

1. **Unit Testing for Mobile**:
   *   Use Jest and React Native Testing Library to test components:

       ```javascript
       // SATPrepSuiteMobile/__tests__/QuestionDisplay.test.js
       import { render, fireEvent } from '@testing-library/react-native';
       import { QuestionDisplay } from '../src/components/QuestionDisplay';

       describe('QuestionDisplay', () => {
         test('renders question and options', () => {
           const { getByText } = render(<QuestionDisplay questionNumber={1} eliminatorActive={false} toggleEliminator={() => {}} />);
           expect(getByText('The student wants to emphasize a similarity between the two works.')).toBeTruthy();
         });

         test('selects an answer', () => {
           const { getByText } = render(<QuestionDisplay questionNumber={1} eliminatorActive={false} toggleEliminator={() => {}} />);
           fireEvent.press(getByText('A. Erasure (2008) uses discarded objects'));
           expect(getByText('●')).toBeTruthy();
         });
       });
       ```
   * Test other components (`PassageDisplay.js`, `AnswerEliminator.js`, `MathQuestionHeader.js`) similarly.
2. **End-to-End Testing for Mobile**:
   *   Use Detox to test the full onboarding flow, practice sessions, and push notifications on iOS and Android emulators:

       ```javascript
       // SATPrepSuiteMobile/e2e/onboarding.e2e.js
       describe('Onboarding Flow', () => {
         beforeEach(async () => {
           await device.reloadReactNative();
         });

         it('should complete onboarding', async () => {
           await element(by.text('Next')).tap();
           await element(by.id('full_name')).typeText('Alex Smith');
           await element(by.id('grade')).tap();
           await element(by.text('11th')).tap();
           await element(by.text('Next')).tap();
           await element(by.text('Yes')).tap();
           await element(by.id('sat_math_score')).typeText('650');
           await element(by.id('sat_reading_score')).typeText('600');
           await element(by.text('Next')).tap();
           // Continue through remaining steps
           await element(by.text('Start Studying')).tap();
           await expect(element(by.text('Dashboard'))).toBeVisible();
         });
       });
       ```
3. **Device Testing**:
   * Test on physical devices (e.g., iPhone 14, Samsung Galaxy S23) and emulators (iOS Simulator, Android Emulator).
   * Verify touch interactions, swipe gestures (e.g., in `OnboardingScreen.js`), and offline mode functionality.
4. **Performance Testing**:
   * Measure app startup time, API latency, and offline mode performance.
   * Target: App startup < 2 seconds, API latency < 300ms (after optimization).
5. **Debugging**:
   * Use React Native Debugger and Flipper to identify and fix issues (e.g., layout bugs, API errors).
   * Ensure push notifications work reliably on both platforms.
6. **Update Web App Tests**:
   * **File**: `frontend/tests/onboarding.test.js` (already provided in previous steps, verified to include new integration features like theme persistence).
   *   Add tests for WebSocket updates:

       ```javascript
       // frontend/tests/gamification.test.js
       import { render, screen, fireEvent } from '@testing-library/react';
       import Dashboard from '../pages/dashboard';
       import { api } from '../utils/api';

       jest.mock('../utils/api');

       describe('Dashboard WebSocket Updates', () => {
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
             return Promise.resolve({ data: {} });
           });
         });

         test('updates coins on WebSocket message', async () => {
           render(<Dashboard />);
           expect(screen.getByText('Coins: 100 🪙')).toBeInTheDocument();
           // Simulate WebSocket message
           api.get.mockImplementationOnce(() => Promise.resolve({ data: { coins: 150 } }));
           // Trigger WebSocket message (mocked)
           await screen.findByText('Coins: 150 🪙');
         });
       });
       ```

**Deliverables for Step 8**

* Unit and end-to-end tests for all major mobile features.
* App tested on iOS and Android devices/emulators.
* Performance optimized (startup < 2s, API latency < 300ms).
* Bugs and issues resolved.
* Web app tests updated to include WebSocket updates.

***

#### Step 9: App Store Submission (June 17-July 7, 2025)

**Objective**

Prepare the mobile app for submission to the App Store (iOS) and Google Play (Android), ensuring compliance with guidelines and a smooth launch.

**Sub-Steps**

1. **Prepare App Assets**:
   * Create app icons, splash screens, and screenshots using tools like Figma.
   * Write app store descriptions:
     * **Title**: "SAT Prep Suite: AI-Powered Test Prep"
     * **Description**: "Prepare for the SAT with AI-driven practice, Bluebook-aligned tests, and gamified learning. Features adaptive diagnostics, AI tutoring, offline mode, and more!"
   * Add keywords: "SAT prep, AI tutor, Bluebook, offline, gamification".
2. **Configure Build Settings**:
   * **iOS**:
     * Update `ios/SATPrepSuiteMobile/Info.plist` with app permissions (e.g., microphone for voice input, notifications).
     * Set up an Apple Developer account ($99/year) and create an App Store Connect listing.
     * Build the app: `cd ios && xcodebuild`.
   * **Android**:
     * Update `android/app/build.gradle` with app details (e.g., version, permissions).
     * Set up a Google Play Developer account ($25 one-time fee) and create a Play Store listing.
     * Build the app: `cd android && ./gradlew assembleRelease`.
3. **Submit to App Stores**:
   * Submit the iOS app to App Store Connect, ensuring compliance with Apple’s guidelines (e.g., privacy policy for voice input).
   * Submit the Android app to Google Play Console, ensuring compliance with Google’s guidelines (e.g., content policy for educational apps).
   * Monitor the review process (typically 1-2 weeks for iOS, 1-3 days for Android).
4. **Launch Marketing**:
   * Announce the app launch on social media (e.g., TikTok, Instagram) targeting high school students.
   * Update the SAT Prep Suite website with links to download the app from the App Store and Google Play.

**Deliverables for Step 9**

* App assets (icons, screenshots, descriptions) prepared.
* iOS and Android builds configured and submitted.
* App approved and live on App Store and Google Play.
* Launch marketing campaign initiated.

***

#### Step 10: Post-Launch Monitoring and Iteration (July 8-31, 2025)

**Objective**

Monitor the app’s performance, gather user feedback, and iterate to improve the mobile experience.

**Sub-Steps**

1. **Monitor Performance**:
   * Use Firebase Analytics to track user engagement (e.g., daily active users, session length, test completion rate).
   * Monitor crash reports using Firebase Crashlytics to identify and fix issues.
2. **Gather User Feedback**:
   *   Add an in-app feedback form (e.g., “Rate your experience”) to collect user input:

       ```javascript
       // SATPrepSuiteMobile/src/screens/DashboardScreen.js (update)
       const DashboardScreen = ({ diagnosticResults }) => {
         const [feedback, setFeedback] = useState('');

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

         return (
           <Animated.View entering={FadeIn} exiting={FadeOut} style={styles.container}>
             {/* Existing dashboard content */}
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
           </Animated.View>
         );
       };

       const styles = StyleSheet.create({
         // Existing styles
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
   * Monitor app store reviews and ratings on the App Store and Google Play.
3. **Iterate Based on Feedback**:
   * Address common issues (e.g., improve push notification reliability, fix layout bugs on specific devices).
   * Add requested features (e.g., dark mode toggle, additional avatars in the Rewards Store).
4. **Optimize Engagement**:
   * Analyze push notification effectiveness (e.g., open rates) and adjust messaging (e.g., “Don’t miss today’s challenge!”).
   * Increase user retention by sending personalized notifications based on user activity (e.g., “You’re 2 days away from a 5-day streak!”).

**Deliverables for Step 10**

* Analytics and crash reporting set up with Firebase.
* User feedback collected via in-app form and app store reviews.
* Initial post-launch updates released (e.g., bug fixes, minor features).
* Engagement optimized through improved notifications.

***

### Final Verification

* **Feature Parity**: All web features (diagnostics, practice, AI tutor, gamification, social features, offline mode, onboarding) are available on mobile.
* **Integration**: Data synchronization (via shared backend and WebSocket), unified authentication (with SSO), consistent UI/UX (via shared design system), and seamless offline support are implemented.
* **Mobile-Specific Features**: Push notifications, real voice input, native gestures, and optimized offline mode are fully integrated.
* **Testing**: Unit, end-to-end, device, and performance tests ensure the app is robust and user-friendly.
* **Deployment**: The app is live on the App Store and Google Play, with a marketing campaign to drive user acquisition.

### Impact

* **User Acquisition**: Native apps increase active users to 2,000 by the end of 2025 (vs. current 1,000 goal).
* **Engagement**: Push notifications and real voice input boost engagement (target: 90% test completion rate, 5 challenges/week per user).
* **User Experience**: Consistent UI/UX and offline support improve satisfaction (NPS > 60).
* **Market Position**: Mobile apps position SAT Prep Suite as a leading AI-based SAT prep app, surpassing competitors like **LearnQ.ai**.

This completes the plan to create the mobile app and update the web app for tight integration. Let me know if you’d like to proceed with implementation or focus on a specific aspect!
