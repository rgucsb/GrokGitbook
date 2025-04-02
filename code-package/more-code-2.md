# More Code 2

Below, I’ll provide the missing files for the **SAT Prep Suite Code Package**: `api/routes/__init__.py`, `api/routes/tutor.py` (assuming "tytor.py" was a typo for "tutor.py"), `mobile/lib/screens/study_plan.dart`, `mobile/lib/screens/community.dart`, and `mobile/lib/test/widget_test.dart`. These files complete the backend routing structure and add the necessary Flutter screens and tests, aligning with the hybrid mobile/web app structure as of March 26, 2025. I’ll integrate them with the existing code, ensuring consistency with prior implementations like `api_service.dart` and the updated `utils.py`.

***

## SAT Prep Suite: Missing Files Document

### Backend Files

#### `api/routes/__init__.py`

**Purpose**: Initializes the routes module, enabling import of all route files in `main.py`.

```python
# api/routes/__init__.py
from .practice_module import router as practice_module_router
from .study_plan import router as study_plan_router
from .review import router as review_router
from .community import router as community_router
from .tutor import router as tutor_router
from .test import router as test_router

# Export routers for inclusion in main.py
__all__ = [
    "practice_module_router",
    "study_plan_router",
    "review_router",
    "community_router",
    "tutor_router",
    "test_router"
]
```

**Notes**:

* This file allows `main.py` to import routes cleanly via `from api.routes import *`.
*   Update `main.py` imports if not already using this pattern:

    ```python
    from api.routes import practice_module, study_plan, review, community, tutor, test
    ```

***

#### `api/routes/tutor.py`

**Purpose**: Handles tutor-specific endpoints for progress tracking and feedback submission.

```python
# api/routes/tutor.py
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from api.database import get_db
from api.models import Tutor, User
from api.utils import log_tutor_feedback, get_student_progress
from typing import Dict, Optional

router = APIRouter(prefix="/tutor", tags=["tutor"])

@router.get("/progress/{tutor_id}")
async def get_tutor_student_progress(tutor_id: str, db: Session = Depends(get_db)):
    """Retrieve progress for all students assigned to a tutor."""
    if not db.query(Tutor).filter(Tutor.tutor_id == tutor_id).first():
        raise HTTPException(status_code=404, detail="Tutor not found")
    progress = get_student_progress(db, tutor_id)
    return progress

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
    """Submit feedback for a student."""
    if context_type not in ["practice", "question", "plan", "general", "test"]:
        raise HTTPException(status_code=400, detail="Invalid context_type")
    if not db.query(Tutor).filter(Tutor.tutor_id == tutor_id).first():
        raise HTTPException(status_code=404, detail="Tutor not found")
    if not db.query(User).filter(User.user_id == user_id).first():
        raise HTTPException(status_code=404, detail="User not found")
    
    log_tutor_feedback(db, tutor_id, user_id, feedback_text, context_type, question_id, plan_id, performance_data)
    return {"user_id": user_id, "feedback_text": feedback_text, "context_type": context_type}
```

**Notes**:

* **Features**: Retrieves student progress and logs tutor feedback with context (e.g., practice, test).
* **Integration**: Uses `log_tutor_feedback` and `get_student_progress` from `utils.py`.

***

### Frontend Files (Flutter)

#### `mobile/lib/screens/study_plan.dart`

**Purpose**: Displays and manages the student’s study plan on the mobile app.

```dart
// mobile/lib/screens/study_plan.dart
import 'package:flutter/material.dart';
import '../services/api_service.dart';

class StudyPlanScreen extends StatefulWidget {
  @override
  _StudyPlanScreenState createState() => _StudyPlanScreenState();
}

class _StudyPlanScreenState extends State<StudyPlanScreen> {
  final ApiService api = ApiService();
  Map<String, dynamic>? plan;
  String userId = 'user123'; // Hardcoded for demo, replace with auth

  @override
  void initState() {
    super.initState();
    api.createStudyPlan(userId, '2025-06-01').then((data) => setState(() => plan = data));
  }

  void _logAction(int planId, Map<String, dynamic> task, String action) {
    api.logStudyPlanAction(planId, {
      "task": task,
      "action": action,
      "performance_update": {"accuracy": 0.8} // Example data
    }).then((res) {
      ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text('Action logged: ${res['feedback']}')));
      api.createStudyPlan(userId, '2025-06-01').then((data) => setState(() => plan = data)); // Refresh plan
    });
  }

  @override
  Widget build(BuildContext context) {
    if (plan == null) return Center(child: CircularProgressIndicator());
    return Scaffold(
      appBar: AppBar(title: Text('Study Plan')),
      body: ListView.builder(
        itemCount: plan!['plan']['weeks'].length,
        itemBuilder: (context, weekIndex) {
          var week = plan!['plan']['weeks'][weekIndex];
          return ExpansionTile(
            title: Text('Week ${week['week']}'),
            children: week['tasks'].map<Widget>((task) => ListTile(
              title: Text('${task['day']}: ${task['task']} (${task['skill']})'),
              trailing: Row(
                mainAxisSize: MainAxisSize.min,
                children: [
                  IconButton(
                    icon: Icon(Icons.check),
                    onPressed: () => _logAction(plan!['plan_id'], task, 'completed'),
                  ),
                  IconButton(
                    icon: Icon(Icons.skip_next),
                    onPressed: () => _logAction(plan!['plan_id'], task, 'skipped'),
                  ),
                ],
              ),
            )).toList(),
          );
        },
      ),
    );
  }
}
```

**Notes**:

* **Features**: Shows a weekly task list, allows marking tasks as completed/skipped, refreshes plan after actions.
* **Integration**: Calls `createStudyPlan` and `logStudyPlanAction` from `api_service.dart`.

***

#### `mobile/lib/screens/community.dart`

**Purpose**: Enables community note/question posting and viewing on the mobile app.

```dart
// mobile/lib/screens/community.dart
import 'package:flutter/material.dart';
import '../services/api_service.dart';

class CommunityScreen extends StatefulWidget {
  @override
  _CommunityScreenState createState() => _CommunityScreenState();
}

class _CommunityScreenState extends State<CommunityScreen> {
  final ApiService api = ApiService();
  final TextEditingController _noteController = TextEditingController();
  String userId = 'user123'; // Hardcoded for demo
  List<Map<String, dynamic>> notes = [];

  @override
  void initState() {
    super.initState();
    _fetchNotes();
  }

  void _fetchNotes() {
    api.getUserNotes(userId).then((data) => setState(() => notes = List<Map<String, dynamic>>.from(data['notes'])));
  }

  void _postNote() {
    api.postCommunityNote(userId, {"note_text": _noteController.text}).then((res) {
      ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text('Note posted: ${res['ai_response']}')));
      _noteController.clear();
      _fetchNotes();
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Community')),
      body: Column(
        children: [
          Padding(
            padding: EdgeInsets.all(8.0),
            child: Row(
              children: [
                Expanded(
                  child: TextField(
                    controller: _noteController,
                    decoration: InputDecoration(labelText: 'Post a note or question'),
                  ),
                ),
                IconButton(
                  icon: Icon(Icons.send),
                  onPressed: _postNote,
                ),
              ],
            ),
          ),
          Expanded(
            child: ListView.builder(
              itemCount: notes.length,
              itemBuilder: (context, index) => ListTile(
                title: Text(notes[index]['note_text']),
                subtitle: Text('AI: ${notes[index]['ai_response']}'),
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```

**Notes**:

* **Features**: Allows posting notes/questions, displays user’s note history with AI responses.
* **Integration**: Uses `postCommunityNote` and `getUserNotes` from `api_service.dart` (add `getUserNotes` below).

***

#### `mobile/lib/test/widget_test.dart`

**Purpose**: Provides basic widget tests for Flutter screens.

```dart
// mobile/lib/test/widget_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:sat_prep_suite/main.dart';
import 'package:sat_prep_suite/screens/dashboard.dart';
import 'package:sat_prep_suite/screens/practice.dart';

void main() {
  testWidgets('Dashboard loads with title', (WidgetTester tester) async {
    await tester.pumpWidget(MyApp());
    expect(find.text('SAT Prep'), findsOneWidget);
    expect(find.text('Practice'), findsOneWidget);
  });

  testWidgets('Practice screen loads questions', (WidgetTester tester) async {
    await tester.pumpWidget(MaterialApp(home: PracticeScreen()));
    await tester.pump(Duration(seconds: 2)); // Wait for API call
    expect(find.text('Solve: 2x + 3 = 7'), findsOneWidget); // Assumes seed data
  });
}
```

**Notes**:

* **Features**: Tests Dashboard and Practice screen rendering.
* **Expansion**: Add tests for `StudyPlanScreen`, `CommunityScreen`, etc., as needed.

***

### Updated `mobile/lib/services/api_service.dart`

**Purpose**: Add missing `getUserNotes` method to support `community.dart`.

```dart
// mobile/lib/services/api_service.dart (updated snippet)
import 'dart:convert';
import 'package:http/http.dart' as http;

class ApiService {
  static const String baseUrl = 'http://localhost:8000';

  // Existing methods: startPractice, submitPractice, startTest, submitTest, getTutorStudentProgress, submitTutorFeedback

  Future<Map<String, dynamic>> createStudyPlan(String userId, String testDate) async {
    final response = await http.post(Uri.parse('$baseUrl/study_plan/create/$userId?test_date=$testDate'));
    return jsonDecode(response.body);
  }

  Future<Map<String, dynamic>> logStudyPlanAction(int planId, Map<String, dynamic> actionData) async {
    final response = await http.post(Uri.parse('$baseUrl/study_plan/action/$planId'), body: jsonEncode(actionData));
    return jsonDecode(response.body);
  }

  Future<Map<String, dynamic>> getSkillPredictions(String userId) async {
    final response = await http.get(Uri.parse('$baseUrl/review/skills/$userId'));
    return jsonDecode(response.body);
  }

  Future<Map<String, dynamic>> postCommunityNote(String userId, Map<String, dynamic> noteData) async {
    final response = await http.post(Uri.parse('$baseUrl/community/note/$userId'), body: jsonEncode(noteData));
    return jsonDecode(response.body);
  }

  Future<Map<String, dynamic>> getUserNotes(String userId) async {
    final response = await http.get(Uri.parse('$baseUrl/community/notes/$userId'));
    return jsonDecode(response.body);
  }
}
```

***

### Integration Notes

* **`__init__.py`**: Ensures all routes are importable in `main.py`. No changes needed if already using individual imports.
* **`tutor.py`**: Completes tutor functionality, pairs with `tutor_dashboard.dart`.
* **`study_plan.dart`**: Mobile screen for task management, syncs with `study_plan.py`.
* **`community.dart`**: Mobile screen for community interaction, syncs with `community.py`.
* **`widget_test.dart`**: Basic tests—expand for full coverage.

### Next Steps

1. **Add to Package**: Copy these into the existing structure.
2. **Update Frontend**: Ensure `DashboardScreen` navigates to these screens.
3. **Test**: Run `flutter test` and `pytest` to verify integration.
4. **Deploy**: Follow the prior testing/deployment plan.

These files complete the missing pieces—any further refinements or additional files needed? Let me know!
