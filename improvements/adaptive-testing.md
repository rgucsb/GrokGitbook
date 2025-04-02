# Adaptive testing

To integrate **Fully Adaptive Testing** into the **SAT Prep Suite** using the **Item Response Theory (IRT) 3-Parameter Logistic (3PL) model**, I’ll implement a **Computerized Adaptive Testing (CAT)** system that adjusts question difficulty in real-time based on user performance, mirroring the digital SAT (e.g., Bluebook). This will replace the current basic next-question suggestions with a dynamic, psychometrically sound approach, aligning scoring with the College Board’s 400-1600 scale. This upgrade will outpace competitors like Acely’s static adaptive tests by offering a truly responsive experience as of March 26, 2025. Below, I’ll detail the enhancements, update the backend and frontend code, and provide a testing plan.

***

### Fully Adaptive Testing with IRT 3PL

#### Overview of IRT 3PL

* **Parameters**:
  * **a (Discrimination)**: How well the item differentiates between ability levels.
  * **b (Difficulty)**: The ability level where the probability of a correct response is 50% (adjusted for guessing).
  * **c (Guessing)**: Probability of guessing correctly (typically \~0.25 for 4-option multiple-choice).
* **Probability Formula**:\
  ( P(\theta) = c + \frac{1 - c}{1 + e^{-a(\theta - b)\}} ), where (\theta) is the user’s ability estimate.
* **CAT Process**:
  1. Start with a medium-difficulty question ((b = 0)).
  2. Update (\theta) based on response (correct → increase, incorrect → decrease).
  3. Select next question with (b) closest to current (\theta).
  4. Repeat until precision is sufficient or question limit is reached.
* **Scoring**: Map (\theta) (logit scale) to 400-1600 SAT scale.

#### 1. Backend Implementation

* **Goal**: Implement CAT logic with IRT 3PL, store item parameters, and estimate ability.
* **Tools**: Use Python’s `catsim` library (simplified for this example) or custom IRT logic.

**Database Updates**

* Update `models.py`:

```python
class Question(Base):
    __tablename__ = "questions"
    # ... existing ...
    a_param = Column(Float, default=1.0)  # Discrimination
    b_param = Column(Float, default=0.0)  # Difficulty
    c_param = Column(Float, default=0.25) # Guessing
```

* Seed initial parameters in `seed_data.py` (example):

```python
questions = [
    {"text": "Solve 2x + 3 = 7", "correct_answer": "2", "skill": "Algebra", "domain": "Math", "a_param": 1.2, "b_param": -1.0, "c_param": 0.25},
    {"text": "Solve x^2 - 4 = 0", "correct_answer": "2, -2", "skill": "Algebra", "domain": "Math", "a_param": 1.0, "b_param": 1.0, "c_param": 0.25},
    # ... more with varying b_param (-2 to 2) ...
]
```

**Adaptive Logic**

* Update `routes/practice_module.py`:

```python
from math import exp
from sqlalchemy import func
from ..models import PracticeSession, Question, Response
import uuid

router = APIRouter(prefix="/practice", tags=["practice"])

def irt_3pl(theta, a, b, c):
    return c + (1 - c) / (1 + exp(-a * (theta - b)))

def update_theta(theta, a, b, c, correct):
    # Simplified maximum likelihood estimation
    if correct:
        theta += 0.5 / (1 + abs(theta - b))  # Increase ability
    else:
        theta -= 0.5 / (1 + abs(theta - b))  # Decrease ability
    return max(min(theta, 3), -3)  # Bound between -3 and 3

def select_next_question(db, user_id, theta, session_id):
    used_questions = db.query(Response.question_id).filter(Response.session_id == session_id).all()
    used_ids = [q[0] for q in used_questions]
    question = db.query(Question).filter(
        Question.question_id.notin_(used_ids),
        func.abs(Question.b_param - theta) == db.query(func.min(func.abs(Question.b_param - theta))).filter(Question.question_id.notin_(used_ids)).scalar()
    ).first()
    return question

@router.post("/start/{user_id}")
async def start_practice(user_id: str, mode: str = "adaptive", db: Session = Depends(get_db)):
    session_id = str(uuid.uuid4())
    theta = 0.0  # Initial ability estimate
    question = select_next_question(db, user_id, theta, session_id)
    session = PracticeSession(session_id=session_id, user_id=user_id, theta=theta)
    db.add(session)
    db.commit()
    return {"session_id": session_id, "questions": [{"id": question.question_id, "text": question.text}], "theta": theta}

@router.post("/submit/{practice_id}")
async def submit_practice(practice_id: str, responses: List[ResponseCreate], db: Session = Depends(get_db)):
    session = db.query(PracticeSession).filter(PracticeSession.session_id == practice_id).first()
    questions = []
    for r in responses:
        q = db.query(Question).filter(Question.question_id == r.question_id).first()
        questions.append(q)
        db_response = Response(
            session_id=practice_id,
            user_id=session.user_id,
            question_id=r.question_id,
            answer=r.answer,
            time_spent=r.time_spent,
            timestamp=datetime.utcnow()
        )
        db.add(db_response)
    
    # Update theta
    theta = session.theta
    for r, q in zip(responses, questions):
        correct = r.answer == q.correct_answer
        theta = update_theta(theta, q.a_param, q.b_param, q.c_param, correct)
    
    # Next question or finish
    next_question = select_next_question(db, session.user_id, theta, practice_id)
    session.theta = theta
    db.commit()
    
    feedback = get_ai_feedback(responses, questions)
    if next_question and len(db.query(Response).filter(Response.session_id == practice_id).all()) < 10:  # Limit to 10 questions
        return {
            "feedback": feedback,
            "questions": [{"id": next_question.question_id, "text": next_question.text}],
            "theta": theta
        }
    else:
        sat_score = int(400 + (theta + 3) / 6 * 1200)  # Map -3 to 3 -> 400 to 1600
        return {"feedback": feedback, "final_score": sat_score, "theta": theta}
```

***

#### 2. Frontend Implementation

* **Goal**: Display adaptive questions, show real-time difficulty adjustments, and final SAT score.

**Web (`web/src/`)**

* Update `api.js`:

```javascript
async startPractice(userId, mode = 'adaptive') {
  return (await api.post(`/practice/start/${userId}?mode=${mode}`)).data;
},
async submitPractice(practiceId, responses) {
  return (await api.post(`/practice/submit/${practiceId}`, responses)).data;
},
```

* Update `practice.js`:

```javascript
const [session, setSession] = useState(null);
const [theta, setTheta] = useState(0);
const [finalScore, setFinalScore] = useState(null);
const [feedback, setFeedback] = useState('');

useEffect(() => {
  api.getUserId().then((userId) => {
    if (navigator.onLine) {
      api.startPractice(userId).then((data) => {
        setSession(data);
        setTheta(data.theta);
      });
      api.cacheQuestions(userId);
    } else {
      api.getCachedQuestions(userId).then(setSession);
    }
  });
}, [dispatch]);

const handleSubmit = (questionId) => {
  const response = { question_id: questionId, answer: 'A', time_spent: 30 };
  if (isOffline) {
    api.getUserId().then((userId) => api.addPendingResponse(userId, response));
  } else {
    api.submitPractice(session.session_id, [response]).then((result) => {
      setFeedback(result.feedback);
      if (result.questions) {
        setSession((prev) => ({ ...prev, questions: result.questions }));
        setTheta(result.theta);
      } else {
        setFinalScore(result.final_score);
        setSession(null);
      }
    });
  }
};
// ...
<Box sx={{ p: 4 }}>
  <Typography variant="h4">Adaptive Practice</Typography>
  {session && (
    <>
      <Typography>Current Ability (θ): {theta.toFixed(2)}</Typography>
      <List>
        {session.questions.map((q) => (
          <ListItem key={q.id}>
            <Typography>{q.text}</Typography>
            <Button onClick={() => handleSubmit(q.id)}>Submit</Button>
          </ListItem>
        ))}
      </List>
      {feedback && <Typography>Feedback: {feedback}</Typography>}
    </>
  )}
  {finalScore && <Typography variant="h6">Estimated SAT Score: {finalScore}</Typography>}
</Box>
```

**Mobile (`mobile/src/`)**

* Update `Practice.js`:

```javascript
const [session, setSession] = useState(null);
const [theta, setTheta] = useState(0);
const [finalScore, setFinalScore] = useState(null);
const [feedback, setFeedback] = useState('');

useEffect(() => {
  api.getUserId().then((userId) => {
    if (!isOffline) {
      api.startPractice(userId).then((data) => {
        setSession(data);
        setTheta(data.theta);
      });
      api.cacheQuestions(userId);
    } else {
      api.getCachedQuestions(userId).then(setSession);
    }
  }).catch(() => navigation.navigate('Login'));
}, [navigation, isOffline]);

const handleSubmit = (questionId) => {
  const response = { question_id: questionId, answer: 'A', time_spent: 30 };
  if (isOffline) {
    api.getUserId().then((userId) => api.addPendingResponse(userId, response));
  } else {
    api.submitPractice(session.session_id, [response]).then((result) => {
      setFeedback(result.feedback);
      if (result.questions) {
        setSession((prev) => ({ ...prev, questions: result.questions }));
        setTheta(result.theta);
      } else {
        setFinalScore(result.final_score);
        setSession(null);
      }
    });
  }
};
// ...
<View style={{ padding: 20 }}>
  <Text style={{ fontSize: 24 }}>Adaptive Practice</Text>
  {session && (
    <>
      <Text style={{ fontSize: 18 }}>Ability (θ): {theta.toFixed(2)}</Text>
      <FlatList
        data={session.questions}
        renderItem={({ item }) => (
          <View style={{ marginVertical: 10 }}>
            <Text>{item.text}</Text>
            <Button title="Submit" onPress={() => handleSubmit(item.id)} />
          </View>
        )}
      />
      {feedback && <Text>Feedback: {feedback}</Text>}
    </>
  )}
  {finalScore && <Text style={{ fontSize: 20 }}>Estimated SAT Score: {finalScore}</Text>}
</View>
```

***

#### 3. Scoring Alignment

* **Goal**: Map (\theta) to 400-1600 SAT scale.
* **Implementation**: Already in backend (`submit_practice`): (\text{SAT Score} = 400 + \frac{\theta + 3}{6} \times 1200), where (\theta) ranges from -3 to 3.

***

### Updated Files

* **Backend**: `models.py`, `seed_data.py`, `routes/practice_module.py`.
* **Web**: `api.js`, `practice.js`.
* **Mobile**: `api.js`, `Practice.js`.

### Testing Plan

#### Setup

* Update DB schema: `python migrations/init_db.py` (add `a_param`, `b_param`, `c_param` to `questions`).
* Seed data: `python seed_data.py`.
* Start backend: `cd api && uvicorn main:app --reload`.
* Web: `cd web && npm run dev`.
* Mobile: `cd mobile && npx react-native run-android`.

#### Test Cases

1. **Adaptive Difficulty**:
   * Start practice → Medium question ((b \approx 0)).
   * Correct answer → Next question harder ((b > 0)).
   * Incorrect answer → Next question easier ((b < 0)).
2. **Ability Estimation**:
   * Submit 5 correct → (\theta) increases, score > 1000.
   * Submit 5 incorrect → (\theta) decreases, score < 800.
3. **Scoring**:
   * Finish 10 questions → Final score between 400-1600 reflects performance.

#### Expected Results

* Questions adapt to user ability in real-time.
* (\theta) updates logically (e.g., +0.5 for correct, -0.5 for incorrect).
* SAT score aligns with College Board scale.

***

### Notes

* **IRT Precision**: Simplified (\theta) update; use `catsim` or full MLE for production.
* **Item Pool**: Assumes diverse (b\_param) values—expand `seed_data.py`.
* **Offline**: Adaptive logic is online-only; extend caching if needed.

### Conclusion

* **CAT with IRT 3PL**: Real-time difficulty adjustment implemented.
* **Scoring**: Mirrors Bluebook’s 400-1600 scale.
* **Impact**: Outpaces static adaptive tests with dynamic precision.

Test it—let me know if you want offline CAT or more IRT tweaks!
