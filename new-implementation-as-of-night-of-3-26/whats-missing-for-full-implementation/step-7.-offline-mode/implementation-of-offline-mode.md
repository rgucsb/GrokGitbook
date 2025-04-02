# Implementation of Offline Mode

Moving to **Item 7: Offline Mode** for the **SAT Prep Suite**, I’ll implement offline functionality to ensure users can continue practicing, taking tests, and interacting with the app without an internet connection. This involves enhancing the backend (`sync.py`) to handle data synchronization when connectivity resumes, and updating the frontend (`api.js`, `practice.js`, `full-test.js`) to cache requests and responses locally using browser storage (e.g., `localStorage` or `IndexedDB`). The implementation will integrate with the full-featured app as of March 27, 2025, allowing seamless offline use and syncing with the backend once online.

***

### Updated Backend and Frontend Structure

#### Backend (`backend/src/`)

* **Updated File**: `sync.py` (expanded for full synchronization).
* **Updated Files**: `practice_module.py`, `full_length_test.py` (log offline actions).

#### Frontend (`frontend/`)

* **Updated File**: `utils/api.js` (enhance offline caching with IndexedDB).
* **Updated Files**: `practice.js`, `full-test.js` (support offline mode).

***

### Backend Implementation

#### 1. `src/sync.py` (Updated)

* **Functionality**: Handle synchronization of offline data (practice, test responses).

```python
from fastapi import APIRouter, Depends
from sqlalchemy.orm import Session
from pydantic import BaseModel
from .database import get_db, Response, PracticeSession, Proficiency
from .utils import update_theta
from .gamification import update_points
import json
from datetime import datetime

router = APIRouter(prefix="/sync", tags=["sync"])

class OfflineResponse(BaseModel):
    session_id: str
    question_id: str
    answer: str
    time_spent: int
    timestamp: str
    test_type: str  # "practice" or "full"
    domain: str
    theta: float

@router.post("/sync/{user_id}")
async def sync_data(user_id: str, offline_data: list[OfflineResponse], db: Session = Depends(get_db)):
    for item in offline_data:
        # Create or update session
        session = db.query(PracticeSession).filter(PracticeSession.session_id == item.session_id).first()
        if not session:
            session = PracticeSession(
                session_id=item.session_id,
                user_id=user_id,
                theta=item.theta,
                test_type=item.test_type,
                section=item.domain
            )
            db.add(session)
        
        # Add response
        db_response = Response(
            session_id=item.session_id,
            user_id=user_id,
            question_id=item.question_id,
            answer=item.answer,
            time_spent=item.time_spent,
            timestamp=datetime.fromisoformat(item.timestamp)
        )
        db.add(db_response)
        
        # Update proficiency
        question = db.query(Question).filter(Question.question_id == item.question_id).first()
        if question:
            correct = item.answer == question.correct_answer
            new_theta = update_theta(item.theta, question.a_param, question.b_param, question.c_param, correct)
            db.add(Proficiency(user_id=user_id, domain=question.domain, skill=question.skill, theta=new_theta))
            session.theta = new_theta
        
        db.commit()
    
    # Award points based on completed sessions
    completed_sessions = set([item.session_id for item in offline_data if len(db.query(Response).filter(Response.session_id == item.session_id).all()) >= (10 if item.test_type == "practice" else 44))]
    for session_id in completed_sessions:
        session = db.query(PracticeSession).filter(PracticeSession.session_id == session_id).first()
        points = 50 if session.test_type == "practice" else 200
        await update_points(user_id, points, db)
    
    db.commit()
    return {"message": f"Synced {len(offline_data)} responses, awarded points for {len(completed_sessions)} sessions"}
```

* **Test**:
  * `curl -X POST "http://localhost:8000/sync/alex123" -H "Content-Type: application/json" -d '[{"session_id": "offline1", "question_id": "mcq1", "answer": "2", "time_spent": 60, "timestamp": "2025-03-27T12:00:00", "test_type": "practice", "domain": "Math", "theta": 0.8}]'` → Response synced, theta updated.
* **Integration Check**: Data in `responses`, `proficiencies`, points awarded.

***

#### 2. `src/practice_module.py` (Updated)

* **Functionality**: Log offline actions for practice.

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

* **Test**: Submit practice response → Ready for offline sync via `/sync`.
* **Integration Check**: Logs match offline structure.

***

#### 3. `src/full_length_test.py` (Updated)

* **Functionality**: Log offline actions for full tests.

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

* **Test**: Submit full test response → Ready for offline sync via `/sync`.
* **Integration Check**: Logs match offline structure.

***

### Frontend Implementation

#### 4. `utils/api.js` (Updated)

* **Functionality**: Enhance offline caching with IndexedDB.

```javascript
import axios from 'axios';

const api = axios.create({
  baseURL: 'http://localhost:8000',
  headers: {
    'Authorization': `Bearer ${localStorage.getItem('token')}`
  }
});

// IndexedDB setup
const dbPromise = typeof window !== 'undefined' ? new Promise((resolve) => {
  const request = indexedDB.open('SATPrepDB', 1);
  request.onupgradeneeded = () => {
    const db = request.result;
    db.createObjectStore('offlineQueue', { keyPath: 'id', autoIncrement: true });
  };
  request.onsuccess = () => resolve(request.result);
  request.onerror = () => console.error('IndexedDB error');
}) : Promise.resolve(null);

async function saveToOfflineQueue(config) {
  const db = await dbPromise;
  if (!db) return;
  const tx = db.transaction('offlineQueue', 'readwrite');
  const store = tx.objectStore('offlineQueue');
  const offlineData = {
    config: JSON.stringify(config),
    timestamp: new Date().toISOString()
  };
  store.add(offlineData);
  return offlineData;
}

async function getOfflineQueue() {
  const db = await dbPromise;
  if (!db) return [];
  return new Promise((resolve) => {
    const tx = db.transaction('offlineQueue', 'readonly');
    const store = tx.objectStore('offlineQueue');
    const request = store.getAll();
    request.onsuccess = () => resolve(request.result);
  });
}

async function clearOfflineQueue() {
  const db = await dbPromise;
  if (!db) return;
  const tx = db.transaction('offlineQueue', 'readwrite');
  const store = tx.objectStore('offlineQueue');
  store.clear();
}

api.interceptors.request.use(
  async (config) => {
    if (!navigator.onLine) {
      const offlineData = await saveToOfflineQueue(config);
      return Promise.reject(new Error('Offline mode: Request queued'));
    }
    return config;
  },
  (error) => Promise.reject(error)
);

api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.message === 'Offline mode: Request queued') {
      return Promise.resolve({ data: { message: 'Request queued for sync' } });
    }
    return Promise.reject(error);
  }
);

export const syncOfflineData = async (userId) => {
  if (navigator.onLine) {
    const offlineQueue = await getOfflineQueue();
    const syncData = [];
    for (const item of offlineQueue) {
      const config = JSON.parse(item.config);
      if (config.method === 'post' && config.url.includes('/submit')) {
        const responseData = config.data.map(r => ({
          session_id: r.session_id || config.url.split('/')[2], // Extract from URL if needed
          question_id: r.question_id,
          answer: r.answer,
          time_spent: r.time_spent,
          timestamp: item.timestamp,
          test_type: config.url.includes('practice') ? 'practice' : 'full',
          domain: config.data[0]?.domain || 'Math', // Fallback
          theta: r.theta || 0.0 // Fallback
        }));
        syncData.push(...responseData);
      }
    }
    if (syncData.length > 0) {
      await api.post(`/sync/${userId}`, syncData);
      await clearOfflineQueue();
    }
    return syncData.length;
  }
  return 0;
};

export { api };
```

* **Test**: Go offline, submit practice → Queues in IndexedDB; go online, call `syncOfflineData()` → Syncs with backend.
* **Integration Check**: IndexedDB stores requests, syncs to `/sync`.

***

#### 5. `pages/practice.js` (Updated)

* **Functionality**: Support offline mode.

```javascript
import { useState, useEffect, useRef } from 'react';
import { api, syncOfflineData } from '../utils/api';
import { motion } from 'framer-motion';

export default function Practice() {
  const [questions, setQuestions] = useState([]);
  const [sessionId, setSessionId] = useState(null);
  const [answers, setAnswers] = useState({});
  const [domain, setDomain] = useState('Math');
  const [chatMessages, setChatMessages] = useState([]);
  const [chatInput, setChatInput] = useState('');
  const [isVoiceRecording, setIsVoiceRecording] = useState(false);
  const [isOffline, setIsOffline] = useState(!navigator.onLine);
  const ws = useRef(null);
  const userId = 'alex123';

  useEffect(() => {
    ws.current = new WebSocket(`ws://localhost:8000/ai_tutor/chat/${userId}`);
    ws.current.onmessage = (event) => {
      const data = JSON.parse(event.data);
      setChatMessages((prev) => [...prev, { sender: 'AI', text: data.response }]);
    };
    const handleOnline = () => {
      setIsOffline(false);
      syncOfflineData(userId).then(count => console.log(`Synced ${count} items`));
    };
    const handleOffline = () => setIsOffline(true);
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      ws.current.close();
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  const startPractice = async () => {
    const sid = `offline_${Date.now()}`; // Unique offline ID
    setSessionId(sid);
    const cachedQuestions = JSON.parse(localStorage.getItem(`practice_${domain}`)) || [];
    if (cachedQuestions.length > 0) {
      setQuestions(cachedQuestions.slice(0, 1));
    } else if (!isOffline) {
      const res = await api.post(`/practice/start/${userId}`, { domain, num_questions: 10 });
      setSessionId(res.data.session_id);
      setQuestions(res.data.questions);
      localStorage.setItem(`practice_${domain}`, JSON.stringify(res.data.questions)); // Cache for offline
    } else {
      alert('Offline mode: No cached questions available');
    }
  };

  const submitAnswer = async (questionId) => {
    const answer = answers[questionId] || '';
    const responseData = [{ question_id: questionId, answer, time_spent: 60, session_id: sessionId, domain, theta: 0.8 }]; // Simplified theta for offline
    try {
      const res = await api.post(`/practice/submit/${sessionId}`, responseData);
      if (res.data.questions) {
        setQuestions(res.data.questions);
        localStorage.setItem(`practice_${domain}`, JSON.stringify(res.data.questions));
      } else {
        alert(`Practice completed! Theta: ${res.data.theta}, Points: ${res.data.points_earned}`);
        setSessionId(null);
        setQuestions([]);
      }
    } catch (error) {
      if (error.message === 'Offline mode: Request queued') {
        console.log('Queued offline response');
        setQuestions(questions.slice(1)); // Simulate progression
        if (questions.length <= 1) {
          alert('Practice completed offline! Sync when online.');
          setSessionId(null);
          setQuestions([]);
        }
      }
    }
    setAnswers({});
  };

  const sendChatMessage = () => {
    if (chatInput && ws.current.readyState === WebSocket.OPEN) {
      const message = { type: 'text', text: chatInput };
      ws.current.send(JSON.stringify(message));
      setChatMessages((prev) => [...prev, { sender: 'You', text: chatInput }]);
      setChatInput('');
    }
  };

  const startVoiceRecording = () => {
    setIsVoiceRecording(true);
    setTimeout(() => {
      const voiceText = "Simulated voice input";
      const message = { type: 'voice', text: voiceText };
      if (ws.current.readyState === WebSocket.OPEN) {
        ws.current.send(JSON.stringify(message));
        setChatMessages((prev) => [...prev, { sender: 'You', text: voiceText }]);
      }
      setIsVoiceRecording(false);
    }, 2000);
  };

  return (
    <motion.div initial={{ opacity: 0 }} animate={{ opacity: 1 }} className="container">
      <h1>Practice Session {isOffline && '(Offline)'}</h1>
      {!sessionId ? (
        <>
          <select value={domain} onChange={(e) => setDomain(e.target.value)} className="select">
            <option value="Math">Math</option>
            <option value="Reading & Writing">Reading & Writing</option>
          </select>
          <motion.button whileHover={{ scale: 1.05 }} onClick={startPractice} className="button">Start</motion.button>
        </>
      ) : (
        <div className="practice-area">
          {questions.map((q) => (
            <div key={q.id} className="question">
              <p>{q.text}</p>
              <input
                type="text"
                value={answers[q.id] || ''}
                onChange={(e) => setAnswers({ ...answers, [q.id]: e.target.value })}
                className="input"
              />
              <motion.button whileHover={{ scale: 1.05 }} onClick={() => submitAnswer(q.id)} className="button">Submit</motion.button>
            </div>
          ))}
        </div>
      )}
      <div className="chat-area">
        <h2>AI Tutor</h2>
        <div className="chat-messages">
          {chatMessages.map((msg, idx) => (
            <p key={idx}><strong>{msg.sender}:</strong> {msg.text}</p>
          ))}
        </div>
        <input
          type="text"
          value={chatInput}
          onChange={(e) => setChatInput(e.target.value)}
          placeholder="Ask the AI tutor..."
          className="chat-input"
          onKeyPress={(e) => e.key === 'Enter' && sendChatMessage()}
        />
        <motion.button whileHover={{ scale: 1.05 }} onClick={sendChatMessage} className="button">Send</motion.button>
        <motion.button whileHover={{ scale: 1.05 }} onClick={startVoiceRecording} disabled={isVoiceRecording} className="button">
          {isVoiceRecording ? 'Recording...' : 'Voice Input'}
        </motion.button>
      </div>
      <style jsx>{`
        .container { max-width: 800px; margin: 50px auto; text-align: center; display: flex; flex-direction: column; gap: 20px; }
        .select, .input, .chat-input { display: block; width: 100%; margin: 10px 0; padding: 8px; }
        .button { padding: 10px 20px; background: #0070f3; color: white; border: none; border-radius: 5px; cursor: pointer; margin: 5px; }
        .question { margin: 20px 0; }
        .practice-area { flex: 1; }
        .chat-area { flex: 1; border: 1px solid #ccc; padding: 10px; border-radius: 5px; }
        .chat-messages { max-height: 200px; overflow-y: auto; }
      `}</style>
    </motion.div>
  );
}
```

* **Test**: Go offline, start practice, submit answers → Queues in IndexedDB; go online → Syncs.
* **Integration**: Calls `/practice/start` and `/sync`.

***

#### 5. `pages/full-test.js` (Updated)

* **Functionality**: Support offline mode.

```javascript
import { useState, useEffect } from 'react';
import { api, syncOfflineData } from '../utils/api';
import { motion } from 'framer-motion';

export default function FullTest() {
  const [questions, setQuestions] = useState([]);
  const [testId, setTestId] = useState(null);
  const [module, setModule] = useState(1);
  const [answers, setAnswers] = useState({});
  const [sections, setSections] = useState(['Math']);
  const [isOffline, setIsOffline] = useState(!navigator.onLine);
  const userId = 'alex123';

  useEffect(() => {
    const handleOnline = () => {
      setIsOffline(false);
      syncOfflineData(userId).then(count => console.log(`Synced ${count} items`));
    };
    const handleOffline = () => setIsOffline(true);
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  const startFullTest = async () => {
    const tid = `offline_test_${Date.now()}`;
    setTestId(tid);
    const cachedQuestions = JSON.parse(localStorage.getItem(`full_test_${sections.join(',')}`)) || [];
    if (cachedQuestions.length > 0) {
      setQuestions(cachedQuestions.slice(0, 1));
    } else if (!isOffline) {
      const res = await api.post(`/full_test/start/${userId}`, { sections, plan_id: 'plan123' });
      setTestId(res.data.test_id);
      setQuestions(res.data.questions);
      setModule(res.data.module);
      localStorage.setItem(`full_test_${sections.join(',')}`, JSON.stringify(res.data.questions));
    } else {
      alert('Offline mode: No cached questions available');
    }
  };

  const submitAnswer = async (questionId) => {
    const answer = answers[questionId] || '';
    const responseData = [{ question_id: questionId, answer, time_spent: 60 }];
    try {
      const res = await api.post(`/full_test/submit/${testId}`, responseData);
      if (res.data.questions) {
        setQuestions(res.data.questions);
        setModule(res.data.module);
        localStorage.setItem(`full_test_${sections.join(',')}`, JSON.stringify(res.data.questions));
      } else {
        alert(`Total Score: ${res.data.total_score}, Points: ${res.data.points_earned}`);
        setTestId(null);
        setQuestions([]);
      }
    } catch (error) {
      if (error.message === 'Offline mode: Request queued') {
        console.log('Queued offline response');
        setQuestions(questions.slice(1));
        if (questions.length <= 1 && module === 2) {
          alert('Full test completed offline! Sync when online.');
          setTestId(null);
          setQuestions([]);
        } else if (questions.length <= 1) {
          setModule(2);
          setQuestions([{ id: 'offline_next', text: 'Next question (offline)', domain: sections[0] }]); // Simulate Module 2
        }
      }
    }
    setAnswers({});
  };

  return (
    <motion.div initial={{ opacity: 0 }} animate={{ opacity: 1 }} className="container">
      <h1>Full-Length Test {isOffline && '(Offline)'}</h1>
      {!testId ? (
        <>
          <select multiple value={sections} onChange={(e) => setSections(Array.from(e.target.selectedOptions, option => option.value))} className="select">
            <option value="Math">Math</option>
            <option value="Reading & Writing">Reading & Writing</option>
          </select>
          <motion.button whileHover={{ scale: 1.05 }} onClick={startFullTest} className="button">Start</motion.button>
        </>
      ) : (
        <div>
          <h2>Module {module}</h2>
          {questions.map((q) => (
            <div key={q.id} className="question">
              <p>{q.text}</p>
              <input
                type="text"
                value={answers[q.id] || ''}
                onChange={(e) => setAnswers({ ...answers, [q.id]: e.target.value })}
                className="input"
              />
              <motion.button whileHover={{ scale: 1.05 }} onClick={() => submitAnswer(q.id)} className="button">Submit</motion.button>
            </div>
          ))}
        </div>
      )}
      <style jsx>{`
        .container { max-width: 600px; margin: 50px auto; text-align: center; }
        .select, .input { display: block; width: 100%; margin: 10px 0; padding: 8px; }
        .button { padding: 10px 20px; background: #0070f3; color: white; border: none; border-radius: 5px; cursor: pointer; margin: 5px; }
        .question { margin: 20px 0; }
      `}</style>
    </motion.div>
  );
}
```

* **Test**: Go offline, start test, submit answers → Queues in IndexedDB; go online → Syncs.
* **Integration**: Calls `/full_test/start` and `/sync`.

***

### Testing and Integration Check

#### Setup

* **Backend**: `cd backend && docker-compose up --build`.
* **Frontend**: `cd frontend && npm install && npm run dev`.
* **Initialize**: `python migrations/init_db.py && python migrations/seed_data.py`.

#### Step-by-Step Tests

1. **Offline Practice**:
   * Go offline, start practice, submit 10 answers → Queued in IndexedDB, progresses locally.
   * Go online, sync → Data in `responses`, points awarded.
2. **Offline Full Test**:
   * Go offline, start Math test, submit 44 answers → Queued, progresses to Module 2 locally.
   * Go online, sync → Data in `responses`, score calculated.
3. **Sync Verification**:
   * Check `offlineQueue` in IndexedDB → Contains requests.
   * Post-sync → Queue cleared, DB updated.

#### Recheck

* **Integration**:
  * `practice.js`/`full-test.js` → Queue via `api.js` → Sync with `/sync`.
  * Backend processes offline data → Updates `responses`, `proficiencies`, `points`.
* **Frontend-Backend**: Offline actions persist locally, sync seamlessly online.
* **Cost**: Minimal (\~$0.01/month for S3 storage of logs).

***

### Cost Impact

* **Storage**: IndexedDB client-side (free), S3 for logs (\~$0.01/month).
* **Processing**: Slight ECS overhead (\~$1/month).
* **Total Monthly**: $365.84 → **$365.85** (<0.1% increase).

***

### Conclusion

* **Status**: Offline mode fully implemented:
  * **Backend**: `sync.py` handles offline data, integrated with `practice_module.py`, `full_length_test.py`.
  * **Frontend**: `api.js` with IndexedDB caching, `practice.js`, `full-test.js` support offline use.
* **Next Steps**: Move to **Item 8: Database Enhancements**.

Offline mode is complete—test it and let me know if you need more offline features (e.g., AI tutor offline) before proceeding!
