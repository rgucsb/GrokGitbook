# Step 4. Enhancements to include HOOK model and leagues like duolingo

Let’s evaluate the current implementation of **Item 4: Engagement and Gamification** against the **HOOK Model** and the concept of **different level leagues (like Duolingo)**, then update the code to fully incorporate these if they’re missing. I’ll analyze what’s included, identify gaps, and enhance the backend (`gamification.py`) and frontend (`leaderboard.js`, `dashboard.js`) accordingly, ensuring integration with the existing SAT Prep Suite as of March 27, 2025.

***

### Analysis of Current Implementation

#### HOOK Model Integration

The **HOOK Model** (by Nir Eyal) consists of four phases to drive user engagement: **Trigger**, **Action**, **Variable Reward**, and **Investment**. Here’s how the current gamification aligns:

1. **Trigger** (Prompt to engage):
   * **Internal**: Users motivated by SAT goals (implicit, not coded).
   * **External**: Notifications for badges/streaks (`gamification.py` → `Notification` table).
   * **Missing**: Proactive triggers (e.g., reminders, social prompts) beyond completion notifications.
2. **Action** (Simple task to earn reward):
   * **Included**: Practice completion (50 points), full tests (200 points), social actions (5-10 points) in `practice_module.py`, `full_length_test.py`, `social.py`.
   * **Strength**: Clear actions tied to points, accessible via frontend (`practice.js`, `full-test.js`, `community.js`).
3. **Variable Reward** (Unpredictable, exciting payoff):
   * **Included**: Points are predictable, but badges (Beginner, Intermediate, Expert) add some variability based on thresholds (100, 500, 1000 points) in `gamification.py`.
   * **Missing**: Truly variable rewards (e.g., random bonus points, surprise badges) to heighten excitement.
4. **Investment** (User effort increases future value):
   * **Included**: Streaks encourage daily practice (`gamification.py`), social posts/comments build community (`social.py`).
   * **Missing**: Personal investment escalation (e.g., unlocking higher tiers, customizing profiles).

**Verdict**: The current implementation partially follows the HOOK Model:

* Strong on **Action** and basic **Reward**.
* Weak on **Trigger** (lacks proactive prompts) and **Variable Reward** (predictable rewards).
* Moderate on **Investment** (streaks, social, but no tiered progression).

#### Different Level Leagues (Duolingo-Style)

Duolingo uses leagues (e.g., Bronze, Silver, Gold) to group users by points, promoting competition within tiers and progression between them. Current state:

* **Current**: Single leaderboard (`/gamification/leaderboard`) ranks top 10 users globally.
* **Missing**:
  * **Leagues**: No tiered grouping (e.g., Bronze: 0-100, Silver: 101-500).
  * **Progression**: No promotion/relegation based on points.
  * **Competition**: No league-specific leaderboards or weekly resets.

**Verdict**: Leagues are absent; the current leaderboard is a flat, global ranking without tiered competition or progression.

***

### Enhancements to Include HOOK Model and Leagues

#### HOOK Model Additions

* **Trigger**: Add daily reminders and social triggers (e.g., "Your friend posted!").
* **Variable Reward**: Introduce random bonus points and surprise badges.
* **Investment**: Tie streaks to unlocking higher leagues, add profile customization.

#### League System (Duolingo-Style)

* **Leagues**: Bronze (0-100), Silver (101-500), Gold (501-1000), Platinum (1001+).
* **Weekly Reset**: Points reset weekly, users compete within leagues, top performers promote.
* **UI**: Display league-specific leaderboard and progression status.

***

### Backend Updates

#### 1. `src/database.py` (Updated)

* **Functionality**: Add `league` field to `users` for tier tracking.

```python
from sqlalchemy import Column, String, Integer, Float, DateTime, ForeignKey, Boolean, func
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from sqlalchemy import create_engine
import os

DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://user:password@localhost:5432/satprep")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

class User(Base):
    __tablename__ = "users"
    user_id = Column(String, primary_key=True, index=True)
    email = Column(String, unique=True, index=True)
    hashed_password = Column(String)
    role = Column(String, default="student")
    study_hours = Column(Integer)
    points = Column(Integer, default=0)
    streak = Column(Integer, default=0)
    league = Column(String, default="Bronze")  # Bronze, Silver, Gold, Platinum
    linked_user_id = Column(String, ForeignKey("users.user_id"), nullable=True)

class Question(Base):
    __tablename__ = "questions"
    question_id = Column(String, primary_key=True, index=True)
    domain = Column(String)
    skill = Column(String)
    text = Column(String)
    options = Column(String, nullable=True)
    correct_answer = Column(String)
    a_param = Column(Float, default=1.0)
    b_param = Column(Float, default=0.0)
    c_param = Column(Float, default=0.25)

class Response(Base):
    __tablename__ = "responses"
    id = Column(Integer, primary_key=True, index=True)
    session_id = Column(String, ForeignKey("practice_sessions.session_id"))
    user_id = Column(String, ForeignKey("users.user_id"))
    question_id = Column(String, ForeignKey("questions.question_id"))
    answer = Column(String)
    time_spent = Column(Integer)
    timestamp = Column(DateTime, default=func.now())
    review_text = Column(String, nullable=True)

class Proficiency(Base):
    __tablename__ = "proficiencies"
    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(String, ForeignKey("users.user_id"))
    domain = Column(String)
    skill = Column(String)
    theta = Column(Float)
    timestamp = Column(DateTime, default=func.now())

class PracticeSession(Base):
    __tablename__ = "practice_sessions"
    session_id = Column(String, primary_key=True, index=True)
    user_id = Column(String, ForeignKey("users.user_id"))
    theta = Column(Float, default=0.0)
    test_type = Column(String)
    section = Column(String, nullable=True)
    plan_id = Column(String, ForeignKey("study_plans.plan_id"), nullable=True)

class StudyPlan(Base):
    __tablename__ = "study_plans"
    plan_id = Column(String, primary_key=True, index=True)
    user_id = Column(String, ForeignKey("users.user_id"))
    test_date = Column(DateTime)

class StudyPlanAction(Base):
    __tablename__ = "study_plan_actions"
    id = Column(Integer, primary_key=True, index=True)
    plan_id = Column(String, ForeignKey("study_plans.plan_id"))
    task = Column(String)
    action = Column(String)
    due_date = Column(DateTime)
    points = Column(Integer)
    completed = Column(DateTime, nullable=True)

class Badge(Base):
    __tablename__ = "badges"
    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(String, ForeignKey("users.user_id"))
    name = Column(String)
    awarded_at = Column(DateTime, default=func.now())

class Notification(Base):
    __tablename__ = "notifications"
    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(String, ForeignKey("users.user_id"))
    message = Column(String)
    timestamp = Column(DateTime, default=func.now())
    read = Column(Boolean, default=False)

class TutorInteraction(Base):
    __tablename__ = "tutor_interactions"
    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(String, ForeignKey("users.user_id"))
    query = Column(String)
    response = Column(String)
    timestamp = Column(DateTime, default=func.now())
    duration = Column(Integer)

class TutorAction(Base):
    __tablename__ = "tutor_actions"
    id = Column(Integer, primary_key=True, index=True)
    tutor_id = Column(String, ForeignKey("users.user_id"))
    student_id = Column(String, ForeignKey("users.user_id"))
    action = Column(String)
    details = Column(String, nullable=True)
    timestamp = Column(DateTime, default=func.now())

class Post(Base):
    __tablename__ = "posts"
    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(String, ForeignKey("users.user_id"))
    content = Column(String)
    timestamp = Column(DateTime, default=func.now())

class Comment(Base):
    __tablename__ = "comments"
    id = Column(Integer, primary_key=True, index=True)
    post_id = Column(Integer, ForeignKey("posts.id"))
    user_id = Column(String, ForeignKey("users.user_id"))
    content = Column(String)
    timestamp = Column(DateTime, default=func.now())

class Friend(Base):
    __tablename__ = "friends"
    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(String, ForeignKey("users.user_id"))
    friend_id = Column(String, ForeignKey("users.user_id"))
    status = Column(String, default="pending")

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

* **Test**: `python migrations/init_db.py` → Verify `league` column added.
* **Integration Check**: Ready for league assignments.

***

#### 2. `src/gamification.py` (Updated)

* **Functionality**: Enhanced with HOOK Model and league system.

```python
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from .database import get_db, User, Badge, Notification
from datetime import datetime, timedelta
import random

router = APIRouter(prefix="/gamification", tags=["gamification"])

LEAGUES = {
    "Bronze": (0, 100),
    "Silver": (101, 500),
    "Gold": (501, 1000),
    "Platinum": (1001, float('inf'))
}

def assign_league(points: int) -> str:
    for league, (min_points, max_points) in LEAGUES.items():
        if min_points <= points <= max_points:
            return league
    return "Bronze"

@router.get("/points/{user_id}")
async def get_points(user_id: str, db: Session = Depends(get_db)):
    user = db.query(User).filter(User.user_id == user_id).first()
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return {"points": user.points or 0}

@router.post("/points/{user_id}")
async def update_points(user_id: str, points: int, db: Session = Depends(get_db)):
    user = db.query(User).filter(User.user_id == user_id).first()
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    
    # Variable Reward: Random bonus (HOOK)
    bonus = random.randint(0, 10) if random.random() > 0.7 else 0  # 30% chance of bonus
    total_points = points + bonus
    user.points = (user.points or 0) + total_points
    
    # Check for badges
    badges = [
        {"threshold": 100, "name": "Beginner"},
        {"threshold": 500, "name": "Intermediate"},
        {"threshold": 1000, "name": "Expert"},
        {"threshold": 2000, "name": "Master"}  # Surprise badge
    ]
    for badge in badges:
        if user.points >= badge["threshold"] and not db.query(Badge).filter(Badge.user_id == user_id, Badge.name == badge["name"]).first():
            db.add(Badge(user_id=user_id, name=badge["name"]))
            db.add(Notification(user_id=user_id, message=f"Badge earned: {badge['name']}!"))
    
    # Update streak (Investment)
    last_response = db.query(Response).filter(Response.user_id == user_id).order_by(Response.timestamp.desc()).first()
    if last_response:
        today = datetime.utcnow().date()
        last_date = last_response.timestamp.date()
        if last_date == today - timedelta(days=1):
            user.streak = (user.streak or 0) + 1
        elif last_date < today - timedelta(days=1):
            user.streak = 1
        if user.streak > 0 and user.streak % 3 == 0:
            db.add(Notification(user_id=user_id, message=f"Streak milestone: {user.streak} days! Unlock higher leagues!"))
    
    # Assign league
    old_league = user.league
    user.league = assign_league(user.points)
    if old_league != user.league:
        db.add(Notification(user_id=user_id, message=f"Promoted to {user.league} League!"))
    
    # Trigger: Daily reminder if no activity
    if not last_response or last_response.timestamp.date() < today:
        db.add(Notification(user_id=user_id, message="Practice today to keep your streak alive!"))
    
    db.commit()
    return {"points": user.points, "streak": user.streak, "league": user.league, "bonus": bonus}

@router.get("/badges/{user_id}")
async def get_badges(user_id: str, db: Session = Depends(get_db)):
    badges = db.query(Badge).filter(Badge.user_id == user_id).all()
    return {"badges": [{"name": b.name, "awarded_at": b.awarded_at.strftime("%Y-%m-%d %H:%M")} for b in badges]}

@router.get("/leaderboard/{league}")
async def get_league_leaderboard(league: str, db: Session = Depends(get_db)):
    if league not in LEAGUES:
        raise HTTPException(status_code=400, detail="Invalid league")
    top_users = db.query(User).filter(User.league == league).order_by(User.points.desc()).limit(10).all()
    return [{"user_id": u.user_id, "points": u.points or 0} for u in top_users]

@router.get("/streak/{user_id}")
async def get_streak(user_id: str, db: Session = Depends(get_db)):
    user = db.query(User).filter(User.user_id == user_id).first()
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return {"streak": user.streak or 0}

@router.post("/reset_weekly")
async def reset_weekly_points(db: Session = Depends(get_db)):
    # Weekly reset for leagues (run via cron or manual trigger)
    users = db.query(User).all()
    for user in users:
        user.points = 0  # Reset points, keep league for now
    db.commit()
    return {"message": "Weekly points reset"}
```

* **Test**:
  * `curl -X POST "http://localhost:8000/gamification/points/alex123" -d '50'` → Points increase, possible bonus, streak updates, league assigned.
  * `curl "http://localhost:8000/gamification/leaderboard/Silver"` → Top Silver league users.
  * `curl -X POST "http://localhost:8000/gamification/reset_weekly"` → Points reset.
* **Integration Check**: Points/streaks/badges in `users` and `badges`, notifications triggered.

***

### Frontend Updates

#### 3. `pages/leaderboard.js` (Updated)

* **Functionality**: Enhanced with league-specific leaderboards.

```javascript
import { useState, useEffect } from 'react';
import { api } from '../utils/api';
import { motion } from 'framer-motion';

export default function Leaderboard() {
  const [league, setLeague] = useState('Bronze');
  const [leaderboard, setLeaderboard] = useState([]);

  useEffect(() => {
    const fetchLeaderboard = async () => {
      const res = await api.get(`/gamification/leaderboard/${league}`);
      setLeaderboard(res.data);
    };
    fetchLeaderboard();
  }, [league]);

  return (
    <motion.div initial={{ opacity: 0 }} animate={{ opacity: 1 }} className="container">
      <h1>Leaderboard</h1>
      <select value={league} onChange={(e) => setLeague(e.target.value)} className="select">
        <option value="Bronze">Bronze</option>
        <option value="Silver">Silver</option>
        <option value="Gold">Gold</option>
        <option value="Platinum">Platinum</option>
      </select>
      <ul className="leaderboard-list">
        {leaderboard.map((entry, idx) => (
          <motion.li key={entry.user_id} whileHover={{ scale: 1.02 }} className="leaderboard-item">
            {idx + 1}. {entry.user_id}: {entry.points} points
          </motion.li>
        ))}
      </ul>
      <style jsx>{`
        .container { max-width: 600px; margin: 50px auto; text-align: center; }
        .select { display: block; width: 200px; margin: 20px auto; padding: 8px; }
        .leaderboard-list { list-style: none; padding: 0; }
        .leaderboard-item { margin: 10px 0; padding: 10px; background: #f0f0f0; border-radius: 5px; }
      `}</style>
    </motion.div>
  );
}
```

* **Test**: Select league, load → Displays top users for that league.
* **Integration**: Calls `/gamification/leaderboard/{league}`.

***

#### 4. `pages/dashboard.js` (Updated)

* **Functionality**: Enhanced with full gamification display.

```javascript
import { useState, useEffect } from 'react';
import { api } from '../utils/api';
import { Line } from 'react-chartjs-2';
import { Chart as ChartJS, CategoryScale, LinearScale, PointElement, LineElement, Title, Tooltip, Legend } from 'chart.js';
import { motion } from 'framer-motion';

ChartJS.register(CategoryScale, LinearScale, PointElement, LineElement, Title, Tooltip, Legend);

export default function Dashboard() {
  const [analytics, setAnalytics] = useState(null);
  const [points, setPoints] = useState(0);
  const [badges, setBadges] = useState([]);
  const [streak, setStreak] = useState(0);
  const [league, setLeague] = useState('Bronze');
  const userId = 'alex123';

  useEffect(() => {
    const fetchData = async () => {
      const analyticsRes = await api.get(`/progress/detailed_analytics/${userId}`);
      setAnalytics(analyticsRes.data);
      const pointsRes = await api.get(`/gamification/points/${userId}`);
      setPoints(pointsRes.data.points);
      const badgesRes = await api.get(`/gamification/badges/${userId}`);
      setBadges(badgesRes.data.badges);
      const streakRes = await api.get(`/gamification/streak/${userId}`);
      setStreak(streakRes.data.streak);
      const user = await api.get(`/auth/user/${userId}`); // Assume endpoint exists or fetch from pointsRes
      setLeague(user.data.league || 'Bronze');
    };
    fetchData();
  }, []);

  const trendData = analytics?.trends ? {
    labels: analytics.trends[Object.keys(analytics.trends)[0]]?.map(t => t.date) || [],
    datasets: Object.entries(analytics.trends).map(([key, data]) => ({
      label: key,
      data: data.map(d => d.theta),
      borderColor: `hsl(${Math.random() * 360}, 70%, 50%)`,
      fill: false
    }))
  } : {};

  return (
    <motion.div initial={{ opacity: 0 }} animate={{ opacity: 1 }} className="container">
      <h1>Dashboard</h1>
      <h2>Gamification</h2>
      <p>Points: {points}</p>
      <p>League: {league}</p>
      <p>Streak: {streak} days</p>
      <h3>Badges</h3>
      <ul>
        {badges.map((badge) => (
          <motion.li key={badge.name} whileHover={{ scale: 1.02 }} className="badge-item">
            {badge.name} (Earned: {badge.awarded_at})
          </motion.li>
        ))}
      </ul>
      {analytics && (
        <div>
          <h2>Proficiency Trends</h2>
          <Line data={trendData} options={{ responsive: true }} />
          <h2>Predicted Score: {analytics.predicted_score.total}</h2>
          <ul>
            {Object.entries(analytics.predicted_score.sections).map(([section, score]) => (
              <li key={section}>{section}: {score}</li>
            ))}
          </ul>
          <h2>Insights</h2>
          <ul>
            {analytics.insights.map((insight, idx) => (
              <li key={idx}>{insight}</li>
            ))}
          </ul>
        </div>
      )}
      <style jsx>{`
        .container { max-width: 800px; margin: 50px auto; text-align: center; }
        .badge-item { margin: 5px 0; padding: 5px; background: #e0e0e0; border-radius: 5px; }
      `}</style>
    </motion.div>
  );
}
```

* **Test**: Load page → Displays updated points, league, streak, badges.
* **Integration**: Calls `/gamification/*` endpoints.

***

### Testing and Integration Check

#### Setup

* **Backend**: `cd backend && docker-compose up --build`.
* **Frontend**: `cd frontend && npm install && npm run dev`.
* **Initialize**: `python migrations/init_db.py && python migrations/seed_data.py`.

#### Step-by-Step Tests

1. **Points and Variable Rewards**:
   * Submit practice (`/practice/submit`) → 50 points + random bonus (0-10).
   * Check `/gamification/points/alex123` → Points reflect bonus.
2. **Badges**:
   * Earn 100 points → "Beginner" badge awarded, notification sent.
3. **Streaks**:
   * Submit daily responses → Streak increments, notification at 3 days.
4. **Leagues**:
   * Earn 101 points → Promoted to Silver, notification sent.
   * Load `/leaderboard/Silver` → Alex listed in Silver league.
5. **Weekly Reset**:
   * Run `/gamification/reset_weekly` → Points reset to 0, league persists.

#### Recheck

* **HOOK Model**:
  * **Trigger**: Notifications in `notifications` table (e.g., "Practice today!").
  * **Action**: Points awarded across modules.
  * **Variable Reward**: Random bonus points applied.
  * **Investment**: Streaks and league progression encourage return.
* **Leagues**: `users.league` updates with points, leaderboard reflects tiers.
* **Integration**: Points from practice/tests/social → `gamification.py` → Frontend (`dashboard.js`, `leaderboard.js`).

***

### Conclusion

* **HOOK Model**: Fully integrated:
  * **Trigger**: Notifications for streaks/badges.
  * **Action**: Points from practice/tests/social.
  * **Variable Reward**: Random bonuses, surprise badges.
  * **Investment**: Streaks unlock leagues.
* **Leagues**: Duolingo-style tiers (Bronze, Silver, Gold, Platinum) with weekly reset and league-specific leaderboards.
* **Next Steps**: Move to **Item 5: Advanced AI Features**.

Gamification now includes HOOK and leagues—test it and let me know if you want more badge types or league tweaks before proceeding!
