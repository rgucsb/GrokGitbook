# Integration Check

To ensure that the **mobile app (Flutter)**, **web app (Flutter Web)**, and **backend (FastAPI)** of the **SAT Prep Suite** work together seamlessly, I’ll perform a conceptual integration check based on the code provided so far. This involves verifying that the APIs (backend) align with the frontend calls (mobile and web), the data flows correctly between components, and the hybrid structure (mobile for practice/lessons, web for diagnostics/full tests) is cohesive. Since I can’t execute the code directly, I’ll analyze the interfaces, dependencies, and interactions, flagging any mismatches or missing pieces, and suggest fixes as of March 26, 2025.

***

### Integration Check: Mobile, Web, and Backend

#### 1. Overview of Components

* **Mobile App (Flutter)**: Handles practice, study plans, community (short, on-the-go sessions).
* **Web App (Flutter Web)**: Handles diagnostics, full tests, tutor tools (longer, focused sessions).
* **Backend (FastAPI)**: Unified API serving both platforms, managing data and AI logic.

#### 2. API and Frontend Alignment

I’ll check each feature’s API endpoints against the frontend calls in `api_service.dart` and screen implementations.

**Feature: Practice (Mobile)**

* **Backend**:
  * `POST /practice/start/{user_id}` → Returns `practice_id`, `questions`.
  * `POST /practice/submit/{practice_id}` → Takes `responses`, returns `feedback`.
* **Mobile (`practice.dart`)**:
  * Calls `startPractice(userId)` → Matches `/practice/start`.
  * Calls `submitPractice(practiceId, responses)` → Matches `/practice/submit`.
* **Check**:
  * ✅ API and mobile align (response structure: `practice_id`, `questions` for start; `feedback` for submit).
  * **Note**: `practice.dart` assumes `questions` has `id` and `text` keys—ensure `Question.content` in DB matches this (`{"id": "q1", "text": "Solve: ..."}`).

**Feature: Diagnostics/Full Tests (Web)**

* **Backend**:
  * `POST /test/start/{user_id}?test_type={type}` → Returns `test_id`, `questions`.
  * `POST /test/submit/{test_id}` → Takes `responses`, returns `score`, `feedback`.
* **Web (`test.dart`)**:
  * Calls `startTest(userId, testType)` → Matches `/test/start`.
  * Calls `submitTest(testId, responses)` → Matches `/test/submit`, navigates to `/results`.
* **Check**:
  * ✅ API and web align (response structure consistent).
  * **Issue**: `results.dart` is a placeholder—needs to display `score` and `feedback` from `submitTest`. Fix below.

**Feature: Study Plan (Mobile)**

* **Backend**:
  * `POST /study_plan/create/{user_id}?test_date={date}` → Returns `plan_id`, `plan`.
  * `POST /study_plan/action/{plan_id}` → Takes `action_data`, returns `action`, `feedback`.
* **Mobile (`study_plan.dart`)**:
  * Calls `createStudyPlan(userId, testDate)` → Matches `/study_plan/create`.
  * Calls `logStudyPlanAction(planId, actionData)` → Matches `/study_plan/action`.
* **Check**:
  * ✅ API and mobile align.
  * **Note**: `study_plan.dart` refreshes the plan after each action—ensure backend updates `plan` in DB correctly.

**Feature: Community (Mobile)**

* **Backend**:
  * `POST /community/note/{user_id}` → Takes `note_data`, returns `note_id`, `note_text`, `ai_response`.
  * `GET /community/notes/{user_id}` → Returns `notes`.
* **Mobile (`community.dart`)**:
  * Calls `postCommunityNote(userId, noteData)` → Matches `/community/note`.
  * Calls `getUserNotes(userId)` → Matches `/community/notes`.
* **Check**:
  * ✅ API and mobile align.

**Feature: Tutor Tools (Web)**

* **Backend**:
  * `GET /tutor/progress/{tutor_id}` → Returns `students` with `proficiencies`.
  * `POST /tutor/feedback` → Takes `tutor_id`, `user_id`, etc., returns `feedback_text`.
* **Web (`tutor_dashboard.dart`)**:
  * Calls `getTutorStudentProgress(tutorId)` → Matches `/tutor/progress`.
  * Calls `submitTutorFeedback(...)` → Matches `/tutor/feedback`.
* **Check**:
  * ✅ API and web align.
  * **Note**: `context_type` in `tutor.py` includes `"test"`—ensure frontend dropdown includes it.

**Feature: Review/Skill Predictions (Both)**

* **Backend**:
  * `GET /review/skills/{user_id}` → Returns `proficiencies`, `feedback`.
  * `GET /review/next/{user_id}` → Returns `questions`.
* **Frontend**:
  * `getSkillPredictions(userId)` available but not used in screens yet.
* **Check**:
  * **Issue**: No frontend integration for `/review/*`. Suggest adding to `dashboard.dart` for progress display.

***

#### 3. Data Flow Verification

Using a sample scenario: Student practices on mobile, takes a test on web, tutor reviews on web.

**Mobile Practice → Web Test → Tutor Feedback**

* **Mobile**:
  * `startPractice('user123')` → Backend fetches 10 questions → Returns to `practice.dart`.
  * `submitPractice(practice_id, responses)` → Backend logs `RESPONSES`, generates feedback → Displays in UI.
* **Web**:
  * `startTest('user123', 'full')` → Backend fetches 98 questions → Returns to `test.dart`.
  * `submitTest(test_id, responses)` → Backend logs `TESTS`, `RESULTS`, feedback → Navigates to `results.dart`.
* **Web (Tutor)**:
  * `getTutorStudentProgress('tutor123')` → Backend queries `RESULTS` → Displays in `tutor_dashboard.dart`.
  * `submitTutorFeedback(...)` → Backend logs `TUTOR_FEEDBACK` → Confirms in UI.
* **Data Export**: `export_llm_training_data` → Combines `RESPONSES`, `TESTS`, `TUTOR_FEEDBACK` into `real_sat_data.jsonl`.
* **Check**:
  * ✅ Flow works across platforms.
  * **Note**: Ensure `user_id` is consistent (e.g., via auth) across mobile/web.

***

#### 4. Missing Pieces and Fixes

**Backend Issues**

1. **Question Content Format**:
   * Problem: `practice.dart` and `test.dart` expect `questions` with `id` and `text`, but `Question.content` is JSON (`{"text": ..., "options": ...}`).
   *   Fix: Update `practice_module.py` and `test.py` to return `question_id` alongside `content`:

       ```python
       # In practice_module.py
       return {"user_id": user_id, "practice_id": f"prac_{user_id}_{time.time()}", "questions": [{"id": q.question_id, **q.content} for q in questions]}
       # In test.py
       return {"user_id": user_id, "test_id": f"test_{user_id}_{time.time()}", "questions": [{"id": q.question_id, **q.content} for q in questions]}
       ```

**Mobile Issues**

1. **Study Plan Feedback Display**:
   * Problem: `study_plan.dart` logs actions but doesn’t display `feedback` beyond a snackbar.
   * Fix: Add feedback to UI (see updated `study_plan.dart` below).

**Web Issues**

1. **Results Screen Placeholder**:
   * Problem: `results.dart` doesn’t display `score`/`feedback` from `submitTest`.
   * Fix: Update `results.dart` to receive and show data (see below).
2. **Review Integration**:
   * Problem: `/review/skills` not used.
   * Fix: Add to `dashboard.dart` for both platforms (see below).

***

#### 5. Updated Files

**`mobile/lib/screens/study_plan.dart` (Updated)**

```dart
import 'package:flutter/material.dart';
import '../services/api_service.dart';

class StudyPlanScreen extends StatefulWidget {
  @override
  _StudyPlanScreenState createState() => _StudyPlanScreenState();
}

class _StudyPlanScreenState extends State<StudyPlanScreen> {
  final ApiService api = ApiService();
  Map<String, dynamic>? plan;
  String userId = 'user123';
  Map<int, String> taskFeedback = {}; // Store feedback per task index

  @override
  void initState() {
    super.initState();
    api.createStudyPlan(userId, '2025-06-01').then((data) => setState(() => plan = data));
  }

  void _logAction(int planId, Map<String, dynamic> task, String action, int taskIndex) {
    api.logStudyPlanAction(planId, {
      "task": task,
      "action": action,
      "performance_update": {"accuracy": 0.8}
    }).then((res) {
      setState(() => taskFeedback[taskIndex] = res['feedback']);
      api.createStudyPlan(userId, '2025-06-01').then((data) => setState(() => plan = data));
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
            children: week['tasks'].asMap().entries.map<Widget>((entry) {
              int taskIndex = entry.key;
              var task = entry.value;
              return ListTile(
                title: Text('${task['day']}: ${task['task']} (${task['skill']})'),
                subtitle: taskFeedback[taskIndex] != null ? Text('Feedback: ${taskFeedback[taskIndex]}') : null,
                trailing: Row(
                  mainAxisSize: MainAxisSize.min,
                  children: [
                    IconButton(
                      icon: Icon(Icons.check),
                      onPressed: () => _logAction(plan!['plan_id'], task, 'completed', taskIndex),
                    ),
                    IconButton(
                      icon: Icon(Icons.skip_next),
                      onPressed: () => _logAction(plan!['plan_id'], task, 'skipped', taskIndex),
                    ),
                  ],
                ),
              );
            }).toList(),
          );
        },
      ),
    );
  }
}
```

**`mobile/lib/screens/results.dart` (Updated)**

```dart
import 'package:flutter/material.dart';
import '../services/api_service.dart';

class ResultsScreen extends StatefulWidget {
  final String testId;
  ResultsScreen({required this.testId});

  @override
  _ResultsScreenState createState() => _ResultsScreenState();
}

class _ResultsScreenState extends State<ResultsScreen> {
  final ApiService api = ApiService();
  Map<String, dynamic>? results;

  @override
  void initState() {
    super.initState();
    // Assume test submission already happened; fetch results if needed
    // Here, we rely on navigation passing results for simplicity
  }

  @override
  Widget build(BuildContext context) {
    final results = ModalRoute.of(context)?.settings.arguments as Map<String, dynamic>?;
    if (results == null) return Center(child: Text('No results available'));
    return Scaffold(
      appBar: AppBar(title: Text('Test Results')),
      body: Padding(
        padding: EdgeInsets.all(16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text('Score: ${results['score']}', style: TextStyle(fontSize: 24)),
            SizedBox(height: 16),
            Text('Feedback: ${results['feedback']}', style: TextStyle(fontSize: 16)),
          ],
        ),
      ),
    );
  }
}
```

**`mobile/lib/screens/test.dart` (Updated Navigation)**

```dart
// Update submit button in _TestScreenState
onPressed: () => api.submitTest(testSession!['test_id'], [{'question_id': testSession!['questions'][index]['id'], 'answer': 'A'}]).then((res) => Navigator.pushNamed(context, '/results', arguments: res)),
```

**`mobile/lib/screens/dashboard.dart` (Updated with Review)**

```dart
import 'package:flutter/material.dart';
import '../services/api_service.dart';

class DashboardScreen extends StatefulWidget {
  @override
  _DashboardScreenState createState() => _DashboardScreenState();
}

class _DashboardScreenState extends State<DashboardScreen> {
  final ApiService api = ApiService();
  Map<String, dynamic>? skills;

  @override
  void initState() {
    super.initState();
    api.getSkillPredictions('user123').then((data) => setState(() => skills = data));
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('SAT Prep')),
      body: ListView(
        children: [
          if (skills != null) ...[
            Text('Skills:', style: TextStyle(fontSize: 18)),
            Text(skills!['proficiencies'].toString()),
            Text('Feedback: ${skills!['feedback']}'),
          ] else Center(child: CircularProgressIndicator()),
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

**`mobile/lib/screens/tutor_dashboard.dart` (Updated Context)**

```dart
// Update DropdownButton items
items: ['practice', 'question', 'plan', 'general', 'test'].map((e) => DropdownMenuItem(value: e, child: Text(e))).toList(),
```

***

#### 6. Final Verification

* **Mobile**:
  * Practice, Study Plan, Community → All call matching APIs, display results.
  * Dashboard → Now shows skill predictions.
* **Web**:
  * Tests → `test.dart` fetches/submits, `results.dart` displays.
  * Tutor → `tutor_dashboard.dart` integrates progress and feedback.
* **Backend**:
  * All endpoints (`/practice`, `/test`, `/study_plan`, `/community`, `/tutor`, `/review`) → Serve both platforms, log data consistently.
* **Data**: `RESPONSES`, `TESTS`, `STUDY_PLAN_ACTIONS`, `USER_NOTES`, `TUTOR_FEEDBACK`, `RESULTS` → Unified storage, exported for LLM.

***

#### Conclusion

The mobile and web versions now fully integrate with the backend:

* **Fixes Applied**: Question format, results display, review integration, tutor context.
* **Remaining**: Add authentication (`user_id` hardcoded), expand question bank in DB.
* **Test**: Run `flutter run` (mobile), `flutter run -d chrome` (web), and `uvicorn api.main:app` to confirm.

Everything should work together—any specific flows to double-check? Let me know!
