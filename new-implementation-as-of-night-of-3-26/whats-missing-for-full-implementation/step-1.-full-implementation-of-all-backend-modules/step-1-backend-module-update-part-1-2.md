# Step 1: Backend Module update Part 1/2

To address **Item 1: Full Implementation of All Backend Modules** for the **SAT Prep Suite**, I’ll provide complete, fully functional implementations for all backend modules previously referenced (`auth.py`, `diagnostic.py`, `full_length_test.py`, `study_plan.py`, `practice_module.py`, `progress_monitoring.py`, `question_review.py`, `gamification.py`, `ai_tutor.py`, `tutor_parent.py`, `social.py`, `sync.py`, `utils.py`, and `main.py`). I’ll ensure each module integrates seamlessly with the others, includes all required functionality, and aligns with the full-featured app scope (including social, engagement, AI, and data collection). After each module, I’ll outline a testing step to verify integration, followed by a recheck to confirm everything works together. Given the complexity, I’ll provide full code for each file and test incrementally.

***

### Updated Directory Structure (Backend Focus)

```
sat-prep-suite/backend/
├── migrations/
│   ├── init_db.py
│   └── seed_data.py
├── src/
│   ├── __init__.py
│   ├── auth.py
│   ├── database.py
│   ├── diagnostic.py
│   ├── full_length_test.py
│   ├── practice_module.py
│   ├── progress_monitoring.py
│   ├── question_review.py
│   ├── study_plan.py
│   ├── gamification.py
│   ├── ai_tutor.py
│   ├── tutor_parent.py
│   ├── social.py
│   ├── sync.py
│   ├── utils.py
│   └── main.py
├── ai_data/
│   └── interactions.jsonl  # JSON Lines for AI training data
├── .env.example
├── Dockerfile
├── docker-compose.yml
└── requirements.txt
```

***

### Backend Module Implementations

#### 1. `src/auth.py`

* **Functionality**: User authentication (signup, login) with JWT, role management.

```python
from fastapi import APIRouter, Depends, HTTPException
from pydantic import BaseModel
from jose import jwt, JWTError
from sqlalchemy.orm import Session
from .database import get_db, User
import os

router = APIRouter(prefix="/auth", tags=["auth"])
SECRET_KEY = os.getenv("SECRET_KEY", "your-secret-key")

class UserCreate(BaseModel):
    email: str
    password: str
    study_hours: int
    role: str = "student"
    linked_user_id: str = None

class UserLogin(BaseModel):
    email: str
    password: str

@router.post("/signup")
async def signup(user: UserCreate, db: Session = Depends(get_db)):
    existing_user = db.query(User).filter(User.email == user.email).first()
    if existing_user:
        raise HTTPException(status_code=400, detail="Email already registered")
    db_user = User(
        user_id=user.email.split("@")[0],
        email=user.email,
        hashed_password=user.password,  # In production, hash this
        role=user.role,
        study_hours=user.study_hours,
        linked_user_id=user.linked_user_id
    )
    db.add(db_user)
    db.commit()
    token = jwt.encode({"user_id": db_user.user_id}, SECRET_KEY, algorithm="HS256")
    return {"access_token": token}

@router.post("/login")
async def login(user: UserLogin, db: Session = Depends(get_db)):
    db_user = db.query(User).filter(User.email == user.email).first()
    if not db_user or db_user.hashed_password != user.password:
        raise HTTPException(status_code=401, detail="Invalid credentials")
    token = jwt.encode({"user_id": db_user.user_id}, SECRET_KEY, algorithm="HS256")
    return {"access_token": token}

def get_current_user(token: str = Depends(lambda x: x.headers.get("Authorization").split()[1]), db: Session = Depends(get_db)):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
        user_id = payload.get("user_id")
        user = db.query(User).filter(User.user_id == user_id).first()
        if not user:
            raise HTTPException(status_code=401, detail="User not found")
        return user
    except JWTError:
        raise HTTPException(status_code=401, detail="Invalid token")
```

*   **Test**:

    ```bash
    curl -X POST "http://localhost:8000/auth/signup" -H "Content-Type: application/json" -d '{"email": "alex@example.com", "password": "pass123", "study_hours": 15}'
    ```

    * **Expected**: `{"access_token": "..."}`, user in `users` table.
* **Integration Check**: Token works with subsequent API calls.

***

#### 2. `src/database.py`

* **Functionality**: Defines all database models and connection.

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
    linked_user_id = Column(String, ForeignKey("users.user_id"), nullable=True)

class Question(Base):
    __tablename__ = "questions"
    question_id = Column(String, primary_key=True, index=True)
    domain = Column(String)
    skill = Column(String)
    text = Column(String)
    options = Column(String, nullable=True)  # JSON string for MCQs
    correct_answer = Column(String)
    a_param = Column(Float, default=1.0)  # IRT discrimination
    b_param = Column(Float, default=0.0)  # IRT difficulty
    c_param = Column(Float, default=0.25)  # IRT guessing

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
    test_type = Column(String)  # "diagnostic", "full", "practice"
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

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

* **Test**: Run `python migrations/init_db.py` → Verify all tables created in PostgreSQL.

***

#### 3. `src/diagnostic.py`

* **Functionality**: Runs adaptive diagnostic test (Math: 22 questions, R\&W: 27 questions).

```python
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from pydantic import BaseModel
from .database import get_db, PracticeSession, Response, Proficiency
from .utils import select_next_question, update_theta
import uuid

router = APIRouter(prefix="/diagnostic", tags=["diagnostic"])

class DiagnosticRequest(BaseModel):
    section: str

class ResponseCreate(BaseModel):
    question_id: str
    answer: str
    time_spent: int

@router.post("/start/{user_id}")
async def start_diagnostic(user_id: str, request: DiagnosticRequest, db: Session = Depends(get_db)):
    if request.section not in ["Math", "Reading & Writing"]:
        raise HTTPException(status_code=400, detail="Invalid section")
    session_id = str(uuid.uuid4())
    theta = db.query(Proficiency).filter(Proficiency.user_id == user_id, Proficiency.domain == request.section).order_by(Proficiency.timestamp.desc()).first()
    theta = theta.theta if theta else 0.0
    question = select_next_question(db, user_id, theta, session_id, request.section)
    session = PracticeSession(session_id=session_id, user_id=user_id, theta=theta, test_type="diagnostic", section=request.section)
    db.add(session)
    db.commit()
    return {"session_id": session_id, "questions": [{"id": question.question_id, "text": question.text, "domain": question.domain}]}

@router.post("/submit/{session_id}")
async def submit_diagnostic(session_id: str, responses: list[ResponseCreate], db: Session = Depends(get_db)):
    session = db.query(PracticeSession).filter(PracticeSession.session_id == session_id).first()
    if not session:
        raise HTTPException(status_code=404, detail="Session not found")
    theta = session.theta
    questions = [db.query(Question).filter(Question.question_id == r.question_id).first() for r in responses]
    for r, q in zip(responses, questions):
        db_response = Response(
            session_id=session_id,
            user_id=session.user_id,
            question_id=r.question_id,
            answer=r.answer,
            time_spent=r.time_spent,
            timestamp=datetime.utcnow()
        )
        db.add(db_response)
        correct = r.answer == q.correct_answer
        theta = update_theta(theta, q.a_param, q.b_param, q.c_param, correct)
        db.add(Proficiency(user_id=session.user_id, domain=q.domain, skill=q.skill, theta=theta))
    
    total_questions = len(db.query(Response).filter(Response.session_id == session_id).all())
    target_questions = 22 if session.section == "Math" else 27
    db.commit()
    if total_questions < target_questions:
        next_question = select_next_question(db, session.user_id, theta, session_id, session.section)
        session.theta = theta
        db.commit()
        return {"questions": [{"id": next_question.question_id, "text": next_question.text, "domain": next_question.domain}], "theta": theta}
    else:
        score = int(200 + (theta + 3) / 6 * 600)  # 200-800 scale
        session.theta = theta
        db.commit()
        return {"score": score, "theta": theta}
```

*   **Test**:

    ```bash
    curl -X POST "http://localhost:8000/diagnostic/start/alex123" -H "Content-Type: application/json" -d '{"section": "Math"}'
    ```

    * Submit responses incrementally until 22 questions → Expect `{"score": ~620, "theta": ~0.8}`.
* **Integration Check**: `proficiencies` table updated, no repeated questions.

***

#### 4. `src/full_length_test.py`

* **Functionality**: Simulates full SAT with adaptive modules, updates study plan.

```python
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from pydantic import BaseModel
from .database import get_db, PracticeSession, Response, Proficiency
from .utils import select_next_question, update_theta
from .study_plan import update_study_plan
import uuid

router = APIRouter(prefix="/full_test", tags=["full_test"])

class FullTestRequest(BaseModel):
    sections: list[str]
    plan_id: str = None

class ResponseCreate(BaseModel):
    question_id: str
    answer: str
    time_spent: int

@router.post("/start/{user_id}")
async def start_full_test(user_id: str, request: FullTestRequest, db: Session = Depends(get_db)):
    if not all(s in ["Math", "Reading & Writing", "Both"] for s in request.sections):
        raise HTTPException(status_code=400, detail="Invalid sections")
    sections = ["Math", "Reading & Writing"] if "Both" in request.sections else request.sections
    session_id = str(uuid.uuid4())
    theta = {s: db.query(Proficiency).filter(Proficiency.user_id == user_id, Proficiency.domain == s).order_by(Proficiency.timestamp.desc()).first().theta if db.query(Proficiency).filter(Proficiency.user_id == user_id, Proficiency.domain == s).first() else 0.0 for s in sections}
    questions = []
    for section in sections:
        q = select_next_question(db, user_id, theta[section], session_id, section)
        if q:
            questions.append(q)
    session = PracticeSession(session_id=session_id, user_id=user_id, theta=sum(theta.values()) / len(theta), test_type="full", section=",".join(sections), plan_id=request.plan_id)
    db.add(session)
    db.commit()
    return {"test_id": session_id, "module": 1, "questions": [{"id": q.question_id, "text": q.text, "domain": q.domain} for q in questions], "theta": theta}

@router.post("/submit/{test_id}")
async def submit_full_test(test_id: str, responses: list[ResponseCreate], db: Session = Depends(get_db)):
    session = db.query(PracticeSession).filter(PracticeSession.session_id == test_id).first()
    if not session:
        raise HTTPException(status_code=404, detail="Session not found")
    sections = session.section.split(",")
    theta = {s: session.theta for s in sections} if len(sections) == 1 else {}
    questions = []
    for r in responses:
        q = db.query(Question).filter(Question.question_id == r.question_id).first()
        questions.append(q)
        db_response = Response(session_id=test_id, user_id=session.user_id, question_id=r.question_id, answer=r.answer, time_spent=r.time_spent)
        db.add(db_response)
        correct = r.answer == q.correct_answer
        theta[q.domain] = update_theta(theta.get(q.domain, 0.0), q.a_param, q.b_param, q.c_param, correct)
        db.add(Proficiency(user_id=session.user_id, domain=q.domain, skill=q.skill, theta=theta[q.domain]))
    
    db.commit()
    total_questions = len(db.query(Response).filter(Response.session_id == test_id).all())
    target_per_module = {"Math": 22, "Reading & Writing": 27}
    module_complete = all(len(db.query(Response).filter(Response.session_id == test_id, Response.question_id.in_([q.question_id for q in db.query(Question).filter(Question.domain == s).all()])).all()) >= target_per_module[s] for s in sections)
    
    if total_questions < sum(target_per_module[s] for s in sections):  # Module 1
        remaining = {s: target_per_module[s] - len(db.query(Response).filter(Response.session_id == test_id, Response.question_id.in_([q.question_id for q in db.query(Question).filter(Question.domain == s).all()])).all()) for s in sections}
        next_questions = [select_next_question(db, session.user_id, theta[s], test_id, s) for s in sections for _ in range(min(1, remaining[s]))]
        session.theta = sum(theta.values()) / len(theta)
        db.commit()
        return {"module": 1, "questions": [{"id": q.question_id, "text": q.text, "domain": q.domain} for q in next_questions if q], "theta": theta}
    elif total_questions < sum(target_per_module[s] * 2 for s in sections):  # Module 2
        if module_complete:
            next_questions = [select_next_question(db, session.user_id, theta[s], test_id, s) for s in sections for _ in range(1)]
            session.theta = sum(theta.values()) / len(theta)
            db.commit()
            return {"module": 2, "questions": [{"id": q.question_id, "text": q.text, "domain": q.domain} for q in next_questions if q], "theta": theta, "break": "10 minutes after R&W Module 2" if "Reading & Writing" in sections else None}
    else:
        scores = {s: int(200 + (theta[s] + 3) / 6 * 600) for s in sections}
        total_score = sum(scores.values())
        session.theta = sum(theta.values()) / len(theta)
        db.commit()
        if session.plan_id:
            await update_study_plan(session.plan_id, db)
        return {"scores": scores, "total_score": total_score if len(sections) > 1 else scores[sections[0]], "theta": theta}
```

*   **Test**:

    ```bash
    curl -X POST "http://localhost:8000/full_test/start/alex123" -H "Content-Type: application/json" -d '{"sections": ["Math"], "plan_id": "plan123"}'
    ```

    * Submit 44 questions in batches → Expect `{"scores": {"Math": ~680}, "total_score": 680}`.
* **Integration Check**: `study_plan.py` updates tasks post-test.

***

#### 5. `src/practice_module.py`

* **Functionality**: Delivers adaptive practice sessions.

```python
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from pydantic import BaseModel
from .database import get_db, PracticeSession, Response, Proficiency
from .utils import select_next_question, update_theta
import uuid

router = APIRouter(prefix="/practice", tags=["practice"])

class PracticeRequest(BaseModel):
    domain: str
    num_questions: int = 10

class ResponseCreate(BaseModel):
    question_id: str
    answer: str
    time_spent: int

@router.post("/start/{user_id}")
async def start_practice(user_id: str, request: PracticeRequest, db: Session = Depends(get_db)):
    session_id = str(uuid.uuid4())
    theta = db.query(Proficiency).filter(Proficiency.user_id == user_id, Proficiency.domain == request.domain).order_by(Proficiency.timestamp.desc()).first()
    theta = theta.theta if theta else 0.0
    question = select_next_question(db, user_id, theta, session_id, request.domain)
    session = PracticeSession(session_id=session_id, user_id=user_id, theta=theta, test_type="practice", section=request.domain)
    db.add(session)
    db.commit()
    return {"session_id": session_id, "questions": [{"id": question.question_id, "text": question.text, "domain": question.domain}], "theta": theta}

@router.post("/submit/{session_id}")
async def submit_practice(session_id: str, responses: list[ResponseCreate], db: Session = Depends(get_db)):
    session = db.query(PracticeSession).filter(PracticeSession.session_id == session_id).first()
    if not session:
        raise HTTPException(status_code=404, detail="Session not found")
    theta = session.theta
    questions = [db.query(Question).filter(Question.question_id == r.question_id).first() for r in responses]
    for r, q in zip(responses, questions):
        db_response = Response(session_id=session_id, user_id=session.user_id, question_id=r.question_id, answer=r.answer, time_spent=r.time_spent)
        db.add(db_response)
        correct = r.answer == q.correct_answer
        theta = update_theta(theta, q.a_param, q.b_param, q.c_param, correct)
        db.add(Proficiency(user_id=session.user_id, domain=q.domain, skill=q.skill, theta=theta))
    
    total_questions = len(db.query(Response).filter(Response.session_id == session_id).all())
    db.commit()
    if total_questions < 10:  # Example limit
        next_question = select_next_question(db, session.user_id, theta, session_id, session.section)
        session.theta = theta
        db.commit()
        return {"questions": [{"id": next_question.question_id, "text": next_question.text, "domain": next_question.domain}], "theta": theta}
    else:
        session.theta = theta
        db.commit()
        return {"theta": theta}
```

*   **Test**:

    ```bash
    curl -X POST "http://localhost:8000/practice/start/alex123" -H "Content-Type: application/json" -d '{"domain": "Math"}'
    ```

    * Submit 10 questions → Expect updated (\theta).
* **Integration Check**: `proficiencies` reflects practice results.

***

#### Remaining Modules

Due to length constraints, I’ll provide full implementations for remaining modules in subsequent responses if needed. For now, I’ll proceed with testing and integration checks for these core modules, then continue with others.

***

### Testing and Integration Check

#### Initial Setup

* **Run**: `docker-compose up --build` (ensure PostgreSQL is running).
* **Initialize DB**: `python migrations/init_db.py && python migrations/seed_data.py`.

#### Step-by-Step Tests

1. **Auth**:
   * Signup → Login → Token works for `/diagnostic/start`.
2. **Diagnostic**:
   * Start Math diagnostic → Submit 22 responses → Score returned, `proficiencies` updated.
3. **Full-Length Test**:
   * Start Math test → Submit 44 responses → Score returned, `study_plan.py` triggered.
4. **Practice**:
   * Start practice → Submit 10 responses → (\theta) updated in `proficiencies`.

#### Recheck

* **Integration**: After each test, query `proficiencies` and `responses` tables to confirm data consistency.
* **No Repeats**: Verify `select_next_question` avoids used questions across sessions.

***

### Next Steps

* **Continue Implementation**: Add `progress_monitoring.py`, `question_review.py`, etc., in the next response.
* **Full Test**: Once all modules are added, re-run the full integration test suite.

Let me know if you’d like to proceed with the next module (e.g., `progress_monitoring.py`) or adjust anything here before moving forward!
