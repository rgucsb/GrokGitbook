# Update with Missing Files

Let’s create the missing files identified in the compatibility and integrations analysis for the **SAT Smart Prep App** by **Learner Labs**. These files will address gaps in the web app (`frontend/`) and mobile app (`SATSmartPrepApp/`), ensuring better compatibility, integration support, and user experience. I’ll create the following files:

#### Web App Missing Files (`frontend/`)

1. **`public/service-worker.js`**: For web push notifications.
2. **`pages/unsupported-browser.js`**: A fallback page for unsupported browsers.
3. **`config/integrations.js`**: To store API keys and configuration for integrations.

#### Mobile App Missing Files (`SATSmartPrepApp/`)

1. **Updated App Icon and Splash Screen Assets**: Placeholder instructions for updating app icons and splash screens (since I cannot generate images).
2. **`src/config/integrations.js`**: To store API keys and configuration for integrations.
3. **`src/components/VoiceInputFallback.js`**: A fallback UI component for voice input failures (e.g., in offline mode).

***

### Web App Missing Files (`frontend/`)

#### 1. `public/service-worker.js` (For Web Push Notifications)

This file enables web push notifications using Firebase Cloud Messaging (FCM), similar to the mobile app’s implementation. It will handle incoming push messages and display notifications in the browser.

* **File**: `public/service-worker.js`
*   **Code**:

    ```javascript
    // Import Firebase Messaging (assumes firebase-messaging-sw.js is set up)
    importScripts('https://www.gstatic.com/firebasejs/9.0.0/firebase-app-compat.js');
    importScripts('https://www.gstatic.com/firebasejs/9.0.0/firebase-messaging-compat.js');

    // Initialize Firebase in the service worker
    firebase.initializeApp({
      apiKey: process.env.FIREBASE_API_KEY,
      authDomain: process.env.FIREBASE_AUTH_DOMAIN,
      projectId: process.env.FIREBASE_PROJECT_ID,
      storageBucket: process.env.FIREBASE_STORAGE_BUCKET,
      messagingSenderId: process.env.FIREBASE_MESSAGING_SENDER_ID,
      appId: process.env.FIREBASE_APP_ID,
    });

    const messaging = firebase.messaging();

    // Handle incoming push notifications
    self.addEventListener('push', event => {
      const data = event.data.json();
      const { title, body } = data.notification;

      const notificationOptions = {
        body,
        icon: '/favicon.ico', // Use the SAT Smart Prep App logo
        badge: '/favicon.ico',
      };

      event.waitUntil(
        self.registration.showNotification(title, notificationOptions)
      );
    });

    // Handle notification clicks (e.g., open the app)
    self.addEventListener('notificationclick', event => {
      event.notification.close();
      event.waitUntil(
        clients.openWindow('/')
      );
    });
    ```
* **Integration in `pages/_app.js`**: Register the service worker and set up Firebase for web push notifications.
  *   **Updated Code** (Add to `pages/_app.js`):

      ```javascript
      import { useEffect } from 'react';
      import firebase from 'firebase/app';
      import 'firebase/messaging';

      const firebaseConfig = {
        apiKey: process.env.FIREBASE_API_KEY,
        authDomain: process.env.FIREBASE_AUTH_DOMAIN,
        projectId: process.env.FIREBASE_PROJECT_ID,
        storageBucket: process.env.FIREBASE_STORAGE_BUCKET,
        messagingSenderId: process.env.FIREBASE_MESSAGING_SENDER_ID,
        appId: process.env.FIREBASE_APP_ID,
      };

      if (!firebase.apps.length) {
        firebase.initializeApp(firebaseConfig);
      }

      export default function MyApp({ Component, pageProps }) {
        useEffect(() => {
          // Register service worker for push notifications
          if ('serviceWorker' in navigator) {
            navigator.serviceWorker.register('/service-worker.js')
              .then(registration => {
                console.log('Service Worker registered:', registration);
              })
              .catch(error => {
                console.error('Service Worker registration failed:', error);
              });
          }

          // Request permission for push notifications
          if (Notification.permission !== 'granted') {
            Notification.requestPermission().then(permission => {
              if (permission === 'granted') {
                const messaging = firebase.messaging();
                messaging.getToken().then(token => {
                  // Send token to backend to store for push notifications
                  fetch('http://localhost:8000/auth/update-device-token/' + localStorage.getItem('user_id'), {
                    method: 'PUT',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ device_token: token })
                  });
                });
              }
            });
          }
        }, []);

        return <Component {...pageProps} />;
      }
      ```
* **Dependencies**: Add Firebase to the web app.
  * **Command**: `npm install firebase`
  *   **Environment Variables**: Add Firebase configuration to `.env`:

      ```
      FIREBASE_API_KEY=your-api-key
      FIREBASE_AUTH_DOMAIN=your-auth-domain
      FIREBASE_PROJECT_ID=your-project-id
      FIREBASE_STORAGE_BUCKET=your-storage-bucket
      FIREBASE_MESSAGING_SENDER_ID=your-sender-id
      FIREBASE_APP_ID=your-app-id
      ```
  *   **Update `next.config.js`** to load environment variables:

      ```javascript
      module.exports = {
        env: {
          FIREBASE_API_KEY: process.env.FIREBASE_API_KEY,
          FIREBASE_AUTH_DOMAIN: process.env.FIREBASE_AUTH_DOMAIN,
          FIREBASE_PROJECT_ID: process.env.FIREBASE_PROJECT_ID,
          FIREBASE_STORAGE_BUCKET: process.env.FIREBASE_STORAGE_BUCKET,
          FIREBASE_MESSAGING_SENDER_ID: process.env.FIREBASE_MESSAGING_SENDER_ID,
          FIREBASE_APP_ID: process.env.FIREBASE_APP_ID,
        },
      };
      ```

#### 2. `pages/unsupported-browser.js` (Fallback for Unsupported Browsers)

This page displays a message to users on unsupported browsers (e.g., Internet Explorer 11).

* **File**: `pages/unsupported-browser.js`
*   **Code**:

    ```javascript
    import { colors, typography } from '../styles';

    export default function UnsupportedBrowser() {
      return (
        <div className="unsupported-browser">
          <h1 style={typography.heading}>Unsupported Browser</h1>
          <p style={typography.body}>
            Your browser is not supported. Please use a modern browser like Chrome, Firefox, Safari, or Edge to access SAT Smart Prep App.
          </p>
          <p style={typography.body}>
            We recommend updating your browser or switching to a supported device for the best experience.
          </p>
          <style jsx>{`
            .unsupported-browser {
              max-width: 600px;
              margin: 50px auto;
              text-align: center;
              background: ${colors.white};
              padding: 20px;
              border-radius: 10px;
              box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
            }
          `}</style>
        </div>
      );
    }
    ```
* **Integration in `pages/_app.js`**: Add a browser compatibility check using the `bowser` library to redirect unsupported browsers to this page.
  * **Install Dependency**: `npm install bowser`
  *   **Updated Code** (Add to `pages/_app.js`):

      ```javascript
      import { useEffect } from 'react';
      import { useRouter } from 'next/router';
      import Bowser from 'bowser';

      export default function MyApp({ Component, pageProps }) {
        const router = useRouter();

        useEffect(() => {
          // Check browser compatibility
          const browser = Bowser.getParser(window.navigator.userAgent);
          const isUnsupported = browser.satisfies({
            'internet explorer': '<=11',
          });

          if (isUnsupported) {
            router.push('/unsupported-browser');
          }

          // Existing service worker and Firebase setup
          if ('serviceWorker' in navigator) {
            navigator.serviceWorker.register('/service-worker.js')
              .then(registration => {
                console.log('Service Worker registered:', registration);
              })
              .catch(error => {
                console.error('Service Worker registration failed:', error);
              });
          }

          if (Notification.permission !== 'granted') {
            Notification.requestPermission().then(permission => {
              if (permission === 'granted') {
                const messaging = firebase.messaging();
                messaging.getToken().then(token => {
                  fetch('http://localhost:8000/auth/update-device-token/' + localStorage.getItem('user_id'), {
                    method: 'PUT',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ device_token: token })
                  });
                });
              }
            });
          }
        }, [router]);

        return <Component {...pageProps} />;
      }
      ```

#### 3. `config/integrations.js` (Integration Configuration)

This file centralizes API keys and configuration for integrations, improving security and maintainability.

* **File**: `config/integrations.js`
*   **Code**:

    ```javascript
    export const integrations = {
      googleSSO: {
        clientId: process.env.GOOGLE_CLIENT_ID,
        redirectUrl: process.env.GOOGLE_REDIRECT_URL,
        scopes: ['email', 'profile'],
        serviceConfiguration: {
          authorizationEndpoint: 'https://accounts.google.com/o/oauth2/v2/auth',
          tokenEndpoint: 'https://oauth2.googleapis.com/token',
        },
      },
      firebase: {
        apiKey: process.env.FIREBASE_API_KEY,
        authDomain: process.env.FIREBASE_AUTH_DOMAIN,
        projectId: process.env.FIREBASE_PROJECT_ID,
        storageBucket: process.env.FIREBASE_STORAGE_BUCKET,
        messagingSenderId: process.env.FIREBASE_MESSAGING_SENDER_ID,
        appId: process.env.FIREBASE_APP_ID,
      },
      bluebook: {
        apiKey: process.env.BLUEBOOK_API_KEY,
        enabled: false, // Enable once integration is implemented
      },
      khanAcademy: {
        apiKey: process.env.KHAN_ACADEMY_API_KEY,
        enabled: false, // Enable once integration is implemented
      },
      canvasLMS: {
        apiKey: process.env.CANVAS_API_KEY,
        baseUrl: process.env.CANVAS_BASE_URL,
        enabled: false, // Enable once integration is implemented
      },
    };
    ```
*   **Update `.env`**:

    ```
    GOOGLE_CLIENT_ID=your-google-client-id
    GOOGLE_REDIRECT_URL=your-redirect-url
    BLUEBOOK_API_KEY=your-bluebook-api-key
    KHAN_ACADEMY_API_KEY=your-khan-academy-api-key
    CANVAS_API_KEY=your-canvas-api-key
    CANVAS_BASE_URL=your-canvas-base-url
    ```
*   **Update `next.config.js`** (already updated for Firebase, ensure all variables are included):

    ```javascript
    module.exports = {
      env: {
        FIREBASE_API_KEY: process.env.FIREBASE_API_KEY,
        FIREBASE_AUTH_DOMAIN: process.env.FIREBASE_AUTH_DOMAIN,
        FIREBASE_PROJECT_ID: process.env.FIREBASE_PROJECT_ID,
        FIREBASE_STORAGE_BUCKET: process.env.FIREBASE_STORAGE_BUCKET,
        FIREBASE_MESSAGING_SENDER_ID: process.env.FIREBASE_MESSAGING_SENDER_ID,
        FIREBASE_APP_ID: process.env.FIREBASE_APP_ID,
        GOOGLE_CLIENT_ID: process.env.GOOGLE_CLIENT_ID,
        GOOGLE_REDIRECT_URL: process.env.GOOGLE_REDIRECT_URL,
        BLUEBOOK_API_KEY: process.env.BLUEBOOK_API_KEY,
        KHAN_ACADEMY_API_KEY: process.env.KHAN_ACADEMY_API_KEY,
        CANVAS_API_KEY: process.env.CANVAS_API_KEY,
        CANVAS_BASE_URL: process.env.CANVAS_BASE_URL,
      },
    };
    ```
*   **Update Usage** (e.g., in `pages/login.js` for Google SSO):

    ```javascript
    import { integrations } from '../config/integrations';

    const handleGoogleLogin = () => {
      // Use integrations.googleSSO.clientId, etc.
      alert('Google SSO not implemented in this example');
    };
    ```

***

### Mobile App Missing Files (`SATSmartPrepApp/`)

#### 1. Updated App Icon and Splash Screen Assets

Since I cannot generate images, I’ll provide placeholder instructions for updating the app icon and splash screen assets to reflect the **SAT Smart Prep App** logo (as described earlier: a brain with a graduation cap, circuit patterns, blue gradient, "SAT Smart Prep" text).

**App Icon**

* **Description**: The app icon should be a simplified version of the logo (brain with graduation cap, blue gradient) on a white background, sized for various resolutions.
* **File Locations**:
  * **Android**: `SATSmartPrepApp/android/app/src/main/res/mipmap-*` (e.g., `mipmap-hdpi`, `mipmap-xhdpi`, `mipmap-xxhdpi`, `mipmap-xxxhdpi`).
    * Sizes: 48x48 (hdpi), 72x72 (xhdpi), 96x96 (xxhdpi), 144x144 (xxxhdpi).
  * **iOS**: `SATSmartPrepApp/ios/SATSmartPrepApp/Images.xcassets/AppIcon.appiconset/`.
    * Sizes: 20x20, 29x29, 40x40, 60x60, 76x76, 83.5x83.5, 1024x1024 (per Apple’s Human Interface Guidelines).
* **Instructions**:
  1. Use a design tool (e.g., Figma, Adobe Illustrator) to create the app icon in the required sizes.
  2. Export PNG files and place them in the respective folders.
  3.  Update `SATSmartPrepApp/android/app/src/main/AndroidManifest.xml` to reference the new icon:

      ```xml
      <application android:icon="@mipmap/ic_launcher">
      ```
  4. Update `SATSmartPrepApp/ios/SATSmartPrepApp/Images.xcassets/AppIcon.appiconset/Contents.json` to include the new icon files.

**Splash Screen**

* **Description**: The splash screen should display the full **SAT Smart Prep App** logo (brain with graduation cap, blue gradient, "SAT Smart Prep" text) centered on a white background.
* **File Locations**:
  * **Android**: `SATSmartPrepApp/android/app/src/main/res/drawable/splash.png`.
    * Size: 512x512 (scalable for different densities).
  * **iOS**: `SATSmartPrepApp/ios/SATSmartPrepApp/SplashScreen.storyboard`.
* **Instructions**:
  1. Create the splash screen image in a design tool.
  2. For Android:
     * Export `splash.png` and place it in `SATSmartPrepApp/android/app/src/main/res/drawable/`.
     *   Update `SATSmartPrepApp/android/app/src/main/res/values/styles.xml`:

         ```xml
         <style name="SplashTheme" parent="Theme.AppCompat.Light.NoActionBar">
           <item name="android:windowBackground">@drawable/splash</item>
         </style>
         ```
     *   Update `SATSmartPrepApp/android/app/src/main/AndroidManifest.xml` to use the splash theme:

         ```xml
         <activity android:name=".MainActivity" android:theme="@style/SplashTheme">
         ```
  3. For iOS:
     * Open `SplashScreen.storyboard` in Xcode.
     * Add an `ImageView` centered in the view controller, set the image to the new splash screen (add the image to `Images.xcassets` as `SplashImage`).
     *   Update `SATSmartPrepApp/ios/SATSmartPrepApp/Info.plist` to reference the storyboard:

         ```xml
         <key>UILaunchStoryboardName</key>
         <string>SplashScreen</string>
         ```

#### 2. `src/config/integrations.js` (Integration Configuration)

This file centralizes API keys and configuration for mobile app integrations.

* **File**: `SATSmartPrepApp/src/config/integrations.js`
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
        enabled: false, // Enable once integration is implemented
      },
      khanAcademy: {
        enabled: false, // Enable once integration is implemented
      },
      canvasLMS: {
        enabled: false, // Enable once integration is implemented
      },
    };
    ```
* **Install Dependency**: Use `react-native-dotenv` to load environment variables.
  * **Command**: `npm install react-native-dotenv`
  *   **Update `babel.config.js`**:

      ```javascript
      module.exports = {
        presets: ['module:metro-react-native-babel-preset'],
        plugins: [
          ['module:react-native-dotenv', {
            moduleName: '@env',
            path: '.env',
          }],
        ],
      };
      ```
  *   **Create `.env`**:

      ```
      FIREBASE_API_KEY=your-api-key
      FIREBASE_AUTH_DOMAIN=your-auth-domain
      FIREBASE_PROJECT_ID=your-project-id
      FIREBASE_STORAGE_BUCKET=your-storage-bucket
      FIREBASE_MESSAGING_SENDER_ID=your-sender-id
      FIREBASE_APP_ID=your-app-id
      GOOGLE_CLIENT_ID=your-google-client-id
      ```
*   **Update Usage** (e.g., in `App.js` for Firebase):

    ```javascript
    import { FIREBASE_API_KEY, FIREBASE_AUTH_DOMAIN, FIREBASE_PROJECT_ID, FIREBASE_STORAGE_BUCKET, FIREBASE_MESSAGING_SENDER_ID, FIREBASE_APP_ID } from '@env';
    import messaging from '@react-native-firebase/messaging';

    const firebaseConfig = {
      apiKey: FIREBASE_API_KEY,
      authDomain: FIREBASE_AUTH_DOMAIN,
      projectId: FIREBASE_PROJECT_ID,
      storageBucket: FIREBASE_STORAGE_BUCKET,
      messagingSenderId: FIREBASE_MESSAGING_SENDER_ID,
      appId: FIREBASE_APP_ID,
    };

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

#### 3. `src/components/VoiceInputFallback.js` (Fallback UI for Voice Input)

This component provides a fallback UI for users unable to use voice input (e.g., in offline mode).

* **File**: `SATSmartPrepApp/src/components/VoiceInputFallback.js`
*   **Code**:

    ```javascript
    import React from 'react';
    import { View, Text, TextInput, TouchableOpacity, StyleSheet } from 'react-native';
    import { colors, typography, commonStyles } from '../styles';

    const VoiceInputFallback = ({ chatInput, setChatInput, sendChatMessage, isOffline }) => {
      return (
        <View style={styles.container}>
          {isOffline && (
            <Text style={typography.body}>Voice input unavailable offline. Use text input instead.</Text>
          )}
          <TextInput
            style={styles.chatInput}
            value={chatInput}
            onChangeText={setChatInput}
            placeholder="Ask the AI tutor..."
            onSubmitEditing={() => sendChatMessage(chatInput)}
          />
          <TouchableOpacity style={commonStyles.button} onPress={() => sendChatMessage(chatInput)}>
            <Text style={commonStyles.buttonText}>Send</Text>
          </TouchableOpacity>
        </View>
      );
    };

    const styles = StyleSheet.create({
      container: {
        padding: 10,
      },
      chatInput: {
        width: '100%',
        marginVertical: 10,
        padding: 8,
        borderWidth: 1,
        borderColor: colors.gray,
        borderRadius: 5,
      },
    });

    export default VoiceInputFallback;
    ```
*   **Integration in `PracticeScreen.js`**: Already updated in the previous response to use this component:

    ```javascript
    import VoiceInputFallback from '../components/VoiceInputFallback';

    const PracticeScreen = ({ navigation }) => {
      return (
        <Animated.View entering={FadeIn} exiting={FadeOut} style={styles.container}>
          {/* Existing content */}
          <View style={styles.chatArea}>
            <Text style={typography.heading}>AI Tutor</Text>
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
        </Animated.View>
      );
    };
    ```

***

### Summary of Created Files

* **Web App (`frontend/`)**:
  * `public/service-worker.js`: Enables web push notifications using Firebase.
  * `pages/unsupported-browser.js`: Displays a message for unsupported browsers.
  * `config/integrations.js`: Centralizes API keys and configuration for integrations.
* **Mobile App (`SATSmartPrepApp/`)**:
  * App Icon and Splash Screen Assets: Provided instructions for updating with the new **SAT Smart Prep App** logo.
  * `src/config/integrations.js`: Centralizes API keys and configuration for mobile integrations.
  * `src/components/VoiceInputFallback.js`: Provides a fallback UI for voice input failures.

These files address the identified gaps, improving compatibility, integration support, and user experience for the **SAT Smart Prep App**. Let me know if you’d like to test these files, integrate additional features, or proceed with deployment!
