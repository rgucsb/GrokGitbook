# Offline Mode

To enhance the **SAT Prep Suite** with an **Offline Mode**, I’ll implement the suggested improvements: caching questions and study plans locally for offline practice (using `localStorage` for web and `AsyncStorage` with a SQLite-like approach via `@react-native-async-storage/async-storage` for mobile), and storing AI feedback with an offline fallback (leveraging Redis online and pre-generated tips offline). This addresses the current reliance on online API calls, ensuring usability without internet connectivity as of March 26, 2025. Since the app is now React-based (web with Next.js, mobile with React Native), I’ll adapt the Flutter-specific `sqflite` suggestion to fit the React ecosystem. Below, I’ll detail the enhancements, update the code, and provide a testing plan.

***

### Offline Mode Enhancements

#### 1. Cache Questions and Study Plans Locally

* **Goal**: Enable offline practice and study plan access with sync on reconnect.
* **Implementation**: Use `localStorage` (web) and `AsyncStorage` (mobile) to cache data, sync when online.

**Backend**

* No major changes needed—API endpoints remain the same.
* Add a `sync` endpoint to handle updates:

```python
# routes/sync.py (new)
from fastapi import APIRouter, Depends
from sqlalchemy.orm import Session
from ..database import get_db
from ..models import Response
from typing import List

router = APIRouter(prefix="/sync", tags=["sync"])

@router.post("/responses/{user_id}")
async def sync_responses(user_id: str, responses: List[dict], db: Session = Depends(get_db)):
    for r in responses:
        db_response = Response(
            user_id=user_id,
            question_id=r["question_id"],
            answer=r["answer"],
            time_spent=r["time_spent"],
            timestamp=r["timestamp"]
        )
        db.add(db_response)
    db.commit()
    return {"status": "synced"}
```

* Register in `main.py`:

```python
from routes import sync
app.include_router(sync.router)
```

**Web (`web/src/`)**

* Update `api.js`:

```javascript
async cacheQuestions(userId) {
  const questions = await this.startPractice(userId);
  localStorage.setItem(`practice_${userId}`, JSON.stringify(questions));
  return questions;
},
async getCachedQuestions(userId) {
  return JSON.parse(localStorage.getItem(`practice_${userId}`)) || null;
},
async cacheStudyPlan(userId, testDate) {
  const plan = await this.createStudyPlan(userId, testDate);
  localStorage.setItem(`study_plan_${userId}`, JSON.stringify(plan));
  return plan;
},
async getCachedStudyPlan(userId) {
  return JSON.parse(localStorage.getItem(`study_plan_${userId}`)) || null;
},
async syncResponses(userId, responses) {
  return await api.post(`/sync/responses/${userId}`, responses);
},
async getPendingResponses(userId) {
  return JSON.parse(localStorage.getItem(`pending_responses_${userId}`)) || [];
},
async addPendingResponse(userId, response) {
  const pending = await this.getPendingResponses(userId);
  pending.push(response);
  localStorage.setItem(`pending_responses_${userId}`, JSON.stringify(pending));
},
```

* Update `practice.js`:

```javascript
const [isOffline, setIsOffline] = useState(false);
const [pendingResponses, setPendingResponses] = useState([]);

useEffect(() => {
  const handleOnline = () => {
    setIsOffline(false);
    api.getUserId().then((userId) => {
      api.getPendingResponses(userId).then((pending) => {
        if (pending.length) {
          api.syncResponses(userId, pending).then(() => {
            localStorage.removeItem(`pending_responses_${userId}`);
            setPendingResponses([]);
          });
        }
      });
    });
  };
  window.addEventListener('online', handleOnline);
  window.addEventListener('offline', () => setIsOffline(true));
  return () => {
    window.removeEventListener('online', handleOnline);
    window.removeEventListener('offline');
  };
}, [dispatch, mode]);

useEffect(() => {
  api.getUserId().then((userId) => {
    if (navigator.onLine) {
      dispatch(fetchPractice(userId, mode));
      api.cacheQuestions(userId);
    } else {
      api.getCachedQuestions(userId).then((cached) => {
        if (cached) setSession(cached);
      });
    }
  });
}, [dispatch, mode]);

const handleSubmit = (questionId) => {
  const response = { question_id: questionId, answer: 'A', time_spent: 30, timestamp: new Date().toISOString() };
  if (isOffline) {
    api.getUserId().then((userId) => {
      api.addPendingResponse(userId, response);
      setPendingResponses((prev) => [...prev, response]);
    });
  } else {
    dispatch(submitPractice({ practiceId: session.practice_id, responses: [response] })).then(() => {
      api.getUserId().then((userId) => dispatch(addPoints({ userId, points: 10 })));
    });
  }
};
// ...
<Typography>{isOffline ? "Offline Mode" : "Online"}</Typography>
```

**Mobile (`mobile/src/`)**

* Update `api.js`:

```javascript
async cacheQuestions(userId) {
  const questions = await this.startPractice(userId);
  await AsyncStorage.setItem(`practice_${userId}`, JSON.stringify(questions));
  return questions;
},
async getCachedQuestions(userId) {
  const cached = await AsyncStorage.getItem(`practice_${userId}`);
  return cached ? JSON.parse(cached) : null;
},
async cacheStudyPlan(userId, testDate) {
  const plan = await this.createStudyPlan(userId, testDate);
  await AsyncStorage.setItem(`study_plan_${userId}`, JSON.stringify(plan));
  return plan;
},
async getCachedStudyPlan(userId) {
  const cached = await AsyncStorage.getItem(`study_plan_${userId}`);
  return cached ? JSON.parse(cached) : null;
},
async syncResponses(userId, responses) {
  return await api.post(`/sync/responses/${userId}`, responses);
},
async getPendingResponses(userId) {
  const pending = await AsyncStorage.getItem(`pending_responses_${userId}`);
  return pending ? JSON.parse(pending) : [];
},
async addPendingResponse(userId, response) {
  const pending = await this.getPendingResponses(userId);
  pending.push(response);
  await AsyncStorage.setItem(`pending_responses_${userId}`, JSON.stringify(pending));
},
```

* Update `Practice.js`:

```javascript
import NetInfo from "@react-native-community/netinfo";
// ...
const [isOffline, setIsOffline] = useState(false);
const [pendingResponses, setPendingResponses] = useState([]);

useEffect(() => {
  const unsubscribe = NetInfo.addEventListener(state => {
    setIsOffline(!state.isConnected);
    if (state.isConnected) {
      api.getUserId().then((userId) => {
        api.getPendingResponses(userId).then((pending) => {
          if (pending.length) {
            api.syncResponses(userId, pending).then(() => {
              AsyncStorage.removeItem(`pending_responses_${userId}`);
              setPendingResponses([]);
            });
          }
        });
      });
    }
  });
  api.getUserId().then((userId) => {
    if (!isOffline) {
      api.startPractice(userId, mode).then(setSession);
      api.cacheQuestions(userId);
    } else {
      api.getCachedQuestions(userId).then(setSession);
    }
  }).catch(() => navigation.navigate('Login'));
  return () => unsubscribe();
}, [navigation, mode]);

const handleSubmit = (questionId) => {
  const response = { question_id: questionId, answer: 'A', time_spent: 30, timestamp: new Date().toISOString() };
  if (isOffline) {
    api.getUserId().then((userId) => {
      api.addPendingResponse(userId, response);
      setPendingResponses((prev) => [...prev, response]);
    });
  } else {
    api.submitPractice(session.practice_id, [response]).then(() => {
      api.getUserId().then((userId) => api.updatePoints(userId, 10).then(setPoints));
    });
  }
};
// ...
<Text style={{ fontSize: 18 }}>{isOffline ? "Offline Mode" : "Online"}</Text>
```

* Update `StudyPlan.js`:

```javascript
useEffect(() => {
  api.getUserId().then((userId) => {
    if (!isOffline) {
      api.createStudyPlan(userId, '2025-06-01').then(setPlan);
      api.cacheStudyPlan(userId, '2025-06-01');
    } else {
      api.getCachedStudyPlan(userId).then(setPlan);
    }
  }).catch(() => navigation.navigate('Login'));
}, [navigation, isOffline]);
```

***

#### 2. Store AI Feedback with Offline Fallback

* **Goal**: Cache AI feedback in Redis (online) and use pre-generated tips offline.
* **Implementation**: Modify AI tutor to cache responses, fallback to static tips.

**Backend**

* Update `utils.py`:

```python
import redis

redis_client = redis.Redis(host='localhost', port=6379, db=0)

def get_ai_feedback(responses, questions):
    feedback = ai_model(...)  # Existing logic
    user_id = responses[0].user_id
    redis_client.set(f"feedback_{user_id}", feedback)
    return feedback

def get_cached_feedback(user_id):
    return redis_client.get(f"feedback_{user_id}") or "Focus on reviewing your mistakes."
```

* Update `ai_tutor.py`:

```python
async def chat_websocket(websocket: WebSocket, user_id: str, db: Session = Depends(get_db)):
    await websocket.accept()
    if user_id not in chat_sessions:
        chat_sessions[user_id] = []
    while True:
        message = await websocket.receive_text()
        chat_sessions[user_id].append({"role": "user", "content": message})
        context = "\n".join([f"{m['role']}: {m['content']}" for m in chat_sessions[user_id][-5:]])
        prompt = f"Act as an SAT tutor...\n{context}"
        response = ai_respond(prompt)
        chat_sessions[user_id].append({"role": "assistant", "content": response})
        redis_client.set(f"chat_{user_id}", json.dumps(chat_sessions[user_id]))
        await websocket.send_text(response)
```

* Add offline fallback:

```python
def get_offline_tip():
    tips = ["Review key concepts daily.", "Practice pacing your answers.", "Focus on weak skills."]
    return random.choice(tips)
```

**Web**

* Update `api.js`:

```javascript
async getCachedChat(userId) {
  return JSON.parse(localStorage.getItem(`chat_${userId}`)) || [];
},
async cacheChat(userId, messages) {
  localStorage.setItem(`chat_${userId}`, JSON.stringify(messages));
},
```

* Update `practice.js`:

```javascript
const offlineTips = [
  "Review key concepts daily.",
  "Practice pacing your answers.",
  "Focus on weak skills."
];
useEffect(() => {
  api.getUserId().then((userId) => {
    if (navigator.onLine) {
      dispatch(fetchPractice(userId, mode));
      api.cacheQuestions(userId);
      wsRef.current = api.connectChat(userId);
      wsRef.current.onmessage = (event) => {
        setChatMessages((prev) => [...prev, { sender: 'AI', text: event.data }]);
        api.cacheChat(userId, [...chatMessages, { sender: 'AI', text: event.data }]);
      };
    } else {
      api.getCachedQuestions(userId).then((cached) => setSession(cached));
      api.getCachedChat(userId).then(setChatMessages);
    }
  });
}, [dispatch, mode]);

const sendChat = (text = chatInput) => {
  if (isOffline) {
    const tip = offlineTips[Math.floor(Math.random() * offlineTips.length)];
    setChatMessages((prev) => [...prev, { sender: 'You', text }, { sender: 'AI', text: tip }]);
    api.cacheChat(userId, [...chatMessages, { sender: 'You', text }, { sender: 'AI', text: tip }]);
  } else if (wsRef.current && text) {
    wsRef.current.send(text);
    setChatMessages((prev) => [...prev, { sender: 'You', text }]);
  }
  setChatInput('');
};
```

**Mobile**

* Update `api.js`:

```javascript
async getCachedChat(userId) {
  const cached = await AsyncStorage.getItem(`chat_${userId}`);
  return cached ? JSON.parse(cached) : [];
},
async cacheChat(userId, messages) {
  await AsyncStorage.setItem(`chat_${userId}`, JSON.stringify(messages));
},
```

* Update `Practice.js`:

```javascript
const offlineTips = [
  "Review key concepts daily.",
  "Practice pacing your answers.",
  "Focus on weak skills."
];
useEffect(() => {
  const unsubscribe = NetInfo.addEventListener(state => {
    setIsOffline(!state.isConnected);
    // ... sync logic ...
  });
  api.getUserId().then((userId) => {
    if (!isOffline) {
      api.startPractice(userId, mode).then(setSession);
      api.cacheQuestions(userId);
      wsRef.current = api.connectChat(userId);
      wsRef.current.onmessage = (event) => {
        setChatMessages((prev) => [...prev, { sender: 'AI', text: event.data }]);
        api.cacheChat(userId, [...chatMessages, { sender: 'AI', text: event.data }]);
      };
    } else {
      api.getCachedQuestions(userId).then(setSession);
      api.getCachedChat(userId).then(setChatMessages);
    }
  }).catch(() => navigation.navigate('Login'));
  return () => unsubscribe();
}, [navigation, mode]);

const sendChat = (text = chatInput) => {
  if (isOffline) {
    const tip = offlineTips[Math.floor(Math.random() * offlineTips.length)];
    setChatMessages((prev) => [...prev, { sender: 'You', text }, { sender: 'AI', text: tip }]);
    api.cacheChat(userId, [...chatMessages, { sender: 'You', text }, { sender: 'AI', text: tip }]);
  } else if (wsRef.current && text) {
    wsRef.current.send(text);
    setChatMessages((prev) => [...prev, { sender: 'You', text }]);
  }
  setChatInput('');
};
```

***

### Updated Files

* **Backend**: `routes/sync.py`, `main.py`, `utils.py`, `ai_tutor.py`.
* **Web**: `api.js`, `practice.js`.
* **Mobile**: `api.js`, `Practice.js`, `StudyPlan.js`, `package.json` (add `@react-native-community/netinfo`).

### Testing Plan

#### Setup

* Install dependencies: `cd mobile && npm install @react-native-community/netinfo`.
* Start backend: `cd api && uvicorn main:app --reload` (with Redis running).
* Web: `cd web && npm run dev`.
* Mobile: `cd mobile && npx react-native run-android`.

#### Test Cases

1. **Offline Practice**:
   * Web/Mobile: Go offline, load practice → Cached questions appear.
   * Submit answers → Pending responses stored locally.
2. **Offline Study Plan**:
   * Mobile: Go offline → Cached study plan loads.
3. **AI Feedback**:
   * Online: Chat with AI → Responses cached.
   * Offline: Ask question → Random tip appears.
4. **Sync**:
   * Go online → Pending responses sync, cache updates.

#### Expected Results

* Offline mode functional with cached data.
* AI tips display offline, sync online.
* No errors on reconnect.

***

### Notes

* **Storage Limits**: `localStorage` (5-10MB) and `AsyncStorage` may cap data; prune old caches if needed.
* **Redis**: Assumes local setup—use cloud service for production.
* **Complexity**: Simplified SQLite replacement; use `react-native-sqlite-storage` for larger datasets.

### Conclusion

* **Caching**: Questions and study plans available offline, synced on reconnect.
* **AI Feedback**: Redis online, static tips offline.

Test it—let me know if you want deeper offline features (e.g., full SQLite) or adjustments!
