---
hidden: true
---

# Code Package

Below, I’ll provide a complete **code package** for the **SAT Prep Suite** based on all prior discussions. This includes the backend (FastAPI with SQLite), mobile app (Flutter for iOS/Android), and web app (Flutter Web), structured to support the hybrid approach—mobile for practice/lessons and web for diagnostics/full tests. I’ll organize it as a directory structure with key files fully coded, ready for your development team to implement, test, and deploy as of March 26, 2025. Due to space constraints, I’ll include essential files with functional code and placeholders for complex logic (e.g., AI integrations), which you can expand.

***

## SAT Prep Suite: Code Package

### Directory Structure

```
sat_prep_suite/
├── api/                    # Backend (FastAPI)
│   ├── main.py
│   ├── models.py
│   ├── utils.py
│   ├── database.py
│   ├── requirements.txt
│   ├── migrations/
│   │   └── add_tutor_tables.py
│   ├── routes/
│   │   ├── __init__.py
│   │   ├── practice_module.py
│   │   ├── study_plan.py
│   │   ├── review.py
│   │   ├── community.py
│   │   ├── tutor.py
│   │   └── test.py
│   └── tests/
│       └── test_endpoints.py
├── mobile/                 # Flutter Mobile + Web App
│   ├── lib/
│   │   ├── main.dart
│   │   ├── screens/
│   │   │   ├── dashboard.dart
│   │   │   ├── practice.dart
│   │   │   ├── study_plan.dart
│   │   │   ├── community.dart
│   │   │   ├── tutor_dashboard.dart
│   │   │   ├── test.dart
│   │   │   └── results.dart
│   │   └── services/
│   │       └── api_service.dart
│   ├── pubspec.yaml
│   └── test/
│       └── widget_test.dart
├── real_sat_data.jsonl     # Data for LLM training
├── README.md
└── seed_data.py
```

***

### Backend (FastAPI)

#### `api/main.py`

```python
from fastapi import FastAPI
from api.routes import practice_module, study_plan, review, community, tutor, test

app = FastAPI(title="SAT Prep Suite API")

app.include_router(practice_module.router)
app.include_router(study_plan.router)
app.include_router(review.router)
app.include_router(community.router)
app.include_router(tutor.router)
app.include_router(test.router)

@app.get("/")
def read_root():
    return {"message": "Welcome to SAT Prep Suite API"}
```

#### `api/database.py`

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

DATABASE_URL = "sqlite:///sat_prep.db"
engine = create_engine(DATABASE_URL, connect_args={"check_same_thread": False})
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

#### `api/models.py`

```python
from sqlalchemy import Column, Integer, String, JSON, DateTime, ForeignKey, Boolean
from sqlalchemy.ext.declarative import declarative_base
from datetime import datetime

Base = declarative_base()

class User(Base):
    __tablename__ = "users"
    user_id = Column(String, primary_key=True)
    name = Column(String, nullable=False)
    email = Column(String, nullable=False)
    tutor_id = Column(String, ForeignKey("tutors.tutor_id"), nullable=True)

class Tutor(Base):
    __tablename__ = "tutors"
    tutor_id = Column(String, primary_key=True)
    name = Column(String, nullable=False)
    email = Column(String, nullable=False)

class Question(Base):
    __tablename__ = "questions"
    question_id = Column(String, primary_key=True)
    test = Column(String)  # e.g., "diagnostic", "full"
    domain = Column(String)
    skill = Column(String)
    content = Column(JSON)

class Response(Base):
    __tablename__ = "responses"
    id = Column(Integer, primary_key=True, autoincrement=True)
    user_id = Column(String, ForeignKey("users.user_id"))
    question_id = Column(String, ForeignKey("questions.question_id"))
    answer = Column(String)
    is_correct = Column(Boolean)
    time_spent = Column(Integer)

class FeedbackRating(Base):
    __tablename__ = "feedback_ratings"
    id = Column(Integer, primary_key=True, autoincrement=True)
    user_id = Column(String, ForeignKey("users.user_id"))
    feedback_text = Column(String)
    rating = Column(Integer)
    performance_data = Column(JSON)
    timestamp = Column(DateTime, default=datetime.utcnow)

class HelpRequest(Base):
    __tablename__ = "help_requests"
    id = Column(Integer, primary_key=True, autoincrement=True)
    user_id = Column(String, ForeignKey("users.user_id"))
    question_id = Column(String, ForeignKey("questions.question_id"))
    user_query = Column(String)
    ai_response = Column(String)
    timestamp = Column(DateTime, default=datetime.utcnow)

class StudyPlan(Base):
    __tablename__ = "study_plans"
    plan_id = Column(Integer, primary_key=True, autoincrement=True)
    user_id = Column(String, ForeignKey("users.user_id"))
    plan = Column(JSON)
    test_date = Column(DateTime)

class StudyPlanAction(Base):
    __tablename__ = "study_plan_actions"
    id = Column(Integer, primary_key=True, autoincrement=True)
    user_id = Column(String, ForeignKey("users.user_id"))
    plan_id = Column(Integer, ForeignKey("study_plans.plan_id"))
    task = Column(JSON)
    action = Column(String)
    performance_update = Column(JSON)
    timestamp = Column(DateTime, default=datetime.utcnow)

class UserNote(Base):
    __tablename__ = "user_notes"
    id = Column(Integer, primary_key=True, autoincrement=True)
    user_id = Column(String, ForeignKey("users.user_id"))
    question_id = Column(String, ForeignKey("questions.question_id"))
    note_text = Column(String)
    ai_response = Column(String)
    timestamp = Column(DateTime, default=datetime.utcnow)

class TutorFeedback(Base):
    __tablename__ = "tutor_feedback"
    id = Column(Integer, primary_key=True, autoincrement=True)
    tutor_id = Column(String, ForeignKey("tutors.tutor_id"))
    user_id = Column(String, ForeignKey("users.user_id"))
    question_id = Column(String, ForeignKey("questions.question_id"))
    plan_id = Column(Integer, ForeignKey("study_plans.plan_id"))
    performance_data = Column(JSON)
    feedback_text = Column(String)
    context_type = Column(String)
    timestamp = Column(DateTime, default=datetime.utcnow)

class Test(Base):
    __tablename__ = "tests"
    test_id = Column(String, primary_key=True)
    user_id = Column(String, ForeignKey("users.user_id"))
    test_type = Column(String)
    responses = Column(JSON)
    score = Column(Integer)
    completed_at = Column(DateTime, default=datetime.utcnow)

class Result(Base):
    __tablename__ = "results"
    result_id = Column(Integer, primary_key=True, autoincrement=True)
    user_id = Column(String, ForeignKey("users.user_id"))
    proficiencies = Column(JSON)
    completed_at = Column(DateTime, default=datetime.utcnow)
```

#### `api/utils.py`

```python
from sqlalchemy.orm import Session
from typing import Dict, List
import json
import redis

redis_client = redis.Redis(host='localhost', port=6379, db=0)

def log_tutor_feedback(db: Session, tutor_id: str, user_id: str, feedback_text: str, context_type: str, question_id: str = None, plan_id: int = None, performance_data: Dict = None):
    feedback = TutorFeedback(tutor_id=tutor_id, user_id=user_id, question_id=question_id, plan_id=plan_id, performance_data=performance_data, feedback_text=feedback_text, context_type=context_type)
    db.add(feedback)
    db.commit()

def generate_feedback_with_custom_llm(input_data: Dict, context: Dict) -> str:
    # Placeholder: Replace with actual LLM integration
    return "Great job! Keep practicing."  # Initially use GPT-4o via API

def get_student_progress(db: Session, tutor_id: str) -> Dict:
    students = db.query(User).filter(User.tutor_id == tutor_id).all()
    progress = [{"user_id": s.user_id, "name": s.name, "proficiencies": db.query(Result).filter(Result.user_id == s.user_id).order_by(Result.completed_at.desc()).first().proficiencies if db.query(Result).filter(Result.user_id == s.user_id).count() > 0 else {}} for s in students]
    return {"tutor_id": tutor_id, "students": progress}

def export_llm_training_data(db: Session, output_file: str):
    with open(output_file, "a") as f:
        for fr in db.query(FeedbackRating).all():
            f.write(json.dumps({"input": fr.performance_data, "output": fr.feedback_text}) + "\n")
        for tf in db.query(TutorFeedback).all():
            if tf.performance_data:
                f.write(json.dumps({"input": tf.performance_data, "output": tf.feedback_text}) + "\n")
        for t in db.query(Test).all():
            f.write(json.dumps({"input": t.responses, "output": f"Score: {t.score}"}) + "\n")
```

#### `api/routes/practice_module.py`

```python
from fastapi import APIRouter, Depends
from sqlalchemy.orm import Session
from api.database import get_db
from api.models import Response, FeedbackRating
from api.utils import generate_feedback_with_custom_llm
from typing import List, Dict

router = APIRouter(prefix="/practice", tags=["practice"])

@router.post("/start/{user_id}")
async def start_practice(user_id: str, db: Session = Depends(get_db)):
    questions = db.query(Question).filter(Question.test == "practice").limit(10).all()
    return {"user_id": user_id, "practice_id": f"prac_{user_id}_{time.time()}", "questions": [q.content for q in questions]}

@router.post("/submit/{practice_id}")
async def submit_practice(practice_id: str, responses: List[Dict], db: Session = Depends(get_db)):
    feedback = []
    for r in responses:
        resp = Response(user_id=practice_id.split("_")[1], question_id=r["question_id"], answer=r["answer"], is_correct=r.get("is_correct", False), time_spent=r.get("time_spent", 0))
        db.add(resp)
        fb = generate_feedback_with_custom_llm({"question_id": r["question_id"], "answer": r["answer"]}, {})
        feedback.append({"question_id": r["question_id"], "feedback": fb})
        db.add(FeedbackRating(user_id=resp.user_id, feedback_text=fb, rating=0, performance_data={"question_id": r["question_id"], "is_correct": r["is_correct"]}))
    db.commit()
    return {"practice_id": practice_id, "feedback": feedback}
```

#### `api/routes/test.py`

```python
from fastapi import APIRouter, Depends
from sqlalchemy.orm import Session
from api.database import get_db
from api.models import Test, Result
from api.utils import generate_feedback_with_custom_llm
from typing import List, Dict
import time

router = APIRouter(prefix="/test", tags=["test"])

@router.post("/start/{user_id}")
async def start_test(user_id: str, test_type: str, db: Session = Depends(get_db)):
    questions = db.query(Question).filter(Question.test == test_type).all()  # e.g., 98 for "full"
    return {"user_id": user_id, "test_id": f"test_{user_id}_{time.time()}", "questions": [q.content for q in questions]}

@router.post("/submit/{test_id}")
async def submit_test(test_id: str, responses: List[Dict], db: Session = Depends(get_db)):
    score = sum(1 for r in responses if r.get("is_correct", False))  # Simple scoring
    test = Test(test_id=test_id, user_id=test_id.split("_")[1], test_type="full", responses=responses, score=score)
    feedback = generate_feedback_with_custom_llm({"responses": responses}, {"score": score})
    result = Result(user_id=test.user_id, proficiencies={"total_score": score}, completed_at=datetime.utcnow())
    db.add_all([test, result])
    db.commit()
    return {"test_id": test_id, "score": score, "feedback": feedback}
```

#### `api/requirements.txt`

```
fastapi==0.95.0
uvicorn==0.21.1
sqlalchemy==1.4.47
redis==4.5.4
requests==2.28.2
boto3==1.26.100  # For AWS Polly
pytest==7.2.2
```

#### `api/migrations/add_tutor_tables.py`

```python
from sqlalchemy import create_engine, MetaData, Table, Column, Integer, String, JSON, DateTime, ForeignKey, Boolean
from api.models import Base

engine = create_engine("sqlite:///sat_prep.db")
Base.metadata.create_all(engine)
```



***



### Frontend (Flutter Mobile + Web)

#### `mobile/pubspec.yaml`

```yaml
name: sat_prep_suite
description: SAT Prep Suite App
version: 1.0.0+1

environment:
  sdk: ">=2.17.0 <3.0.0"

dependencies:
  flutter:
    sdk: flutter
  http: ^0.13.5
  sqflite: ^2.0.0+4
  path_provider: ^2.0.11
  firebase_core: ^1.24.0
  firebase_messaging: ^13.0.4
  charts_flutter: ^0.12.0

dev_dependencies:
  flutter_test:
    sdk: flutter

flutter:
  uses-material-design: true
```

#### `mobile/lib/main.dart`

```dart
import 'package:flutter/material.dart';
import 'screens/dashboard.dart';
import 'screens/practice.dart';
import 'screens/study_plan.dart';
import 'screens/community.dart';
import 'screens/tutor_dashboard.dart';
import 'screens/test.dart';
import 'screens/results.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'SAT Prep Suite',
      initialRoute: '/dashboard',
      routes: {
        '/dashboard': (_) => DashboardScreen(),
        '/practice': (_) => PracticeScreen(),
        '/study_plan': (_) => StudyPlanScreen(),
        '/community': (_) => CommunityScreen(),
        '/tutor': (_) => TutorDashboardScreen(),
        '/test': (_) => TestScreen(),
        '/results': (_) => ResultsScreen(),
      },
    );
  }
}
```

#### `mobile/lib/services/api_service.dart`

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;

class ApiService {
  static const String baseUrl = 'http://localhost:8000'; // Update for production

  Future<Map<String, dynamic>> startPractice(String userId) async {
    final response = await http.post(Uri.parse('$baseUrl/practice/start/$userId'));
    return jsonDecode(response.body);
  }

  Future<Map<String, dynamic>> submitPractice(String practiceId, List<Map<String, dynamic>> responses) async {
    final response = await http.post(Uri.parse('$baseUrl/practice/submit/$practiceId'), body: jsonEncode(responses));
    return jsonDecode(response.body);
  }

  Future<Map<String, dynamic>> startTest(String userId, String testType) async {
    final response = await http.post(Uri.parse('$baseUrl/test/start/$userId?test_type=$testType'));
    return jsonDecode(response.body);
  }

  Future<Map<String, dynamic>> submitTest(String testId, List<Map<String, dynamic>> responses) async {
    final response = await http.post(Uri.parse('$baseUrl/test/submit/$testId'), body: jsonEncode(responses));
    return jsonDecode(response.body);
  }

  Future<Map<String, dynamic>> getTutorStudentProgress(String tutorId) async {
    final response = await http.get(Uri.parse('$baseUrl/tutor/progress/$tutorId'));
    return jsonDecode(response.body);
  }

  Future<void> submitTutorFeedback(String tutorId, String userId, String feedbackText, String contextType, {Map<String, dynamic>? performanceData}) async {
    await http.post(Uri.parse('$baseUrl/tutor/feedback'), body: jsonEncode({
      'tutor_id': tutorId, 'user_id': userId, 'feedback_text': feedbackText, 'context_type': contextType, 'performance_data': performanceData
    }));
  }
}
```

#### `mobile/lib/screens/dashboard.dart`

```dart
import 'package:flutter/material.dart';
import '../services/api_service.dart';

class DashboardScreen extends StatelessWidget {
  final ApiService api = ApiService();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('SAT Prep')),
      body: ListView(
        children: [
          Text('Progress: Placeholder'), // Add real progress later
          ElevatedButton(onPressed: () => Navigator.pushNamed(context, '/practice'), child: Text('Practice')),
          ElevatedButton(onPressed: () => Navigator.pushNamed(context, '/study_plan'), child: Text('Study Plan')),
          ElevatedButton(onPressed: () => Navigator.pushNamed(context, '/community'), child: Text('Community')),
          ElevatedButton(onPressed: () => Navigator.pushNamed(context, '/test'), child: Text('Take Test')),
          ElevatedButton(onPressed: () => Navigator.pushNamed(context, '/tutor'), child: Text('Tutor Dashboard')),
        ],
      ),
    );
  }
}
```

#### `mobile/lib/screens/practice.dart`

```dart
import 'package:flutter/material.dart';
import '../services/api_service.dart';

class PracticeScreen extends StatefulWidget {
  @override
  _PracticeScreenState createState() => _PracticeScreenState();
}

class _PracticeScreenState extends State<PracticeScreen> {
  final ApiService api = ApiService();
  Map<String, dynamic>? session;

  @override
  void initState() {
    super.initState();
    api.startPractice('user123').then((data) => setState(() => session = data));
  }

  @override
  Widget build(BuildContext context) {
    if (session == null) return Center(child: CircularProgressIndicator());
    return Scaffold(
      appBar: AppBar(title: Text('Practice')),
      body: ListView.builder(
        itemCount: session!['questions'].length,
        itemBuilder: (context, index) => ListTile(
          title: Text(session!['questions'][index]['text']),
          subtitle: ElevatedButton(
            onPressed: () => api.submitPractice(session!['practice_id'], [{'question_id': session!['questions'][index]['id'], 'answer': 'A'}]).then((res) => print(res)),
            child: Text('Submit Answer'),
          ),
        ),
      ),
    );
  }
}
```

#### `mobile/lib/screens/test.dart`

```dart
import 'package:flutter/material.dart';
import '../services/api_service.dart';

class TestScreen extends StatefulWidget {
  @override
  _TestScreenState createState() => _TestScreenState();
}

class _TestScreenState extends State<TestScreen> {
  final ApiService api = ApiService();
  Map<String, dynamic>? testSession;

  @override
  void initState() {
    super.initState();
    api.startTest('user123', 'full').then((data) => setState(() => testSession = data));
  }

  @override
  Widget build(BuildContext context) {
    if (testSession == null) return Center(child: CircularProgressIndicator());
    return Scaffold(
      appBar: AppBar(title: Text('Full SAT Test')),
      body: ListView.builder(
        itemCount: testSession!['questions'].length,
        itemBuilder: (context, index) => ListTile(
          title: Text(testSession!['questions'][index]['text']),
          subtitle: ElevatedButton(
            onPressed: () => api.submitTest(testSession!['test_id'], [{'question_id': testSession!['questions'][index]['id'], 'answer': 'A'}]).then((res) => Navigator.pushNamed(context, '/results')),
            child: Text('Submit Test'),
          ),
        ),
      ),
    );
  }
}
```

#### `mobile/lib/screens/results.dart`

```dart
import 'package:flutter/material.dart';

class ResultsScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Test Results')),
      body: Center(child: Text('Score: Placeholder')), // Add real data later
    );
  }
}
```

#### `mobile/lib/screens/tutor_dashboard.dart`

```dart
import 'package:flutter/material.dart';
import '../services/api_service.dart';

class TutorDashboardScreen extends StatefulWidget {
  @override
  _TutorDashboardScreenState createState() => _TutorDashboardScreenState();
}

class _TutorDashboardScreenState extends State<TutorDashboardScreen> {
  final ApiService api = ApiService();
  final TextEditingController _feedbackController = TextEditingController();
  String _contextType = 'practice';
  Map<String, dynamic>? progress;

  @override
  void initState() {
    super.initState();
    api.getTutorStudentProgress('tutor123').then((data) => setState(() => progress = data));
  }

  @override
  Widget build(BuildContext context) {
    if (progress == null) return Center(child: CircularProgressIndicator());
    return Scaffold(
      appBar: AppBar(title: Text('Tutor Dashboard')),
      body: ListView.builder(
        itemCount: progress!['students'].length,
        itemBuilder: (context, index) {
          var student = progress!['students'][index];
          return ExpansionTile(
            title: Text(student['name']),
            children: [
              TextField(controller: _feedbackController, decoration: InputDecoration(labelText: 'Feedback')),
              DropdownButton<String>(
                value: _contextType,
                items: ['practice', 'question', 'plan', 'general'].map((e) => DropdownMenuItem(value: e, child: Text(e))).toList(),
                onChanged: (value) => setState(() => _contextType = value!),
              ),
              ElevatedButton(
                onPressed: () => api.submitTutorFeedback('tutor123', student['user_id'], _feedbackController.text, _contextType, performanceData: student['proficiencies']),
                child: Text('Submit'),
              ),
            ],
          );
        },
      ),
    );
  }
}
```

***

### Additional Files

#### `seed_data.py`

```python
from sqlalchemy.orm import Session
from api.models import User, Tutor, Question
from api.database import get_db

def seed(db: Session):
    db.add_all([
        User(user_id="user123", name="Alice", email="alice@example.com", tutor_id="tutor123"),
        Tutor(tutor_id="tutor123", name="Jane Doe", email="jane@example.com"),
        Question(question_id="q1", test="practice", domain="Algebra", skill="Linear Functions", content={"text": "Solve: 2x + 3 = 7", "options": ["A: 2", "B: 3"]}),
        Question(question_id="q2", test="full", domain="Reading", skill="Comprehension", content={"text": "What is the main idea?", "options": ["A", "B"]}),
    ])
    db.commit()

db = next(get_db())
seed(db)
```

#### `README.md`

```markdown
# SAT Prep Suite

Hybrid app for SAT preparation:
- **Mobile**: Practice, lessons, community (Flutter iOS/Android).
- **Web**: Diagnostics, full tests, tutor tools (Flutter Web).
- **Backend**: FastAPI with SQLite, data collection for custom LLM.

## Setup
1. Backend: `pip install -r api/requirements.txt`, `uvicorn api.main:app --reload`
2. Mobile: `cd mobile && flutter pub get && flutter run`
3. Web: `cd mobile && flutter build web && firebase deploy`

## Testing
- Local: `pytest api/tests/`, `flutter test`
- Staging: Deploy to EC2/Firebase, test flows.
```

***

### Notes

* **Placeholders**: AI logic (`generate_feedback_with_custom_llm`), full implementations for `study_plan.py`, `review.py`, `community.py` are omitted for brevity—expand as needed.
* **Build Commands**:
  * Mobile: `flutter build apk` (Android), `flutter build ios` (iOS).
  * Web: `flutter build web`.
* **Next Steps**: Add detailed question banks, implement LLM training script, flesh out remaining screens.

This package provides a functional starting point—copy into your repo, run setup commands, and test per the previous plan. Any specific files or expansions needed? Let me know!
