# UX improvements

To further improve the **user experience (UX)** of the **SAT Prep Suite**, we can focus on making the app more intuitive, engaging, visually appealing, and personalized beyond the current enhancements (gamification, advanced AI tutor, comprehensive analytics). The goal is to create a seamless, enjoyable, and effective experience for students and tutors as of March 26, 2025. I’ll propose a set of improvements targeting **interface design**, **interactivity**, **personalization**, and **accessibility**, then detail how to implement them in the React-based web and mobile apps. These enhancements will build on the existing functionality and address potential friction points.

***

### UX Improvement Plan

#### 1. Polished and Modern Interface Design

* **Current**: Functional UI with Material-UI (web) and basic React Native components (mobile), but lacks visual flair.
* **Improvement**: Redesign with modern aesthetics, animations, dark mode, and customizable themes to make it visually appealing and comfortable.

**Web (`web/src/`)**

*   **Dark Mode**:

    * Update `package.json`:

    ```json
    "dependencies": {
      // ... existing ...
      "@mui/system": "^5.14.0"
    }
    ```

    * Create `theme.js`:

    ```javascript
    import { createTheme } from '@mui/material/styles';

    export const lightTheme = createTheme({
      palette: { mode: 'light', primary: { main: '#1976d2' } },
    });

    export const darkTheme = createTheme({
      palette: { mode: 'dark', primary: { main: '#90caf9' } },
    });
    ```

    * Wrap app in `pages/_app.js`:

    ```javascript
    import { ThemeProvider } from '@mui/material/styles';
    import { useState } from 'react';
    import { CssBaseline, Switch, Box } from '@mui/material';
    import { Provider } from 'react-redux';
    import store from '../store';
    import { lightTheme, darkTheme } from '../theme';

    export default function MyApp({ Component, pageProps }) {
      const [isDark, setIsDark] = useState(false);
      return (
        <Provider store={store}>
          <ThemeProvider theme={isDark ? darkTheme : lightTheme}>
            <CssBaseline />
            <Box sx={{ p: 2 }}>
              <Switch checked={isDark} onChange={() => setIsDark(!isDark)} /> Dark Mode
              <Component {...pageProps} />
            </Box>
          </ThemeProvider>
        </Provider>
      );
    }
    ```
*   **Animations**:

    * Install `framer-motion`: `npm install framer-motion`.
    * Update `dashboard.js`:

    ```javascript
    import { motion } from 'framer-motion';
    // ...
    <motion.div initial={{ opacity: 0 }} animate={{ opacity: 1 }} transition={{ duration: 0.5 }}>
      <Typography variant="h4">SAT Prep</Typography>
      {/* ... rest of content ... */}
    </motion.div>
    ```

**Mobile (`mobile/src/`)**

*   **Styling**:

    * Install `react-native-paper`: `npm install react-native-paper`.
    * Update `App.js`:

    ```javascript
    import { Provider as PaperProvider, DefaultTheme, DarkTheme } from 'react-native-paper';
    import { useState } from 'react';
    // ...
    const [isDark, setIsDark] = useState(false);
    return (
      <PaperProvider theme={isDark ? DarkTheme : DefaultTheme}>
        <NavigationContainer>
          <Stack.Navigator initialRouteName="Login">
            {/* ... screens ... */}
          </Stack.Navigator>
          <Button title={isDark ? "Light Mode" : "Dark Mode"} onPress={() => setIsDark(!isDark)} />
        </NavigationContainer>
      </PaperProvider>
    );
    ```

    * Update `Practice.js`:

    ```javascript
    import { Card, Paragraph } from 'react-native-paper';
    // ...
    <FlatList
      data={session.questions}
      renderItem={({ item }) => (
        <Card style={{ marginVertical: 10 }}>
          <Card.Content>
            <Paragraph>{item.text}</Paragraph>
            <Button title="Submit Answer" onPress={() => handleSubmit(item.id)} />
          </Card.Content>
        </Card>
      )}
    />
    ```

***

#### 2. Enhanced Interactivity

* **Current**: Basic navigation and static feedback.
* **Improvement**: Add guided onboarding, interactive tooltips, and real-time progress indicators.

**Web**

*   **Onboarding**:

    * Create `components/Onboarding.js`:

    ```javascript
    import { useState } from 'react';
    import { Typography, Button, Box } from '@mui/material';

    export default function Onboarding({ onComplete }) {
      const [step, setStep] = useState(0);
      const steps = [
        "Welcome! Start with practice questions.",
        "Track your progress on the dashboard.",
        "Chat with the AI tutor for help.",
      ];

      return (
        <Box sx={{ position: 'fixed', top: 0, left: 0, right: 0, bottom: 0, bgcolor: 'rgba(0,0,0,0.5)', display: 'flex', alignItems: 'center', justifyContent: 'center' }}>
          <Box sx={{ bgcolor: 'white', p: 4, borderRadius: 2 }}>
            <Typography variant="h6">{steps[step]}</Typography>
            <Button onClick={() => step < steps.length - 1 ? setStep(step + 1) : onComplete()} sx={{ mt: 2 }}>
              {step < steps.length - 1 ? 'Next' : 'Start'}
            </Button>
          </Box>
        </Box>
      );
    }
    ```

    * Update `login.js`:

    ```javascript
    import Onboarding from '../components/Onboarding';
    // ...
    const [showOnboarding, setShowOnboarding] = useState(false);
    const handleSubmit = async () => {
      // ... login logic ...
      if (isLogin) setShowOnboarding(true);
    };
    // ...
    {showOnboarding && <Onboarding onComplete={() => setShowOnboarding(false)} />}
    ```
*   **Tooltips**:

    * Install `react-tooltip`: `npm install react-tooltip`.
    * Update `practice.js`:

    ```javascript
    import { Tooltip } from 'react-tooltip';
    // ...
    <Button variant="contained" onClick={startBlitz} data-tooltip-id="blitz-tip" data-tooltip-content="Answer 5 questions in 60 seconds!">
      Start Question Blitz
    </Button>
    <Tooltip id="blitz-tip" />
    ```

**Mobile**

*   **Progress Indicator**:

    * Update `Practice.js`:

    ```javascript
    import { ProgressBar } from 'react-native-paper';
    // ...
    const progress = session ? session.questions.filter(q => q.answer).length / session.questions.length : 0;
    // ...
    <ProgressBar progress={progress} color="#1976d2" style={{ marginVertical: 10 }} />
    ```

***

#### 3. Personalization

* **Current**: Generic experience with some adaptive features.
* **Improvement**: Add user profiles, goal setting, and tailored notifications.

**Backend**

* Update `models.py`:

```python
class User(Base):
    # ... existing ...
    goal_score = Column(Integer)
    study_hours = Column(Integer)  # Weekly target
```

* Update `auth.py`:

```python
@router.post("/register")
async def register(user: UserCreate, db: Session = Depends(get_db)):
    # ... existing ...
    db_user.goal_score = user.goal_score or 1200
    db_user.study_hours = user.study_hours or 10
```

**Web**

* Update `login.js` (register form):

```javascript
const [goalScore, setGoalScore] = useState(1200);
const [studyHours, setStudyHours] = useState(10);
// ...
{!isLogin && (
  <>
    <TextField label="Goal Score" type="number" value={goalScore} onChange={(e) => setGoalScore(e.target.value)} fullWidth margin="normal" />
    <TextField label="Weekly Study Hours" type="number" value={studyHours} onChange={(e) => setStudyHours(e.target.value)} fullWidth margin="normal" />
  </>
)}
// ...
await api.register(userId, name, email, password, { goal_score: goalScore, study_hours: studyHours });
```

* Update `dashboard.js`:

```javascript
const [userProfile, setUserProfile] = useState(null);
useEffect(() => {
  api.getUserId().then(async (userId) => {
    // ... existing ...
    const profile = await api.getUserProfile(userId);  // New endpoint
    setUserProfile(profile);
  });
}, [dispatch]);
// ...
{userProfile && (
  <Typography>Goal: {userProfile.goal_score} | Weekly Hours: {userProfile.study_hours}</Typography>
)}
```

* Add to `api.js`:

```javascript
async getUserProfile(userId) {
  return (await api.get(`/auth/profile/${userId}`)).data;
},
```

**Mobile**

* Add profile display to `Practice.js`:

```javascript
const [profile, setProfile] = useState(null);
useEffect(() => {
  // ... existing ...
  api.getUserProfile(userId).then(setProfile);
});
// ...
{profile && <Text>Goal: {profile.goal_score} | Hours: {profile.study_hours}</Text>}
```

***

#### 4. Accessibility

* **Current**: Basic accessibility (e.g., keyboard navigation).
* **Improvement**: Ensure WCAG compliance, screen reader support, and high-contrast mode.

**Web**

* Update `practice.js`:

```javascript
<ListItem key={q.id} role="listitem" aria-label={`Question: ${q.text}`}>
  <Typography>{q.text}</Typography>
  <Button aria-label="Submit answer" onClick={() => handleSubmit(q.id)}>Submit</Button>
</ListItem>
```

* Add high-contrast toggle in `_app.js`:

```javascript
const highContrastTheme = createTheme({
  palette: { mode: 'light', contrastThreshold: 4.5 },
});
// ...
const [isHighContrast, setIsHighContrast] = useState(false);
// ...
<Switch checked={isHighContrast} onChange={() => setIsHighContrast(!isHighContrast)} /> High Contrast
<ThemeProvider theme={isHighContrast ? highContrastTheme : (isDark ? darkTheme : lightTheme)}>
```

**Mobile**

* Update `Practice.js`:

```javascript
<Text accessibilityLabel={`Question: ${item.text}`} style={{ fontSize: 16 }}>{item.text}</Text>
<Button title="Submit Answer" accessibilityLabel="Submit answer" onPress={() => handleSubmit(item.id)} />
```

***

### Testing Plan

#### Setup

* Install dependencies: `cd web && npm install framer-motion react-tooltip @mui/system`.
* Mobile: `cd mobile && npm install react-native-paper`.
* Start backend: `cd api && uvicorn main:app --reload`.
* Web: `cd web && npm run dev`.
* Mobile: `cd mobile && npx react-native run-android`.

#### Test Cases

1. **Interface**:
   * Web: Toggle dark mode → UI switches, animations fade in.
   * Mobile: Switch themes → Cards render with new colors.
2. **Interactivity**:
   * Web: Login → Onboarding appears, tooltips show on hover.
   * Mobile: Submit practice → Progress bar updates.
3. **Personalization**:
   * Register with goal (1400) → Displays on dashboard/practice.
4. **Accessibility**:
   * Web: Tab through practice → All elements focusable, screen reader reads questions.
   * Mobile: VoiceOver (iOS) reads question text.

#### Expected Results

* Smooth transitions, no lag.
* Onboarding guides new users.
* Profile data persists.
* WCAG compliance (e.g., 4.5:1 contrast in high-contrast mode).

***

### Conclusion

* **Interface**: Modern, animated, themeable.
* **Interactivity**: Onboarding, tooltips, progress indicators.
* **Personalization**: Goals and hours tailor the experience.
* **Accessibility**: Inclusive for all users.

These changes elevate UX significantly. Test them—let me know if you want more (e.g., push notifications) or refinements!
