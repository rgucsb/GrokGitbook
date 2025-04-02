# IMproved AI tutor

To enhance the **SAT Prep Suite** with an **Advanced AI Tutor**, I’ll implement the suggested improvements: training the custom LLM on a larger SAT dataset, adding a conversational AI tutor with real-time Q\&A, hints, and step-by-step explanations, and integrating predictive analytics for SAT score forecasts. These upgrades will build on the current system (custom LLM with GPT-4o fallback) to outperform competitors like LearnQ.ai’s “Mia” as of March 26, 2025. I’ll modify the backend to handle AI interactions and predictive analytics, update the web and mobile frontends for conversational features, and ensure seamless integration. Below, I’ll detail the enhancements, provide updated code, and outline a testing plan.

***

### Advanced AI Tutor Enhancements

#### 1. Train Custom LLM on Larger SAT Dataset

* **Goal**: Improve specificity over GPT-4o using `real_sat_data.jsonl` and public SAT resources.
* **Implementation**: Enhance backend AI logic, assuming a fine-tuned model (e.g., LLaMA) hosted locally or via AWS.

**Backend**

* Update `utils.py` (AI feedback generation):

```python
import json
from transformers import pipeline  # Hypothetical fine-tuned model
from datetime import datetime

# Simulated fine-tuned LLM (replace with actual model)
ai_model = pipeline("text-generation", model="path/to/fine-tuned-sat-llm")  # Fine-tuned on real_sat_data.jsonl

def get_ai_feedback(responses, questions):
    # Collect data for fine-tuning
    with open("real_sat_data.jsonl", "a") as f:
        for r, q in zip(responses, questions):
            f.write(json.dumps({"prompt": q.text, "response": r.answer, "correct": q.correct_answer, "timestamp": str(datetime.utcnow())}))
    # Generate specific feedback
    input_text = "\n".join([f"Q: {q.text}\nA: {r.answer}\nCorrect: {q.correct_answer}" for r, q in zip(responses, questions)])
    feedback = ai_model(f"Provide detailed feedback for SAT practice:\n{input_text}", max_length=200)[0]["generated_text"]
    return feedback.strip()

def get_ai_response(note_text):
    return ai_model(f"Respond to this SAT-related note:\n{note_text}", max_length=100)[0]["generated_text"].strip()
```

* **Training** (hypothetical, run offline):
  * Collect `real_sat_data.jsonl` (populated by usage).
  * Augment with public SAT datasets (e.g., College Board samples).
  * Fine-tune LLaMA: `transformers.Trainer` on AWS SageMaker (e.g., 10 epochs, SAT-specific prompts).
  * Deploy model to EC2 (e.g., `g4dn.xlarge`) with FastAPI endpoint.

**Notes**

* Replace `pipeline` with actual inference code post-training.
* Fallback to GPT-4o remains if model isn’t ready:

```python
import openai
if not ai_model:
    def get_ai_feedback(responses, questions):
        return openai.Completion.create(engine="gpt-4o", prompt=..., max_tokens=200).choices[0].text
```

***

#### 2. Conversational AI Tutor

* **Goal**: Add a “Mia-like” tutor for real-time Q\&A, hints, and explanations.
* **Implementation**: WebSocket endpoint for live chat, integrated into practice and mobile screens.

**Backend**

* Add `routes/ai_tutor.py`:

```python
from fastapi import APIRouter, WebSocket, Depends
from sqlalchemy.orm import Session
from ..database import get_db
from ..utils import get_ai_response as ai_respond

router = APIRouter(prefix="/ai_tutor", tags=["ai_tutor"])

@router.websocket("/chat/{user_id}")
async def chat_websocket(websocket: WebSocket, user_id: str, db: Session = Depends(get_db)):
    await websocket.accept()
    while True:
        message = await websocket.receive_text()
        response = ai_respond(f"User: {message}\nContext: SAT practice question assistance")
        await websocket.send_text(response)
```

* Register in `main.py`:

```python
from routes import ai_tutor
app.include_router(ai_tutor.router)
```

**Web (`web/src/`)**

* Update `api.js`:

```javascript
connectChat(userId) {
  const ws = new WebSocket(`ws://localhost:8000/ai_tutor/chat/${userId}`);
  return ws;
},
```

* Update `practice.js`:

```javascript
import { useRef } from 'react';
import { TextField, Button, List, ListItem, Typography, Box } from '@mui/material';
// ...
const [chatMessages, setChatMessages] = useState([]);
const [chatInput, setChatInput] = useState('');
const wsRef = useRef(null);

useEffect(() => {
  api.getUserId().then((userId) => {
    dispatch(fetchPractice(userId, mode));
    wsRef.current = api.connectChat(userId);
    wsRef.current.onmessage = (event) => {
      setChatMessages((prev) => [...prev, { sender: 'AI', text: event.data }]);
    };
    return () => wsRef.current.close();
  });
}, [dispatch, mode]);

const sendChat = () => {
  if (wsRef.current && chatInput) {
    wsRef.current.send(chatInput);
    setChatMessages((prev) => [...prev, { sender: 'You', text: chatInput }]);
    setChatInput('');
  }
};
// ...
<Box sx={{ mt: 4 }}>
  <Typography variant="h6">AI Tutor Chat</Typography>
  <List sx={{ maxHeight: 200, overflow: 'auto' }}>
    {chatMessages.map((msg, idx) => (
      <ListItem key={idx}>
        <Typography>{`${msg.sender}: ${msg.text}`}</Typography>
      </ListItem>
    ))}
  </List>
  <TextField
    label="Ask a question"
    value={chatInput}
    onChange={(e) => setChatInput(e.target.value)}
    fullWidth
    onKeyPress={(e) => e.key === 'Enter' && sendChat()}
  />
  <Button variant="contained" onClick={sendChat} sx={{ mt: 1 }}>Send</Button>
</Box>
```

**Mobile (`mobile/src/`)**

* Update `api.js`:

```javascript
connectChat(userId) {
  const ws = new WebSocket(`ws://localhost:8000/ai_tutor/chat/${userId}`);
  return ws;
},
```

* Update `Practice.js`:

```javascript
import { TextInput } from 'react-native';
// ...
const [chatMessages, setChatMessages] = useState([]);
const [chatInput, setChatInput] = useState('');
const wsRef = useRef(null);

useEffect(() => {
  api.getUserId().then((userId) => {
    api.startPractice(userId, mode).then(setSession);
    api.getPoints(userId).then(setPoints);
    api.getBadges(userId).then(setBadges);
    wsRef.current = api.connectChat(userId);
    wsRef.current.onmessage = (event) => {
      setChatMessages((prev) => [...prev, { sender: 'AI', text: event.data }]);
    };
    return () => wsRef.current.close();
  }).catch(() => navigation.navigate('Login'));
}, [navigation, mode]);

const sendChat = () => {
  if (wsRef.current && chatInput) {
    wsRef.current.send(chatInput);
    setChatMessages((prev) => [...prev, { sender: 'You', text: chatInput }]);
    setChatInput('');
  }
};
// ...
<FlatList
  data={chatMessages}
  keyExtractor={(_, idx) => idx.toString()}
  renderItem={({ item }) => (
    <Text style={{ fontSize: 16 }}>{`${item.sender}: ${item.text}`}</Text>
  )}
  style={{ maxHeight: 200 }}
/>
<TextInput
  placeholder="Ask the AI Tutor"
  value={chatInput}
  onChangeText={setChatInput}
  style={{ borderBottomWidth: 1, marginVertical: 10 }}
  onSubmitEditing={sendChat}
/>
<Button title="Send" onPress={sendChat} />
```

***

#### 3. Predictive Analytics

* **Goal**: Forecast SAT scores based on practice trends.
* **Implementation**: Add endpoint, display on dashboard.

**Backend**

* Add to `review.py`:

```python
@router.get("/predict_score/{user_id}")
async def predict_score(user_id: str, db: Session = Depends(get_db)):
    results = db.query(Result).filter(Result.user_id == user_id).all()
    if not results:
        return {"predicted_score": 400, "message": "Insufficient data"}
    avg_proficiency = sum(sum(p.values()) for p in [r.proficiencies for r in results]) / (len(results) * 7)
    predicted_score = int(400 + (avg_proficiency * 1200))  # Scale 400-1600
    test_date = (datetime.utcnow() + timedelta(days=90)).strftime("%Y-%m-%d")  # Hypothetical 3 months
    return {"predicted_score": predicted_score, "message": f"At this rate, you’ll hit {predicted_score} by {test_date}"}
```

**Web**

* Update `api.js`:

```javascript
async predictScore(userId) {
  return (await api.get(`/review/predict_score/${userId}`)).data;
},
```

* Update `dashboard.js`:

```javascript
const [prediction, setPrediction] = useState(null);
useEffect(() => {
  api.getUserId().then((userId) => {
    dispatch(fetchSkills(userId));
    dispatch(fetchPoints(userId));
    api.getBadges(userId).then(setBadges);
    api.getLeaderboard().then(setLeaderboard);
    api.predictScore(userId).then(setPrediction);
  });
}, [dispatch]);
// ...
{prediction && (
  <Typography variant="h6">
    Predicted Score: {prediction.predicted_score} - {prediction.message}
  </Typography>
)}
```

**Mobile**

* Not implemented (dashboard not in mobile scope).

***

### Updated Files

* **Backend**: `utils.py`, `routes/ai_tutor.py`, `review.py`, `main.py`, `models.py`.
* **Web**: `api.js`, `practice.js`, `dashboard.js`.
* **Mobile**: `api.js`, `Practice.js`.

### Testing Plan

#### Setup

* Update DB schema: `python migrations/init_db.py` (no changes needed for this feature).
* Start backend: `cd api && uvicorn main:app --reload`.
* Web: `cd web && npm run dev`.
* Mobile: `cd mobile && npx react-native run-android`.

#### Test Cases

1. **AI Feedback**:
   * Submit practice (web/mobile) → Feedback is specific (e.g., “Your algebra answer was off due to misapplying the quadratic formula”).
2. **Conversational Tutor**:
   * Web/Mobile: Ask “How do I solve x^2 + 5x + 6 = 0?” → AI responds with step-by-step explanation.
   * Test hints: “Give me a hint for this” → AI provides a nudge.
3. **Predictive Analytics**:
   * Web: Submit multiple practices → Dashboard shows “At this rate, you’ll hit 1400 by June 2025”.

#### Expected Results

* AI responses are SAT-specific, not generic.
* Chat persists in real-time, no disconnects.
* Prediction updates with practice data.

***

### Conclusion

* **Custom LLM**: Enhanced specificity (assuming fine-tuning).
* **Conversational Tutor**: Real-time Q\&A/hints added to practice.
* **Predictive Analytics**: Score forecasts on web dashboard.

These upgrades make the AI tutor a standout feature. Test it out—let me know if you want more AI capabilities (e.g., voice input) or tweaks!
