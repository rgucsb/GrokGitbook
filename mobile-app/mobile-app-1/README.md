# Mobile app -1

I apologize for the repeated interruptions in completing the task. The complexity of creating a mobile app and updating the web app for tight integration, combined with the need to ensure no steps are missed, has led to challenges in completing the response in a single go. To address this, I’ll break the task into smaller, more manageable steps, ensuring each step is concise and focused. This approach will allow us to systematically build the mobile app and update the web app while maintaining tight integration. I’ll double-check each step to ensure completeness and clarity, and I’ll proceed step-by-step to avoid getting stuck.

The task is to create a mobile app for the **SAT Prep Suite** (also referred to as **LearnerLabs SAT Prep App**) for iOS and Android, while updating the existing web app to ensure a seamless user experience across both platforms. The plan will cover project setup, component reuse, mobile-specific features, backend updates, screen creation, testing, deployment, and post-launch iteration, aligning with the April-July 2025 timeline.

***

### Step-by-Step Plan to Create the Mobile App and Update the Web App for Tight Integration

#### Step 1: Project Setup and Initial Backend Updates (April 1-7, 2025)

**Objective**

Set up the mobile app project, update the web app for integration, and prepare the backend to support real-time updates and mobile-specific features.

**Sub-Steps**

1. **Choose React Native for Mobile App**:
   * Use React Native to build cross-platform iOS and Android apps, leveraging existing React components from the web app (`frontend/components/`).
   * **Reason**: React Native allows code reuse, reducing development time while providing native performance.
2. **Set Up Mobile Development Environment**:
   * Install Node.js (v18+), React Native CLI, and dependencies on development machines.
   * Set up iOS (Xcode 15+) and Android (Android Studio 2023+) development environments.
   * Install required libraries: `react-native`, `react-native-navigation`, `axios`, `react-native-async-storage`, `react-native-firebase`, `react-native-voice`, `react-native-gesture-handler`, `react-native-reanimated`, `react-native-background-fetch`, `react-native-app-auth`.
   * Command: `npx react-native init SATPrepSuiteMobile`.
3. **Create Mobile Project Structure**:
   *   Structure the project:

       ```
       SATPrepSuiteMobile/
       ├── src/
       │   ├── components/       # Reused React components (e.g., QuestionDisplay, PassageDisplay)
       │   ├── screens/          # Mobile screens (e.g., OnboardingScreen, DashboardScreen)
       │   ├── utils/            # API and offline utilities
       │   ├── context/          # Shared state management (AppContext)
       │   ├── assets/           # Images, icons, fonts
       │   ├── navigation/       # Navigation setup
       │   └── styles/           # Shared design system (colors, typography)
       ├── App.js                # Main app entry point
       ├── android/              # Android-specific files
       ├── ios/                  # iOS-specific files
       └── package.json
       ```
4. **Update Backend for Integration**:
   * Add `theme` and `device_token` fields to the `users` table to support theme persistence and push notifications:
     * **File**: `backend/migrations/init_db.py`
     *   **Code**:

         ```python
         import psycopg2
         from psycopg2 import sql

         def init_db():
             conn = psycopg2.connect(
                 dbname="satprep",
                 user="user",
                 password="password",
                 host="localhost",
                 port="5432"
             )
             cursor = conn.cursor()

             # Existing tables
             cursor.execute("""
                 CREATE TABLE IF NOT EXISTS users (
                     user_id VARCHAR(255) PRIMARY KEY,
                     email VARCHAR(255) UNIQUE,
                     hashed_password VARCHAR(255),
                     role VARCHAR(50) DEFAULT 'student',
                     study_hours INTEGER,
                     points INTEGER DEFAULT 0,
                     streak INTEGER DEFAULT 0,
                     league VARCHAR(50) DEFAULT 'Bronze',
                     linked_user_id VARCHAR(255) REFERENCES users(user_id),
                     coins INTEGER DEFAULT 0,
                     full_name VARCHAR(255),
                     grade VARCHAR(50),
                     school VARCHAR(255),
                     sat_taken BOOLEAN DEFAULT FALSE,
                     sat_math_score INTEGER,
                     sat_reading_score INTEGER,
                     sat_test_date TIMESTAMP,
                     study_hours_per_week INTEGER,
                     study_days_per_week INTEGER,
                     preferred_study_time VARCHAR(50),
                     onboarding_completed BOOLEAN DEFAULT FALSE
                 );
             """)

             # Add new columns for theme and device token
             cursor.execute("""
                 ALTER TABLE users
                 ADD COLUMN IF NOT EXISTS theme VARCHAR(50) DEFAULT 'light',
                 ADD COLUMN IF NOT EXISTS device_token VARCHAR(255);
             """)

             # Other existing tables (abridged for brevity)
             cursor.execute("""
                 CREATE TABLE IF NOT EXISTS questions (
                     question_id VARCHAR(255) PRIMARY KEY,
                     domain VARCHAR(255),
                     skill VARCHAR(255),
                     text TEXT,
                     options TEXT,
                     correct_answer TEXT,
                     a_param FLOAT DEFAULT 1.0,
                     b_param FLOAT DEFAULT 0.0,
                     c_param FLOAT DEFAULT 0.25
                 );
             """)

             cursor.execute("""
                 CREATE TABLE IF NOT EXISTS responses (
                     id SERIAL PRIMARY KEY,
                     session_id VARCHAR(255),
                     user_id VARCHAR(255) REFERENCES users(user_id),
                     question_id VARCHAR(255) REFERENCES questions(question_id),
                     answer TEXT,
                     time_spent INTEGER,
                     timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                     review_text TEXT
                 );
             """)

             cursor.execute("""
                 CREATE TABLE IF NOT EXISTS proficiencies (
                     id SERIAL PRIMARY KEY,
                     user_id VARCHAR(255) REFERENCES users(user_id),
                     domain VARCHAR(255),
                     skill VARCHAR(255),
                     theta FLOAT,
                     timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                 );
             """)

             cursor.execute("""
                 CREATE TABLE IF NOT EXISTS practice_sessions (
                     session_id VARCHAR(255) PRIMARY KEY,
                     user_id VARCHAR(255) REFERENCES users(user_id),
                     theta FLOAT DEFAULT 0.0,
                     test_type VARCHAR(50),
                     section VARCHAR(50),
                     plan_id VARCHAR(255)
                 );
             """)

             cursor.execute("""
                 CREATE TABLE IF NOT EXISTS study_plans (
                     plan_id VARCHAR(255) PRIMARY KEY,
                     user_id VARCHAR(255) REFERENCES users(user_id),
                     test_date TIMESTAMP
                 );
             """)

             cursor.execute("""
                 CREATE TABLE IF NOT EXISTS study_plan_actions (
                     id SERIAL PRIMARY KEY,
                     plan_id VARCHAR(255) REFERENCES study_plans(plan_id),
                     task VARCHAR(255),
                     action TEXT,
                     due_date TIMESTAMP,
                     points INTEGER,
                     completed TIMESTAMP
                 );
             """)

             cursor.execute("""
                 CREATE TABLE IF NOT EXISTS badges (
                     id SERIAL PRIMARY KEY,
                     user_id VARCHAR(255) REFERENCES users(user_id),
                     name VARCHAR(255),
                     awarded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                 );
             """)

             cursor.execute("""
                 CREATE TABLE IF NOT EXISTS notifications (
                     id SERIAL PRIMARY KEY,
                     user_id VARCHAR(255) REFERENCES users(user_id),
                     message TEXT,
                     timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                     read BOOLEAN DEFAULT FALSE
                 );
             """)

             cursor.execute("""
                 CREATE TABLE IF NOT EXISTS tutor_interactions (
                     id SERIAL PRIMARY KEY,
                     user_id VARCHAR(255) REFERENCES users(user_id),
                     query TEXT,
                     response TEXT,
                     timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                     duration INTEGER
                 );
             """)

             cursor.execute("""
                 CREATE TABLE IF NOT EXISTS tutor_actions (
                     id SERIAL PRIMARY KEY,
                     tutor_id VARCHAR(255) REFERENCES users(user_id),
                     student_id VARCHAR(255) REFERENCES users(user_id),
                     action VARCHAR(255),
                     details TEXT,
                     timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                 );
             """)

             cursor.execute("""
                 CREATE TABLE IF NOT EXISTS posts (
                     id SERIAL PRIMARY KEY,
                     user_id VARCHAR(255) REFERENCES users(user_id),
                     content TEXT,
                     timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                 );
             """)

             cursor.execute("""
                 CREATE TABLE IF NOT EXISTS comments (
                     id SERIAL PRIMARY KEY,
                     post_id INTEGER REFERENCES posts(id),
                     user_id VARCHAR(255) REFERENCES users(user_id),
                     content TEXT,
                     timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                 );
             """)

             cursor.execute("""
                 CREATE TABLE IF NOT EXISTS friends (
                     id SERIAL PRIMARY KEY,
                     user_id VARCHAR(255) REFERENCES users(user_id),
                     friend_id VARCHAR(255) REFERENCES users(user_id),
                     status VARCHAR(50) DEFAULT 'pending',
                     timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                 );
             """)

             cursor.execute("""
                 CREATE TABLE IF NOT EXISTS challenges (
                     id SERIAL PRIMARY KEY,
                     user_id VARCHAR(255) REFERENCES users(user_id),
                     challenge_type VARCHAR(50),
                     target INTEGER,
                     progress INTEGER DEFAULT 0,
                     completed BOOLEAN DEFAULT FALSE,
                     date DATE DEFAULT CURRENT_DATE
                 );
             """)

             cursor.execute("""
                 CREATE TABLE IF NOT EXISTS rewards (
                     id SERIAL PRIMARY KEY,
                     user_id VARCHAR(255) REFERENCES users(user_id),
                     reward_type VARCHAR(50),
                     reward_id VARCHAR(255),
                     unlocked_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                 );
             """)

             cursor.execute("""
                 CREATE TABLE IF NOT EXISTS team_leagues (
                     id SERIAL PRIMARY KEY,
                     team_name VARCHAR(255),
                     points INTEGER DEFAULT 0,
                     created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                 );
             """)

             cursor.execute("""
                 CREATE TABLE IF NOT EXISTS team_members (
                     id SERIAL PRIMARY KEY,
                     team_id INTEGER REFERENCES team_leagues(id),
                     user_id VARCHAR(255) REFERENCES users(user_id)
                 );
             """)

             conn.commit()
             cursor.close()
             conn.close()
             print("Database tables created.")

         if __name__ == "__main__":
             init_db()
         ```
   * Update the backend to handle theme updates and device tokens:
     * **File**: `backend/src/auth.py`
     *   **Code**:

         ```python
         from fastapi import APIRouter, HTTPException
         from pydantic import BaseModel
         import psycopg2
         from psycopg2.extras import RealDictCursor
         import uuid
         import bcrypt

         router = APIRouter()

         def get_db_connection():
             return psycopg2.connect(
                 dbname="satprep",
                 user="user",
                 password="password",
                 host="localhost",
                 port="5432"
             )

         class UserSignup(BaseModel):
             email: str
             password: str
             study_hours: int

         class UserLogin(BaseModel):
             email: str
             password: str

         class UserProfileUpdate(BaseModel):
             full_name: str
             grade: str
             school: str | None = None
             sat_taken: bool
             sat_math_score: int | None = None
             sat_reading_score: int | None = None
             sat_test_date: str
             study_hours_per_week: int
             study_days_per_week: int
             preferred_study_time: str
             onboarding_completed: bool

         @router.post("/signup")
         async def signup(user: UserSignup):
             conn = get_db_connection()
             cursor = conn.cursor(cursor_factory=RealDictCursor)
             try:
                 cursor.execute("SELECT * FROM users WHERE email = %s", (user.email,))
                 if cursor.fetchone():
                     raise HTTPException(status_code=400, detail="Email already exists")

                 hashed_password = bcrypt.hashpw(user.password.encode('utf-8'), bcrypt.gensalt()).decode('utf-8')
                 user_id = str(uuid.uuid4())
                 cursor.execute("""
                     INSERT INTO users (user_id, email, hashed_password, role, study_hours, points, streak, league, coins)
                     VALUES (%s, %s, %s, 'student', %s, 0, 0, 'Bronze', 0)
                     RETURNING user_id;
                 """, (user_id, user.email, hashed_password, user.study_hours))
                 conn.commit()
                 return {"user_id": user_id}
             except Exception as e:
                 conn.rollback()
                 raise HTTPException(status_code=500, detail=str(e))
             finally:
                 cursor.close()
                 conn.close()

         @router.post("/login")
         async def login(user: UserLogin):
             conn = get_db_connection()
             cursor = conn.cursor(cursor_factory=RealDictCursor)
             try:
                 cursor.execute("SELECT * FROM users WHERE email = %s", (user.email,))
                 db_user = cursor.fetchone()
                 if not db_user:
                     raise HTTPException(status_code=400, detail="Invalid email or password")

                 if not bcrypt.checkpw(user.password.encode('utf-8'), db_user['hashed_password'].encode('utf-8')):
                     raise HTTPException(status_code=400, detail="Invalid email or password")

                 return {"user_id": db_user['user_id']}
             except Exception as e:
                 raise HTTPException(status_code=500, detail=str(e))
             finally:
                 cursor.close()
                 conn.close()

         @router.put("/update-profile/{user_id}")
         async def update_profile(user_id: str, update: UserProfileUpdate):
             conn = get_db_connection()
             cursor = conn.cursor(cursor_factory=RealDictCursor)
             try:
                 cursor.execute("""
                     UPDATE users
                     SET full_name = %s,
                         grade = %s,
                         school = %s,
                         sat_taken = %s,
                         sat_math_score = %s,
                         sat_reading_score = %s,
                         sat_test_date = %s,
                         study_hours_per_week = %s,
                         study_days_per_week = %s,
                         preferred_study_time = %s,
                         onboarding_completed = %s
                     WHERE user_id = %s
                     RETURNING *;
                 """, (
                     update.full_name,
                     update.grade,
                     update.school,
                     update.sat_taken,
                     update.sat_math_score,
                     update.sat_reading_score,
                     update.sat_test_date,
                     update.study_hours_per_week,
                     update.study_days_per_week,
                     update.preferred_study_time,
                     update.onboarding_completed,
                     user_id
                 ))
                 updated_user = cursor.fetchone()
                 if not updated_user:
                     raise HTTPException(status_code=404, detail="User not found")
                 conn.commit()
                 return updated_user
             except Exception as e:
                 conn.rollback()
                 raise HTTPException(status_code=500, detail=str(e))
             finally:
                 cursor.close()
                 conn.close()

         @router.get("/onboarding-status/{user_id}")
         async def get_onboarding_status(user_id: str):
             conn = get_db_connection()
             cursor = conn.cursor(cursor_factory=RealDictCursor)
             try:
                 cursor.execute("""
                     SELECT onboarding_completed FROM users WHERE user_id = %s;
                 """, (user_id,))
                 user = cursor.fetchone()
                 if not user:
                     raise HTTPException(status_code=404, detail="User not found")
                 return {"onboarding_completed": user['onboarding_completed']}
             except Exception as e:
                 raise HTTPException(status_code=500, detail=str(e))
             finally:
                 cursor.close()
                 conn.close()

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

         @router.put("/update-device-token/{user_id}")
         async def update_device_token(user_id: str, device_token: str):
             conn = get_db_connection()
             cursor = conn.cursor(cursor_factory=RealDictCursor)
             try:
                 cursor.execute("""
                     UPDATE users
                     SET device_token = %s
                     WHERE user_id = %s
                     RETURNING device_token;
                 """, (device_token, user_id))
                 updated_user = cursor.fetchone()
                 if not updated_user:
                     raise HTTPException(status_code=404, detail="User not found")
                 conn.commit()
                 return {"device_token": updated_user['device_token']}
             except Exception as e:
                 conn.rollback()
                 raise HTTPException(status_code=500, detail=str(e))
             finally:
                 cursor.close()
                 conn.close()
         ```
5. **Add WebSocket Support for Real-Time Updates**:
   * Update the backend to support WebSocket updates:
     * **File**: `backend/src/gamification.py`
     *   **Code**:

         ```python
         from fastapi import APIRouter, HTTPException, WebSocket
         from pydantic import BaseModel
         from datetime import datetime, date
         import psycopg2
         from psycopg2.extras import RealDictCursor
         from typing import Dict

         router = APIRouter()

         active_connections: Dict[str, WebSocket] = {}

         def get_db_connection():
             return psycopg2.connect(
                 dbname="satprep",
                 user="user",
                 password="password",
                 host="localhost",
                 port="5432"
             )

         class Challenge(BaseModel):
             user_id: str
             challenge_type: str
             target: int

         class ChallengeUpdate(BaseModel):
             progress: int

         class CoinEarn(BaseModel):
             user_id: str
             amount: int

         class RewardUnlock(BaseModel):
             user_id: str
             reward_id: str
             reward_type: str
             cost: int

         class TeamCreate(BaseModel):
             team_name: str
             user_id: str

         class TeamJoin(BaseModel):
             team_id: int
             user_id: str

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
                 message = f"New Daily Challenge: {new_challenge['challenge_type']} - Complete {new_challenge['target']} tasks!"
                 await send_notification(new_challenge['user_id'], message)

                 return new_challenge
             except Exception as e:
                 conn.rollback()
                 raise HTTPException(status_code=500, detail=str(e))
             finally:
                 cursor.close()
                 conn.close()

         @router.put("/challenges/{challenge_id}")
         async def update_challenge(challenge_id: int, update: ChallengeUpdate):
             conn = get_db_connection()
             cursor = conn.cursor(cursor_factory=RealDictCursor)
             try:
                 cursor.execute("""
                     UPDATE challenges
                     SET progress = %s,
                         completed = CASE WHEN progress >= target THEN TRUE ELSE FALSE END
                     WHERE id = %s
                     RETURNING *;
                 """, (update.progress, challenge_id))
                 updated_challenge = cursor.fetchone()
                 if not updated_challenge:
                     raise HTTPException(status_code=404, detail="Challenge not found")

                 if updated_challenge['completed']:
                     cursor.execute("""
                         UPDATE users
                         SET points = points + 50,
                             coins = coins + 50
                         WHERE user_id = %s;
                     """, (updated_challenge['user_id'],))
                     cursor.execute("""
                         INSERT INTO badges (user_id, name, awarded_at)
                         VALUES (%s, %s, CURRENT_TIMESTAMP);
                     """, (updated_challenge['user_id'], f"{updated_challenge['challenge_type']}_badge"))
                     await broadcast_gamification_update(updated_challenge['user_id'], "Challenge completed! You earned 50 coins!")

                 conn.commit()
                 return updated_challenge
             except Exception as e:
                 conn.rollback()
                 raise HTTPException(status_code=500, detail=str(e))
             finally:
                 cursor.close()
                 conn.close()

         @router.get("/challenges/{user_id}")
         async def get_user_challenges(user_id: str):
             conn = get_db_connection()
             cursor = conn.cursor(cursor_factory=RealDictCursor)
             try:
                 cursor.execute("""
                     SELECT * FROM challenges
                     WHERE user_id = %s AND date = CURRENT_DATE;
                 """, (user_id,))
                 challenges = cursor.fetchall()
                 return challenges
             except Exception as e:
                 raise HTTPException(status_code=500, detail=str(e))
             finally:
                 cursor.close()
                 conn.close()

         @router.get("/leaderboards/{skill}")
         async def get_leaderboard(skill: str, user_id: str):
             conn = get_db_connection()
             cursor = conn.cursor(cursor_factory=RealDictCursor)
             try:
                 cursor.execute("""
                     SELECT u.user_id, u.email, p.theta AS score
                     FROM proficiencies p
                     JOIN users u ON p.user_id = u.user_id
                     WHERE p.skill = %s
                     ORDER BY p.theta DESC
                     LIMIT 100;
                 """, (skill,))
                 global_leaderboard = cursor.fetchall()

                 cursor.execute("""
                     SELECT u.user_id, u.email, p.theta AS score
                     FROM proficiencies p
                     JOIN users u ON p.user_id = u.user_id
                     JOIN friends f ON (f.user_id = %s AND f.friend_id = u.user_id) OR (f.friend_id = %s AND f.user_id = u.user_id)
                     WHERE p.skill = %s AND f.status = 'accepted'
                     ORDER BY p.theta DESC;
                 """, (user_id, user_id, skill))
                 friends_leaderboard = cursor.fetchall()

                 return {"global": global_leaderboard, "friends": friends_leaderboard}
             except Exception as e:
                 raise HTTPException(status_code=500, detail=str(e))
             finally:
                 cursor.close()
                 conn.close()

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

         @router.get("/coins/{user_id}")
         async def get_coins(user_id: str):
             conn = get_db_connection()
             cursor = conn.cursor(cursor_factory=RealDictCursor)
             try:
                 cursor.execute("""
                     SELECT coins FROM users WHERE user_id = %s;
                 """, (user_id,))
                 user = cursor.fetchone()
                 if not user:
                     raise HTTPException(status_code=404, detail="User not found")
                 return {"coins": user['coins']}
             except Exception as e:
                 raise HTTPException(status_code=500, detail=str(e))
             finally:
                 cursor.close()
                 conn.close()

         @router.get("/rewards/available")
         async def get_available_rewards():
             rewards = [
                 {"id": "math_wizard_avatar", "name": "Math Wizard Avatar", "type": "avatar", "cost": 200, "image": "https://via.placeholder.com/150"},
                 {"id": "dark_mode_theme", "name": "Dark Mode Theme", "type": "theme", "cost": 300, "image": "https://via.placeholder.com/150"},
                 {"id": "double_points_powerup", "name": "Double Points (24h)", "type": "power_up", "cost": 500, "image": "https://via.placeholder.com/150"}
             ]
             return rewards

         @router.post("/rewards/unlock")
         async def unlock_reward(reward: RewardUnlock):
             conn = get_db_connection()
             cursor = conn.cursor(cursor_factory=RealDictCursor)
             try:
                 cursor.execute("""
                     SELECT coins FROM users WHERE user_id = %s;
                 """, (reward.user_id,))
                 user = cursor.fetchone()
                 if not user:
                     raise HTTPException(status_code=404, detail="User not found")
                 if user['coins'] < reward.cost:
                     raise HTTPException(status_code=400, detail="Not enough coins")

                 cursor.execute("""
                     UPDATE users
                     SET coins = coins - %s
                     WHERE user_id = %s;
                 """, (reward.cost, reward.user_id))
                 cursor.execute("""
                     INSERT INTO rewards (user_id, reward_type, reward_id, unlocked_at)
                     VALUES (%s, %s, %s, CURRENT_TIMESTAMP);
                 """, (reward.user_id, reward.reward_type, reward.reward_id))
                 conn.commit()
                 await broadcast_gamification_update(reward.user_id, f"You unlocked {reward.reward_id}!")
                 return {"message": "Reward unlocked"}
             except Exception as e:
                 conn.rollback()
                 raise HTTPException(status_code=500, detail=str(e))
             finally:
                 cursor.close()
                 conn.close()

         @router.post("/teams/create")
         async def create_team(team: TeamCreate):
             conn = get_db_connection()
             cursor = conn.cursor(cursor_factory=RealDictCursor)
             try:
                 cursor.execute("""
                     INSERT INTO team_leagues (team_name, points, created_at)
                     VALUES (%s, 0, CURRENT_TIMESTAMP)
                     RETURNING id;
                 """, (team.team_name,))
                 team_id = cursor.fetchone()['id']
                 cursor.execute("""
                     INSERT INTO team_members (team_id, user_id)
                     VALUES (%s, %s);
                 """, (team_id, team.user_id))
                 conn.commit()
                 return {"team_id": team_id}
             except Exception as e:
                 conn.rollback()
                 raise HTTPException(status_code=500, detail=str(e))
             finally:
                 cursor.close()
                 conn.close()

         @router.post("/teams/join")
         async def join_team(join: TeamJoin):
             conn = get_db_connection()
             cursor = conn.cursor(cursor_factory=RealDictCursor)
             try:
                 cursor.execute("""
                     INSERT INTO team_members (team_id, user_id)
                     VALUES (%s, %s);
                 """, (join.team_id, join.user_id))
                 conn.commit()
                 return {"message": "Joined team"}
             except Exception as e:
                 conn.rollback()
                 raise HTTPException(status_code=500, detail=str(e))
             finally:
                 cursor.close()
                 conn.close()

         @router.get("/teams/leaderboard")
         async def get_team_leaderboard():
             conn = get_db_connection()
             cursor = conn.cursor(cursor_factory=RealDictCursor)
             try:
                 cursor.execute("""
                     SELECT * FROM team_leagues
                     ORDER BY points DESC
                     LIMIT 10;
                 """,)
                 leaderboard = cursor.fetchall()
                 return leaderboard
             except Exception as e:
                 raise HTTPException(status_code=500, detail=str(e))
             finally:
                 cursor.close()
                 conn.close()

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
6. **Update Web App for WebSocket Integration**:
   * **File**: `frontend/utils/api.js`
   *   **Code**:

       ```javascript
       export const api = {
         async get(endpoint) {
           try {
             const response = await fetch(`http://localhost:8000${endpoint}`);
             const data = await response.json();
             localStorage.setItem(endpoint, JSON.stringify(data));
             return { data };
           } catch (error) {
             const cachedData = localStorage.getItem(endpoint);
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
             const queuedRequests = localStorage.getItem('queuedRequests') || '[]';
             const requests = JSON.parse(queuedRequests);
             requests.push({ endpoint, body });
             localStorage.setItem('queuedRequests', JSON.stringify(requests));
             throw new Error('Offline mode: Request queued');
           }
         },
         async sync() {
           const queuedRequests = localStorage.getItem('queuedRequests') || '[]';
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
           localStorage.setItem('queuedRequests', JSON.stringify(requests.slice(syncedCount)));
           return syncedCount;
         }
       };

       export const connectWebSocket = (userId, onMessage) => {
         const ws = new WebSocket(`ws://localhost:8000/gamification/updates/${userId}`);
         ws.onmessage = (message) => onMessage(message.data);
         return ws;
       };

       const handleOnline = () => {
         api.sync().then(count => console.log(`Synced ${count} items`));
       };

       const handleOffline = () => {
         console.log('Offline mode activated');
       };

       window.addEventListener('online', handleOnline);
       window.addEventListener('offline', handleOffline);
       ```
7. **Define Shared Architecture**:
   * **Backend**: Use the existing FastAPI backend (`backend/src/`) for both platforms.
   * **State Management**: Use React Context for shared state (e.g., user data, theme) on both platforms.
   * **Data Synchronization**: Use WebSocket for real-time updates, `localStorage` (web) and `react-native-async-storage` (mobile) for offline caching.
   * **Push Notifications**: Integrate Firebase Cloud Messaging (FCM) for mobile, with backend support to send notifications.
   * **UI/UX**: Create a shared design system (`styles.js`) for consistent colors, typography, and components.
8. **Plan Feature Parity**:
   * Ensure all web features (diagnostics, practice, AI tutor, gamification, social features, offline mode, onboarding) are available on mobile.
   * Add mobile-specific features: push notifications, real voice input, native gestures.

**Deliverables**

* React Native project initialized (`SATPrepSuiteMobile`).
* Development environment set up for iOS and Android.
* Backend updated with `theme` and `device_token` fields, WebSocket support.
* Web app updated with WebSocket integration.
* Shared architecture and design system defined.

***

#### Step 2: Create Shared Design System and Adapt Components (April 8-14, 2025)

**Objective**

Create a shared design system for consistent UI/UX across platforms, port existing React components to React Native, and update web components to use the design system.

**Sub-Steps**

1. **Create Shared Design System**:
   * **File**: `SATPrepSuiteMobile/src/styles.js`
   *   **Code**:

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
   * Copy to `frontend/src/styles.js` for the web app.
2. **Copy Reusable Components**:
   * Copy components from `frontend/components/` (e.g., `QuestionDisplay.js`, `PassageDisplay.js`, `MathQuestionHeader.js`, `AnswerEliminator.js`) to `SATPrepSuiteMobile/src/components/`.
   * Copy layout components from `frontend/components/layouts/` (e.g., `ReadingWritingTest.js`, `MathBasicTest.js`, `MathGraphTest.js`, `MathTableTest.js`) to `SATPrepSuiteMobile/src/components/layouts/`.
3. **Adapt Components for React Native**:
   * Replace web-specific elements with React Native equivalents:
     * `div` → `View`
     * `p` → `Text`
     * `input` → `TextInput`
     * `button` → `TouchableOpacity` or `Button`
     * `style jsx` → `StyleSheet`
   *   Example: Update `QuestionDisplay.js`:

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
                 The
       ```
