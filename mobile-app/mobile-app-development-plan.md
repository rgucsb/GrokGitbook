# Mobile App Development Plan

To create a mobile app for the **SAT Prep Suite** (also referred to as **LearnerLabs SAT Prep App** in the onboarding context), we need a step-by-step plan that leverages the existing web-based application (built with Next.js and FastAPI) and addresses the gaps identified in the mobile platform capabilities. The goal is to develop native iOS and Android apps that provide a seamless, engaging, and accessible experience for users, aligning with the strategic objective of becoming the best AI-based SAT test prep app by the end of 2025. The plan will use **React Native** to reuse existing React components, integrate with the existing FastAPI backend, and ensure feature parity with the web app while adding mobile-specific enhancements (e.g., push notifications, native gestures, optimized offline mode). The timeline aligns with the previously proposed April-July 2025 window.

***

### Step-by-Step Plan to Create a Mobile App for SAT Prep Suite

#### 1. Project Setup and Planning (April 1-7, 2025)

**Objective**

Set up the development environment, define the project structure, and plan the mobile app architecture to ensure feature parity with the web app.

**Steps**

1. **Choose React Native**:
   * Use React Native to build cross-platform iOS and Android apps, leveraging the existing React components (e.g., `QuestionDisplay.js`, `PassageDisplay.js`, `onboarding.js`).
   * **Reason**: React Native allows code reuse from the Next.js frontend, reducing development time while providing native performance.
2. **Set Up Development Environment**:
   * Install Node.js, React Native CLI, and dependencies on development machines.
   * Set up iOS (Xcode) and Android (Android Studio) development environments.
   * Install required libraries: `react-native`, `react-native-navigation`, `axios` (for API calls), `react-native-async-storage` (for offline caching), `react-native-firebase` (for push notifications).
3. **Create Project Structure**:
   * Initialize a new React Native project: `npx react-native init SATPrepSuiteMobile`.
   *   Structure the project:

       ```
       SATPrepSuiteMobile/
       ├── src/
       │   ├── components/       # Reused React components (e.g., QuestionDisplay, PassageDisplay)
       │   ├── screens/          # Mobile screens (e.g., OnboardingScreen, DashboardScreen)
       │   ├── utils/            # API and offline utilities
       │   ├── assets/           # Images, icons, fonts
       │   └── navigation/       # Navigation setup
       ├── App.js                # Main app entry point
       ├── android/              # Android-specific files
       ├── ios/                  # iOS-specific files
       └── package.json
       ```
4. **Define Architecture**:
   * **Frontend**: React Native with React Navigation for routing, reusing existing components where possible.
   * **Backend**: Reuse the existing FastAPI backend (`backend/src/`), ensuring API endpoints are mobile-friendly (e.g., support for push notification tokens).
   * **State Management**: Use React Context or Redux for managing app state (e.g., user data, onboarding progress).
   * **Offline Mode**: Use `react-native-async-storage` for caching questions and responses, with background sync when online.
   * **Push Notifications**: Integrate Firebase Cloud Messaging (FCM) for reminders (e.g., daily challenges, streaks).
5. **Plan Feature Parity**:
   * Ensure all web features (diagnostics, practice, AI tutor, gamification, social features, offline mode, onboarding) are available on mobile.
   * Add mobile-specific features: push notifications, native gestures (e.g., swipe to navigate), real voice input for AI tutor.

**Deliverables**

* React Native project initialized (`SATPrepSuiteMobile`).
* Development environment set up for iOS and Android.
* Project structure and architecture defined.

***

#### 2. Reuse and Adapt Existing Components (April 8-21, 2025)

**Objective**

Port existing React components from the web app to React Native, adapting them for mobile-specific requirements (e.g., touch events, native styling).

**Steps**

1. **Copy Reusable Components**:
   * Copy components from `frontend/components/` (e.g., `QuestionDisplay.js`, `PassageDisplay.js`, `MathQuestionHeader.js`, `AnswerEliminator.js`) to `SATPrepSuiteMobile/src/components/`.
   * Copy layout components from `frontend/components/layouts/` (e.g., `ReadingWritingTest.js`, `MathBasicTest.js`) to `SATPrepSuiteMobile/src/components/layouts/`.
2. **Adapt Components for React Native**:
   * Replace web-specific elements with React Native equivalents:
     * `div` → `View`
     * `p` → `Text`
     * `input` → `TextInput`
     * `button` → `TouchableOpacity` or `Button`
     * `style jsx` → `StyleSheet` (React Native styling)
   *   Example: Update `QuestionDisplay.js`:

       ```javascript
       import React, { useState, useEffect } from 'react';
       import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
       import AnswerEliminator from './AnswerEliminator';

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

             <View style={styles.card}>
               <Text style={styles.questionText}>
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
                     <View style={styles.optionContent} onPress={() => handleAnswerClick(option.value)}>
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
           borderBottomColor: '#ddd',
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
         card: {
           padding: 20,
           marginBottom: 16,
           borderWidth: 1,
           borderColor: '#ddd',
           borderRadius: 5,
         },
         questionText: {
           fontSize: 18,
           fontWeight: '500',
           marginBottom: 24,
           color: '#1e40af',
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
           borderColor: '#ddd',
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
3. **Update Navigation**:
   * Replace web navigation (Next.js routing) with React Navigation.
   * Example: Update `onboarding.js` to use `StackNavigator` for step navigation.
4. **Handle Touch Events**:
   * Replace `onClick` with `onPress` for touch interactions.
   * Add gesture support (e.g., swipe to navigate between steps in the onboarding wizard) using `react-native-gesture-handler`.

**Deliverables**

* Reused components (`QuestionDisplay.js`, `PassageDisplay.js`, etc.) adapted for React Native.
* Navigation setup with React Navigation.
* Touch events and gestures implemented.

***

#### 3. Implement Mobile-Specific Features (April 22-May 5, 2025)

**Objective**

Add mobile-specific features to enhance the user experience, including push notifications, real voice input, and optimized offline mode.

**Steps**

1. **Push Notifications**:
   * Integrate Firebase Cloud Messaging (FCM) for push notifications.
   *   Install `react-native-firebase`:

       ```
       npm install @react-native-firebase/app @react-native-firebase/messaging
       ```
   * Configure FCM for iOS and Android (e.g., add `google-services.json` for Android, APNs for iOS).
   *   Add a background handler to process notifications:

       ```javascript
       import messaging from '@react-native-firebase/messaging';

       messaging().setBackgroundMessageHandler(async remoteMessage => {
         console.log('Message handled in the background!', remoteMessage);
       });

       messaging().onMessage(async remoteMessage => {
         Alert.alert('New Notification', remoteMessage.notification.body);
       });
       ```
   * Update the backend to send notifications for daily challenges, streaks, and study sessions:
     * **File**: `backend/src/notifications.py`
     *   **Code**:

         ```python
         from fastapi import APIRouter
         import firebase_admin
         from firebase_admin import credentials, messaging

         router = APIRouter()

         # Initialize Firebase Admin SDK
         cred = credentials.Certificate("path/to/firebase-adminsdk.json")
         firebase_admin.initialize_app(cred)

         @router.post("/send-notification")
         async def send_notification(user_id: str, message: str):
             try:
                 # Fetch user's device token (stored in users table)
                 conn = get_db_connection()
                 cursor = conn.cursor(cursor_factory=RealDictCursor)
                 cursor.execute("SELECT device_token FROM users WHERE user_id = %s", (user_id,))
                 user = cursor.fetchone()
                 if not user or not user['device_token']:
                     raise HTTPException(status_code=404, detail="Device token not found")

                 # Send notification
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
   * Add a device token field to the `users` table:
     * **File**: `backend/migrations/init_db.py`
     *   **Code**:

         ```sql
         ALTER TABLE users ADD COLUMN IF NOT EXISTS device_token VARCHAR(255);
         ```
   * Send notifications for daily challenges:
     * **File**: `backend/src/gamification.py`
     *   **Code**:

         ```python
         @router.post("/challenges")
         async def create_challenge(challenge: Challenge):
             conn = get_db_connection()
             cursor = conn.cursor(cursor_factory=RealDictCursor)
             try:
                 cursor.execute("""
                     INSERT INTO challenges (user_id, challenge_type, target, progress, completed, date)
                     VALUES (%s, %s, %s, 0, FALSE, CURRENT_DATE)
                     RETURNING *;
                 """, (challenge.user_id, challenge.challenge_type, challenge.target))
                 new_challenge = cursor.fetchone()
                 conn.commit()

                 # Send push notification
                 message = f"New Daily Challenge: {challenge.challenge_type} - Complete {challenge.target} tasks!"
                 await send_notification(challenge.user_id, message)

                 return new_challenge
             except Exception as e:
                 conn.rollback()
                 raise HTTPException(status_code=500, detail=str(e))
             finally:
                 cursor.close()
                 conn.close()
         ```
2. **Real Voice Input for AI Tutor**:
   *   Use `react-native-voice` for speech-to-text:

       ```
       npm install @react-native-voice/voice
       ```
   * Update the AI tutor in practice sessions to use real voice input:
     * **File**: `SATPrepSuiteMobile/src/screens/PracticeScreen.js`
     *   **Code**:

         ```javascript
         import Voice from '@react-native-voice/voice';

         const PracticeScreen = () => {
           const [isRecording, setIsRecording] = useState(false);
           const [voiceText, setVoiceText] = useState('');

           useEffect(() => {
             Voice.onSpeechResults = (e) => {
               setVoiceText(e.value[0]);
               // Send voice text to AI tutor
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
   *   Use `react-native-async-storage` for caching:

       ```
       npm install @react-native-async-storage/async-storage
       ```
   *   Update `utils/api.js` to support background sync:

       ```javascript
       import AsyncStorage from '@react-native-async-storage/async-storage';

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
       ```
   *   Add a sync trigger when the app comes online:

       ```javascript
       import NetInfo from "@react-native-community/netinfo";

       NetInfo.addEventListener(state => {
         if (state.isConnected) {
           api.sync().then(count => console.log(`Synced ${count} items`));
         }
       });
       ```

**Deliverables**

* Push notifications implemented with FCM.
* Real voice input for AI tutor using `react-native-voice`.
* Optimized offline mode with background sync using `react-native-async-storage`.

***

#### 4. Create Mobile Screens (May 6-26, 2025)

**Objective**

Create mobile screens for all major features, ensuring feature parity with the web app and a native mobile experience.

**Steps**

1. **Set Up Navigation**:
   *   Use React Navigation for a stack navigator:

       ```javascript
       import { NavigationContainer } from '@react-navigation/native';
       import { createStackNavigator } from '@react-navigation/stack';
       import OnboardingScreen from './screens/OnboardingScreen';
       import DashboardScreen from './screens/DashboardScreen';
       import PracticeScreen from './screens/PracticeScreen';
       import StudyPlanScreen from './screens/StudyPlanScreen';
       import CommunityScreen from './screens/CommunityScreen';
       import LeaderboardScreen from './screens/LeaderboardScreen';
       import RewardsStoreScreen from './screens/RewardsStoreScreen';

       const Stack = createStackNavigator();

       export default function App() {
         return (
           <NavigationContainer>
             <Stack.Navigator initialRouteName="Onboarding">
               <Stack.Screen name="Onboarding" component={OnboardingScreen} />
               <Stack.Screen name="Dashboard" component={DashboardScreen} />
               <Stack.Screen name="Practice" component={PracticeScreen} />
               <Stack.Screen name="StudyPlan" component={StudyPlanScreen} />
               <Stack.Screen name="Community" component={CommunityScreen} />
               <Stack.Screen name="Leaderboard" component={LeaderboardScreen} />
               <Stack.Screen name="RewardsStore" component={RewardsStoreScreen} />
             </Stack.Navigator>
           </NavigationContainer>
         );
       }
       ```
2. **Create Screens**:
   * **OnboardingScreen (`screens/OnboardingScreen.js`)**:
     * Port `onboarding.js` to React Native, using `StackNavigator` for step navigation.
     * Add swipe gestures for navigation between steps using `react-native-gesture-handler`.
   * **DashboardScreen (`screens/DashboardScreen.js`)**:
     * Port `dashboard.js`, displaying daily challenges, achievements, and diagnostic results.
     * Add a bottom navigation bar for quick access to Practice, Study Plan, Community, and Leaderboard.
   * **PracticeScreen (`screens/PracticeScreen.js`)**:
     * Port `practice-bluebook.js`, integrating Bluebook layouts (`ReadingWritingTest.js`, `MathBasicTest.js`).
     * Add voice input for AI tutor (as implemented above).
   * **StudyPlanScreen (`screens/StudyPlanScreen.js`)**:
     * Port `study-plan.js`, allowing users to view and adjust their study plan.
   * **CommunityScreen (`screens/CommunityScreen.js`)**:
     * Port `community.js`, supporting posts, comments, friend challenges, and team leagues.
   * **LeaderboardScreen (`screens/LeaderboardScreen.js`)**:
     * Port `leaderboard.js`, displaying skill-specific leaderboards.
   * **RewardsStoreScreen (`screens/RewardsStoreScreen.js`)**:
     * Port `rewards-store.js`, allowing users to unlock avatars and themes.
3. **Add Mobile-Specific UX**:
   * Use `react-native-gesture-handler` for swipe gestures (e.g., swipe to navigate between leaderboard tabs).
   * Add a splash screen and app icon using `react-native-splash-screen` and `react-native-vector-icons`.

**Deliverables**

* All major screens implemented (`OnboardingScreen`, `DashboardScreen`, etc.).
* Bottom navigation bar for quick access to features.
* Swipe gestures and mobile-specific UX enhancements.

***

#### 5. Testing and Debugging (May 27-June 16, 2025)

**Objective**

Test the mobile app on iOS and Android to ensure feature parity, performance, and usability, addressing any bugs or issues.

**Steps**

1. **Unit Testing**:
   *   Use Jest and React Native Testing Library to test components:

       ```javascript
       import { render, fireEvent } from '@testing-library/react-native';
       import { QuestionDisplay } from '../components/QuestionDisplay';

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
2. **End-to-End Testing**:
   * Use Detox to test the full onboarding flow, practice sessions, and push notifications on iOS and Android emulators.
   * Example: Test onboarding completion and redirection to dashboard.
3. **Device Testing**:
   * Test on physical devices (e.g., iPhone 14, Samsung Galaxy S23) and emulators (e.g., iOS Simulator, Android Emulator).
   * Verify touch interactions, swipe gestures, and offline mode functionality.
4. **Performance Testing**:
   * Measure app startup time, API latency, and offline mode performance.
   * Target: App startup < 2 seconds, API latency < 300ms (after optimization).
5. **Debugging**:
   * Use React Native Debugger and Flipper to identify and fix issues (e.g., layout bugs, API errors).
   * Ensure push notifications work reliably on both platforms.

**Deliverables**

* Unit and end-to-end tests for all major features.
* App tested on iOS and Android devices/emulators.
* Performance optimized (startup < 2s, API latency < 300ms).
* Bugs and issues resolved.

***

#### 6. App Store Submission (June 17-July 7, 2025)

**Objective**

Prepare the app for submission to the App Store (iOS) and Google Play (Android), ensuring compliance with guidelines and a smooth launch.

**Steps**

1. **Prepare App Assets**:
   * Create app icons, splash screens, and screenshots using tools like Figma.
   * Write app store descriptions:
     * **Title**: "SAT Prep Suite: AI-Powered Test Prep"
     * **Description**: "Prepare for the SAT with AI-driven practice, Bluebook-aligned tests, and gamified learning. Features adaptive diagnostics, AI tutoring, offline mode, and more!"
   * Add keywords: "SAT prep, AI tutor, Bluebook, offline, gamification".
2. **Configure Build Settings**:
   * For iOS:
     * Update `ios/SATPrepSuiteMobile/Info.plist` with app permissions (e.g., microphone for voice input, notifications).
     * Set up an Apple Developer account ($99/year) and create an App Store Connect listing.
     * Build the app: `cd ios && xcodebuild`.
   * For Android:
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

**Deliverables**

* App assets (icons, screenshots, descriptions) prepared.
* iOS and Android builds configured and submitted.
* App approved and live on App Store and Google Play.
* Launch marketing campaign initiated.

***

#### 7. Post-Launch Monitoring and Iteration (July 8-31, 2025)

**Objective**

Monitor the app’s performance, gather user feedback, and iterate to improve the mobile experience.

**Steps**

1. **Monitor Performance**:
   * Use Firebase Analytics to track user engagement (e.g., daily active users, session length, test completion rate).
   * Monitor crash reports using Firebase Crashlytics to identify and fix issues.
2. **Gather User Feedback**:
   * Add an in-app feedback form (e.g., “Rate your experience”) to collect user input.
   * Monitor app store reviews and ratings on the App Store and Google Play.
3. **Iterate Based on Feedback**:
   * Address common issues (e.g., improve push notification reliability, fix layout bugs on specific devices).
   * Add requested features (e.g., dark mode toggle, additional avatars in the Rewards Store).
4. **Optimize Engagement**:
   * Analyze push notification effectiveness (e.g., open rates) and adjust messaging (e.g., “Don’t miss today’s challenge!”).
   * Increase user retention by sending personalized notifications based on user activity (e.g., “You’re 2 days away from a 5-day streak!”).

**Deliverables**

* Analytics and crash reporting set up with Firebase.
* User feedback collected via in-app form and app store reviews.
* Initial post-launch updates released (e.g., bug fixes, minor features).
* Engagement optimized through improved notifications.

***

### Timeline

* **April 1-7, 2025**: Project setup and planning (1 week).
* **April 8-21, 2025**: Reuse and adapt existing components (2 weeks).
* **April 22-May 5, 2025**: Implement mobile-specific features (2 weeks).
* **May 6-26, 2025**: Create mobile screens (3 weeks).
* **May 27-June 16, 2025**: Testing and debugging (3 weeks).
* **June 17-July 7, 2025**: App store submission (3 weeks).
* **July 8-31, 2025**: Post-launch monitoring and iteration (3 weeks).

### Impact

* **User Acquisition**: Native apps increase active users to 2,000 by the end of 2025 (vs. current 1,000 goal), as 60% of teens prefer mobile apps (Statista, 2023).
* **Engagement**: Push notifications boost engagement by 20% (e.g., 5 challenges/week per user), improving test completion rate to 90% (vs. current 80%).
* **User Experience**: Native gestures, real voice input, and optimized offline mode improve satisfaction (NPS > 60, vs. current target of >50).
* **Market Position**: Mobile apps position SAT Prep Suite as a direct competitor to **LearnQ.ai**, which offers iOS/Android apps, enhancing its appeal to mobile-first users.

### Risks and Mitigation

* **Risk**: App store rejection due to guideline violations.
  * **Mitigation**: Ensure compliance with Apple/Google guidelines (e.g., privacy policy for voice input, content policy for educational apps) during development.
* **Risk**: Performance issues on low-end devices.
  * **Mitigation**: Test on a range of devices (e.g., iPhone 8, budget Android phones) and optimize performance (e.g., reduce image sizes, lazy-load components).
* **Risk**: User adoption slower than expected.
  * **Mitigation**: Launch a targeted marketing campaign (e.g., TikTok ads) and offer a 30-day free trial to attract users.

This plan ensures the SAT Prep Suite mobile app is developed efficiently, leveraging existing components while adding mobile-specific enhancements. Let me know if you’d like to proceed with the implementation or adjust the plan!
