---
noIndex: true
---

# Set 1 of 4 Webapp Backend and config files, start of Front End

Apologies for the interruption! Let’s break the task into smaller, manageable groups to create the complete package for the **SAT Smart Prep App** by **Learner Labs**. I’ll divide the files into four sets, ensuring all files are included and are the latest versions as of March 27, 2025. Each set will include a subset of the project files, and I’ll provide detailed instructions for installation and testing in a local development environment at the end. This approach will make the process more manageable and ensure clarity.

***

### Overview of File Sets

The repository will be split into four sets:

1. **Set 1: Backend and Core Configuration Files** (`frontend/backend/`, root files like `README.md`, `docker-compose.yml`, etc.)
2. **Set 2: Web App Frontend Files** (`frontend/components/`, `frontend/pages/`, `frontend/styles/`, `frontend/utils/`, `frontend/config/`)
3. **Set 3: Web App Testing and Static Files** (`frontend/tests/`, `frontend/cypress/`, `frontend/public/`, `frontend/package.json`, `frontend/next.config.js`, `frontend/Dockerfile`)
4. **Set 4: Mobile App Files** (`SATSmartPrepApp/`)

Each set will include the latest versions of the files, reflecting all changes (e.g., nudge modal in `PracticeScreen.js`, Bluebook-like UI in `practice-bluebook.js`, device tracking in `practice.py`).

***

### Set 1: Backend and Core Configuration Files

This set includes the FastAPI backend for the web app (`frontend/backend/`), along with root-level configuration files for the repository.

#### Project Structure for Set 1

```
SATSmartPrepAppRepo/
├── frontend/
│   ├── backend/
│   │   ├── migrations/
│   │   │   └── init_db.py
│   │   ├── src/
│   │   │   ├── __init__.py
│   │   │   ├── main.py
│   │   │   ├── auth.py
│   │   │   ├── diagnostic.py
│   │   │   ├── practice.py
│   │   │   ├── study_plan.py
│   │   │   ├── progress_monitoring.py
│   │   │   ├── ai_tutor.py
│   │   │   ├── social.py
│   │   │   ├── gamification.py
│   │   │   └── notifications.py
│   │   ├── tests/
│   │   │   ├── test_auth.py
│   │   │   ├── test_diagnostic.py
│   │   │   ├── test_practice.py
│   │   │   ├── test_study_plan.py
│   │   │   ├── test_progress.py
│   │   │   ├── test_ai_tutor.py
│   │   │   ├── test_social.py
│   │   │   ├── test_gamification.py
│   │   │   └── test_notifications.py
│   │   ├── requirements.txt
│   │   └── Dockerfile
├── README.md
├── docker-compose.yml
└── setup.sh
```

#### Files for Set 1

**`frontend/backend/migrations/init_db.py`**

* **Purpose**: Initializes the database schema.
* **Latest Version**: Includes `device` column in `practice_sessions` and `subscription_status` in `users`.
*   **Code**:

    ```python
    import psycopg2
    from psycopg2.extras import RealDictCursor

    def init_db():
        conn = psycopg2.connect(
            dbname="satprep",
            user="user",
            password="password",
            host="localhost",
            port="5432"
        )
        cursor = conn.cursor(cursor_factory=RealDictCursor)

        # Create users table
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS users (
                user_id VARCHAR(36) PRIMARY KEY,
                email VARCHAR(255) UNIQUE NOT NULL,
                hashed_password VARCHAR(255) NOT NULL,
                full_name VARCHAR(255),
                grade VARCHAR(50),
                school VARCHAR(255),
                sat_taken BOOLEAN,
                sat_math_score INTEGER,
                sat_reading_score INTEGER,
                sat_test_date VARCHAR(50),
                study_hours_per_week INTEGER,
                study_days_per_week INTEGER,
                preferred_study_time VARCHAR(50),
                onboarding_completed BOOLEAN DEFAULT FALSE,
                theme VARCHAR(50) DEFAULT 'light',
                device_token VARCHAR(255),
                points INTEGER DEFAULT 0,
                streak INTEGER DEFAULT 0,
                league VARCHAR(50) DEFAULT 'Bronze',
                coins INTEGER DEFAULT 0,
                referral_code VARCHAR(8),
                referral_redeemed BOOLEAN DEFAULT FALSE,
                subscription_status VARCHAR(50) DEFAULT 'inactive'
            );
        """)

        # Create questions table
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS questions (
                id VARCHAR(36) PRIMARY KEY,
                domain VARCHAR(50),
                skill VARCHAR(50),
                difficulty INTEGER,
                question_text TEXT,
                options JSONB,
                correct_answer VARCHAR(255),
                explanation TEXT,
                test_type VARCHAR(50) DEFAULT 'SAT',
                question_type VARCHAR(50)
            );
        """)

        # Create practice_sessions table
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS practice_sessions (
                session_id VARCHAR(36) PRIMARY KEY,
                user_id VARCHAR(36) REFERENCES users(user_id),
                test_type VARCHAR(50),
                section VARCHAR(50),
                device VARCHAR(50),
                theta FLOAT,
                timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            );
        """)

        # Create responses table
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS responses (
                response_id VARCHAR(36) PRIMARY KEY,
                session_id VARCHAR(36) REFERENCES practice_sessions(session_id),
                question_id VARCHAR(36) REFERENCES questions(id),
                answer VARCHAR(255),
                time_spent INTEGER,
                domain VARCHAR(50),
                theta FLOAT
            );
        """)

        # Create proficiencies table
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS proficiencies (
                id VARCHAR(36) PRIMARY KEY,
                user_id VARCHAR(36) REFERENCES users(user_id),
                domain VARCHAR(50),
                skill VARCHAR(50),
                theta FLOAT,
                timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            );
        """)

        # Create study_plans table
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS study_plans (
                plan_id VARCHAR(36) PRIMARY KEY,
                user_id VARCHAR(36) REFERENCES users(user_id),
                test_date VARCHAR(50)
            );
        """)

        # Create study_plan_actions table
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS study_plan_actions (
                id VARCHAR(36) PRIMARY KEY,
                plan_id VARCHAR(36) REFERENCES study_plans(plan_id),
                task VARCHAR(255),
                action VARCHAR(50),
                due_date VARCHAR(50),
                points INTEGER,
                completed TIMESTAMP
            );
        """)

        # Create tutor_interactions table
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS tutor_interactions (
                id VARCHAR(36) PRIMARY KEY,
                user_id VARCHAR(36) REFERENCES users(user_id),
                query TEXT,
                response TEXT,
                duration INTEGER,
                timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            );
        """)

        # Create posts table
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS posts (
                id VARCHAR(36) PRIMARY KEY,
                user_id VARCHAR(36) REFERENCES users(user_id),
                content TEXT,
                timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            );
        """)

        # Create friends table
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS friends (
                id VARCHAR(36) PRIMARY KEY,
                user_id VARCHAR(36) REFERENCES users(user_id),
                friend_id VARCHAR(36) REFERENCES users(user_id),
                status VARCHAR(50),
                timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            );
        """)

        # Create teams table
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS teams (
                id VARCHAR(36) PRIMARY KEY,
                team_name VARCHAR(255),
                user_id VARCHAR(36) REFERENCES users(user_id),
                points INTEGER DEFAULT 0,
                timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            );
        """)

        conn.commit()
        cursor.close()
        conn.close()

    if __name__ == "__main__":
        init_db()
    ```

**`frontend/backend/src/__init__.py`**

* **Purpose**: Package initialization.
*   **Code**:

    ```python
    # Empty file to make src a package
    ```

**`frontend/backend/src/main.py`**

* **Purpose**: Main FastAPI app entry point.
* **Latest Version**: Includes all routers.
*   **Code**:

    ```python
    from fastapi import FastAPI
    from . import auth, diagnostic, practice, study_plan, progress_monitoring, ai_tutor, social, gamification, notifications

    app = FastAPI()

    app.include_router(auth.router, prefix="/auth")
    app.include_router(diagnostic.router, prefix="/diagnostic")
    app.include_router(practice.router, prefix="/practice")
    app.include_router(study_plan.router, prefix="/study_plan")
    app.include_router(progress_monitoring.router, prefix="/progress_monitoring")
    app.include_router(ai_tutor.router, prefix="/ai_tutor")
    app.include_router(social.router, prefix="/social")
    app.include_router(gamification.router, prefix="/gamification")
    app.include_router(notifications.router, prefix="/notifications")

    @app.get("/")
    async def root():
        return {"message": "SAT Smart Prep App Backend"}
    ```

**`frontend/backend/src/auth.py`**

* **Purpose**: Handles authentication (signup, login, SSO).
* **Latest Version**: Includes tutor signup and device token updates.
*   **Code**:

    ```python
    from fastapi import APIRouter, HTTPException
    from pydantic import BaseModel
    from psycopg2.extras import RealDictCursor
    import psycopg2
    import bcrypt
    import uuid

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

    class UserLogin(BaseModel):
        email: str
        password: str

    class SSORequest(BaseModel):
        token: str

    class DeviceTokenUpdate(BaseModel):
        device_token: str

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
                INSERT INTO users (user_id, email, hashed_password, study_hours, points, streak, league, coins)
                VALUES (%s, %s, %s, 0, 0, 0, 'Bronze', 0)
                RETURNING user_id;
            """, (user_id, user.email, hashed_password))
            conn.commit()
            return {"user_id": user_id}
        finally:
            cursor.close()
            conn.close()

    @router.post("/signup/tutor")
    async def signup_tutor(email: str, password: str):
        conn = get_db_connection()
        cursor = conn.cursor(cursor_factory=RealDictCursor)
        try:
            cursor.execute("SELECT * FROM users WHERE email = %s", (email,))
            if cursor.fetchone():
                raise HTTPException(status_code=400, detail="Email already exists")
            hashed_password = bcrypt.hashpw(password.encode('utf-8'), bcrypt.gensalt()).decode('utf-8')
            user_id = str(uuid.uuid4())
            cursor.execute("""
                INSERT INTO users (user_id, email, hashed_password, role, study_hours, points, streak, league, coins)
                VALUES (%s, %s, %s, 'tutor', 0, 0, 0, 'Bronze', 0)
                RETURNING user_id;
            """, (user_id, email, hashed_password))
            conn.commit()
            return {"user_id": user_id}
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
            if not db_user or not bcrypt.checkpw(user.password.encode('utf-8'), db_user['hashed_password'].encode('utf-8')):
                raise HTTPException(status_code=401, detail="Invalid credentials")
            return {"user_id": db_user['user_id']}
        finally:
            cursor.close()
            conn.close()

    @router.post("/sso")
    async def sso(sso_request: SSORequest):
        conn = get_db_connection()
        cursor = conn.cursor(cursor_factory=RealDictCursor)
        try:
            # Placeholder for SSO token validation (e.g., Google OAuth)
            user_id = str(uuid.uuid4())
            cursor.execute("""
                INSERT INTO users (user_id, email, hashed_password, study_hours, points, streak, league, coins)
                VALUES (%s, %s, %s, 0, 0, 0, 'Bronze', 0)
                ON CONFLICT (email) DO UPDATE SET user_id = EXCLUDED.user_id
                RETURNING user_id;
            """, (user_id, "sso_user@example.com", "sso"))
            conn.commit()
            return {"user_id": user_id}
        finally:
            cursor.close()
            conn.close()

    @router.get("/user/{user_id}")
    async def get_user(user_id: str):
        conn = get_db_connection()
        cursor = conn.cursor(cursor_factory=RealDictCursor)
        try:
            cursor.execute("SELECT * FROM users WHERE user_id = %s", (user_id,))
            user = cursor.fetchone()
            if not user:
                raise HTTPException(status_code=404, detail="User not found")
            return user
        finally:
            cursor.close()
            conn.close()

    @router.put("/update-profile/{user_id}")
    async def update_profile(user_id: str, profile: dict):
        conn = get_db_connection()
        cursor = conn.cursor(cursor_factory=RealDictCursor)
        try:
            cursor.execute("SELECT * FROM users WHERE user_id = %s", (user_id,))
            if not cursor.fetchone():
                raise HTTPException(status_code=404, detail="User not found")
            update_query = "UPDATE users SET "
            update_values = []
            for key, value in profile.items():
                update_query += f"{key} = %s, "
                update_values.append(value)
            update_query = update_query.rstrip(", ") + " WHERE user_id = %s"
            update_values.append(user_id)
            cursor.execute(update_query, update_values)
            conn.commit()
            return {"message": "Profile updated"}
        finally:
            cursor.close()
            conn.close()

    @router.put("/update-device-token/{user_id}")
    async def update_device_token(user_id: str, token: DeviceTokenUpdate):
        conn = get_db_connection()
        cursor = conn.cursor(cursor_factory=RealDictCursor)
        try:
            cursor.execute("UPDATE users SET device_token = %s WHERE user_id = %s", (token.device_token, user_id))
            conn.commit()
            return {"message": "Device token updated"}
        finally:
            cursor.close()
            conn.close()
    ```

**`frontend/backend/src/diagnostic.py`**

* **Purpose**: Handles diagnostic test logic.
* **Latest Version**: Supports multiple test types (SAT, PSAT, etc.).
*   **Code**:

    ```python
    from fastapi import APIRouter, HTTPException
    from psycopg2.extras import RealDictCursor
    import psycopg2
    import uuid

    router = APIRouter()

    def get_db_connection():
        return psycopg2.connect(
            dbname="satprep",
            user="user",
            password="password",
            host="localhost",
            port="5432"
        )

    @router.post("/diagnostic/start/{user_id}")
    async def start_diagnostic(user_id: str, num_questions: int, test_type: str = 'SAT'):
        conn = get_db_connection()
        cursor = conn.cursor(cursor_factory=RealDictCursor)
        try:
            session_id = str(uuid.uuid4())
            cursor.execute("""
                INSERT INTO practice_sessions (session_id, user_id, test_type)
                VALUES (%s, %s, %s);
            """, (session_id, user_id, test_type))
            cursor.execute("""
                SELECT * FROM questions
                WHERE test_type = %s
                ORDER BY RANDOM()
                LIMIT %s;
            """, (test_type, num_questions))
            questions = cursor.fetchall()
            conn.commit()
            return {"session_id": session_id, "questions": questions}
        except Exception as e:
            conn.rollback()
            raise HTTPException(status_code=500, detail=str(e))
        finally:
            cursor.close()
            conn.close()

    @router.post("/diagnostic/submit/{session_id}")
    async def submit_diagnostic(session_id: str, responses: list):
        conn = get_db_connection()
        cursor = conn.cursor(cursor_factory=RealDictCursor)
        try:
            for response in responses:
                response_id = str(uuid.uuid4())
                cursor.execute("""
                    INSERT INTO responses (response_id, session_id, question_id, answer, time_spent, domain, theta)
                    VALUES (%s, %s, %s, %s, %s, %s, %s);
                """, (response_id, session_id, response['question_id'], response['answer'], response['time_spent'], response['domain'], response['theta']))
            cursor.execute("SELECT user_id FROM practice_sessions WHERE session_id = %s", (session_id,))
            user_id = cursor.fetchone()['user_id']
            cursor.execute("SELECT * FROM responses WHERE session_id = %s", (session_id,))
            user_responses = cursor.fetchall()
            theta = sum(resp['theta'] for resp in user_responses) / len(user_responses)
            cursor.execute("UPDATE practice_sessions SET theta = %s WHERE session_id = %s", (theta, session_id))
            for response in user_responses:
                cursor.execute("""
                    INSERT INTO proficiencies (id, user_id, domain, skill, theta)
                    VALUES (%s, %s, %s, %s, %s);
                """, (str(uuid.uuid4()), user_id, response['domain'], response['domain'], response['theta']))
            conn.commit()
            return {"theta": theta}
        except Exception as e:
            conn.rollback()
            raise HTTPException(status_code=500, detail=str(e))
        finally:
            cursor.close()
            conn.close()
    ```

**`frontend/backend/src/practice.py`**

* **Purpose**: Handles practice session logic.
* **Latest Version**: Includes device tracking and analytics endpoint.
*   **Code**:

    ```python
    from fastapi import APIRouter, HTTPException
    from pydantic import BaseModel
    from psycopg2.extras import RealDictCursor
    import psycopg2
    import uuid

    router = APIRouter()

    def get_db_connection():
        return psycopg2.connect(
            dbname="satprep",
            user="user",
            password="password",
            host="localhost",
            port="5432"
        )

    class PracticeStart(BaseModel):
        domain: str
        num_questions: int
        test_type: str
        device: str

    class PracticeResponse(BaseModel):
        question_id: str
        answer: str
        time_spent: int
        session_id: str
        domain: str
        theta: float

    @router.post("/practice/start/{user_id}")
    async def start_practice(user_id: str, practice: PracticeStart):
        conn = get_db_connection()
        cursor = conn.cursor(cursor_factory=RealDictCursor)
        try:
            session_id = str(uuid.uuid4())
            cursor.execute("""
                INSERT INTO practice_sessions (session_id, user_id, test_type, section, device)
                VALUES (%s, %s, %s, %s, %s);
            """, (session_id, user_id, practice.test_type, practice.domain, practice.device))
            cursor.execute("""
                SELECT * FROM questions
                WHERE domain = %s AND test_type = %s
                ORDER BY RANDOM()
                LIMIT %s;
            """, (practice.domain, practice.test_type, practice.num_questions))
            questions = cursor.fetchall()
            conn.commit()
            return {"session_id": session_id, "questions": questions}
        except Exception as e:
            conn.rollback()
            raise HTTPException(status_code=500, detail=str(e))
        finally:
            cursor.close()
            conn.close()

    @router.post("/practice/submit/{session_id}")
    async def submit_practice(session_id: str, responses: list[PracticeResponse]):
        conn = get_db_connection()
        cursor = conn.cursor(cursor_factory=RealDictCursor)
        try:
            for response in responses:
                response_id = str(uuid.uuid4())
                cursor.execute("""
                    INSERT INTO responses (response_id, session_id, question_id, answer, time_spent, domain, theta)
                    VALUES (%s, %s, %s, %s, %s, %s, %s);
                """, (response_id, session_id, response.question_id, response.answer, response.time_spent, response.domain, response.theta))
            cursor.execute("SELECT user_id FROM practice_sessions WHERE session_id = %s", (session_id,))
            user_id = cursor.fetchone()['user_id']
            cursor.execute("SELECT * FROM responses WHERE session_id = %s", (session_id,))
            user_responses = cursor.fetchall()
            theta = sum(resp['theta'] for resp in user_responses) / len(user_responses)
            cursor.execute("UPDATE practice_sessions SET theta = %s WHERE session_id = %s", (theta, session_id))
            for response in user_responses:
                cursor.execute("""
                    INSERT INTO proficiencies (id, user_id, domain, skill, theta)
                    VALUES (%s, %s, %s, %s, %s);
                """, (str(uuid.uuid4()), user_id, response['domain'], response['domain'], response['theta']))
            cursor.execute("UPDATE users SET points = points + 10 WHERE user_id = %s", (user_id,))
            conn.commit()
            return {"theta": theta, "points_earned": 10}
        except Exception as e:
            conn.rollback()
            raise HTTPException(status_code=500, detail=str(e))
        finally:
            cursor.close()
            conn.close()

    @router.get("/practice/device-stats")
    async def get_device_stats():
        conn = get_db_connection()
        cursor = conn.cursor(cursor_factory=RealDictCursor)
        try:
            cursor.execute("""
                SELECT device, COUNT(*) as count
                FROM practice_sessions
                WHERE device IS NOT NULL
                GROUP BY device;
            """)
            stats = cursor.fetchall()
            return stats
        except Exception as e:
            raise HTTPException(status_code=500, detail=str(e))
        finally:
            cursor.close()
            conn.close()
    ```

**`frontend/backend/src/study_plan.py`**

* **Purpose**: Manages study plan creation and retrieval.
* **Latest Version**: Supports granular study plans.
*   **Code**:

    ```python
    from fastapi import APIRouter, HTTPException
    from pydantic import BaseModel
    from psycopg2.extras import RealDictCursor
    import psycopg2
    import uuid
    from datetime import datetime, timedelta

    router = APIRouter()

    def get_db_connection():
        return psycopg2.connect(
            dbname="satprep",
            user="user",
            password="password",
            host="localhost",
            port="5432"
        )

    class StudyPlanRequest(BaseModel):
        test_date: str
        study_hours: int
        study_days: int

    @router.post("/study_plan/create/{user_id}")
    async def create_study_plan(user_id: str, request: StudyPlanRequest):
        conn = get_db_connection()
        cursor = conn.cursor(cursor_factory=RealDictCursor)
        try:
            cursor.execute("SELECT * FROM proficiencies WHERE user_id = %s", (user_id,))
            proficiencies = cursor.fetchall()
            focus_areas = [prof['skill'] for prof in proficiencies if prof['theta'] < 0.5]
            if not focus_areas:
                focus_areas = [prof['skill'] for prof in proficiencies]

            sessions_per_week = request.study_days * (request.study_hours // request.study_days)
            total_sessions = len(focus_areas) * sessions_per_week
            test_date = datetime.fromisoformat(request.test_date.replace('Z', '+00:00'))
            days_until_test = (test_date - datetime.now()).days
            weeks_until_test = max(1, days_until_test // 7)
            sessions_per_skill = total_sessions // len(focus_areas)

            milestones = [f"Complete {sessions_per_skill} sessions in {skill}" for skill in focus_areas]

            plan_id = str(uuid.uuid4())
            cursor.execute("""
                INSERT INTO study_plans (plan_id, user_id, test_date)
                VALUES (%s, %s, %s);
            """, (plan_id, user_id, request.test_date))

            for i, milestone in enumerate(milestones):
                due_date = (datetime.now() + timedelta(days=(i + 1) * (days_until_test // len(milestones)))).isoformat()
                cursor.execute("""
                    INSERT INTO study_plan_actions (id, plan_id, task, action, due_date, points)
                    VALUES (%s, %s, %s, %s, %s, 50);
                """, (str(uuid.uuid4()), plan_id, milestone, "Complete", due_date))
            conn.commit()
            return {
                "plan_id": plan_id,
                "sessions_per_week": sessions_per_week,
                "focus_areas": focus_areas,
                "milestones": milestones
            }
        except Exception as e:
            conn.rollback()
            raise HTTPException(status_code=500, detail=str(e))
        finally:
            cursor.close()
            conn.close()

    @router.get("/study_plan/{user_id}")
    async def get_study_plan(user_id: str):
        conn = get_db_connection()
        cursor = conn.cursor(cursor_factory=RealDictCursor)
        try:
            cursor.execute("SELECT * FROM study_plans WHERE user_id = %s ORDER BY test_date DESC LIMIT 1", (user_id,))
            plan = cursor.fetchone()
            if not plan:
                raise HTTPException(status_code=404, detail="Study plan not found")
            cursor.execute("SELECT * FROM study_plan_actions WHERE plan_id = %s", (plan['plan_id'],))
            actions = cursor.fetchall()
            return {
                "plan_id": plan['plan_id'],
                "test_date": plan['test_date'],
                "actions": actions
            }
        except Exception as e:
            raise HTTPException(status_code=500, detail=str(e))
        finally:
            cursor.close()
            conn.close()
    ```

**`frontend/backend/src/progress_monitoring.py`**

* **Purpose**: Tracks user progress and score guarantee eligibility.
* **Latest Version**: Includes starting score check for guarantee (1250 or lower).
*   **Code**:

    ```python
    from fastapi import APIRouter, HTTPException
    from psycopg2.extras import RealDictCursor
    from datetime import datetime, timedelta
    import psycopg2

    router = APIRouter()

    def get_db_connection():
        return psycopg2.connect(
            dbname="satprep",
            user="user",
            password="password",
            host="localhost",
            port="5432"
        )

    @router.get("/progress_monitoring/proficiencies/{user_id}")
    async def get_proficiencies(user_id: str):
        conn = get_db_connection()
        cursor = conn.cursor(cursor_factory=RealDictCursor)
        try:
            cursor.execute("SELECT * FROM proficiencies WHERE user_id = %s", (user_id,))
            proficiencies = cursor.fetchall()
            return proficiencies
        except Exception as e:
            raise HTTPException(status_code=500, detail=str(e))
        finally:
            cursor.close()
            conn.close()

    @router.get("/progress_monitoring/scores/{user_id}")
    async def get_score_history(user_id: str):
        conn = get_db_connection()
        cursor = conn.cursor(cursor_factory=RealDictCursor)
        try:
            cursor.execute("""
                SELECT theta, timestamp
                FROM practice_sessions
                WHERE user_id = %s AND test_type = 'SAT'
                ORDER BY timestamp;
            """, (user_id,))
            sessions = cursor.fetchall()
            return [{"score": session['theta'] * 400 + 400, "timestamp": session['timestamp']} for session in sessions]
        except Exception as e:
            raise HTTPException(status_code=500, detail=str(e))
        finally:
            cursor.close()
            conn.close()

    @router.get("/progress_monitoring/guarantee/{user_id}")
    async def check_score_guarantee(user_id: str):
        conn = get_db_connection()
        cursor = conn.cursor(cursor_factory=RealDictCursor)
        try:
            cursor.execute("SELECT subscription_status, sat_math_score, sat_reading_score FROM users WHERE user_id = %s", (user_id,))
            user = cursor.fetchone()
            if not user:
                raise HTTPException(status_code=404, detail="User not found")
            if user['subscription_status'] != 'active':
                return {"eligible": False, "message": "You must have an active paid subscription to be eligible for the guarantee."}

            starting_score = None
            if user['sat_math_score'] is not None and user['sat_reading_score'] is not None:
                starting_score = user['sat_math_score'] + user['sat_reading_score']
            else:
                cursor.execute("""
                    SELECT theta
                    FROM practice_sessions
                    WHERE user_id = %s AND test_type = 'SAT'
                    ORDER BY timestamp
                    LIMIT 1;
                """, (user_id,))
                first_session = cursor.fetchone()
                if first_session:
                    starting_score = first_session['theta'] * 400 + 400
                else:
                    return {"eligible": False, "message": "No starting score available. Complete at least one full-length test to determine eligibility."}

            if starting_score > 1250:
                return {"eligible": False, "message": "Your starting score is above 1250. The score improvement guarantee is only available for users starting at 1250 or lower."}

            cursor.execute("""
                SELECT theta, timestamp
                FROM practice_sessions
                WHERE user_id = %s AND test_type = 'SAT'
                ORDER BY timestamp;
            """, (user_id,))
            sessions = cursor.fetchall()
            if len(sessions) < 2:
                return {"eligible": False, "message": "Not enough test data to evaluate guarantee. Complete at least two full-length tests."}

            first_test_date = sessions[0]['timestamp']
            if (datetime.now() - first_test_date).days > 60:
                return {"eligible": False, "message": "Refund request period has expired. Requests must be submitted within 60 days of your first test."}

            cursor.execute("""
                SELECT COUNT(*) as total, COUNT(CASE WHEN completed IS NOT NULL THEN 1 END) as completed
                FROM study_plan_actions
                WHERE plan_id IN (SELECT plan_id FROM study_plans WHERE user_id = %s)
                AND due_date BETWEEN %s AND %s;
            """, (user_id, first_test_date, first_test_date + timedelta(days=30)))
            study_plan_stats = cursor.fetchone()
            completion_rate = study_plan_stats['completed'] / study_plan_stats['total'] if study_plan_stats['total'] > 0 else 0
            if completion_rate < 0.8:
                return {"eligible": False, "message": "You must complete at least 80% of your study plan tasks within 30 days of your first test."}

            initial_score = sessions[0]['theta'] * 400 + 400
            latest_score = max(session['theta'] * 400 + 400 for session in sessions[1:])
            improvement = latest_score - initial_score
            if improvement < 150:
                return {
                    "eligible": True,
                    "improvement": improvement,
                    "message": "You qualify for a refund! Contact support at support@learnerlabs.com."
                }
            return {
                "eligible": False,
                "improvement": improvement,
                "message": "Great job! You’ve improved by 150+ points."
            }
        except Exception as e:
            raise HTTPException(status_code=500, detail=str(e))
        finally:
            cursor.close()
            conn.close()
    ```

**`frontend/backend/src/ai_tutor.py`**

* **Purpose**: Manages AI tutor interactions.
* **Latest Version**: Includes OpenAI integration for advanced NLP.
*   **Code**:

    ```python
    from fastapi import APIRouter, HTTPException
    from psycopg2.extras import RealDictCursor
    import psycopg2
    import uuid
    import openai

    router = APIRouter()

    openai.api_key = "your-openai-api-key"

    def get_db_connection():
        return psycopg2.connect(
            dbname="satprep",
            user="user",
            password="password",
            host="localhost",
            port="5432"
        )

    @router.post("/ai_tutor/ask")
    async def ask_ai_tutor(user_id: str, query: str):
        conn = get_db_connection()
        cursor = conn.cursor(cursor_factory=RealDictCursor)
        try:
            cursor.execute("SELECT * FROM proficiencies WHERE user_id = %s", (user_id,))
            proficiencies = cursor.fetchall()
            context = f"User proficiencies: {proficiencies}. Query: {query}"
            response = openai.Completion.create(
                model="gpt-4",
                prompt=context,
                max_tokens=150
            )
            interaction_id = str(uuid.uuid4())
            cursor.execute("""
                INSERT INTO tutor_interactions (id, user_id, query, response, duration)
                VALUES (%s, %s, %s, %s, %s);
            """, (interaction_id, user_id, query, response.choices[0].text, 0))
            conn.commit()
            return {"response": response.choices[0].text}
        except Exception as e:
            conn.rollback()
            raise HTTPException(status_code=500, detail=str(e))
        finally:
            cursor.close()
            conn.close()
    ```

**`frontend/backend/src/social.py`**

* **Purpose**: Manages social features (posts, friends, teams).
* **Latest Version**: Includes all social endpoints.
*   **Code**:

    ```python
    from fastapi import APIRouter, HTTPException
    from psycopg2.extras import RealDictCursor
    import psycopg2
    import uuid

    router = APIRouter()

    def get_db_connection():
        return psycopg2.connect(
            dbname="satprep",
            user="user",
            password="password",
            host="localhost",
            port="5432"
        )

    @router.post("/social/posts")
    async def create_post(user_id: str, content: str):
        conn = get_db_connection()
        cursor = conn.cursor(cursor_factory=RealDictCursor)
        try:
            post_id = str(uuid.uuid4())
            cursor.execute("""
                INSERT INTO posts (id, user_id, content)
                VALUES (%s, %s, %s);
            """, (post_id, user_id, content))
            conn.commit()
            return {"message": "Post created"}
        except Exception as e:
            conn.rollback()
            raise HTTPException(status_code=500, detail=str(e))
        finally:
            cursor.close()
            conn.close()

    @router.get("/social/posts")
    async def get_posts():
        conn = get_db_connection()
        cursor = conn.cursor(cursor_factory=RealDictCursor)
        try:
            cursor.execute("SELECT * FROM posts ORDER BY timestamp DESC")
            posts = cursor.fetchall()
            return posts
        except Exception as e:
            raise HTTPException(status_code=500, detail=str(e))
        finally:
            cursor.close()
            conn.close()

    @router.get("/social/friends/{user_id}")
    async def get_friends(user_id: str):
        conn = get_db_connection()
        cursor = conn.cursor(cursor_factory=RealDictCursor)
        try:
            cursor.execute("SELECT * FROM friends WHERE user_id = %s", (user_id,))
            friends = cursor.fetchall()
            return friends
        except Exception as e:
            raise HTTPException(status_code=500, detail=str(e))
        finally:
            cursor.close()
            conn.close()

    @router.post("/teams/create")
    async def create_team(team_name: str, user_id: str):
        conn = get_db_connection()
        cursor = conn.cursor(cursor_factory=RealDictCursor)
        try:
            team_id = str(uuid.uuid4())
            cursor.execute("""
                INSERT INTO teams (id, team_name, user_id)
                VALUES (%s, %s, %s);
            """, (team_id, team_name, user_id))
            conn.commit()
            return {"message": "Team created"}
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
            cursor.execute("SELECT * FROM teams ORDER BY points DESC LIMIT 10")
            leaderboard = cursor.fetchall()
            return leaderboard
        except Exception as e:
            raise HTTPException(status_code=500, detail=str(e))
        finally:
            cursor.close()
            conn.close()
    ```

**`frontend/backend/src/gamification.py`**

* **Purpose**: Manages gamification features (coins, challenges, rewards, leaderboards).
* **Latest Version**: Includes WebSocket updates.
*   **Code**:

    ```python
    from fastapi import APIRouter, WebSocket, HTTPException
    from psycopg2.extras import RealDictCursor
    import psycopg2
    import uuid
    import asyncio

    router = APIRouter()

    def get_db_connection():
        return psycopg2.connect(
            dbname="satprep",
            user="user",
            password="password",
            host="localhost",
            port="5432"
        )

    active_connections = {}

    @router.websocket("/gamification/updates/{user_id}")
    async def websocket_endpoint(websocket: WebSocket, user_id: str):
        await websocket.accept()
        active_connections[user_id] = websocket
        try:
            while True:
                await websocket.receive_text()
        except Exception as e:
            print(f"WebSocket error: {e}")
        finally:
            del active_connections[user_id]
            await websocket.close()

    async def notify_user(user_id: str, message: str):
        if user_id in active_connections:
            await active_connections[user_id].send_text(message)

    @router.get("/gamification/coins/{user_id}")
    async def get_coins(user_id: str):
        conn = get_db_connection()
        cursor = conn.cursor(cursor_factory=RealDictCursor)
        try:
            cursor.execute("SELECT coins FROM users WHERE user_id = %s", (user_id,))
            user = cursor.fetchone()
            if not user:
                raise HTTPException(status_code=404, detail="User not found")
            return {"coins": user['coins']}
        finally:
            cursor.close()
            conn.close()

    @router.post("/gamification/coins/earn")
    async def earn_coins(user_id: str, amount: int):
        conn = get_db_connection()
        cursor = conn.cursor(cursor_factory=RealDictCursor)
        try:
            cursor.execute("UPDATE users SET coins = coins + %s WHERE user_id = %s", (amount, user_id))
            cursor.execute("SELECT coins FROM users WHERE user_id = %s", (user_id,))
            user = cursor.fetchone()
            conn.commit()
            await notify_user(user_id, f"You earned {amount} coins! Total: {user['coins']}")
            return {"message": "Coins earned", "new_balance": user['coins']}
        finally:
            cursor.close()
            conn.close()

    @router.post("/gamification/rewards/unlock")
    async def unlock_reward(user_id: str, reward_id: str, reward_type: str, cost: int):
        conn = get_db_connection()
        cursor = conn.cursor(cursor_factory=RealDictCursor)
        try:
            cursor.execute("SELECT coins FROM users WHERE user_id = %s", (user_id,))
            user = cursor.fetchone()
            if user['coins'] < cost:
                raise HTTPException(status_code=400, detail="Not enough coins")
            cursor.execute("UPDATE users SET coins = coins - %s WHERE user_id = %s", (cost, user_id))
            conn.commit()
            await notify_user(user_id, f"You unlocked the {reward_id} {reward_type}!")
            return {"message": "Reward unlocked"}
        finally:
            cursor.close()
            conn.close()

    @router.get("/rewards/available")
    async def get_available_rewards():
        return [
            {"id": "learner_star_avatar", "name": "Learner Star Avatar", "type": "avatar", "cost": 0, "image": "learner_star.png"},
            {"id": "math_prodigy_badge", "name": "Math Prodigy Badge", "type": "badge", "cost": 100, "image": "math_prodigy.png"}
        ]

    @router.post("/gamification/challenges")
    async def create_challenge(user_id: str, challenge_type: str, target: int):
        conn = get_db_connection()
        cursor = conn.cursor(cursor_factory=RealDictCursor)
        try:
            challenge_id = str(uuid.uuid4())
            cursor.execute("""
                INSERT INTO challenges (id, user_id, challenge_type, target, progress)
                VALUES (%s, %s, %s, %s, 0);
            """, (challenge_id, user_id, challenge_type, target))
            conn.commit()
            await notify_user(user_id, f"New challenge: {challenge_type} (Target: {target})")
            return {"message": "Challenge created"}
        finally:
            cursor.close()
            conn.close()

    @router.post("/gamification/challenges/friend")
    async def challenge_friend(friendId: str):
        return {"message": f"Challenge sent to {friendId}"}

    @router.get("/challenges/{user_id}")
    async def get_challenges(user_id: str):
        return [
            {"id": "challenge1", "challenge_type": "daily_questions", "target": 10, "progress": 5, "completed": false},
            {"id": "challenge2", "challenge_type": "speed_session", "target": 5, "progress": 3, "completed": false}
        ]

    @router.get("/leaderboards/{skill}")
    async def get_leaderboards(skill: str, user_id: str):
        return {
            "global": [
                {"user_id": "user1", "email": "user1@example.com", "score": 1500},
                {"user_id": "user2", "email": "user2@example.com", "score": 1450}
            ],
            "friends": [
                {"user_id": "friend1", "email": "friend1@example.com", "score": 1400}
            ]
        }
    ```

**`frontend/backend/src/notifications.py`**

* **Purpose**: Manages push notifications.
* **Latest Version**: Includes FCM integration.
*   **Code**:

    ```python
    from fastapi import APIRouter, HTTPException
    import firebase_admin
    from firebase_admin import credentials, messaging
    from psycopg2.extras import RealDictCursor
    import psycopg2

    router = APIRouter()

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
                    title="SAT Smart Prep App",
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

**`frontend/backend/requirements.txt`**

* **Purpose**: Lists backend dependencies.
*   **Code**:

    ```
    fastapi==0.68.0
    uvicorn==0.15.0
    psycopg2-binary==2.9.1
    bcrypt==3.2.0
    python-jose==3.3.0
    firebase-admin==5.0.1
    openai==0.10.5
    ```

**`frontend/backend/Dockerfile`**

* **Purpose**: Docker configuration for the backend.
*   **Code**:

    ```dockerfile
    FROM python:3.9-slim

    WORKDIR /app

    COPY requirements.txt .
    RUN pip install -r requirements.txt

    COPY . .

    CMD ["uvicorn", "backend.src.main:app", "--host", "0.0.0.0", "--port", "8000"]
    ```

#### Backend Tests (`frontend/backend/tests/`)

**`frontend/backend/tests/test_auth.py`**

* **Purpose**: Tests authentication endpoints.
*   **Code**:

    ```python
    from fastapi.testclient import TestClient
    from backend.src.main import app

    client = TestClient(app)

    def test_signup():
        response = client.post("/auth/signup", json={"email": "test@example.com", "password": "password123"})
        assert response.status_code == 200
        assert "user_id" in response.json()

    def test_login():
        client.post("/auth/signup", json={"email": "test@example.com", "password": "password123"})
        response = client.post("/auth/login", json={"email": "test@example.com", "password": "password123"})
        assert response.status_code == 200
        assert "user_id" in response.json()
    ```

**`frontend/backend/tests/test_diagnostic.py`**

* **Purpose**: Tests diagnostic endpoints.
*   **Code**:

    ```python
    from fastapi.testclient import TestClient
    from backend.src.main import app

    client = TestClient(app)

    def test_start_diagnostic():
        response = client.post("/diagnostic/start/user1", params={"num_questions": 22, "test_type": "SAT"})
        assert response.status_code == 200
        data = response.json()
        assert "session_id" in data
        assert "questions" in data
    ```

**`frontend/backend/tests/test_practice.py`**

* **Purpose**: Tests practice endpoints.
* **Latest Version**: Includes device tracking test.
*   **Code**:

    ```python
    from fastapi.testclient import TestClient
    from backend.src.main import app
    from psycopg2.extras import RealDictCursor
    import psycopg2

    client = TestClient(app)

    def get_db_connection():
        return psycopg2.connect(
            dbname="satprep",
            user="user",
            password="password",
            host="localhost",
            port="5432"
        )

    def test_start_practice_with_device():
        response = client.post("/practice/start/user1", json={
            "domain": "Math",
            "num_questions": 44,
            "test_type": "SAT",
            "device": "mobile"
        })
        assert response.status_code == 200
        data = response.json()
        assert "session_id" in data
        assert "questions" in data

        conn = get_db_connection()
        cursor = conn.cursor(cursor_factory=RealDictCursor)
        cursor.execute("SELECT device FROM practice_sessions WHERE session_id = %s", (data["session_id"],))
        session = cursor.fetchone()
        assert session["device"] == "mobile"
        cursor.close()
        conn.close()

    def test_device_stats():
        client.post("/practice/start/user1", json={
            "domain": "Math",
            "num_questions": 44,
            "test_type": "SAT",
            "device": "mobile"
        })
        client.post("/practice/start/user1", json={
            "domain": "Reading & Writing",
            "num_questions": 44,
            "test_type": "SAT",
            "device": "web"
        })

        response = client.get("/practice/device-stats")
        assert response.status_code == 200
        data = response.json()
        assert len(data) == 2
        assert any(stat["device"] == "mobile" and stat["count"] == 1 for stat in data)
        assert any(stat["device"] == "web" and stat["count"] == 1 for stat in data)
    ```

**`frontend/backend/tests/test_study_plan.py`**

* **Purpose**: Tests study plan endpoints.
*   **Code**:

    ```python
    from fastapi.testclient import TestClient
    from backend.src.main import app

    client = TestClient(app)

    def test_create_study_plan():
        response = client.post("/study_plan/create/user1", json={
            "test_date": "2025-12-01T00:00:00Z",
            "study_hours": 10,
            "study_days": 5
        })
        assert response.status_code == 200
        data = response.json()
        assert "plan_id" in data
        assert "sessions_per_week" in data
        assert "focus_areas" in data
        assert "milestones" in data

    def test_get_study_plan():
        client.post("/study_plan/create/user1", json={
            "test_date": "2025-12-01T00:00:00Z",
            "study_hours": 10,
            "study_days": 5
        })
        response = client.get("/study_plan/user1")
        assert response.status_code == 200
        data = response.json()
        assert "plan_id" in data
        assert "actions" in data
    ```

**`frontend/backend/tests/test_progress.py`**

* **Purpose**: Tests progress monitoring endpoints.
*   **Code**:

    ```python
    from fastapi.testclient import TestClient
    from backend.src.main import app

    client = TestClient(app)

    def test_get_proficiencies():
        response = client.get("/progress_monitoring/proficiencies/user1")
        assert response.status_code == 200
        assert isinstance(response.json(), list)

    def test_get_score_history():
        response = client.get("/progress_monitoring/scores/user1")
        assert response.status_code == 200
        assert isinstance(response.json(), list)
    ```

**`frontend/backend/tests/test_ai_tutor.py`**

* **Purpose**: Tests AI tutor endpoints.
*   **Code**:

    ```python
    from fastapi.testclient import TestClient
    from backend.src.main import app

    client = TestClient(app)

    def test_ask_ai_tutor():
        response = client.post("/ai_tutor/ask", params={"user_id": "user1", "query": "How do I solve this math problem?"})
        assert response.status_code == 200
        assert "response" in response.json()
    ```

**`frontend/backend/tests/test_social.py`**

* **Purpose**: Tests social endpoints.
*   **Code**:

    ```python
    from fastapi.testclient import TestClient
    from backend.src.main import app

    client = TestClient(app)

    def test_create_post():
        response = client.post("/social/posts", params={"user_id": "user1", "content": "Hello, world!"})
        assert response.status_code == 200
        assert response.json() == {"message": "Post created"}

    def test_get_posts():
        response = client.get("/social/posts")
        assert response.status_code == 200
        assert isinstance(response.json(), list)
    ```

**`frontend/backend/tests/test_gamification.py`**

* **Purpose**: Tests gamification endpoints.
*   **Code**:

    ```python
    from fastapi.testclient import TestClient
    from backend.src.main import app

    client = TestClient(app)

    def test_get_coins():
        response = client.get("/gamification/coins/user1")
        assert response.status_code == 200
        assert "coins" in response.json()

    def test_earn_coins():
        response = client.post("/gamification/coins/earn", params={"user_id": "user1", "amount": 50})
        assert response.status_code == 200
        assert "new_balance" in response.json()
    ```

**`frontend/backend/tests/test_notifications.py`**

* **Purpose**: Tests notification endpoints.
*   **Code**:

    ```python
    from fastapi.testclient import TestClient
    from backend.src.main import app

    client = TestClient(app)

    def test_send_notification():
        # This test requires a valid device token and Firebase setup
        response = client.post("/send-notification", params={"user_id": "user1", "message": "Test notification"})
        assert response.status_code == 404  # Expected to fail without a device token
    ```

#### Root Files

**`README.md`**

* **Purpose**: Provides installation and testing instructions (to be completed in the final step).
*   **Code** (Placeholder for now, will be updated in the final step):

    ```markdown
    # SAT Smart Prep App by Learner Labs

    A digital SAT prep app with AI-driven features, personalized study plans, and gamification.

    ## Installation and Testing Instructions
    (To be completed in the final step)
    ```

**`docker-compose.yml`**

* **Purpose**: Configures Docker services for local development (PostgreSQL, FastAPI backend).
*   **Code**:

    ```yaml
    version: '3.8'

    services:
      db:
        image: postgres:13
        environment:
          POSTGRES_DB: satprep
          POSTGRES_USER: user
          POSTGRES_PASSWORD: password
        ports:
          - "5432:5432"
        volumes:
          - db_data:/var/lib/postgresql/data

      backend:
        build:
          context: ./frontend
          dockerfile: backend/Dockerfile
        ports:
          - "8000:8000"
        depends_on:
          - db
        environment:
          - DATABASE_URL=postgresql://user:password@db:5432/satprep

      web:
        build:
          context: ./frontend
          dockerfile: Dockerfile
        ports:
          - "3000:3000"
        depends_on:
          - backend

    volumes:
      db_data:
    ```

**`setup.sh`**

* **Purpose**: Script to set up the local development environment (to be completed in the final step).
*   **Code** (Placeholder for now):

    ```bash
    #!/bin/bash
    echo "Setting up SAT Smart Prep App..."
    # (To be completed in the final step)
    ```

***

### Set 2: Web App Frontend Files (`frontend/`)

#### Project Structure for Set 2

```
SATSmartPrepAppRepo/
├── frontend/
│   ├── components/
│   │   ├── layouts/
│   │   │   ├── ReadingWritingTest.js
│   │   │   ├── MathBasicTest.js
│   │   │   ├── MathGraphTest.js
│   │   │   └── MathTableTest.js
│   │   ├── QuestionDisplay.js
│   │   ├── PassageDisplay.js
│   │   ├── MathQuestionHeader.js
│   │   └── AnswerEliminator.js
│   ├── pages/
│   │   ├── _app.js
│   │   ├── index.js
│   │   ├── login.js
│   │   ├── onboarding.js
│   │   ├── diagnostic.js
│   │   ├── practice-bluebook.js
│   │   ├── study-plan.js
│   │   ├── dashboard.js
│   │   ├── community.js
│   │   ├── leaderboard.js
│   │   ├── rewards-store.js
│   │   ├── tutor-parent.js
│   │   ├── policy.js
│   │   └── unsupported-browser.js
│   ├── styles/
│   │   └── styles.js
│   ├── utils/
│   │   └── api.js
│   ├── config/
│   │   └── integrations.js
```

#### Components (`frontend/components/`)

**`frontend/components/layouts/ReadingWritingTest.js`**

* **Purpose**: Layout for Reading/Writing tests.
* **Latest Version**: Includes Bluebook-like features (e.g., line reader, zoom).
*   **Code**:

    ```jsx
    import React, { useState } from 'react';
    import { PassageDisplay } from '../PassageDisplay';
    import { QuestionDisplay } from '../QuestionDisplay';
    import { MathQuestionHeader } from '..  /MathQuestionHeader';
    import { colors } from '../styles';

    const ReadingWritingTest = ({
      eliminatorActive,
      setEliminatorActive,
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
      const [zoomLevel, setZoomLevel] = useState(1);
      const [lineReaderPosition, setLineReaderPosition] = useState(0);

      const passageText = `While researching a topic, a student has taken the following notes...`;

      const zoomIn = () => setZoomLevel(zoomLevel + 0.1);
      const zoomOut = () => setZoomLevel(zoomLevel - 0.1);
      const moveLineReader = (direction) => {
        setLineReaderPosition(prev => prev + (direction === 'up' ? -20 : 20));
      };

      return (
        <div className="reading-writing-test">
          <MathQuestionHeader
            questionNumber={questionNumber}
            totalQuestions={totalQuestions}
            timer={timer}
            onNext={onNext}
            onPrevious={onPrevious}
            showCalculator={showCalculator}
            userName={userName}
            onTimeEnd={onTimeEnd}
            customButtons={
              <div className="custom-buttons">
                {customButtons}
                <button onClick={zoomIn}>Zoom In</button>
                <button onClick={zoomOut}>Zoom Out</button>
                <button onClick={() => moveLineReader('up')}>Line Up</button>
                <button onClick={() => moveLineReader('down')}>Line Down</button>
              </div>
            }
          />
          <div className="passage">
            <PassageDisplay passageText={passageText} style={{ transform: `scale(${zoomLevel})` }} />
            {lineReaderPosition > 0 && (
              <div className="line-reader" style={{ top: `${lineReaderPosition}px` }} />
            )}
          </div>
          <div className="question">
            <QuestionDisplay
              questionNumber={questionNumber}
              eliminatorActive={eliminatorActive}
              toggleEliminator={setEliminatorActive}
            />
          </div>

          <style jsx>{`
            .reading-writing-test {
              display: flex;
              flex-direction: column;
              height: 100%;
            }
            .passage {
              flex: 1;
              border-bottom: 1px solid ${colors.gray};
              padding: 20px;
              position: relative;
            }
            .question {
              flex: 1;
              padding: 20px;
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
            .line-reader {
              position: absolute;
              left: 0;
              right: 0;
              height: 2px;
              background-color: yellow;
              opacity: 0.5;
            }
          `}</style>
        </div>
      );
    };

    export default ReadingWritingTest;
    ```

**`frontend/components/layouts/MathBasicTest.js`**

* **Purpose**: Layout for basic Math tests.
* **Latest Version**: No recent changes.
*   **Code**:

    ```javascript
    import React from 'react';
    import { QuestionDisplay } from '../QuestionDisplay';
    import { MathQuestionHeader } from '../MathQuestionHeader';

    const MathBasicTest = ({
      eliminatorActive,
      setEliminatorActive,
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
        <div className="math-basic-test">
          <MathQuestionHeader
            questionNumber={questionNumber}
            totalQuestions={totalQuestions}
            timer={timer}
            onNext={onNext}
            onPrevious={onPrevious}
            showCalculator={showCalculator}
            userName={userName}
            onTimeEnd={onTimeEnd}
            customButtons={customButtons}
          />
          <div className="question">
            <QuestionDisplay
              questionNumber={questionNumber}
              eliminatorActive={eliminatorActive}
              toggleEliminator={setEliminatorActive}
            />
          </div>

          <style jsx>{`
            .math-basic-test {
              display: flex;
              flex-direction: column;
              height: 100%;
            }
            .question {
              flex: 1;
              padding: 20px;
            }
          `}</style>
        </div>
      );
    };

    export default MathBasicTest;
    ```

**`frontend/components/layouts/MathGraphTest.js`**

* **Purpose**: Layout for Math tests with graphs.
* **Latest Version**: No recent changes.
*   **Code**:

    ```javascript
    import React from 'react';
    import { QuestionDisplay } from '../QuestionDisplay';
    import { MathQuestionHeader } from '../MathQuestionHeader';
    import { colors, typography } from '../styles';

    const MathGraphTest = ({
      eliminatorActive,
      setEliminatorActive,
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
        <div className="math-graph-test">
          <MathQuestionHeader
            questionNumber={questionNumber}
            totalQuestions={totalQuestions}
            timer={timer}
            onNext={onNext}
            onPrevious={onPrevious}
            showCalculator={showCalculator}
            userName={userName}
            onTimeEnd={onTimeEnd}
            customButtons={customButtons}
          />
          <div className="graph">
            <p style={typography.body}>[Graph Placeholder]</p>
          </div>
          <div className="question">
            <QuestionDisplay
              questionNumber={questionNumber}
              eliminatorActive={eliminatorActive}
              toggleEliminator={setEliminatorActive}
            />
          </div>

          <style jsx>{`
            .math-graph-test {
              display: flex;
              flex-direction: column;
              height: 100%;
            }
            .graph {
              flex: 1;
              border-bottom: 1px solid ${colors.gray};
              padding: 20px;
              text-align: center;
            }
            .question {
              flex: 1;
              padding: 20px;
            }
          `}</style>
        </div>
      );
    };

    export default MathGraphTest;
    ```

**`frontend/components/layouts/MathTableTest.js`**

* **Purpose**: Layout for Math tests with tables.
* **Latest Version**: No recent changes.
*   **Code**:

    ```javascript
    import React from 'react';
    import { QuestionDisplay } from '../QuestionDisplay';
    import { MathQuestionHeader } from '../MathQuestionHeader';
    import { colors, typography } from '../styles';

    const MathTableTest = ({
      eliminatorActive,
      setEliminatorActive,
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
        <div className="math-table-test">
          <MathQuestionHeader
            questionNumber={questionNumber}
            totalQuestions={totalQuestions}
            timer={timer}
            onNext={onNext}
            onPrevious={onPrevious}
            showCalculator={showCalculator}
            userName={userName}
            onTimeEnd={onTimeEnd}
            customButtons={customButtons}
          />
          <div className="table">
            <p style={typography.body}>[Table Placeholder]</p>
          </div>
          <div className="question">
            <QuestionDisplay
              questionNumber={questionNumber}
              eliminatorActive={eliminatorActive}
              toggleEliminator={setEliminatorActive}
            />
          </div>

          <style jsx>{`
            .math-table-test {
              display: flex;
              flex-direction: column;
              height: 100%;
            }
            .table {
              flex: 1;
              border-bottom: 1px solid ${colors.gray};
              padding: 20px;
              text-align: center;
            }
            .question {
              flex: 1;
              padding: 20px;
            }
          `}</style>
        </div>
      );
    };

    export default MathTableTest;
    ```

**`frontend/components/QuestionDisplay.js`**

* **Purpose**: Displays a question with answer options.
* **Latest Version**: No recent changes.
*   **Code**:

    ```javascript
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

**`frontend/components/PassageDisplay.js`**

* **Purpose**: Displays passages with highlighting.
* **Latest Version**: No recent changes.
*   **Code**:

    ```javascript
    import React, { useState } from 'react';
    import { colors, typography } from '../styles';

    export const PassageDisplay = ({ passageText }) => {
      const [highlighted, setHighlighted] = useState(false);

      const toggleHighlight = () => {
        setHighlighted(!highlighted);
      };

      return (
        <div className="passage-display">
          <p style={[typography.body, highlighted && { backgroundColor: 'yellow' }]}>
            {passageText}
          </p>
          <button onClick={toggleHighlight} className="highlight-button">
            {highlighted ? 'Remove Highlight' : 'Highlight'}
          </button>

          <style jsx>{`
            .passage-display {
              padding: 10px;
            }
            .highlight-button {
              margin-top: 10px;
              padding: 5px;
              background: ${colors.primary};
              color: ${colors.white};
              border: none;
              border-radius: 5px;
              cursor: pointer;
            }
          `}</style>
        </div>
      );
    };
    ```

**`frontend/components/MathQuestionHeader.js`**

* **Purpose**: Displays the header for Math questions.
* **Latest Version**: No recent changes.
*   **Code**:

    ```javascript
    import React from 'react';
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
        <div className="math-question-header">
          <div className="header-left">
            <span className="question-number">{questionNumber}/{totalQuestions}</span>
            <p style={typography.body}>{timer}</p>
          </div>
          <div className="header-right">
            {customButtons}
            {showCalculator && (
              <button className="calculator-button">
                <span style={typography.body}>Calc</span>
              </button>
            )}
            <button onClick={onPrevious} className="nav-button">
              <span style={typography.body}>Previous</span>
            </button>
            <button onClick={onNext} className="nav-button">
              <span style={typography.body}>Next</span>
            </button>
          </div>

          <style jsx>{`
            .math-question-header {
              display: flex;
              justify-content: space-between;
              align-items: center;
              padding: 10px;
              border-bottom: 1px solid ${colors.gray};
            }
            .header-left {
              display: flex;
              gap: 10px;
            }
            .header-right {
              display: flex;
              gap: 10px;
            }
            .question-number {
              font-weight: bold;
              background: #333;
              color: white;
              padding: 8px 12px;
              border
    ```
