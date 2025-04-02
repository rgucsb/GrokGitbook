# Tutor Data Collection Implementation

Let’s implement the features to **capture tutor feedback** in the **SAT Prep Suite**, integrating them into the existing backend and mobile app to collect high-quality data for training our small Language Learning Model (LLM). This builds on the previous plans, adding a `TUTOR_FEEDBACK` table, a `/tutor/feedback` endpoint, and a **Tutor Dashboard** screen, while ensuring compatibility with the data collection pipeline for a seamless transition from synthetic to real data as of March 26, 2025. Below is the detailed implementation.

***

#### Step 1: Backend Implementation

**Database Setup**

* **Migration**: Add `TUTORS` and `TUTOR_FEEDBACK` tables to the existing SQLite/PostgreSQL database (assuming SQLAlchemy).

```python
# api/migrations/add_tutor_tables.py
from sqlalchemy import create_engine, MetaData, Table, Column, Integer, String, JSON, DateTime, ForeignKey

engine = create_engine("sqlite:///sat_prep.db")  # Adjust for your DB
metadata = MetaData()

tutors = Table(
    "tutors",
    metadata,
    Column("tutor_id", String, primary_key=True),
    Column("name", String, nullable=False),
    Column("email", String, nullable=False),
)

tutor_feedback = Table(
    "tutor_feedback",
    metadata,
    Column("id", Integer, primary_key=True, autoincrement=True),
    Column("tutor_id", String, ForeignKey("tutors.tutor_id"), nullable=False),
    Column("user_id", String, ForeignKey("users.user_id"), nullable=False),
    Column("question_id", String, ForeignKey("questions.question_id"), nullable=True),
    Column("plan_id", Integer, ForeignKey("study_plans.plan_id"), nullable=True),
    Column("performance_data", JSON, nullable=True),
    Column("feedback_text", String, nullable=False),
    Column("context_type", String, nullable=False),
    Column("timestamp", DateTime, server_default="CURRENT_TIMESTAMP"),
)

metadata.create_all(engine)
```

**`api/models.py` (Updated)**

```python
from sqlalchemy import Column, Integer, String, JSON, DateTime, ForeignKey
from sqlalchemy.ext.declarative import declarative_base
from datetime import datetime

Base = declarative_base()

class Tutor(Base):
    __tablename__ = "tutors"
    tutor_id = Column(String, primary_key=True)
    name = Column(String, nullable=False)
    email = Column(String, nullable=False)

class TutorFeedback(Base):
    __tablename__ = "tutor_feedback"
    id = Column(Integer, primary_key=True, autoincrement=True)
    tutor_id = Column(String, ForeignKey("tutors.tutor_id"), nullable=False)
    user_id = Column(String, ForeignKey("users.user_id"), nullable=False)
    question_id = Column(String, ForeignKey("questions.question_id"), nullable=True)
    plan_id = Column(Integer, ForeignKey("study_plans.plan_id"), nullable=True)
    performance_data = Column(JSON, nullable=True)
    feedback_text = Column(String, nullable=False)
    context_type = Column(String, nullable=False)  # "practice", "question", "plan", "general"
    timestamp = Column(DateTime, default=datetime.utcnow)
```

**`api/utils.py` (Updated)**

```python
from typing import Dict
from sqlalchemy.orm import Session
import json

def log_tutor_feedback(db: Session, tutor_id: str, user_id: str, feedback_text: str, context_type: str, question_id: str = None, plan_id: int = None, performance_data: Dict = None):
    feedback = TutorFeedback(
        tutor_id=tutor_id,
        user_id=user_id,
        question_id=question_id,
        plan_id=plan_id,
        performance_data=performance_data,
        feedback_text=feedback_text,
        context_type=context_type
    )
    db.add(feedback)
    db.commit()

def get_student_progress(db: Session, tutor_id: str) -> Dict:
    """Fetch progress for students assigned to a tutor (simplified)"""
    students = db.query(User).filter(User.tutor_id == tutor_id).all()  # Assuming tutor_id in USERS
    progress = []
    for student in students:
        latest_result = db.query(Result).filter(Result.user_id == student.user_id).order_by(Result.completed_at.desc()).first()
        progress.append({
            "user_id": student.user_id,
            "name": student.name,
            "proficiencies": latest_result.proficiencies if latest_result else {}
        })
    return {"tutor_id": tutor_id, "students": progress}

def export_llm_training_data(db: Session, output_file: str):
    with open(output_file, "a") as f:
        # Existing exports (FeedbackRating, HelpRequest, etc.) remain unchanged
        
        # Tutor feedback
        for tf in db.query(TutorFeedback).all():
            if tf.context_type == "practice" and tf.performance_data:
                f.write(json.dumps({"input": tf.performance_data, "output": tf.feedback_text}) + "\n")
            elif tf.context_type == "question" and tf.question_id:
                q = db.query(Question).filter(Question.question_id == tf.question_id).first()
                input_data = {"question": q.content["text"], "options": q.content["options"]}
                f.write(json.dumps({"input": json.dumps(input_data), "output": tf.feedback_text}) + "\n")
            elif tf.context_type == "plan" and tf.plan_id:
                p = db.query(StudyPlan).filter(StudyPlan.plan_id == tf.plan_id).first()
                input_data = {"plan": p.plan, "performance": tf.performance_data}
                f.write(json.dumps({"input": json.dumps(input_data), "output": tf.feedback_text}) + "\n")
            elif tf.context_type == "general":
                f.write(json.dumps({"input": f"General observation for {tf.user_id}", "output": tf.feedback_text}) + "\n")
```

**`api/routes/tutor.py` (New Module)**

```python
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from typing import Dict, Optional
from api.models import TutorFeedback
from api.utils import log_tutor_feedback, get_student_progress
from database import get_db

router = APIRouter(prefix="/tutor", tags=["tutor"])

@router.post("/feedback")
async def submit_tutor_feedback(
    tutor_id: str,
    user_id: str,
    feedback_text: str,
    context_type: str,
    question_id: Optional[str] = None,
    plan_id: Optional[int] = None,
    performance_data: Optional[Dict] = None,
    db: Session = Depends(get_db)
):
    if context_type not in ["practice", "question", "plan", "general"]:
        raise HTTPException(status_code=400, detail="Invalid context_type")
    if not db.query(Tutor).filter(Tutor.tutor_id == tutor_id).first():
        raise HTTPException(status_code=404, detail="Tutor not found")
    if not db.query(User).filter(User.user_id == user_id).first():
        raise HTTPException(status_code=404, detail="User not found")
    
    log_tutor_feedback(db, tutor_id, user_id, feedback_text, context_type, question_id, plan_id, performance_data)
    return {"user_id": user_id, "feedback_text": feedback_text, "context_type": context_type}

@router.get("/progress/{tutor_id}")
async def get_tutor_student_progress(tutor_id: str, db: Session = Depends(get_db)):
    if not db.query(Tutor).filter(Tutor.tutor_id == tutor_id).first():
        raise HTTPException(status_code=404, detail="Tutor not found")
    progress = get_student_progress(db, tutor_id)
    return progress
```

**`api/main.py` (Updated)**

```python
from fastapi import FastAPI
from api.routes import practice_module, study_plan, review, community, tutor  # Add tutor

app = FastAPI()
app.include_router(practice_module.router)
app.include_router(study_plan.router)
app.include_router(review.router)
app.include_router(community.router)
app.include_router(tutor.router)  # Include new tutor routes
```

***

#### Step 2: Mobile App Implementation (Flutter)

**`pubspec.yaml` (Updated)**

```yaml
dependencies:
  flutter:
    sdk: flutter
  http: ^0.13.5
  sqflite: ^2.0.0+4
  path_provider: ^2.0.11
  firebase_core: ^1.24.0
  firebase_messaging: ^13.0.4
  charts_flutter: ^0.12.0
  audio_session: ^0.1.10
  record: ^4.4.0
```

**`lib/services/api_service.dart` (Updated)**

```dart
Future<void> submitTutorFeedback(
  String tutorId,
  String userId,
  String feedbackText,
  String contextType, {
  String? questionId,
  int? planId,
  Map<String, dynamic>? performanceData,
}) async {
  final response = await http.post(
    Uri.parse('$baseUrl/tutor/feedback'),
    body: jsonEncode({
      'tutor_id': tutorId,
      'user_id': userId,
      'feedback_text': feedbackText,
      'context_type': contextType,
      'question_id': questionId,
      'plan_id': planId,
      'performance_data': performanceData,
    }),
    headers: {'Content-Type': 'application/json'},
  );
  if (response.statusCode != 200) throw Exception('Failed to submit feedback');
}

Future<Map<String, dynamic>> getTutorStudentProgress(String tutorId) async {
  final response = await http.get(Uri.parse('$baseUrl/tutor/progress/$tutorId'));
  return jsonDecode(response.body);
}
```

**`lib/screens/tutor_dashboard.dart` (New)**

```dart
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;
import 'dart:convert';
import '../services/api_service.dart';

class TutorDashboardScreen extends StatefulWidget {
  @override
  _TutorDashboardScreenState createState() => _TutorDashboardScreenState();
}

class _TutorDashboardScreenState extends State<TutorDashboardScreen> {
  final ApiService api = ApiService();
  final TextEditingController _feedbackController = TextEditingController();
  String _contextType = 'practice';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Tutor Dashboard')),
      body: FutureBuilder(
        future: api.getTutorStudentProgress('tutor123'),  // Example tutor ID
        builder: (context, snapshot) {
          if (!snapshot.hasData) return Center(child: CircularProgressIndicator());
          if (snapshot.hasError) return Center(child: Text('Error: ${snapshot.error}'));
          var progress = snapshot.data! as Map<String, dynamic>;
          var students = progress['students'] as List<dynamic>;
          return ListView.builder(
            itemCount: students.length,
            itemBuilder: (context, index) {
              var student = students[index];
              return ExpansionTile(
                title: Text(student['name']),
                subtitle: Text('Progress: ${jsonEncode(student['proficiencies'])}'),
                children: [
                  Padding(
                    padding: EdgeInsets.all(8.0),
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        TextField(
                          controller: _feedbackController,
                          decoration: InputDecoration(
                            labelText: 'Add Feedback',
                            border: OutlineInputBorder(),
                          ),
                          maxLines: 3,
                        ),
                        SizedBox(height: 10),
                        DropdownButton<String>(
                          value: _contextType,
                          items: ['practice', 'question', 'plan', 'general']
                              .map((e) => DropdownMenuItem(value: e, child: Text(e)))
                              .toList(),
                          onChanged: (value) => setState(() => _contextType = value!),
                        ),
                        SizedBox(height: 10),
                        ElevatedButton(
                          onPressed: () async {
                            try {
                              await api.submitTutorFeedback(
                                'tutor123',
                                student['user_id'],
                                _feedbackController.text,
                                _contextType,
                                performanceData: student['proficiencies'],
                              );
                              ScaffoldMessenger.of(context).showSnackBar(
                                SnackBar(content: Text('Feedback submitted!')),
                              );
                              _feedbackController.clear();
                            } catch (e) {
                              ScaffoldMessenger.of(context).showSnackBar(
                                SnackBar(content: Text('Error: $e')),
                              );
                            }
                          },
                          child: Text('Submit Feedback'),
                        ),
                      ],
                    ),
                  ),
                ],
              );
            },
          );
        },
      ),
    );
  }
}
```

**`lib/main.dart` (Updated)**

```dart
import 'package:flutter/material.dart';
import 'screens/dashboard.dart';
import 'screens/practice.dart';
import 'screens/study_plan.dart';
import 'screens/community.dart';
import 'screens/tutor_dashboard.dart';

void main() {
  runApp(MaterialApp(
    initialRoute: '/dashboard',
    routes: {
      '/dashboard': (_) => DashboardScreen(),
      '/practice': (_) => PracticeScreen(),
      '/study_plan': (_) => StudyPlanScreen(),
      '/community': (_) => CommunityScreen(),
      '/tutor': (_) => TutorDashboardScreen(),  // New tutor route
    },
  ));
}
```

**`lib/screens/dashboard.dart` (Updated)**

```dart
class DashboardScreen extends StatelessWidget {
  final ApiService api = ApiService();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('SAT Prep')),
      body: ListView(
        children: [
          // Existing progress display...
          ElevatedButton(
            onPressed: () => Navigator.pushNamed(context, '/practice'),
            child: Text('Start Practice'),
          ),
          ElevatedButton(
            onPressed: () => Navigator.pushNamed(context, '/study_plan'),
            child: Text('Study Plan'),
          ),
          ElevatedButton(
            onPressed: () => Navigator.pushNamed(context, '/community'),
            child: Text('Community'),
          ),
          ElevatedButton(
            onPressed: () => Navigator.pushNamed(context, '/tutor'),  // Tutor access
            child: Text('Tutor Dashboard'),
          ),
        ],
      ),
    );
  }
}
```

***

#### Step 3: Data Pipeline Integration

**Initial Setup**

*   **Seed Data**: Add a few tutors manually:

    ```sql
    INSERT INTO TUTORS (tutor_id, name, email) VALUES
    ('tutor123', 'Jane Doe', 'jane@example.com'),
    ('tutor456', 'John Smith', 'john@example.com');
    ```
* **Assign Tutors**: Hypothetically add `tutor_id` to `USERS` table or assume a many-to-many relation (not implemented here for simplicity).

**Collection**

* **Endpoint**: `/tutor/feedback` logs feedback into `TUTOR_FEEDBACK`.
* **Volume**: 10 tutors × 10 feedbacks/day × 30 days = \~3K pairs/month initially.

**Export**

*   **Script**: Run `export_llm_training_data` weekly:

    ```bash
    python -c "from api.utils import export_llm_training_data; from database import get_db; export_llm_training_data(get_db(), 'real_sat_data.jsonl')"
    ```
*   **Sample Output**:

    ```json
    {"input": "{\"Math\": {\"Algebra\": {\"Linear Functions\": {\"correct\": 7, \"total\": 10}}}}", "output": "You’re doing well with Linear Functions (7/10), but watch your steps on multi-variable problems—nice effort!"}
    ```

**Training Integration**

* **Initial**: Combine with 50K synthetic pairs.
* **1K Users**: \~1K tutor pairs + 5K student pairs + synthetic.
* **10K Users**: \~10K tutor pairs + 37K student pairs → Retrain LLM.

***

#### Step 4: Testing

* **Setup**: Add tutor ‘tutor123’ and student ‘user123’ to DB.
* **Test**:
  1. Navigate to `/tutor` → See student progress.
  2. Submit feedback → Verify entry in `TUTOR_FEEDBACK`.
  3. Export data → Check `real_sat_data.jsonl` for tutor pairs.
* **Expected**: \~100 pairs from 1 tutor in a test week.

***

#### Conclusion

This implementation adds:

* **Backend**: `TUTORS` and `TUTOR_FEEDBACK` tables, `/tutor/feedback` and `/tutor/progress` endpoints.
* **Mobile**: **Tutor Dashboard** screen for feedback submission.
* **Pipeline**: Tutor feedback integrated into LLM training data export.

It enables the SAT Prep Suite to capture expert tutor feedback, starting with \~1K pairs at 1K users and scaling to \~10K at 10K users, enhancing our LLM’s quality and reducing reliance on external APIs by mid-2025. Ready to deploy and test—any adjustments before proceeding?
