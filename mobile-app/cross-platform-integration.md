# Cross Platform Integration

To ensure a seamless experience for users across the **SAT Prep Suite** mobile app (to be developed for iOS and Android) and the existing web app, we need to integrate the two platforms so that they share data, maintain consistent functionality, and provide a unified user experience (UX). The goal is to allow users to switch between the web app (accessed via desktop or mobile browsers) and the mobile app (native iOS/Android apps) without losing progress, settings, or preferences. This integration will leverage the existing FastAPI backend, shared state management, synchronized data, and a consistent UI/UX design. Below, I’ll explain the integration strategy, focusing on **data synchronization**, **authentication**, **UI/UX consistency**, **feature parity**, and **offline support**, ensuring a seamless experience for users as of March 27, 2025.

***

### Integration Strategy for Mobile App and Web App

#### 1. Shared Backend and Data Synchronization

The mobile app and web app will use the same FastAPI backend (`backend/src/`), ensuring that all data (e.g., user profiles, responses, study plans, gamification progress) is stored centrally in the PostgreSQL database. This allows for real-time synchronization across platforms.

**Steps for Data Synchronization**

1. **Centralized Database**:
   * Both the web app and mobile app will interact with the same PostgreSQL database via the FastAPI backend (`backend/src/main.py`).
   * Existing tables (`users`, `responses`, `proficiencies`, `study_plans`, `challenges`, `rewards`, etc.) already store user data centrally, ensuring consistency.
   * Example: When a user completes a practice session on the mobile app, the response is saved to the `responses` table via the `/practice/submit/{session_id}` endpoint, and the web app immediately reflects this progress when the user logs in.
2. **Real-Time Updates with WebSocket**:
   * Use the existing WebSocket implementation (`backend/src/ai_tutor.py`) for real-time updates (e.g., AI tutor chat, gamification notifications).
   * Extend WebSocket to broadcast updates to both platforms:
     * **File**: `backend/src/gamification.py`
     *   **Code**:

         ```python
         from fastapi import WebSocket
         from typing import Dict

         active_connections: Dict[str, WebSocket] = {}

         @router.websocket("/gamification/updates/{user_id}")
         async def gamification_updates(websocket: WebSocket, user_id: str):
             await websocket.accept()
             active_connections[user_id] = websocket
             try:
                 while True:
                     await websocket.receive_text()
             except Exception as e:
                 print(f"WebSocket error: {e}")
             finally:
                 del active_connections[user_id]

         async def broadcast_gamification_update(user_id: str, message: str):
             if user_id in active_connections:
                 await active_connections[user_id].send_text(message)
         ```
   * Update endpoints to broadcast updates:
     *   Example: When a user earns coins, notify both platforms:

         ```python
         @router.post("/coins/earn")
         async def earn_coins(coin_earn: CoinEarn):
             conn = get_db_connection()
             cursor = conn.cursor(cursor_factory=RealDictCursor)
             try:
                 cursor.execute("""
                     UPDATE users
                     SET coins = coins + %s
                     WHERE user_id = %s
                     RETURNING coins;
                 """, (coin_earn.amount, coin_earn.user_id))
                 updated_user = cursor.fetchone()
                 if not updated_user:
                     raise HTTPException(status_code=404, detail="User not found")
                 conn.commit()
                 await broadcast_gamification_update(coin_earn.user_id, f"You earned {coin_earn.amount} coins!")
                 return {"coins": updated_user['coins']}
             except Exception as e:
                 conn.rollback()
                 raise HTTPException(status_code=500, detail=str(e))
             finally:
                 cursor.close()
                 conn.close()
         ```
   * On the mobile app, connect to the WebSocket:
     * **File**: `SATPrepSuiteMobile/src/utils/api.js`
     *   **Code**:

         ```javascript
         import { w3cwebsocket as W3CWebSocket } from 'websocket';

         export const connectWebSocket = (userId, onMessage) => {
           const ws = new W3CWebSocket(`ws://localhost:8000/gamification/updates/${userId}`);
           ws.onmessage = (message) => onMessage(message.data);
           return ws;
         };
         ```
   *   On the web app, update `utils/api.js` to connect to the WebSocket:

       ```javascript
       export const connectWebSocket = (userId, onMessage) => {
         const ws = new WebSocket(`ws://localhost:8000/gamification/updates/${userId}`);
         ws.onmessage = (message) => onMessage(message.data);
         return ws;
       };
       ```
3. **Offline Synchronization**:
   * Both platforms already support offline mode (`utils/api.js` caches questions and queues requests).
   *   Ensure the mobile app uses `react-native-async-storage` for caching, as implemented in the mobile app plan:

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
   * The web app uses local storage for caching, but the logic is identical, ensuring that both platforms sync seamlessly when online.

**Impact**

* Users can start a practice session on the web app, switch to the mobile app, and continue where they left off (e.g., responses are saved in the `responses` table).
* Gamification progress (e.g., coins, badges) updates in real-time across platforms via WebSocket.
* Offline mode ensures users can practice on either platform without interruption, with data syncing when online.

***

#### 2. Unified Authentication

Ensure users can log in once and access their account seamlessly across both platforms using the same credentials.

**Steps for Authentication**

1. **Shared Authentication**:
   * Both platforms use the same `/auth/login` and `/auth/signup` endpoints (`backend/src/auth.py`).
   * Store the `user_id` in local storage (web) and AsyncStorage (mobile) after login:
     * **Web**: `localStorage.setItem('user_id', userId)`
     * **Mobile**: `await AsyncStorage.setItem('user_id', userId)`
   *   Example: Update `SATPrepSuiteMobile/src/screens/LoginScreen.js`:

       ```javascript
       import { useState } from 'react';
       import { View, TextInput, TouchableOpacity, Text, StyleSheet } from 'react-native';
       import { api } from '../utils/api';
       import AsyncStorage from '@react-native-async-storage/async-storage';

       const LoginScreen = ({ navigation }) => {
         const [email, setEmail] = useState('');
         const [password, setPassword] = useState('');

         const handleLogin = async () => {
           try {
             const res = await api.post('/auth/login', { email, password });
             await AsyncStorage.setItem('user_id', res.data.user_id);
             navigation.navigate('Onboarding');
           } catch (error) {
             alert('Login failed: ' + error.message);
           }
         };

         return (
           <View style={styles.container}>
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
             <TouchableOpacity style={styles.button} onPress={handleLogin}>
               <Text style={styles.buttonText}>Login</Text>
             </TouchableOpacity>
           </View>
         );
       };

       const styles = StyleSheet.create({
         container: {
           flex: 1,
           justifyContent: 'center',
           padding: 20,
         },
         input: {
           borderWidth: 1,
           borderColor: '#ddd',
           borderRadius: 5,
           padding: 10,
           marginBottom: 10,
         },
         button: {
           backgroundColor: '#0070f3',
           padding: 10,
           borderRadius: 5,
           alignItems: 'center',
         },
         buttonText: {
           color: 'white',
           fontWeight: 'bold',
         },
       });

       export default LoginScreen;
       ```
2. **Session Persistence**:
   * Use the `user_id` stored in local storage/AsyncStorage to authenticate API requests on both platforms.
   * Add a logout feature to clear the `user_id`:
     * **Web**: `localStorage.removeItem('user_id')`
     * **Mobile**: `await AsyncStorage.removeItem('user_id')`
3. **Single Sign-On (SSO)**:
   * The onboarding wizard already supports SSO (Google, Apple, Meta) on the web (`pages/onboarding.js`).
   *   Implement SSO on mobile using `react-native-app-auth`:

       ```javascript
       import { authorize } from 'react-native-app-auth';

       const config = {
         clientId: 'your-client-id',
         redirectUrl: 'com.satprepsuite:/oauthredirect',
         scopes: ['email', 'profile'],
         serviceConfiguration: {
           authorizationEndpoint: 'https://accounts.google.com/o/oauth2/v2/auth',
           tokenEndpoint: 'https://oauth2.googleapis.com/token',
         },
       };

       const handleGoogleLogin = async () => {
         try {
           const result = await authorize(config);
           const res = await api.post('/auth/sso', { token: result.accessToken });
           await AsyncStorage.setItem('user_id', res.data.user_id);
           navigation.navigate('Onboarding');
         } catch (error) {
           alert('SSO failed: ' + error.message);
         }
       };
       ```

**Impact**

* Users can log in on the web app and immediately access their account on the mobile app (and vice versa) without re-authenticating.
* SSO ensures a consistent login experience across platforms, reducing friction.

***

#### 3. Consistent UI/UX Design

Ensure the mobile app mirrors the web app’s Bluebook-inspired UI while adapting to mobile-specific design patterns for a seamless experience.

**Steps for UI/UX Consistency**

1. **Shared Design System**:
   * Create a shared design system for both platforms:
     * **Colors**: Blue (`#0070f3`), white (`#fff`), gray (`#ddd`), as used in the web app.
     * **Typography**: Use the same font (e.g., `Arial` or a custom font like `Roboto`) with consistent sizes (e.g., 18px for headings, 16px for body text).
     * **Components**: Reuse styles for buttons, inputs, and cards (e.g., `borderRadius: 5`, `padding: 10`).
   *   Example: Create a `styles.js` file for shared styles:

       ```javascript
       export const colors = {
         primary: '#0070f3',
         white: '#fff',
         gray: '#ddd',
         text: '#333',
       };

       export const typography = {
         heading: { fontSize: 18, fontWeight: '500', color: colors.text },
         body: { fontSize: 16, color: colors.text },
       };

       export const commonStyles = {
         button: {
           backgroundColor: colors.primary,
           padding: 10,
           borderRadius: 5,
           alignItems: 'center',
         },
         buttonText: {
           color: colors.white,
           fontWeight: 'bold',
         },
         card: {
           borderWidth: 1,
           borderColor: colors.gray,
           borderRadius: 5,
           padding: 20,
           marginBottom: 16,
         },
       };
       ```
   * Use these styles in both web and mobile components:
     * **Web**: `style={commonStyles.button}`
     * **Mobile**: `style={commonStyles.button}`
2. **Adapt for Mobile**:
   * Adjust layouts for smaller screens (e.g., stack elements vertically instead of side-by-side for Reading/Writing layout).
   * Use mobile-specific design patterns:
     * Bottom navigation bar on mobile (vs. sidebar on web).
     * Swipe gestures for navigation (e.g., in onboarding wizard).
   *   Example: Update `ReadingWritingTest.js` for mobile:

       ```javascript
       import { View, StyleSheet } from 'react-native';
       import { PassageDisplay } from '../PassageDisplay';
       import { QuestionDisplay } from '../QuestionDisplay';
       import { commonStyles, colors } from '../../styles';

       const ReadingWritingTest = ({ eliminatorActive, setEliminatorActive, questionNumber, totalQuestions, timer, onNext, onPrevious, userName, onTimeEnd, customButtons }) => {
         const passageText = `While researching a topic, a student has taken the following notes...`;

         return (
           <View style={styles.container}>
             <View style={styles.passage}>
               <PassageDisplay passageText={passageText} />
             </View>
             <View style={styles.question}>
               <QuestionDisplay
                 questionNumber={questionNumber}
                 eliminatorActive={eliminatorActive}
                 toggleEliminator={setEliminatorActive}
               />
             </View>
           </View>
         );
       };

       const styles = StyleSheet.create({
         container: {
           flex: 1,
         },
         passage: {
           flex: 1,
           borderBottomWidth: 1,
           borderBottomColor: colors.gray,
           padding: 20,
         },
         question: {
           flex: 1,
           padding: 20,
         },
       });

       export default ReadingWritingTest;
       ```
3. **Consistent Animations**:
   * Use Framer Motion on the web and `react-native-reanimated` on mobile for consistent animations (e.g., fade transitions in onboarding).
   *   Example: Update `OnboardingScreen.js` to use `react-native-reanimated`:

       ```javascript
       import Animated, { FadeIn, FadeOut } from 'react-native-reanimated';

       const WelcomeStep = ({ nextStep }) => (
         <Animated.View entering={FadeIn} exiting={FadeOut} style={styles.step}>
           <Text style={styles.heading}>Welcome to LearnerLabs SAT Prep!</Text>
           <Text style={styles.body}>Let’s create your personalized study plan.</Text>
           <TouchableOpacity style={styles.button} onPress={nextStep}>
             <Text style={styles.buttonText}>Next</Text>
           </TouchableOpacity>
         </Animated.View>
       );
       ```
4. **Theme Persistence**:
   * If a user unlocks a theme (e.g., "Dark Mode") on the web app, it should apply on the mobile app.
   * Store the user’s theme preference in the `users` table:
     * **File**: `backend/migrations/init_db.py`
     *   **Code**:

         ```sql
         ALTER TABLE users ADD COLUMN IF NOT EXISTS theme VARCHAR(50) DEFAULT 'light';
         ```
   * Update the backend to save and retrieve the theme:
     * **File**: `backend/src/auth.py`
     *   **Code**:

         ```python
         @router.put("/update-theme/{user_id}")
         async def update_theme(user_id: str, theme: str):
             conn = get_db_connection()
             cursor = conn.cursor(cursor_factory=RealDictCursor)
             try:
                 cursor.execute("""
                     UPDATE users
                     SET theme = %s
                     WHERE user_id = %s
                     RETURNING theme;
                 """, (theme, user_id))
                 updated_user = cursor.fetchone()
                 if not updated_user:
                     raise HTTPException(status_code=404, detail="User not found")
                 conn.commit()
                 return {"theme": updated_user['theme']}
             except Exception as e:
                 conn.rollback()
                 raise HTTPException(status_code=500, detail=str(e))
             finally:
                 cursor.close()
                 conn.close()
         ```
   * Fetch and apply the theme on both platforms:
     * **Web**: Update `pages/_app.js` to fetch the theme on app load.
     * **Mobile**: Update `App.js` to fetch the theme on app start.

**Impact**

* Users experience the same Bluebook-inspired design (colors, typography, layouts) on both platforms.
* Mobile-specific adaptations (e.g., bottom navigation, swipe gestures) ensure a native feel while maintaining consistency.
* Theme preferences (e.g., "Dark Mode") persist across platforms, enhancing the seamless experience.

***

#### 4. Feature Parity and Platform-Specific Enhancements

Ensure all features are available on both platforms, with mobile-specific enhancements to leverage native capabilities.

**Steps for Feature Parity**

1. **Feature Parity**:
   * All web features (diagnostics, practice, AI tutor, gamification, social features, offline mode, onboarding) are already ported to the mobile app as part of the mobile app development plan (e.g., `OnboardingScreen.js`, `PracticeScreen.js`).
   * Example: The onboarding wizard (`OnboardingScreen.js`) mirrors the web version (`onboarding.js`), with the same 8 steps and gamification features (XP, avatar, bonus challenge).
2. **Mobile-Specific Enhancements**:
   * **Push Notifications**: Already implemented in the mobile app plan, sending reminders for daily challenges, streaks, and study sessions.
   * **Real Voice Input**: Added to the AI tutor on mobile using `react-native-voice`, providing a more natural interaction compared to the web app’s simulated voice input.
   * **Native Gestures**: Swipe gestures for navigation (e.g., in onboarding, leaderboards) using `react-native-gesture-handler`.
   * **Device Integration**: Use the mobile device’s microphone for real voice input, with plans to add camera integration for scanning practice tests in the future.
3. **Cross-Platform State Management**:
   * Use React Context to manage app state (e.g., user data, onboarding progress) on both platforms:
     * **File**: `SATPrepSuiteMobile/src/context/AppContext.js`
     *   **Code**:

         ```javascript
         import React, { createContext, useState, useEffect } from 'react';
         import AsyncStorage from '@react-native-async-storage/async-storage';
         import { api } from '../utils/api';

         export const AppContext = createContext();

         export const AppProvider = ({ children }) => {
           const [user, setUser] = useState(null);

           useEffect(() => {
             const loadUser = async () => {
               const userId = await AsyncStorage.getItem('user_id');
               if (userId) {
                 const res = await api.get(`/auth/user/${userId}`);
                 setUser(res.data);
               }
             };
             loadUser();
           }, []);

           return (
             <AppContext.Provider value={{ user, setUser }}>
               {children}
             </AppContext.Provider>
           );
         };
         ```
   * Use the same context on the web app, replacing AsyncStorage with local storage.

**Impact**

* Users can access all features (e.g., practice sessions, AI tutor, gamification) on both platforms with no loss of functionality.
* Mobile-specific enhancements (e.g., push notifications, real voice input) improve the mobile experience while maintaining consistency with the web app.

***

#### 5. Seamless Offline Support

Ensure offline mode works consistently across both platforms, with seamless synchronization when the user comes online.

**Steps for Offline Support**

1. **Unified Offline Logic**:
   * Both platforms use the same offline logic in `utils/api.js`, caching questions and queuing requests:
     * Web: Uses `localStorage` for caching.
     * Mobile: Uses `react-native-async-storage` for caching.
   *   The mobile app adds background sync using `react-native-background-fetch`:

       ```javascript
       import BackgroundFetch from 'react-native-background-fetch';

       const configureBackgroundSync = () => {
         BackgroundFetch.configure({
           minimumFetchInterval: 15, // Fetch every 15 minutes
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
       ```
2. **Consistent User Experience**:
   * Display a consistent offline message on both platforms (e.g., “You’re offline. Your progress will sync when you’re back online.”).
   * Ensure cached content (e.g., practice questions) is available on both platforms, with the mobile app pre-caching more aggressively due to network variability.

**Impact**

* Users can practice offline on either platform, with responses syncing seamlessly when online.
* Background sync on mobile ensures data is updated even when the app is not in use, providing a more reliable experience.

***

#### 6. Testing and Validation

Test the integration to ensure a seamless experience across platforms.

**Steps for Testing**

1. **Cross-Platform Testing**:
   * Test a user journey across platforms:
     * Start onboarding on the web app, complete steps 1-4.
     * Switch to the mobile app, continue from step 5, and complete onboarding.
     * Start a practice session on mobile, switch to web, and verify progress.
   * Verify gamification progress (e.g., coins, badges) updates in real-time on both platforms.
2. **Offline Testing**:
   * Test offline mode on both platforms:
     * Cache questions on the web app, go offline, and complete a practice session.
     * Switch to the mobile app, go offline, and continue the session.
     * Come online on either platform and verify sync.
3. **UI/UX Testing**:
   * Test UI consistency (e.g., colors, typography, layouts) on web (desktop/mobile browser) and mobile (iOS/Android).
   * Verify animations (e.g., onboarding transitions) are smooth on both platforms.
4. **Performance Testing**:
   * Measure API latency on both platforms, targeting < 300ms after optimization.
   * Test app startup time on mobile (target: < 2 seconds).

**Impact**

* Ensures users experience no disruptions when switching between platforms.
* Validates that offline mode, gamification, and UI/UX are consistent across web and mobile.

***

### Summary of Integration

* **Data Synchronization**: Shared FastAPI backend and PostgreSQL database ensure all user data (e.g., responses, gamification progress) is synchronized in real-time via WebSocket and offline sync.
* **Authentication**: Unified login with SSO support allows users to access their account seamlessly across platforms.
* **UI/UX Consistency**: Shared design system (colors, typography, components) and consistent animations ensure a familiar experience, with mobile-specific adaptations (e.g., bottom navigation, swipe gestures).
* **Feature Parity**: All features are available on both platforms, with mobile enhancements (e.g., push notifications, real voice input) improving the mobile experience.
* **Offline Support**: Consistent offline mode with background sync on mobile ensures uninterrupted usage across platforms.

### Impact on User Experience

* **Seamless Transition**: Users can start on the web app (e.g., complete onboarding steps 1-4), switch to the mobile app (e.g., finish onboarding, start a practice session), and continue on the web app without losing progress.
* **Consistent Experience**: The Bluebook-aligned UI, gamification features (e.g., daily challenges, leaderboards), and study plan look and feel the same on both platforms, with mobile-specific enhancements improving usability.
* **Enhanced Engagement**: Push notifications on mobile remind users to engage (e.g., “Don’t miss today’s challenge!”), while real voice input makes the AI tutor more interactive, increasing overall engagement (target: 90% test completion rate, 5 challenges/week per user).
* **Accessibility**: Offline mode ensures users can study anywhere, with seamless sync across platforms, making the app accessible to rural and low-income students.

This integration ensures a seamless experience across the web and mobile platforms, positioning the SAT Prep Suite to compete effectively with mobile-first competitors like **LearnQ.ai**. Let me know if you’d like to proceed with the mobile app development or focus on another aspect of the integration!
