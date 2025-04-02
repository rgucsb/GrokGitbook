# Page 6

Let's continue with the step-by-step plan to create the **SAT Prep Suite** mobile app for iOS and Android while updating the web app for tight integration. We’ll proceed with the remaining sub-steps of **Step 2: Create Shared Design System and Adapt Components**, and then move on to the subsequent steps. I’ll keep each step focused and concise to ensure we can complete the plan without interruptions, while double-checking for completeness.

***

#### Step 2: Create Shared Design System and Adapt Components (April 8-14, 2025) - Continued

**Objective**

Continue adapting components for React Native, update web components to use the shared design system, and implement theme persistence across platforms.

**Sub-Steps (Continued)**

4. **Adapt Components for React Native (Continued)**:
   *   Complete the adaptation of `QuestionDisplay.js` (started in the previous response):

       ```javascript
       // SATPrepSuiteMobile/src/components/QuestionDisplay.js
       import React, { useState, useEffect } from 'react';
       import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
       import AnswerEliminator from './AnswerEliminator';
       import { colors, typography, commonStyles } from '../styles';

       export const QuestionDisplay = ({
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

         const handleAnswerClick = (value) => {
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
             console.log(`Eliminator active: ${eliminatorActive}, toggling elimination for ${value}`);
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
                 <TouchableOpacity onPress={toggleMarkForReview} style={styles.markButton}>
                   <Text style={styles.markText}>
                     Mark for Review {isMarkedForReview ? '★' : '☆'}
                   </Text>
                 </TouchableOpacity>
               </View>
               <View style={styles.headerRight}>
                 <AnswerEliminator active={eliminatorActive} onToggle={toggleEliminator} />
               </View>
             </View>

             <View style={commonStyles.card}>
               <Text style={typography.heading}>
                 The student wants to emphasize a similarity between the two works. Which choice most effectively uses relevant information from the notes to accomplish this goal?
               </Text>

               <View style={styles.options}>
                 {options.map((option) => (
                   <View
                     key={option.value}
                     style={[
                       styles.option,
                       selectedAnswer === option.value && styles.selectedOption,
                       eliminatedAnswers.has(option.value) && styles.eliminatedOption
                     ]}
                   >
                     <View style={styles.optionContent}>
                       <TouchableOpacity
                         onPress={() => handleAnswerClick(option.value)}
                         style={styles.radio}
                       >
                         <Text>{selectedAnswer === option.value ? '●' : '○'}</Text>
                       </TouchableOpacity>
                       <Text
                         style={[
                           styles.optionLabel,
                           eliminatedAnswers.has(option.value) && styles.lineThrough
                         ]}
                       >
                         <Text style={styles.optionLetter}>{option.value}.</Text> {option.label}
                       </Text>
                     </View>
                     {eliminatorActive && (
                       <TouchableOpacity
                         onPress={() => handleAnswerClick(option.value)}
                         style={styles.eliminatorButton}
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
                   </View>
                 ))}
               </View>
             </View>
           </View>
         );
       };

       const styles = StyleSheet.create({
         container: {
           padding: 24,
           flex: 1,
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
         },
         questionNumber: {
           fontWeight: 'bold',
           backgroundColor: '#333',
           color: 'white',
           padding: 8,
           borderRadius: 5,
         },
         markButton: {
           padding: 5,
         },
         markText: {
           color: '#4b5563',
         },
         options: {
           marginTop: 20,
         },
         option: {
           flexDirection: 'row',
           alignItems: 'center',
           justifyContent: 'space-between',
           padding: 12,
           borderWidth: 1,
           borderColor: colors.gray,
           borderRadius: 5,
           marginBottom: 12,
         },
         optionContent: {
           flexDirection: 'row',
           alignItems: 'center',
           flex: 1,
         },
         selectedOption: {
           borderColor: '#2563eb',
           backgroundColor: 'rgba(37, 99, 235, 0.1)',
         },
         eliminatedOption: {
           backgroundColor: '#f3f4f6',
           opacity: 0.6,
         },
         radio: {
           marginRight: 10,
         },
         optionLabel: {
           flex: 1,
         },
         lineThrough: {
           textDecorationLine: 'line-through',
           color: '#6b7280',
         },
         optionLetter: {
           fontWeight: '500',
           marginRight: 8,
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
       ```
   *   Adapt `PassageDisplay.js`:

       ```javascript
       // SATPrepSuiteMobile/src/components/PassageDisplay.js
       import React, { useState } from 'react';
       import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
       import { colors, typography } from '../styles';

       export const PassageDisplay = ({ passageText }) => {
         const [highlighted, setHighlighted] = useState(false);

         const toggleHighlight = () => {
           setHighlighted(!highlighted);
         };

         return (
           <View style={styles.container}>
             <Text style={[typography.body, highlighted && styles.highlighted]}>
               {passageText}
             </Text>
             <TouchableOpacity onPress={toggleHighlight} style={styles.highlightButton}>
               <Text style={styles.buttonText}>{highlighted ? 'Remove Highlight' : 'Highlight'}</Text>
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
           alignItems: 'center',
         },
         buttonText: {
           color: colors.white,
           fontWeight: 'bold',
         },
       });
       ```
   *   Adapt `AnswerEliminator.js`:

       ```javascript
       // SATPrepSuiteMobile/src/components/AnswerEliminator.js
       import React from 'react';
       import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
       import { colors, typography } from '../styles';

       const AnswerEliminator = ({ active, onToggle }) => {
         return (
           <TouchableOpacity onPress={onToggle} style={styles.container}>
             <Text style={[typography.body, active && styles.active]}>
               ABC {active ? 'On' : 'Off'}
             </Text>
           </TouchableOpacity>
         );
       };

       const styles = StyleSheet.create({
         container: {
           padding: 5,
           borderWidth: 1,
           borderColor: colors.gray,
           borderRadius: 5,
         },
         active: {
           color: colors.primary,
           fontWeight: 'bold',
         },
       });

       export default AnswerEliminator;
       ```
   *   Adapt `MathQuestionHeader.js`:

       ```javascript
       // SATPrepSuiteMobile/src/components/MathQuestionHeader.js
       import React from 'react';
       import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
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
           <View style={styles.header}>
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
               <TouchableOpacity onPress={onPrevious} style={styles.navButton}>
                 <Text style={typography.body}>Previous</Text>
               </TouchableOpacity>
               <TouchableOpacity onPress={onNext} style={styles.navButton}>
                 <Text style={typography.body}>Next</Text>
               </TouchableOpacity>
             </View>
           </View>
         );
       };

       const styles = StyleSheet.create({
         header: {
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
           padding: 8,
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
       ```
   * Adapt layout components (`ReadingWritingTest.js`, `MathBasicTest.js`, etc.) similarly, ensuring they use the shared design system.
5. **Update Web Components for Consistency**:
   *   Update `frontend/components/QuestionDisplay.js` to use the shared design system:

       ```javascript
       // frontend/components/QuestionDisplay.js
       import React, { useState, useEffect } from 'react';
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

         const handleAnswerClick = (value) => {
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
             console.log(`Eliminator active: ${eliminatorActive}, toggling elimination for ${value}`);
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
           <div className="question-display">
             <div className="question-header">
               <div className="header-left">
                 <span className="question-number">{questionNumber}</span>
                 <button onClick={toggleMarkForReview} className="mark-button">
                   Mark for Review {isMarkedForReview ? '★' : '☆'}
                 </button>
               </div>
               <div className="header-right">
                 <AnswerEliminator active={eliminatorActive} onToggle={toggleEliminator} />
               </div>
             </div>

             <div className="question-card">
               <p style={typography.heading}>
                 The student wants to emphasize a similarity between the two works. Which choice most effectively uses relevant information from the notes to accomplish this goal?
               </p>

               <div className="answer-options">
                 {options.map((option) => (
                   <div
                     key={option.value}
                     className={`option ${selectedAnswer === option.value ? 'selected' : ''} ${eliminatedAnswers.has(option.value) ? 'eliminated' : ''}`}
                   >
                     <div className="option-content">
                       <input
                         type="radio"
                         name={`question-${questionNumber}`}
                         value={option.value}
                         checked={selectedAnswer === option.value}
                         onChange={() => handleAnswerClick(option.value)}
                       />
                       <label
                         style={eliminatedAnswers.has(option.value) ? { textDecoration: 'line-through', color: '#6b7280' } : {}}
                       >
                         <span className="option-letter">{option.value}.</span> {option.label}
                       </label>
                     </div>
                     {eliminatorActive && (
                       <button onClick={() => handleAnswerClick(option.value)} className="eliminator-button">
                         {eliminatedAnswers.has(option.value) ? (
                           <span className="undo-button">Undo</span>
                         ) : (
                           <div className="cross-out">
                             <span className="cross-out-letter">{option.value}</span>
                             <div className="cross-line"></div>
                           </div>
                         )}
                       </button>
                     )}
                   </div>
                 ))}
               </div>
             </div>

             <style jsx>{`
               .question-display {
                 padding: 24px;
                 overflow-y: auto;
               }
               .question-header {
                 display: flex;
                 justify-content: space-between;
                 align-items: center;
                 margin-bottom: 16px;
                 border-bottom: 1px solid ${colors.gray};
                 padding-bottom: 12px;
               }
               .header-left, .header-right {
                 display: flex;
                 align-items: center;
                 gap: 12px;
               }
               .question-number {
                 font-weight: bold;
                 background: #333;
                 color: white;
                 padding: 8px 12px;
                 border-radius: 5px;
               }
               .mark-button {
                 padding: 5px;
                 background: none;
                 border: none;
                 color: #4b5563;
                 cursor: pointer;
               }
               .question-card {
                 border: 1px solid ${colors.gray};
                 border-radius: 5px;
                 padding: 20px;
                 margin-bottom: 16px;
               }
               .answer-options {
                 margin-top: 20px;
                 display: flex;
                 flex-direction: column;
                 gap: 12px;
               }
               .option {
                 display: flex;
                 align-items: center;
                 padding: 12px;
                 border: 1px solid ${colors.gray};
                 border-radius: 5px;
                 cursor: pointer;
               }
               .option.selected {
                 border-color: #2563eb;
                 background: rgba(37, 99, 235, 0.1);
               }
               .option.eliminated {
                 background: #f3f4f6;
                 opacity: 0.6;
               }
               .option.eliminated label {
                 text-decoration: line-through;
                 color: #6b7280;
               }
               .option-content {
                 display: flex;
                 align-items: center;
                 flex: 1;
               }
               .option label {
                 flex: 1;
                 cursor: pointer;
               }
               .option-letter {
                 font-weight: 500;
                 margin-right: 10px;
               }
               .eliminator-button {
                 width: 24px;
                 height: 24px;
                 display: flex;
                 justify-content: center;
                 align-items: center;
                 background: none;
                 border: none;
                 cursor: pointer;
               }
               .cross-out {
                 position: relative;
                 width: 24px;
                 height: 24px;
                 border: 1px solid #000;
                 border-radius: 12px;
                 display: flex;
                 justify-content: center;
                 align-items: center;
               }
               .cross-out-letter {
                 font-size: 14px;
                 font-weight: bold;
               }
               .cross-line {
                 position: absolute;
                 width: 100%;
                 height: 2px;
                 background: #000;
               }
               .undo-button {
                 font-size: 12px;
                 color: #2563eb;
               }
             `}</style>
           </div>
         );
       };

       export default QuestionDisplay;
       ```
   * Update other web components (`PassageDisplay.js`, `AnswerEliminator.js`, `MathQuestionHeader.js`) similarly to use the shared design system.
6. **Handle Touch Events and Gestures**:
   * Replace `onClick` with `onPress` in mobile components (already done in `QuestionDisplay.js`).
   * Add swipe gestures using `react-native-gesture-handler` (to be implemented in screens like `OnboardingScreen.js` in the next step).
7. **Add Theme Persistence**:
   * Fetch the user’s theme on app load for both platforms:
     * **Web**: Update `pages/_app.js` (already done in Step 1).
     * **Mobile**: Update `App.js` (to be created in the next step, but placeholder code provided in Step 1).

**Deliverables**

* Shared design system (`styles.js`) created and applied to both platforms.
* Reused components (`QuestionDisplay.js`, `PassageDisplay.js`, `AnswerEliminator.js`, `MathQuestionHeader.js`) adapted for React Native.
* Web components updated to use the shared design system.
* Touch events implemented in mobile components.

***

#### Step 3: Implement Mobile-Specific Features and Backend Updates (April 15-28, 2025)

**Objective**

Add mobile-specific features (push notifications, real voice input, optimized offline mode) and update the backend to support real-time updates and notifications, ensuring seamless integration.

**Sub-Steps**

1. **Push Notifications**:
   *   Install `react-native-firebase`:

       ```
       npm install @react-native-firebase/app @react-native-firebase/messaging
       ```
   * Configure FCM for iOS and Android (add `google-services.json` for Android, APNs for iOS).
   *   Add a background handler in `App.js` (to be created in the next step, but placeholder code provided):

       ```javascript
       import messaging from '@react-native-firebase/messaging';
       import AsyncStorage from '@react-native-async-storage/async-storage';

       useEffect(() => {
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
       ```
   * Update the backend to send notifications:
     * **File**: `backend/src/notifications.py`
     *   **Code**:

         ```python
         from fastapi import APIRouter, HTTPException
         import firebase_admin
         from firebase_admin import credentials, messaging
         from psycopg2.extras import RealDictCursor

         router = APIRouter()

         # Initialize Firebase Admin SDK
         cred = credentials.Certificate("path/to/firebase-adminsdk.json")
         firebase_admin.initialize_app(cred)

         def get_db_connection():
             return psycopg2.connect(
                 dbname="satprep",
                 user="user",
                 password="password",
                 host="localhost",
                 port="5432"
             )

         @router.post("/send-notification")
         async def send_notification(user_id: str, message: str):
             conn = get_db_connection()
             cursor = conn.cursor(cursor_factory=RealDictCursor)
             try:
                 cursor.execute("SELECT device_token FROM users WHERE user_id = %s", (user_id,))
                 user = cursor.fetchone()
                 if not user or not user['device_token']:
                     raise HTTPException(status_code=404, detail="Device token not found")

                 message = messaging.Message(
                     notification=messaging.Notification(
                         title="SAT Prep Suite",
                         body=message
                     ),
                     token=user['device_token']
                 )
                 response = messaging.send(message)
                 return {"message": "Notification sent", "response": response}
             except Exception as e:
                 raise HTTPException(status_code=500, detail=str(e))
             finally:
                 cursor.close()
                 conn.close()
         ```
   * Send notifications for daily challenges (already updated in `backend/src/gamification.py` in Step 1).
2. **Real Voice Input for AI Tutor**:
   *   Install `react-native-voice`:

       ```
       npm install @react-native-voice/voice
       ```
   *   Placeholder for implementation in `PracticeScreen.js` (to be created in the next step):

       ```javascript
       import Voice from '@react-native-voice/voice';

       const PracticeScreen = () => {
         const [isRecording, setIsRecording] = useState(false);
         const [voiceText, setVoiceText] = useState('');

         useEffect(() => {
           Voice.onSpeechResults = (e) => {
             setVoiceText(e.value[0]);
             sendChatMessage(e.value[0]);
           };
           return () => Voice.destroy().then(Voice.removeAllListeners);
         }, []);

         const startVoiceRecording = async () => {
           setIsRecording(true);
           await Voice.start('en-US');
         };

         const stopVoiceRecording = async () => {
           setIsRecording(false);
           await Voice.stop();
         };

         return (
           <View>
             <TouchableOpacity onPress={isRecording ? stopVoiceRecording : startVoiceRecording}>
               <Text>{isRecording ? 'Stop Recording' : 'Start Voice Input'}</Text>
             </TouchableOpacity>
             <Text>Voice Input: {voiceText}</Text>
           </View>
         );
       };
       ```
3. **Optimize Offline Mode**:
   *   Update `SATPrepSuiteMobile/src/utils/api.js` for mobile:

       ```javascript
       // SATPrepSuiteMobile/src/utils/api.js
       import AsyncStorage from '@react-native-async-storage/async-storage';
       import BackgroundFetch from 'react-native-background-fetch';
       import NetInfo from "@react-native-community/netinfo";
       import { w3cwebsocket as W3CWebSocket } from 'websocket';

       export const api = {
         async get(endpoint) {
           try {
             const response = await fetch(`http://localhost:8000${endpoint}`);
             const data = await response.json();
             await AsyncStorage.setItem(endpoint, JSON.stringify(data));
             return { data };
           } catch (error) {
             const cachedData = await AsyncStorage.getItem(endpoint);
             if (cachedData) {
               return { data: JSON.parse(cachedData) };
             }
             throw new Error('Offline mode: No cached data available');
           }
         },
         async post(endpoint, body) {
           try {
             const response = await fetch(`http://localhost:8000${endpoint}`, {
               method: 'POST',
               headers: { 'Content-Type': 'application/json' },
               body: JSON.stringify(body)
             });
             const data = await response.json();
             return { data };
           } catch (error) {
             const queuedRequests = await AsyncStorage.getItem('queuedRequests') || '[]';
             const requests = JSON.parse(queuedRequests);
             requests.push({ endpoint, body });
             await AsyncStorage.setItem('queuedRequests', JSON.stringify(requests));
             throw new Error('Offline mode: Request queued');
           }
         },
         async sync() {
           const queuedRequests = await AsyncStorage.getItem('queuedRequests') || '[]';
           const requests = JSON.parse(queuedRequests);
           let syncedCount = 0;
           for (const req of requests) {
             try {
               await fetch(`http://localhost:8000${req.endpoint}`, {
                 method: 'POST',
                 headers: { 'Content-Type': 'application/json' },
                 body: JSON.stringify(req.body)
               });
               syncedCount++;
             } catch (error) {
               console.error('Sync failed for request:', req);
             }
           }
           await AsyncStorage.setItem('queuedRequests', JSON.stringify(requests.slice(syncedCount)));
           return syncedCount;
         }
       };

       export const connectWebSocket = (userId, onMessage) => {
         const ws = new W3CWebSocket(`ws://localhost:8000/gamification/updates/${userId}`);
         ws.onmessage = (message) => onMessage(message.data);
         return ws;
       };

       const configureBackgroundSync = () => {
         BackgroundFetch.configure({
           minimumFetchInterval: 15,
           stopOnTerminate: false,
           startOnBoot: true,
         }, async () => {
           const syncedCount = await api.sync();
           console.log(`Background sync completed: ${syncedCount} items`);
           BackgroundFetch.finish(BackgroundFetch.FETCH_RESULT_NEW_DATA);
         }, (error) => {
           console.error('Background fetch failed:', error);
         });
       };

       NetInfo.addEventListener(state => {
         if (state.isConnected) {
           api.sync().then(count => console.log(`Synced ${count} items`));
         }
       });

       configureBackgroundSync();
       ```
   * Ensure the web app’s `utils/api.js` matches this logic (already updated in Step 1).

**Deliverables**

* Push notifications configured with FCM.
* Real voice input placeholder added for AI tutor.
* Optimized offline mode with background sync on mobile.

***

#### Step 4: Create Mobile Navigation and Login Screen (April 29-May 5, 2025)

**Objective**

Set up navigation for the mobile app and create the login screen, ensuring unified authentication across platforms.

**Sub-Steps**

1. **Set Up Navigation for Mobile**:
   *   Use React Navigation for a stack navigator with a bottom tab navigator:

       ```javascript
       // SATPrepSuiteMobile/App.js
       import React, { useState, useEffect } from 'react';
       import { View, StyleSheet, Alert } from 'react-native';
       import { NavigationContainer } from '@react-navigation/native';
       import { createStackNavigator } from '@react-navigation/stack';
       import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
       import AsyncStorage from '@react-native-async-storage/async-storage';
       import messaging from '@react-native-firebase/messaging';
       import { api } from './src/utils/api';
       import OnboardingScreen from './src/screens/OnboardingScreen';
       import DashboardScreen from './src/screens/DashboardScreen';
       import PracticeScreen from './src/screens/PracticeScreen';
       import StudyPlanScreen from './src/screens/StudyPlanScreen';
       import CommunityScreen from './src/screens/CommunityScreen';
       import LeaderboardScreen from './src/screens/LeaderboardScreen';
       import RewardsStoreScreen from './src/screens/RewardsStoreScreen';
       import LoginScreen from './src/screens/LoginScreen';

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
2. **Create Login Screen**:
   * **File**: `SATPrepSuiteMobile/src/screens/LoginScreen.js`
   *   **Code**:

       ```javascript
       import React, { useState } from 'react';
       import { View, TextInput, TouchableOpacity, Text, StyleSheet, Alert } from 'react-native';
       import { api } from '../utils/api';
       import AsyncStorage from '@react-native-async-storage/async-storage';
       import { authorize } from 'react-native-app-auth';
       import { colors, typography, commonStyles } from '../styles';

       const config = {
         clientId: 'your-client-id',
         redirectUrl: 'com.satprepsuite:/oauthredirect',
         scopes: ['email', 'profile'],
         serviceConfiguration: {
           authorizationEndpoint: 'https://accounts.google.com/o/oauth2/v2/auth',
           tokenEndpoint: 'https://oauth2.googleapis.com/token',
         },
       };

       const LoginScreen = ({ navigation }) => {
         const [email, setEmail] = useState('');
         const [password, setPassword] = useState('');

         const handleLogin = async () => {
           try {
             const res = await api.post('/auth/login', { email, password });
             await AsyncStorage.setItem('user_id', res.data.user_id);
             navigation.navigate('Onboarding');
           } catch (error) {
             Alert.alert('Login failed', error.message);
           }
         };

         const handleGoogleLogin = async () => {
           try {
             const result = await authorize(config);
             const res = await api.post('/auth/sso', { token: result.accessToken });
             await AsyncStorage.setItem('user_id', res.data.user_id);
             navigation.navigate('Onboarding');
           } catch (error) {
             Alert.alert('SSO failed', error.message);
           }
         };

         return (
           <View style={styles.container}>
             <Text style={typography.heading}>Login to SAT Prep Suite</Text>
             <TextInput
               style={styles.input}
               placeholder="Email"
               value={email}
               onChangeText={setEmail}
             />
             <TextInput
               style={styles.input}
               placeholder="Password"
               value={password}
               onChangeText={setPassword}
               secureTextEntry
             />
             <TouchableOpacity style={commonStyles.button} onPress={handleLogin}>
               <Text style={commonStyles.buttonText}>Login</Text>
             </TouchableOpacity>
             <TouchableOpacity style={commonStyles.button} onPress={handleGoogleLogin}>
               <Text style={commonStyles.buttonText}>Login with Google</Text>
             </TouchableOpacity>
           </View>
         );
       };

       const styles = StyleSheet.create({
         container: {
           flex: 1,
           justifyContent: 'center',
           padding: 20,
           backgroundColor: colors.white,
         },
         input: {
           borderWidth: 1,
           borderColor: colors.gray,
           borderRadius: 5,
           padding: 10,
           marginBottom: 10,
         },
       });

       export default LoginScreen;
       ```
3. **Update Web App Login for Consistency**:
   * **File**: `frontend/pages/login.js`
   *   **Code**:

       ```javascript
       import React, { useState } from 'react';
       import { useRouter } from 'next/router';
       import { api } from '../utils/api';
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
           // Implement Google SSO (e.g., using Firebase Auth)
           alert('Google SSO not implemented in this example');
         };

         return (
           <div className="login">
             <h1 style={typography.heading}>Login to SAT Prep Suite</h1>
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

**Deliverables**

* Mobile app navigation set up with React Navigation.
* Login screen created for mobile with SSO support.
* Web app login updated for UI/UX consistency.

***

#### Step 5: Create Onboarding Screen for Mobile (May 6-12, 2025)

**Objective**

Create the onboarding screen for the mobile app, ensuring it matches the web app’s functionality and UI/UX, with swipe gestures for navigation.

**Sub-Steps**

1. **Create Onboarding Screen**:
   * **File**: `SATPrepSuiteMobile/src/screens/OnboardingScreen.js`
   *   **Code**:

       ```javascript
       import React, { useState, useEffect } from 'react';
       import { View, Text, TextInput, TouchableOpacity, StyleSheet, Picker, Slider, Alert } from 'react-native';
       import { GestureHandlerRootView, Swipeable } from 'react-native-gesture-handler';
       import Animated, { FadeIn, FadeOut } from 'react-native-reanimated';
       import { api } from '../utils/api';
       import AsyncStorage from '@react-native-async-storage/async-storage';
       import { colors, typography, commonStyles } from '../styles';
       import DiagnosticScreen from './DiagnosticScreen';
       import DashboardScreen from './DashboardScreen';
       import StudyPlanScreen from './StudyPlanScreen';

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
             <Text style={typography.heading}>Welcome to LearnerLabs SAT Prep!</Text>
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
               <Text style={styles.label}>Full Name</Text>
               <TextInput
                 style={styles.input}
                 value={userData.full_name}
                 onChangeText={(text) => handleInputChange('full_name', text)}
                 placeholder="Enter your full name"
               />
             </View>
             <View style={styles.formGroup}>
               <Text style={styles.label}>Grade Level</Text>
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
                 <Picker.Item label="Gap Year" value="Gap Year" />
               </Picker>
             </View>
             <View style={styles.formGroup}>
               <Text style={styles.label}>School Name (Optional)</Text>
               <TextInput
                 style={styles.input}
                 value={userData.school}
                 onChangeText={(text) => handleInputChange('school', text)}
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

         const SATHistoryStep = () => (
           <Animated.View entering={FadeIn} exiting={FadeOut} style={styles.step}>
             <Text style={typography.heading}>SAT History</Text>
             <View style={styles.formGroup}>
               <Text style={styles.label}>Have you taken the SAT before?</Text>
               <View style={styles.radioGroup}>
                 <TouchableOpacity onPress={() => handleInputChange('sat_taken', true)}>
                   <Text style={userData.sat_taken ? styles.selectedRadio : styles.radio}>Yes</Text>
                 </TouchableOpacity>
                 <TouchableOpacity onPress={() => handleInputChange('sat_taken', false)}>
                   <Text style={!userData.sat_taken ? styles.selectedRadio : styles.radio}>No</Text>
                 </TouchableOpacity>
               </View>
             </View>
             {userData.sat_taken && (
               <>
                 <View style={styles.formGroup}>
                   <Text style={styles.label}>Math Score</Text>
                   <TextInput
                     style={styles.input}
                     value={userData.sat_math_score?.toString() || ''}
                     onChangeText={(text) => handleInputChange('sat_math_score', parseInt(text))}
                     placeholder="Enter your Math score"
                     keyboardType="numeric"
                   />
                 </View>
                 <View style={styles.formGroup}>
                   <Text style={styles.label}>Reading & Writing Score</Text>
                   <TextInput
                     style={styles.input}
                     value={userData.sat_reading_score?.toString() || ''}
                     onChangeText={(text) => handleInputChange('sat_reading_score', parseInt(text))}
                     placeholder="Enter your Reading & Writing score"
                     keyboardType="numeric"
                   />
                 </View>
               </>
             )}
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
               <Text style={styles.label}>When do you plan to take the SAT?</Text>
               <TextInput
                 style={styles.input}
                 value={userData.sat_test_date}
                 onChangeText={(text) => handleInputChange('sat_test_date', text)}
                 placeholder="YYYY-MM-DD"
               />
             </View>
             <View style={styles.formGroup}>
               <Text style={styles.label}>How many hours per week can you study? ({userData.study_hours_per_week} hours)</Text>
               <Slider
                 minimumValue={1}
                 maximumValue={20}
                 step={1}
                 value={userData.study_hours_per_week}
                 onValueChange={(value) => handleInputChange('study_hours_per_week', value)}
                 style={styles.slider}
               />
             </View>
             <View style={styles.formGroup}>
               <Text style={styles.label}>How many days per week do you want to study?</Text>
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
               <Text style={styles.label}>Preferred Study Time?</Text>
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

         const DiagnosticTestStep = () => (
           <Animated.View entering={FadeIn} exiting={FadeOut} style={styles.step}>
             <Text style={typography.heading}>Let’s Assess Your Current Skills!</Text>
             <Text style={typography.body}>The diagnostic test will take approximately 45 minutes and covers Math and Reading/Writing.</Text>
             <Text style={typography.body}>Our AI will adapt the questions based on your responses to accurately assess your skills.</Text>
             <TouchableOpacity style={commonStyles.button} onPress={nextStep}>
               <Text style={commonStyles.buttonText}>Start Test</Text>
             </TouchableOpacity>
             <View style={styles.diagnosticTest}>
               <DiagnosticScreen onComplete={(results) => {
                 setDiagnosticResults(results);
                 nextStep();
               }} />
             </View>
           </Animated.View>
         );

         const ReviewResultsStep = () => (
           <Animated.View entering={FadeIn} exiting={FadeOut} style={styles.step}>
             <Text style={typography.heading}>Review Your Results</Text>
             <DashboardScreen diagnosticResults={diagnosticResults} />
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

         const DashboardIntroStep = () => (
           <Animated.View entering={FadeIn} exiting={FadeOut} style={styles.step}>
             <Text style={typography.heading}>Welcome to Your Dashboard!</Text>
             <Text style={typography.body}>Here’s a quick tour:</Text>
             <View style={styles.list}>
               <Text style={typography.body}>• <Text style={{ fontWeight: 'bold' }}>Daily Tasks:</Text> See your daily challenges and tasks.</Text>
               <Text style={typography.body}>• <Text style={{ fontWeight: 'bold' }}>Streaks:</Text> Track your study streaks to stay motivated.</Text>
               <Text style={typography.body}>• <Text style={{ fontWeight: 'bold' }}>Upcoming Tests:</Text> View your scheduled practice tests.</Text>
               <Text style={typography.body}>• <Text style={{ fontWeight: 'bold' }}>Study Sessions:</Text> Access practice sets and lessons.</Text>
               <Text style={typography.body}>• <Text style={{ fontWeight: 'bold' }}>Performance Tracking:</Text> Monitor your progress with graphs.</Text>
               <Text style={typography.body}>• <Text style={{ fontWeight: 'bold' }}>Social Features:</Text> Challenge friends and view leaderboards.</Text>
             </View>
             <Text style={typography.body}>Bonus Challenge: Complete your first study session to earn 100 XP!</Text>
             <TouchableOpacity style={commonStyles.button} onPress={completeOnboarding}>
               <Text style={commonStyles.buttonText}>Start Studying</Text>
             </TouchableOpacity>
           </Animated.View>
         );

         const renderStep = () => {
           switch (step) {
             case 1: return <WelcomeStep />;
             case 2: return <BasicInfoStep />;
             case 3: return <SATHistoryStep />;
             case 4: return <StudyPreferencesStep />;
             case 5: return <DiagnosticTestStep />;
             case 6: return <ReviewResultsStep />;
             case 7: return <StudyPlanStep />;
             case 8: return <DashboardIntroStep />;
             default: return null;
           }
         };

         return (
           <GestureHandlerRootView style={styles.container}>
             <View style={styles.stepper}>
               <Text style={typography.body}>Step {step} of 8</Text>
               <View style={styles.progressBar}>
                 <View style={[styles.progress, { width: `${(step / 8) * 100}%` }]} />
               </View>
             </View>
             <Swipeable
               onSwipeableRightOpen={prevStep}
               onSwipeableLeftOpen={nextStep}
             >
               {renderStep()}
             </Swipeable>
           </GestureHandlerRootView>
         );
       };

       const styles = StyleSheet.create({
         container: {
           flex: 1,
           padding: 20,
           backgroundColor: colors.white,
         },
         stepper: {
           marginBottom: 20,
         },
         progressBar: {
           backgroundColor: '#f0f0f0',
           borderRadius: 5,
           height: 10,
           position: 'relative',
         },
         progress: {
           backgroundColor: colors.primary,
           height: '100%',
           borderRadius: 5,
         },
         step: {
           padding: 20,
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
         radioGroup: {
           flexDirection: 'row',
           gap: 20,
           justifyContent: 'center',
         },
         radio: {
           fontSize: 16,
         },
         selectedRadio: {
           fontSize: 16,
           fontWeight: 'bold',
           color: colors.primary,
         },
         slider: {
           width: '100%',
         },
         buttons: {
           flexDirection: 'row',
           gap: 10,
           justifyContent: 'center',
           marginTop: 20,
         },
         list: {
           marginVertical: 20,
           alignItems: 'flex-start',
         },
       });

       export default OnboardingScreen;
       ```
2. **Update Web Onboarding for Consistency**:
   * **File**: `frontend/pages/onboarding.js`
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
             <h2 style={typography.heading}>Welcome to LearnerLabs SAT Prep!</h2>
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
                 <option value="Gap Year">Gap Year</option>
               </select>
             </div>
             <div className="form-group">
               <label style={typography.body}>School Name (Optional)</label>
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

         const SATHistoryStep = () => (
           <div className="step">
             <h2 style={typography.heading}>SAT History</h2>
             <div className="form-group">
               <label style={typography.body}>Have you taken the SAT before?</label>
               <div className="radio-group">
                 <label>
                   <input
                     type="radio"
                     value="true"
                     checked={userData.sat_taken === true}
                     onChange={() => handleInputChange('sat_taken', true)}
                   />
                   Yes
                 </label>
                 <label>
                   <input
                     type="radio"
                     value="false"
                     checked={userData.sat_taken === false}
                     onChange={() => handleInputChange('sat_taken', false)}
                   />
                   No
                 </label>
               </div>
             </div>
             {userData.sat_taken && (
               <>
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
               </>
             )}
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
               <label style={typography.body}>When do you plan to take the SAT?</label>
               <input
                 type="date"
                 value={userData.sat_test_date}
                 onChange={(e) => handleInputChange('sat_test_date', e.target.value)}
                 className="input"
               />
             </div>
             <div className="form-group">
               <label style={typography.body}>How many hours per week can you study? ({userData.study_hours_per_week} hours)</label>
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
               <label style={typography.body}>How many days per week do you want to study?</label>
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
               <label style={typography.body}>Preferred Study Time?</label>
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

         const DiagnosticTestStep = () => (
           <div className="step">
             <h2 style={typography.heading}>Let’s Assess Your Current Skills!</h2>
             <p style={typography.body}>The diagnostic test will take approximately 45 minutes and covers Math and Reading/Writing.</p>
             <p style={typography.body}>Our AI will adapt the questions based on your responses to accurately assess your skills.</p>
             <button style={commonStyles.button} onClick={nextStep}>
               <span style={commonStyles.buttonText}>Start Test</span>
             </button>
             <div className="diagnostic-test">
               <Diagnostic onComplete={(results) => {
                 setDiagnosticResults(results);
                 nextStep();
               }} />
             </div>
           </div>
         );

         const ReviewResultsStep = () => (
           <div className="step">
             <h2 style={typography.heading}>Review Your Results</h2>
             <Dashboard diagnosticResults={diagnosticResults} />
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

         const DashboardIntroStep = () => (
           <div className="step">
             <h2 style={typography.heading}>Welcome to Your Dashboard!</h2>
             <p style={typography.body}>Here’s a quick tour:</p>
             <ul style={{ textAlign: 'left', margin: '20px 0' }}>
               <li style={typography.body}><strong>Daily Tasks:</strong> See your daily challenges and tasks.</li>
               <li style={typography.body}><strong>Streaks:</strong> Track your study streaks to stay motivated.</li>
               <li style={typography.body}><strong>Upcoming Tests:</strong> View your scheduled practice tests.</li>
               <li style={typography.body}><strong>Study Sessions:</strong> Access practice sets and lessons.</li>
               <li style={typography.body}><strong>Performance Tracking:</strong> Monitor your progress with graphs.</li>
               <li style={typography.body}><strong>Social Features:</strong> Challenge friends and view leaderboards.</li>
             </ul>
             <p style={typography.body}>Bonus Challenge: Complete your first study session to earn 100 XP!</p>
             <button style={commonStyles.button} onClick={completeOnboarding}>
               <span style={commonStyles.buttonText}>Start Studying</span>
             </button>
           </div>
         );

         return (
           <div className="onboarding">
             <div className="stepper">
               <p style={typography.body}>Step {step} of 8</p>
               <div className="progress-bar">
                 <div style={{ width: `${(step / 8) * 100}%` }}></div>
               </div>
             </div>
             {step === 1 && <WelcomeStep />}
             {step === 2 && <BasicInfoStep />}
             {step === 3 && <SATHistoryStep />}
             {step === 4 && <StudyPreferencesStep />}
             {step === 5 && <DiagnosticTestStep />}
             {step === 6 && <ReviewResultsStep />}
             {step === 7 && <StudyPlanStep />}
             {step === 8 && <DashboardIntroStep />}
             <style jsx>{`
               .onboarding {
                 max-width: 600px;
                 margin: 50px auto;
                 padding: 20px;
                 text-align: center;
                 background: ${colors.white};
                 border-radius: 10px;
                 box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
               }
               .stepper {
                 margin-bottom: 20px;
               }
               .progress-bar {
                 background: #f0f0f0;
                 border-radius: 5px;
                 height: 10px;
                 position: relative;
               }
               .progress-bar div {
                 background: ${colors.primary};
                 height: 100%;
                 border-radius: 5px;
               }
               .step {
                 padding: 20px;
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
               .radio-group {
                 display: flex;
                 gap: 20px;
                 justify-content: center;
               }
               .buttons {
                 display: flex;
                 gap: 10px;
                 justify-content: center;
                 margin-top: 20px;
               }
               ul {
                 text-align: left;
                 margin: 20px 0;
               }
               li {
                 margin-bottom: 10px;
               }
               .slider {
                 width: 100%;
               }
             `}</style>
           </div>
         );
       };

       export default Onboarding;
       ```

**Deliverables**

* Onboarding screen created for mobile with swipe gestures.
* Web onboarding updated for UI/UX consistency.

***

#### Step 6: Create Diagnostic, Dashboard, and Study Plan Screens for Mobile (May 13-19, 2025)

**Objective**

Create the diagnostic, dashboard, and study plan screens for the mobile app, ensuring they match the web app’s functionality and UI/UX.

**Sub-Steps**

1. **Create Diagnostic Screen**:
   * **File**: `SATPrepSuiteMobile/src/screens/DiagnosticScreen.js`
   *   **Code**:

       ```javascript
       import React, { useState, useEffect } from 'react';
       import { View, Text, TouchableOpacity, StyleSheet, Alert } from 'react-native';
       import { api } from '../utils/api';
       import { colors, typography, commonStyles } from '../styles';
       import ReadingWritingTest from '../components/layouts/ReadingWritingTest';
       import MathBasicTest from '../components/layouts/MathBasicTest';

       const DiagnosticScreen = ({ onComplete }) => {
         const [questions, setQuestions] = useState([]);
         const [sessionId, setSessionId] = useState(null);
         const [answers, setAnswers] = useState({});
         const userId = localStorage.getItem('user_id');

         useEffect(() => {
           if (!userId) {
             navigation.navigate('Login');
             return;
           }
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

         const submitAnswer = async (questionId) => {
           const answer = answers[questionId] || '';
           const responseData = [{ question_id: questionId, answer, time_spent: 60, session_id: sessionId, domain: 'Mixed', theta: 0.8 }];
           try {
             const res = await api.post(`/diagnostic/submit/${sessionId}`, responseData);
             if (res.data.questions) {
               setQuestions(res.data.questions);
             } else {
               onComplete(res.data);
             }
             setAnswers({});
           } catch (error) {
             if (error.message === 'Offline mode: Request queued') {
               setQuestions(questions.slice(1));
               if (questions.length <= 1) {
                 Alert.alert('Diagnostic completed offline! Sync when online.');
                 onComplete({ theta: 0.8 }); // Mock result for offline
               }
             } else {
               Alert.alert('Error', 'Failed to submit answer: ' + error.message);
             }
           }
         };

         const renderQuestionLayout = (q) => {
           const props = {
             questionNumber: 1,
             totalQuestions: 22,
             timer: "15:00",
             onNext: () => submitAnswer(q.id),
             onPrevious: () => console.log("Previous question"),
             showCalculator: q.domain === 'Math',
             userName: userId || "Student",
             onTimeEnd: () => console.log("Time ended"),
             customButtons: q.domain === 'Reading & Writing' ? (
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

           if (q.domain === 'Reading & Writing') {
             return <ReadingWritingTest {...props} />;
           } else {
             return <MathBasicTest {...props} />;
           }
         };

         return (
           <View style={styles.container}>
             <Text style={typography.heading}>Diagnostic Test</Text>
             {questions.map((q) => (
               <View key={q.id} style={styles.question}>
                 {renderQuestionLayout(q)}
               </View>
             ))}
           </View>
         );
       };

       const styles = StyleSheet.create({
         container: {
           flex: 1,
           padding: 20,
           backgroundColor: colors.white,
         },
         question: {
           margin: 20,
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
2. **Create Dashboard Screen**:
   * **File**: `SATPrepSuiteMobile/src/screens/DashboardScreen.js`
   *   **Code**:

       ```javascript
       import React, { useState, useEffect } from 'react';
       import { View, Text, TouchableOpacity, StyleSheet, Alert } from 'react-native';
       import Animated, { FadeIn, FadeOut } from 'react-native-re
       ```
