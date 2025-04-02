# Improved Gamification

To enhance the **SAT Prep Suite** by introducing gamification, I’ll implement the suggested improvements—points, badges, leaderboards, mini-games (e.g., “Question Blitz” mode), and streak rewards—into the React-based web and mobile apps. These features will boost engagement, drawing inspiration from LearnQ.ai’s Duolingo-like approach, while maintaining integration with the existing FastAPI backend. Since the current app is functional but lacks engagement hooks, I’ll modify the necessary files to add these elements as of March 26, 2025. Below, I’ll outline the enhancements, update the relevant code, and provide a testing plan to ensure everything works seamlessly.

***

### Gamification Enhancements

#### 1. Points System

* **Goal**: Award points for completing practice questions, tests, and study plan tasks.
* **Implementation**: Track points in Redux (web) and local state (mobile), persist via backend.

**Backend Addition**

* Add `/gamification/points/{user_id}` endpoint to store/retrieve points:

```python
# api/routes/gamification.py (new file)
from fastapi import APIRouter, Depends
from sqlalchemy.orm import Session
from ..database import get_db
from ..models import User

router = APIRouter(prefix="/gamification", tags=["gamification"])

@router.get("/points/{user_id}")
async def get_points(user_id: str, db: Session = Depends(get_db)):
    user = db.query(User).filter(User.user_id == user_id).first()
    return {"points": user.points if user else 0}

@router.post("/points/{user_id}")
async def update_points(user_id: str, points: int, db: Session = Depends(get_db)):
    user = db.query(User).filter(User.user_id == user_id).first()
    if not user:
        user = User(user_id=user_id, points=0)
        db.add(user)
    user.points = (user.points or 0) + points
    db.commit()
    return {"points": user.points}
```

* Update `api/models.py`:

```python
class User(Base):
    __tablename__ = "users"
    # ... existing fields ...
    points = Column(Integer, default=0)
```

* Register in `api/main.py`:

```python
from routes import gamification
app.include_router(gamification.router)
```

**Web (`web/src/`)**

* Update `api.js`:

```javascript
async getPoints(userId) {
  return (await api.get(`/gamification/points/${userId}`)).data.points;
},
async updatePoints(userId, points) {
  return (await api.post(`/gamification/points/${userId}`, points)).data.points;
},
```

* Add `pointsSlice.js`:

```javascript
// web/src/store/slices/pointsSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import api from '../../shared-utils/api';

export const fetchPoints = createAsyncThunk('points/fetch', async (userId) => {
  return await api.getPoints(userId);
});

export const addPoints = createAsyncThunk('points/add', async ({ userId, points }) => {
  return await api.updatePoints(userId, points);
});

const pointsSlice = createSlice({
  name: 'points',
  initialState: { value: 0, loading: false },
  extraReducers: (builder) => {
    builder
      .addCase(fetchPoints.fulfilled, (state, action) => {
        state.value = action.payload;
        state.loading = false;
      })
      .addCase(addPoints.fulfilled, (state, action) => {
        state.value = action.payload;
        state.loading = false;
      });
  },
});

export default pointsSlice.reducer;
```

* Update `store/index.js`:

```javascript
import pointsReducer from './slices/pointsSlice';
// ...
export default configureStore({
  reducer: {
    skills: skillsReducer,
    practice: practiceReducer,
    points: pointsReducer,
  },
});
```

* Update `dashboard.js`:

```javascript
import { fetchPoints, addPoints } from '../store/slices/pointsSlice';
// ...
const points = useSelector((state) => state.points.value);
// ...
useEffect(() => {
  api.getUserId().then((userId) => {
    dispatch(fetchSkills(userId));
    dispatch(fetchPoints(userId));
  });
}, [dispatch]);
// ...
<Typography>Points: {points}</Typography>
```

* Update `practice.js` (award 10 points per correct answer):

```javascript
import { addPoints } from '../store/slices/pointsSlice';
// ...
const handleSubmit = (questionId) => {
  dispatch(submitPractice({ practiceId: session.practice_id, responses: [{ question_id: questionId, answer: 'A' }] })).then(() => {
    api.getUserId().then((userId) => dispatch(addPoints({ userId, points: 10 })));
  });
};
```

**Mobile (`mobile/src/`)**

* Update `api.js`:

```javascript
async getPoints(userId) {
  return (await api.get(`/gamification/points/${userId}`)).data.points;
},
async updatePoints(userId, points) {
  return (await api.post(`/gamification/points/${userId}`, points)).data.points;
},
```

* Update `Practice.js`:

```javascript
const [points, setPoints] = useState(0);
// ...
useEffect(() => {
  api.getUserId().then((userId) => {
    api.startPractice(userId).then(setSession);
    api.getPoints(userId).then(setPoints);
  }).catch(() => navigation.navigate('Login'));
}, [navigation]);
// ...
const handleSubmit = (questionId) => {
  api.submitPractice(session.practice_id, [{ question_id: questionId, answer: 'A' }]).then(() => {
    api.getUserId().then((userId) => api.updatePoints(userId, 10).then(setPoints));
  });
};
// ...
<Text style={{ fontSize: 18 }}>Points: {points}</Text>
```

***

#### 2. Badges

* **Goal**: Award badges like “Algebra Ace” for 90% accuracy in a skill.
* **Implementation**: Store badges in backend, display in UI.

**Backend**

* Update `models.py`:

```python
class Badge(Base):
    __tablename__ = "badges"
    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(String, ForeignKey("users.user_id"))
    name = Column(String)
    earned_at = Column(DateTime, default=func.now())
```

* Add to `gamification.py`:

```python
@router.get("/badges/{user_id}")
async def get_badges(user_id: str, db: Session = Depends(get_db)):
    badges = db.query(Badge).filter(Badge.user_id == user_id).all()
    return {"badges": [{"name": b.name, "earned_at": b.earned_at} for b in badges]}

@router.post("/badges/{user_id}")
async def award_badge(user_id: str, badge_name: str, db: Session = Depends(get_db)):
    badge = Badge(user_id=user_id, name=badge_name)
    db.add(badge)
    db.commit()
    return {"name": badge.name, "earned_at": badge.earned_at}
```

* Update `practice_module.py` (check accuracy):

```python
@router.post("/submit/{practice_id}")
async def submit_practice(practice_id: str, responses: List[ResponseCreate], db: Session = Depends(get_db)):
    # ... existing logic ...
    accuracy = sum(1 for r, q in zip(responses_list, questions) if r.answer == q.correct_answer) / len(questions)
    if accuracy >= 0.9:
        user_id = practice.user_id
        skill = questions[0].skill  # Simplified: assumes all questions same skill
        db.add(Badge(user_id=user_id, name=f"{skill} Ace"))
        db.commit()
    # ...
```

**Web**

* Update `api.js`:

```javascript
async getBadges(userId) {
  return (await api.get(`/gamification/badges/${userId}`)).data.badges;
},
```

* Update `dashboard.js`:

```javascript
const [badges, setBadges] = useState([]);
useEffect(() => {
  api.getUserId().then((userId) => {
    dispatch(fetchSkills(userId));
    dispatch(fetchPoints(userId));
    api.getBadges(userId).then(setBadges);
  });
}, [dispatch]);
// ...
<Typography>Badges: {badges.map(b => b.name).join(', ')}</Typography>
```

**Mobile**

* Update `api.js`:

```javascript
async getBadges(userId) {
  return (await api.get(`/gamification/badges/${userId}`)).data.badges;
},
```

* Update `Practice.js`:

```javascript
const [badges, setBadges] = useState([]);
useEffect(() => {
  api.getUserId().then((userId) => {
    api.startPractice(userId).then(setSession);
    api.getPoints(userId).then(setPoints);
    api.getBadges(userId).then(setBadges);
  }).catch(() => navigation.navigate('Login'));
}, [navigation]);
// ...
<Text style={{ fontSize: 18 }}>Badges: {badges.map(b => b.name).join(', ')}</Text>
```

***

#### 3. Leaderboards

* **Goal**: Display top users by points.
* **Implementation**: Add endpoint, show on dashboard.

**Backend**

* Add to `gamification.py`:

```python
@router.get("/leaderboard")
async def get_leaderboard(db: Session = Depends(get_db)):
    top_users = db.query(User).order_by(User.points.desc()).limit(10).all()
    return [{"user_id": u.user_id, "points": u.points or 0} for u in top_users]
```

**Web**

* Update `api.js`:

```javascript
async getLeaderboard() {
  return (await api.get('/gamification/leaderboard')).data;
},
```

* Update `dashboard.js`:

```javascript
const [leaderboard, setLeaderboard] = useState([]);
useEffect(() => {
  api.getUserId().then((userId) => {
    dispatch(fetchSkills(userId));
    dispatch(fetchPoints(userId));
    api.getBadges(userId).then(setBadges);
    api.getLeaderboard().then(setLeaderboard);
  });
}, [dispatch]);
// ...
<Typography variant="h6">Leaderboard</Typography>
<List>
  {leaderboard.map((entry, index) => (
    <ListItem key={entry.user_id}>
      <Typography>{`${index + 1}. ${entry.user_id}: ${entry.points} points`}</Typography>
    </ListItem>
  ))}
</List>
```

**Mobile**

* Not implemented (mobile scope limited to practice/study plan).

***

#### 4. Mini-Games: Question Blitz Mode

* **Goal**: Timed mode (e.g., 5 questions in 60 seconds).
* **Implementation**: Add mode to practice.

**Backend**

* Update `practice_module.py`:

```python
@router.post("/start/{user_id}")
async def start_practice(user_id: str, mode: str = "normal", db: Session = Depends(get_db)):
    questions = db.query(Question).order_by(func.random()).limit(5 if mode == "blitz" else 10).all()
    # ... rest unchanged ...
```

**Web**

* Update `practice.js`:

```javascript
const [timeLeft, setTimeLeft] = useState(60);
const [mode, setMode] = useState('normal');

useEffect(() => {
  api.getUserId().then((userId) => {
    dispatch(fetchPractice(userId, mode));
    if (mode === 'blitz') {
      const timer = setInterval(() => setTimeLeft((t) => t > 0 ? t - 1 : (clearInterval(timer), 0)), 1000);
      return () => clearInterval(timer);
    }
  });
}, [dispatch, mode]);

const startBlitz = () => setMode('blitz');
// ...
<Button variant="contained" onClick={startBlitz} sx={{ mt: 2 }}>Start Question Blitz</Button>
{mode === 'blitz' && <Typography>Time Left: {timeLeft}s</Typography>}
{timeLeft === 0 && <Typography>Time’s up! Submit your answers.</Typography>}
```

**Mobile**

* Update `Practice.js`:

```javascript
const [timeLeft, setTimeLeft] = useState(60);
const [mode, setMode] = useState('normal');

useEffect(() => {
  api.getUserId().then((userId) => {
    api.startPractice(userId, mode).then(setSession);
    api.getPoints(userId).then(setPoints);
    api.getBadges(userId).then(setBadges);
    if (mode === 'blitz') {
      const timer = setInterval(() => setTimeLeft((t) => t > 0 ? t - 1 : (clearInterval(timer), 0)), 1000);
      return () => clearInterval(timer);
    }
  }).catch(() => navigation.navigate('Login'));
}, [navigation, mode]);

const startBlitz = () => setMode('blitz');
// ...
<Button title="Start Question Blitz" onPress={startBlitz} />
{mode === 'blitz' && <Text style={{ fontSize: 18 }}>Time Left: {timeLeft}s</Text>}
{timeLeft === 0 && <Text>Time’s up! Submit your answers.</Text>}
```

***

#### 5. Streak Rewards

* **Goal**: Reward daily streaks with bonus content (e.g., AI tips).
* **Implementation**: Track streaks, unlock tips.

**Backend**

* Update `models.py`:

```python
class User(Base):
    # ...
    streak = Column(Integer, default=0)
    last_active = Column(DateTime)
```

* Add to `gamification.py`:

```python
@router.post("/streak/{user_id}")
async def update_streak(user_id: str, db: Session = Depends(get_db)):
    user = db.query(User).filter(User.user_id == user_id).first()
    now = datetime.utcnow()
    if not user.last_active or (now - user.last_active).days >= 1:
        user.streak = 0 if (now - user.last_active).days > 1 else (user.streak or 0) + 1
    user.last_active = now
    db.commit()
    tips = ["Focus on pacing during tests."] if user.streak >= 3 else []
    return {"streak": user.streak, "tips": tips}
```

* Update `practice_module.py` (call on submit):

```python
await update_streak(user_id, db)
```

**Web**

* Update `api.js`:

```javascript
async updateStreak(userId) {
  return (await api.post(`/gamification/streak/${userId}`)).data;
},
```

* Update `practice.js`:

```javascript
const [streak, setStreak] = useState(0);
const [tips, setTips] = useState([]);
const handleSubmit = (questionId) => {
  dispatch(submitPractice({ practiceId: session.practice_id, responses: [{ question_id: questionId, answer: 'A' }] })).then(() => {
    api.getUserId().then((userId) => {
      dispatch(addPoints({ userId, points: 10 }));
      api.updateStreak(userId).then(({ streak, tips }) => {
        setStreak(streak);
        setTips(tips);
      });
    });
  });
};
// ...
<Typography>Streak: {streak} days</Typography>
{tips.length > 0 && <Typography>AI Tip: {tips[0]}</Typography>}
```

**Mobile**

* Update `api.js`:

```javascript
async updateStreak(userId) {
  return (await api.post(`/gamification/streak/${userId}`)).data;
},
```

* Update `Practice.js`:

```javascript
const [streak, setStreak] = useState(0);
const [tips, setTips] = useState([]);
const handleSubmit = (questionId) => {
  api.submitPractice(session.practice_id, [{ question_id: questionId, answer: 'A' }]).then(() => {
    api.getUserId().then((userId) => {
      api.updatePoints(userId, 10).then(setPoints);
      api.updateStreak(userId).then(({ streak, tips }) => {
        setStreak(streak);
        setTips(tips);
      });
    });
  });
};
// ...
<Text style={{ fontSize: 18 }}>Streak: {streak} days</Text>
{tips.length > 0 && <Text style={{ fontSize: 16 }}>AI Tip: {tips[0]}</Text>}
```

***

### Testing Plan

#### Setup

* Update DB schema: `python migrations/init_db.py` (add `points`, `streak`, `last_active` to `users`, create `badges` table).
* Start backend: `cd api && uvicorn main:app --reload`.
* Web: `cd web && npm run dev`.
* Mobile: `cd mobile && npx react-native run-android`.

#### Test Cases

1. **Points**:
   * Web/Mobile: Submit practice → Points increase by 10.
2. **Badges**:
   * Submit practice with 90%+ accuracy → “Algebra Ace” appears on dashboard.
3. **Leaderboard**:
   * Web: Multiple users submit practice → Top 10 display on dashboard.
4. **Question Blitz**:
   * Web/Mobile: Start blitz → 5 questions, timer counts down, submits at 0.
5. **Streaks**:
   * Submit practice daily → Streak increments, tip unlocks at 3 days.

#### Expected Results

* No errors in API calls or UI updates.
* Points, badges, streaks persist across sessions.
* Blitz mode enforces time limit.

***

### Conclusion

These enhancements add robust gamification:

* **Points**: Tracked and displayed.
* **Badges**: Awarded for skill mastery.
* **Leaderboards**: Web-only ranking.
* **Question Blitz**: Timed mini-game.
* **Streaks**: Unlock AI tips.

All integrate with the backend and UI. Test it out—let me know if you want more badges, mini-games, or UI polish!
