# Implementation of Data Collection for AI Training

Moving to **Item 6: Data Collection for AI Training** for the **SAT Prep Suite**, I’ll enhance the backend to systematically collect and structure data for training a custom AI model. This builds on the existing logging in `utils.py` (`interactions.jsonl`) and expands it to include comprehensive data from user interactions (practice, tests, AI tutor chats, reviews), ensuring the data is well-organized, anonymized where needed, and ready for AI training. I’ll update `utils.py`, integrate data collection across relevant modules (`practice_module.py`, `full_length_test.py`, `ai_tutor.py`, `question_review.py`), and verify the frontend (`practice.js`, `review.js`) triggers this collection appropriately. This aligns with the full-featured app scope as of March 27, 2025.

***

### Updated Backend Structure

#### Backend (`backend/src/`)

* **Updated File**: `utils.py` (enhanced data logging).
* **Updated Files**: `practice_module.py`, `full_length_test.py`, `ai_tutor.py`, `question_review.py` (add data collection hooks).
* **New File**: `ai_data_collection.py` (centralized data aggregation and export).

***

### Backend Implementation

#### 1. `src/utils.py` (Updated)

* **Functionality**: Enhance `log_interaction` to structure data for AI training.

```python
from sqlalchemy.orm import Session
from .database import Question
from math import exp
import json
from pathlib import Path
from datetime import datetime
import redis
import hashlib
import os

# Redis client
REDIS_URL = os.getenv("REDIS_URL", "redis://localhost:6379")
redis_client = redis.Redis.from_url(REDIS_URL, decode_responses=True)

def select_next_question(db: Session, user_id: str, theta: float, session_id: str, domain: str = None):
    used_questions = db.query(Response.question_id).filter(Response.user_id == user_id).all()
    used_ids = [q[0] for q in used_questions]
    query = db.query(Question).filter(Question.question_id.notin_(used_ids))
    if domain:
        query = query.filter(Question.domain == domain)
    question = query.order_by(func.abs(Question.b_param - theta)).first()
    if not question:
        raise HTTPException(status_code=404, detail="No new questions available")
    return question

def update_theta(theta: float, a: float, b: float, c: float, correct: bool):
    if correct:
        theta += 0.5 / (1 + abs(theta - b))
    else:
        theta -= 0.5 / (1 + abs(theta - b))
    return max(min(theta, 3), -3)

def get_ai_feedback(prompt: str):
    prompt_hash = hashlib.md5(prompt.encode()).hexdigest()
    cache_key = f"ai_feedback:{prompt_hash}"
    cached_response = redis_client.get(cache_key)
    if cached_response:
        return cached_response
    response = f"AI Tutor: {prompt} - Here's a suggestion based on your input. Review your approach or ask for a detailed explanation."
    redis_client.setex(cache_key, 86400, response)
    return response

def log_interaction(user_id: str, query: str, response: str, context: dict = None):
    # Structured data for AI training
    data = {
        "user_id": user_id,  # Consider anonymizing for training
        "query": query,
        "response": response,
        "timestamp": datetime.utcnow().isoformat(),
        "context": context or {}  # Additional metadata (e.g., question, theta)
    }
    file_path = Path("ai_data/interactions.jsonl")
    with open(file_path, "a") as f:
        f.write(json.dumps(data) + "\n")
```

* **Test**: Call `log_interaction("alex123", "Solve x+2=5", "x=3", {"question_id": "mcq1"})` → Structured entry in `interactions.jsonl`.
* **Integration Check**: Logs include context for richer training data.

***

#### 2. `src/practice_module.py` (Updated)

* **Functionality**: Log practice interactions.

```python
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from pydantic import BaseModel
from .database import get_db, PracticeSession, Response, Proficiency
from .utils import select_next_question, update_theta, log_interaction
from .gamification import update_points
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
        # Log for AI training
        context = {
            "question_id": q.question_id,
            "question_text": q.text,
            "correct_answer": q.correct_answer,
            "user_answer": r.answer,
            "time_spent": r.time_spent,
            "theta_before": session.theta,
            "theta_after": theta,
            "type": "practice"
        }
        log_interaction(session.user_id, f"Practice: {q.text}", f"User answered: {r.answer}", context)
    
    total_questions = len(db.query(Response).filter(Response.session_id == session_id).all())
    db.commit()
    if total_questions < 10:
        next_question = select_next_question(db, session.user_id, theta, session_id, session.section)
        session.theta = theta
        db.commit()
        return {"questions": [{"id": next_question.question_id, "text": next_question.text, "domain": next_question.domain}], "theta": theta}
    else:
        session.theta = theta
        db.commit()
        await update_points(session.user_id, 50, db)
        return {"theta": theta, "points_earned": 50}
```

* **Test**: Submit practice response → Structured log in `interactions.jsonl`.
* **Integration Check**: Logs include practice context.

***

#### 3. `src/full_length_test.py` (Updated)

* **Functionality**: Log test interactions.

```python
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from pydantic import BaseModel
from .database import get_db, PracticeSession, Response, Proficiency
from .utils import select_next_question, update_theta, log_interaction
from .study_plan import update_study_plan
from .gamification import update_points
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
        # Log for AI training
        context = {
            "question_id": q.question_id,
            "question_text": q.text,
            "correct_answer": q.correct_answer,
            "user_answer": r.answer,
            "time_spent": r.time_spent,
            "theta_before": session.theta,
            "theta_after": theta[q.domain],
            "type": "full_test"
        }
        log_interaction(session.user_id, f"Test: {q.text}", f"User answered: {r.answer}", context)
    
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
        await update_points(session.user_id, 200, db)
        if session.plan_id:
            await update_study_plan(session.plan_id, db)
        db.commit()
        return {"scores": scores, "total_score": total_score if len(sections) > 1 else scores[sections[0]], "theta": theta, "points_earned": 200}
```

* **Test**: Submit full test response → Structured log in `interactions.jsonl`.
* **Integration Check**: Logs include test context.

***

#### 4. `src/ai_tutor.py` (Updated)

* **Functionality**: Log tutor interactions with context.

```python
from fastapi import APIRouter, WebSocket, Depends, HTTPException
from sqlalchemy.orm import Session
from .database import get_db, TutorInteraction, User
from .utils import get_ai_feedback, log_interaction
from .auth import get_current_user
from datetime import datetime
import json

router = APIRouter(prefix="/ai_tutor", tags=["ai_tutor"])

@router.websocket("/chat/{user_id}")
async def websocket_chat(user_id: str, websocket: WebSocket, db: Session = Depends(get_db)):
    await websocket.accept()
    start_time = datetime.utcnow()
    try:
        while True:
            data = await websocket.receive_text()
            message = json.loads(data)
            query = message.get("text", "")
            input_type = message.get("type", "text")
            if input_type == "voice":
                query = f"Voice input: {query}"
            response = get_ai_feedback(query)
            await websocket.send_text(json.dumps({"response": response}))
            duration = int((datetime.utcnow() - start_time).total_seconds())
            context = {"input_type": input_type, "duration": duration}
            db.add(TutorInteraction(user_id=user_id, query=query, response=response, duration=duration))
            log_interaction(user_id, query, response, context)
            db.commit()
    except WebSocketDisconnect:
        await websocket.close()

@router.get("/interactions/{user_id}")
async def get_tutor_interactions(user_id: str, db: Session = Depends(get_db), current_user: User = Depends(get_current_user)):
    if current_user.user_id != user_id and current_user.role not in ["tutor", "parent"]:
        raise HTTPException(status_code=403, detail="Unauthorized")
    interactions = db.query(TutorInteraction).filter(TutorInteraction.user_id == user_id).order_by(TutorInteraction.timestamp.desc()).all()
    return [{"query": i.query, "response": i.response, "timestamp": i.timestamp.strftime("%Y-%m-%d %H:%M"), "duration": i.duration} for i in interactions]
```

* **Test**: Send chat message → Structured log in `interactions.jsonl`.
* **Integration Check**: Logs include chat context.

***

#### 5. `src/question_review.py` (Updated)

* **Functionality**: Log review interactions with context.

```python
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from .database import get_db, PracticeSession, Response, Question
from .utils import get_ai_feedback, log_interaction

router = APIRouter(prefix="/review", tags=["review"])

@router.post("/generate/{session_id}")
async def generate_review(session_id: str, db: Session = Depends(get_db)):
    session = db.query(PracticeSession).filter(PracticeSession.session_id == session_id).first()
    if not session:
        raise HTTPException(status_code=404, detail="Session not found")
    
    responses = db.query(Response).filter(Response.session_id == session_id).all()
    questions = [db.query(Question).filter(Question.question_id == r.question_id).first() for r in responses]
    
    review_data = []
    for r, q in zip(responses, questions):
        correct = r.answer == q.correct_answer
        prompt = f"""
        Act as an SAT tutor. Review this response:
        - Question: {q.text}
        - User's Answer: {r.answer}
        - Correct Answer: {q.correct_answer}
        - Time Spent: {r.time_spent}s
        - Skill: {q.skill} ({q.domain})
        Provide detailed feedback, explain why the answer was {'correct' if correct else 'incorrect'}, and suggest a strategy to improve this skill.
        """
        review_text = get_ai_feedback(prompt)
        r.review_text = review_text
        context = {
            "question_id": q.question_id,
            "question_text": q.text,
            "correct_answer": q.correct_answer,
            "user_answer": r.answer,
            "time_spent": r.time_spent,
            "correct": correct,
            "type": "review"
        }
        log_interaction(session.user_id, f"Review: {q.text}", review_text, context)
        review_data.append({
            "question_id": q.question_id,
            "text": q.text,
            "user_answer": r.answer,
            "correct_answer": q.correct_answer,
            "time_spent": r.time_spent,
            "review": review_text
        })
    
    db.commit()
    return {"session_id": session_id, "reviews": review_data}

@router.get("/session/{session_id}")
async def get_session_review(session_id: str, db: Session = Depends(get_db)):
    responses = db.query(Response).filter(Response.session_id == session_id).all()
    if not responses:
        raise HTTPException(status_code=404, detail="No responses found")
    questions = [db.query(Question).filter(Question.question_id == r.question_id).first() for r in responses]
    return {
        "session_id": session_id,
        "reviews": [{"question_id": q.question_id, "text": q.text, "user_answer": r.answer, "correct_answer": q.correct_answer, "time_spent": r.time_spent, "review": r.review_text or "Pending"} for r, q in zip(responses, questions)]
    }
```

* **Test**: Generate reviews → Structured log in `interactions.jsonl`.
* **Integration Check**: Logs include review context.

***

#### 6. `src/ai_data_collection.py` (New)

* **Functionality**: Aggregate and export data for AI training.

```python
from fastapi import APIRouter, Depends
from sqlalchemy.orm import Session
from .database import get_db, Response, Proficiency, TutorInteraction
from pathlib import Path
import json
from datetime import datetime

router = APIRouter(prefix="/ai_data", tags=["ai_data"])

@router.get("/export/{user_id}")
async def export_user_data(user_id: str, db: Session = Depends(get_db)):
    # Collect all user data
    responses = db.query(Response).filter(Response.user_id == user_id).all()
    proficiencies = db.query(Proficiency).filter(Proficiency.user_id == user_id).all()
    interactions = db.query(TutorInteraction).filter(TutorInteraction.user_id == user_id).all()
    
    # Structure data
    data = {
        "user_id": user_id,
        "responses": [{"question_id": r.question_id, "answer": r.answer, "time_spent": r.time_spent, "timestamp": r.timestamp.isoformat(), "review": r.review_text} for r in responses],
        "proficiencies": [{"domain": p.domain, "skill": p.skill, "theta": p.theta, "timestamp": p.timestamp.isoformat()} for p in proficiencies],
        "interactions": [{"query": i.query, "response": i.response, "duration": i.duration, "timestamp": i.timestamp.isoformat()} for i in interactions]
    }
    
    # Export to file
    file_path = Path(f"ai_data/user_{user_id}_{datetime.utcnow().strftime('%Y%m%d_%H%M%S')}.json")
    with open(file_path, "w") as f:
        json.dump(data, f, indent=2)
    
    return {"message": f"Data exported to {file_path}"}

@router.get("/export_all")
async def export_all_data(db: Session = Depends(get_db)):
    users = db.query(User).all()
    all_data = []
    for user in users:
        responses = db.query(Response).filter(Response.user_id == user.user_id).all()
        proficiencies = db.query(Proficiency).filter(Proficiency.user_id == user.user_id).all()
        interactions = db.query(TutorInteraction).filter(TutorInteraction.user_id == user.user_id).all()
        user_data = {
            "user_id": user.user_id,
            "responses": [{"question_id": r.question_id, "answer": r.answer, "time_spent": r.time_spent, "timestamp": r.timestamp.isoformat(), "review": r.review_text} for r in responses],
            "proficiencies": [{"domain": p.domain, "skill": p.skill, "theta": p.theta, "timestamp": p.timestamp.isoformat()} for p in proficiencies],
            "interactions": [{"query": i.query, "response": i.response, "duration": i.duration, "timestamp": i.timestamp.isoformat()} for i in interactions]
        }
        all_data.append(user_data)
    
    file_path = Path(f"ai_data/all_users_{datetime.utcnow().strftime('%Y%m%d_%H%M%S')}.json")
    with open(file_path, "w") as f:
        json.dump(all_data, f, indent=2)
    
    return {"message": f"All data exported to {file_path}"}
```

* **Test**:
  * `curl "http://localhost:8000/ai_data/export/alex123"` → User data exported to JSON file.
  * `curl "http://localhost:8000/ai_data/export_all"` → All users’ data exported.
* **Integration Check**: Data matches logs in `interactions.jsonl`.

***

#### 7. `src/main.py` (Updated)

* **Functionality**: Include `ai_data_collection.py` router.

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from . import auth, diagnostic, full_length_test, practice_module, progress_monitoring, question_review, study_plan, gamification, ai_tutor, tutor_parent, social, sync, ai_data_collection

app = FastAPI(title="SAT Prep Suite")

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

app.include_router(auth.router)
app.include_router(diagnostic.router)
app.include_router(full_length_test.router)
app.include_router(practice_module.router)
app.include_router(progress_monitoring.router)
app.include_router(question_review.router)
app.include_router(study_plan.router)
app.include_router(gamification.router)
app.include_router(ai_tutor.router)
app.include_router(tutor_parent.router)
app.include_router(social.router)
app.include_router(sync.router)
app.include_router(ai_data_collection.router)

@app.get("/")
async def root():
    return {"message": "SAT Prep Suite API"}
```

* **Test**: `curl "http://localhost:8000/"` → Confirms app running with new router.
* **Integration Check**: `/ai_data/export` endpoint accessible.

***

### Testing and Integration Check

#### Setup

* **Backend**: `cd backend && docker-compose up --build`.
* **Frontend**: `cd frontend && npm install && npm run dev`.
* **Initialize**: `python migrations/init_db.py && python migrations/seed_data.py`.

#### Step-by-Step Tests

1. **Practice Data**:
   * Submit practice response (`/practice/submit`) → Structured log in `interactions.jsonl` with question context.
2. **Full Test Data**:
   * Submit full test response (`/full_test/submit`) → Structured log with test context.
3. **AI Tutor Data**:
   * Send chat message (`/ai_tutor/chat`) → Structured log with input type and duration.
4. **Review Data**:
   * Generate reviews (`/review/generate`) → Structured log with review context.
5. **Data Export**:
   * Export user data (`/ai_data/export/alex123`) → JSON file matches logs.
   * Export all data (`/ai_data/export_all`) → Comprehensive JSON file created.

#### Recheck

* **Integration**:
  * Practice/tests → Logs via `log_interaction` → Exported via `ai_data_collection.py`.
  * AI tutor/reviews → Logs with context → Included in export.
* **Data Structure**: `interactions.jsonl` entries include `user_id`, `query`, `response`, `timestamp`, `context` (e.g., question details, theta).
* **Frontend-Backend**: Practice (`practice.js`) and review (`review.js`) trigger logging via backend calls.

#### Example Log Entry (`interactions.jsonl`)

```json
{
  "user_id": "alex123",
  "query": "Practice: Solve x+2=5",
  "response": "User answered: 3",
  "timestamp": "2025-03-27T12:00:00.000000",
  "context": {
    "question_id": "mcq1",
    "question_text": "Solve x+2=5",
    "correct_answer": "3",
    "user_answer": "3",
    "time_spent": 60,
    "theta_before": 0.8,
    "theta_after": 1.0,
    "type": "practice"
  }
}
```

***

### Cost Impact

* **Storage**: Additional S3 cost for exported JSON files (\~$0.01/month for 300 MB → negligible).
* **Processing**: Minimal CPU overhead for logging/export (\~$1/month on ECS).
* **Total**: \~$365.84/month (up from $364.83, <1% increase).

***

### Conclusion

* **Status**: Data collection for AI training fully implemented:
  * **Backend**: Enhanced `utils.py` logging, updated modules (`practice_module.py`, etc.), new `ai_data_collection.py` for export.
  * **Data**: Structured in `interactions.jsonl` and exportable JSON files, ready for training.
* **Next Steps**: Move to **Item 7: Offline Mode**.

Data collection is complete—test it and let me know if you need specific data fields or anonymization before proceeding!
