# Adaptive practices based on current level

To enhance the **SAT Prep Suite** by tracking **current proficiency in all domains and skill levels over time** and making **all tests and practices adaptive** using these proficiency levels, I’ll extend the existing IRT 3PL-based adaptive testing framework. This will involve:

1. Tracking proficiency ((\theta)) for each domain (e.g., Math, Reading) and skill (e.g., Algebra, Evidence-Based Analysis) over time.
2. Making both practice sessions and full tests fully adaptive, using current proficiency estimates to select questions.
3. Updating the backend and frontend to store, retrieve, and display these proficiency trends.

This builds on the current adaptive practice implementation, aligning it with the digital SAT’s multi-stage adaptive design while providing detailed longitudinal analytics as of March 26, 2025. Below, I’ll detail the enhancements, update the code, and provide a testing plan.

***

### Tracking Proficiency and Full Adaptivity

#### 1. Track Proficiency Over Time

* **Goal**: Maintain a historical record of (\theta) for each domain and skill, updated after every response.
* **Approach**: Store proficiency in a new table, calculate running averages, and use these to initialize adaptive sessions.

**Backend**

* Update `models.py`:

```python
class Proficiency(Base):
    __tablename__ = "proficiencies"
    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(String, ForeignKey("users.user_id"))
    domain = Column(String)  # e.g., Math, Reading
    skill = Column(String)  # e.g., Algebra, Evidence-Based Analysis
    theta = Column(Float)   # Current ability estimate
    timestamp = Column(DateTime, default=func.now())
```

* Update `practice_module.py`:

```python
def update_proficiency(db, user_id, question, correct):
    theta = db.query(Proficiency).filter(
        Proficiency.user_id == user_id,
        Proficiency.domain == question.domain,
        Proficiency.skill == question.skill
    ).order_by(Proficiency.timestamp.desc()).first()
    theta = theta.theta if theta else 0.0  # Default to 0 if no prior
    a, b, c = question.a_param, question.b_param, question.c_param
    new_theta = update_theta(theta, a, b, c, correct)
    
    proficiency = Proficiency(user_id=user_id, domain=question.domain, skill=question.skill, theta=new_theta)
    db.add(proficiency)
    db.commit()
    return new_theta

def get_proficiency(db, user_id, domain=None, skill=None):
    query = db.query(Proficiency).filter(Proficiency.user_id == user_id)
    if domain:
        query = query.filter(Proficiency.domain == domain)
    if skill:
        query = query.filter(Proficiency.skill == skill)
    return query.order_by(Proficiency.timestamp.desc()).first()

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
        # Update proficiency per skill
        update_proficiency(db, session.user_id, q, r.answer == q.correct_answer)
    
    # Next question based on aggregate theta
    theta = session.theta
    for r, q in zip(responses, questions):
        correct = r.answer == q.correct_answer
        theta = update_theta(theta, q.a_param, q.b_param, q.c_param, correct)
    
    next_question = select_next_question(db, session.user_id, theta, practice_id)
    session.theta = theta
    db.commit()
    
    feedback = get_ai_feedback(responses, questions)
    if next_question and len(db.query(Response).filter(Response.session_id == practice_id).all()) < 10:
        return {
            "feedback": feedback,
            "questions": [{"id": next_question.question_id, "text": next_question.text}],
            "theta": theta
        }
    else:
        sat_score = int(400 + (theta + 3) / 6 * 1200)
        return {"feedback": feedback, "final_score": sat_score, "theta": theta}
```

* Update `routes/test.py` (make tests adaptive):

```python
@router.post("/start/{user_id}")
async def start_test(user_id: str, test_type: str = "full", db: Session = Depends(get_db)):
    session_id = str(uuid.uuid4())
    # Initialize theta per domain
    domains = ["Math", "Reading", "Writing"]
    theta = {d: get_proficiency(db, user_id, domain=d).theta if get_proficiency(db, user_id, domain=d) else 0.0 for d in domains}
    questions = []
    for domain in domains:
        q = select_next_question(db, user_id, theta[domain], session_id)
        if q:
            questions.append(q)
    session = PracticeSession(session_id=session_id, user_id=user_id, theta=sum(theta.values()) / len(theta))
    db.add(session)
    db.commit()
    return {"test_id": session_id, "questions": [{"id": q.question_id, "text": q.text, "domain": q.domain} for q in questions], "theta": theta}

@router.post("/submit/{test_id}")
async def submit_test(test_id: str, responses: List[ResponseCreate], db: Session = Depends(get_db)):
    session = db.query(PracticeSession).filter(PracticeSession.session_id == test_id).first()
    theta = {}
    questions = []
    for r in responses:
        q = db.query(Question).filter(Question.question_id == r.question_id).first()
        questions.append(q)
        db_response = Response(
            session_id=test_id,
            user_id=session.user_id,
            question_id=r.question_id,
            answer=r.answer,
            time_spent=r.time_spent,
            timestamp=datetime.utcnow()
        )
        db.add(db_response)
        theta[q.domain] = update_proficiency(db, session.user_id, q, r.answer == q.correct_answer)
    
    next_questions = []
    for domain in set(q.domain for q in questions):
        next_q = select_next_question(db, session.user_id, theta[domain], test_id)
        if next_q and len(db.query(Response).filter(Response.session_id == test_id, Response.question_id.in_([q.question_id for q in questions if q.domain == domain])).all()) < 5:  # 5 per domain
            next_questions.append(next_q)
    
    session.theta = sum(theta.values()) / len(theta)
    db.commit()
    
    if next_questions:
        return {"questions": [{"id": q.question_id, "text": q.text, "domain": q.domain} for q in next_questions], "theta": theta}
    else:
        sat_score = int(400 + (session.theta + 3) / 6 * 1200)
        return {"score": sat_score, "feedback": get_ai_feedback(responses, questions)}
```

* Add proficiency history endpoint in `routes/review.py`:

```python
@router.get("/proficiency_history/{user_id}")
async def get_proficiency_history(user_id: str, db: Session = Depends(get_db)):
    proficiencies = db.query(Proficiency).filter(Proficiency.user_id == user_id).all()
    history = {}
    for p in proficiencies:
        domain_skill = f"{p.domain}: {p.skill}"
        if domain_skill not in history:
            history[domain_skill] = []
        history[domain_skill].append({"theta": p.theta, "timestamp": p.timestamp.strftime("%Y-%m-%d")})
    return history
```

***

#### 2. Make Tests and Practices Adaptive

* **Goal**: Use current proficiency ((\theta)) per domain/skill to start adaptive sessions.
* **Approach**: Initialize (\theta) from historical data, adjust dynamically.

**Web (`web/src/`)**

* Update `practice.js`:

```javascript
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
```

* Update `test.js`:

```javascript
const [session, setSession] = useState(null);
const [theta, setTheta] = useState({});
const [finalScore, setFinalScore] = useState(null);

useEffect(() => {
  api.getUserId().then((userId) => {
    api.startTest(userId, 'full').then((data) => {
      setSession(data);
      setTheta(data.theta);
    });
  });
}, []);

const submitTest = async () => {
  const responses = session.questions.map((q) => ({ question_id: q.id, answer: 'A', time_spent: 30 }));
  const res = await api.submitTest(session.test_id, responses);
  if (res.questions) {
    setSession((prev) => ({ ...prev, questions: res.questions }));
    setTheta(res.theta);
  } else {
    setFinalScore(res.score);
    setSession(null);
  }
};
// ...
<Typography>Current Ability (θ): {Object.entries(theta).map(([d, t]) => `${d}: ${t.toFixed(2)}`).join(', ')}</Typography>
{finalScore && <Typography variant="h6">Final SAT Score: {finalScore}</Typography>}
```

**Mobile (`mobile/src/`)**

* Update `Practice.js`:

```javascript
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
```

***

#### 3. Display Proficiency Trends

* **Goal**: Show (\theta) history on the analytics dashboard.

**Web**

* Update `api.js`:

```javascript
async getProficiencyHistory(userId) {
  return (await api.get(`/review/proficiency_history/${userId}`)).data;
},
```

* Update `dashboard/analytics.js`:

```javascript
const [profHistory, setProfHistory] = useState({});
useEffect(() => {
  api.getAnalytics(userId).then(setAnalytics);
  api.getDrills(userId).then(setDrills);
  api.getProficiencyHistory(userId).then(setProfHistory);
}, [userId]);

const profTrendData = Object.entries(profHistory).map(([domainSkill, history]) => ({
  label: domainSkill,
  data: history.map(h => h.theta),
  borderColor: `rgba(${Math.random() * 255}, ${Math.random() * 255}, ${Math.random() * 255}, 1)`,
  fill: false,
}));
// ...
<Box sx={{ mt: 2 }}>
  <Typography variant="h6">Proficiency Trends Over Time</Typography>
  <Line data={{ labels: profHistory[Object.keys(profHistory)[0]]?.map(h => h.timestamp) || [], datasets: profTrendData }} options={{ responsive: true }} />
</Box>
```

***

### Updated Files

* **Backend**: `models.py`, `practice_module.py`, `test.py`, `review.py`.
* **Web**: `practice.js`, `test.js`, `dashboard/analytics.js`, `api.js`.
* **Mobile**: `Practice.js`.

### Testing Plan

#### Setup

* Update DB: `python migrations/init_db.py` (add `proficiencies` table).
* Seed data: `python seed_data.py` (ensure diverse domains/skills).
* Start backend: `cd api && uvicorn main:app --reload`.
* Web: `cd web && npm run dev`.
* Mobile: `cd mobile && npx react-native run-android`.

#### Test Cases

1. **Proficiency Tracking**:
   * Submit Algebra question → (\theta) updates for Math: Algebra in `proficiencies`.
   * Check history → Multiple entries for same skill over time.
2. **Adaptive Practice**:
   * Start practice with prior (\theta = 1.0) (Algebra) → Next question (b \approx 1.0).
   * Incorrect answer → (\theta) drops, next (b) easier.
3. **Adaptive Test**:
   * Start test → Questions from Math, Reading, Writing with domain-specific (\theta).
   * Submit → Next questions adapt per domain, final score reflects aggregate.
4. **Trends Display**:
   * Dashboard → Line chart shows (\theta) history per domain:skill.

#### Expected Results

* Proficiency updates per response, stored historically.
* Practice/tests start with current (\theta), adapt dynamically.
* Trends visualize (\theta) changes over time.

***

### Notes

* **Precision**: Simplified (\theta) updates; use `catsim` for full IRT accuracy.
* **Offline**: Adaptivity online-only; extend caching for offline (\theta).
* **Scalability**: Add indexing to `proficiencies` for large datasets.

### Conclusion

* **Tracking**: (\theta) per domain/skill tracked over time.
* **Adaptivity**: All tests/practices use current proficiency for CAT.
* **Impact**: Personalized, SAT-aligned experience.

Test it—let me know if you want offline adaptivity or deeper analytics!
