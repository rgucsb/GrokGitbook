# Mobile App Integration

Not stuck, just ensuring I deliver a thorough and well-structured implementation! I noticed your latest request builds on a previous suggestion to implement a **Mobile App and Offline Mode**, with additional emphasis on social/community features and notifications using the HOOK Model. Since this is a significant undertaking, I’ll break it down into a clear plan and implement it step-by-step, focusing on a **Flutter-based mobile app** (for its cross-platform capability) with offline sync, push notifications, and social features. I’ll assume the backend is already set up with gamification, analytics, and tutor/parent integration from prior steps.

Let’s dive in!

***

#### Step 1: Define Requirements

**Mobile App and Offline Mode**

* **Missing**: Web-only access, no offline capability.
* **Why Useful**: Students need flexibility to study on-the-go; internet reliability varies.
* **Features**:
  * **Mobile Frontend**: Flutter app mirroring the React web UI.
  * **Offline Sync**: Cache practice modules, assignments, and results locally (SQLite), sync when online.
  * **Push Notifications**: Remind users of tasks/tests (e.g., “Practice Algebra today!”).

**Social and Community Features**

* **Missing**: No peer interaction or collaboration.
* **Why Useful**: Peer support boosts motivation and learning.
* **Features**:
  * **Study Groups**: Join/create groups to share progress and chat.
  * **Q\&A Forum**: Post/answer SAT-related questions.
  * **Peer Challenges**: Compete on practice modules (e.g., “Beat my Algebra score!”).

**HOOK Model Integration**

* **Trigger**: External (notifications: “Your streak is at 3!”) and internal (desire to compete with peers).
* **Action**: Study, join groups, post in Q\&A, accept challenges.
* **Variable Reward**: Points, badges, leaderboard ranks, random group bonuses.
* **Investment**: Build streaks, contribute to Q\&A, grow group participation.

***

#### Step 2: Backend Enhancements

**Updated ERD (Mermaid Syntax)**

```mermaid
erDiagram
    USERS ||--o{ RESPONSES : submits
    QUESTIONS ||--o{ RESPONSES : answered_in
    USERS ||--o{ RESULTS : generates
    USERS ||--o{ USED_QUESTIONS : has_used
    USERS ||--o{ STUDY_PLANS : follows
    USERS ||--o{ POINTS : earns
    USERS ||--o{ LEADERBOARD_OPTS : opts_into
    TUTORS ||--o{ USER_TUTORS : mentors
    USERS ||--o{ USER_TUTORS : assigned_to
    USERS ||--o{ USER_PARENTS : monitored_by
    TUTORS ||--o{ TUTOR_ASSIGNMENTS : creates
    USERS ||--o{ TUTOR_ASSIGNMENTS : assigned_to
    USERS ||--o{ GROUPS : joins
    GROUPS ||--o{ GROUP_MEMBERS : includes
    USERS ||--o{ COMMUNITY_QUESTIONS : posts
    USERS ||--o{ COMMUNITY_ANSWERS : answers
    USERS ||--o{ CHALLENGES : issues

    USERS {
        string user_id PK "e.g., 'user123'"
        string username "Student's name"
        string email "Contact email"
        string fcm_token "Firebase Cloud Messaging token"
        datetime created_at "Creation timestamp"
    }

    QUESTIONS {
        string question_id PK "e.g., 'q1'"
        string test "Math or Reading and Writing"
        string domain "e.g., 'Algebra'"
        string skill "e.g., 'Linear Functions'"
        string difficulty "Easy, Medium, Hard"
        json content "Question details"
        json irt_params "IRT parameters {a, b, c}"
    }

    RESPONSES {
        int response_id PK "Auto-incremented"
        string user_id FK "References USERS(user_id)"
        string test_id FK "References RESULTS(test_id)"
        string question_id FK "References QUESTIONS(question_id)"
        string module "e.g., 'Math_1'"
        string answer "e.g., 'A'"
        boolean is_correct "True if correct"
        json content "Snapshot of question"
        float time_spent "Seconds taken to answer"
        datetime timestamp "Submission time"
    }

    RESULTS {
        int result_id PK "Auto-incremented"
        string user_id FK "References USERS(user_id)"
        string test_id "e.g., 'diag_abc123'"
        string test_type "diagnostic or full_length"
        json selected_sections "e.g., ['Math']"
        string section "Math or Reading and Writing"
        float theta "IRT ability score"
        json proficiencies "Skill proficiencies"
        datetime completed_at "Completion timestamp"
    }

    USED_QUESTIONS {
        int id PK "Auto-incremented"
        string user_id FK "References USERS(user_id)"
        string question_id FK "References QUESTIONS(question_id)"
        datetime used_at "Timestamp of use"
    }

    STUDY_PLANS {
        int plan_id PK "Auto-incremented"
        string user_id FK "References USERS(user_id)"
        datetime test_date "SAT exam date"
        int target_score "Desired score (400–1600)"
        int weekly_hours "Hours available per week"
        json plan "Daily/weekly schedule"
        datetime created_at "Plan creation timestamp"
    }

    POINTS {
        int id PK "Auto-incremented"
        string user_id FK "References USERS(user_id)"
        int points "Total points earned"
        json badges "List of earned badges"
        int streak "Consecutive days studied"
        datetime last_activity "Last study date"
    }

    LEADERBOARD_OPTS {
        int id PK "Auto-incremented"
        string user_id FK "References USERS(user_id)"
        boolean global_opt_in "Opt-in to global leaderboard"
        json friend_ids "List of friend user_ids"
    }

    TUTORS {
        string tutor_id PK "e.g., 'tutor456'"
        string name "Tutor's name"
        string email "Contact email"
        string password_hash "Hashed password"
        datetime created_at "Creation timestamp"
    }

    USER_TUTORS {
        int id PK "Auto-incremented"
        string user_id FK "References USERS(user_id)"
        string tutor_id FK "References TUTORS(tutor_id)"
        datetime assigned_at "Assignment timestamp"
    }

    USER_PARENTS {
        int id PK "Auto-incremented"
        string user_id FK "References USERS(user_id)"
        string parent_email "Parent's email"
        string access_token "Unique token for access"
        datetime created_at "Creation timestamp"
    }

    TUTOR_ASSIGNMENTS {
        int assignment_id PK "Auto-incremented"
        string tutor_id FK "References TUTORS(tutor_id)"
        string user_id FK "References USERS(user_id)"
        string type "practice or full_test"
        json content "Assignment details"
        string note "Tutor's instructions"
        datetime due_date "Due date"
        boolean completed "Completion status"
        datetime created_at "Creation timestamp"
    }

    GROUPS {
        int group_id PK "Auto-incremented"
        string name "Group name"
        string creator_id FK "References USERS(user_id)"
        datetime created_at "Creation timestamp"
    }

    GROUP_MEMBERS {
        int id PK "Auto-incremented"
        int group_id FK "References GROUPS(group_id)"
        string user_id FK "References USERS(user_id)"
        datetime joined_at "Join timestamp"
    }

    COMMUNITY_QUESTIONS {
        int id PK "Auto-incremented"
        string user_id FK "References USERS(user_id)"
        string question "Question text"
        datetime timestamp "Post time"
    }

    COMMUNITY_ANSWERS {
        int id PK "Auto-incremented"
        string user_id FK "References USERS(user_id)"
        int question_id FK "References COMMUNITY_QUESTIONS(id)"
        string answer "Answer text"
        datetime timestamp "Post time"
    }

    CHALLENGES {
        int challenge_id PK "Auto-incremented"
        string issuer_id FK "References USERS(user_id)"
        string opponent_id FK "References USERS(user_id)"
        string section "Math or Reading and Writing"
        string domain "e.g., 'Algebra'"
        int num_questions "Number of questions"
        float issuer_theta "Issuer's score"
        float opponent_theta "Opponent's score (null until completed)"
        datetime created_at "Creation timestamp"
        datetime completed_at "Completion timestamp (null until done)"
    }
```

```mermaid
erDiagram
    USERS ||--o{ RESPONSES : submits
    QUESTIONS ||--o{ RESPONSES : answered_in
    USERS ||--o{ RESULTS : generates
    USERS ||--o{ USED_QUESTIONS : has_used
    USERS ||--o{ STUDY_PLANS : follows
    USERS ||--o{ POINTS : earns
    USERS ||--o{ LEADERBOARD_OPTS : opts_into
    TUTORS ||--o{ USER_TUTORS : mentors
    USERS ||--o{ USER_TUTORS : assigned_to
    USERS ||--o{ USER_PARENTS : monitored_by
    TUTORS ||--o{ TUTOR_ASSIGNMENTS : creates
    USERS ||--o{ TUTOR_ASSIGNMENTS : assigned_to
    USERS ||--o{ GROUPS : joins
    GROUPS ||--o{ GROUP_MEMBERS : includes
    USERS ||--o{ COMMUNITY_QUESTIONS : posts
    USERS ||--o{ COMMUNITY_ANSWERS : answers
    USERS ||--o{ CHALLENGES : issues

    USERS {
        string user_id PK "e.g., 'user123'"
        string username "Student's name"
        string email "Contact email"
        string fcm_token "Firebase Cloud Messaging token"
        datetime created_at "Creation timestamp"
    }

    QUESTIONS {
        string question_id PK "e.g., 'q1'"
        string test "Math or Reading and Writing"
        string domain "e.g., 'Algebra'"
        string skill "e.g., 'Linear Functions'"
        string difficulty "Easy, Medium, Hard"
        json content "Question details"
        json irt_params "IRT parameters {a, b, c}"
    }

    RESPONSES {
        int response_id PK "Auto-incremented"
        string user_id FK "References USERS(user_id)"
        string test_id FK "References RESULTS(test_id)"
        string question_id FK "References QUESTIONS(question_id)"
        string module "e.g., 'Math_1'"
        string answer "e.g., 'A'"
        boolean is_correct "True if correct"
        json content "Snapshot of question"
        float time_spent "Seconds taken to answer"
        datetime timestamp "Submission time"
    }

    RESULTS {
        int result_id PK "Auto-incremented"
        string user_id FK "References USERS(user_id)"
        string test_id "e.g., 'diag_abc123'"
        string test_type "diagnostic or full_length"
        json selected_sections "e.g., ['Math']"
        string section "Math or Reading and Writing"
        float theta "IRT ability score"
        json proficiencies "Skill proficiencies"
        datetime completed_at "Completion timestamp"
    }

    USED_QUESTIONS {
        int id PK "Auto-incremented"
        string user_id FK "References USERS(user_id)"
        string question_id FK "References QUESTIONS(question_id)"
        datetime used_at "Timestamp of use"
    }

    STUDY_PLANS {
        int plan_id PK "Auto-incremented"
        string user_id FK "References USERS(user_id)"
        datetime test_date "SAT exam date"
        int target_score "Desired score (400–1600)"
        int weekly_hours "Hours available per week"
        json plan "Daily/weekly schedule"
        datetime created_at "Plan creation timestamp"
    }

    POINTS {
        int id PK "Auto-incremented"
        string user_id FK "References USERS(user_id)"
        int points "Total points earned"
        json badges "List of earned badges"
        int streak "Consecutive days studied"
        datetime last_activity "Last study date"
    }

    LEADERBOARD_OPTS {
        int id PK "Auto-incremented"
        string user_id FK "References USERS(user_id)"
        boolean global_opt_in "Opt-in to global leaderboard"
        json friend_ids "List of friend user_ids"
    }

    TUTORS {
        string tutor_id PK "e.g., 'tutor456'"
        string name "Tutor's name"
        string email "Contact email"
        string password_hash "Hashed password"
        datetime created_at "Creation timestamp"
    }

    USER_TUTORS {
        int id PK "Auto-incremented"
        string user_id FK "References USERS(user_id)"
        string tutor_id FK "References TUTORS(tutor_id)"
        datetime assigned_at "Assignment timestamp"
    }

    USER_PARENTS {
        int id PK "Auto-incremented"
        string user_id FK "References USERS(user_id)"
        string parent_email "Parent's email"
        string access_token "Unique token for access"
        datetime created_at "Creation timestamp"
    }

    TUTOR_ASSIGNMENTS {
        int assignment_id PK "Auto-incremented"
        string tutor_id FK "References TUTORS(tutor_id)"
        string user_id FK "References USERS(user_id)"
        string type "practice or full_test"
        json content "Assignment details"
        string note "Tutor's instructions"
        datetime due_date "Due date"
        boolean completed "Completion status"
        datetime created_at "Creation timestamp"
    }

    GROUPS {
        int group_id PK "Auto-incremented"
        string name "Group name"
        string creator_id FK "References USERS(user_id)"
        datetime created_at "Creation timestamp"
    }

    GROUP_MEMBERS {
        int id PK "Auto-incremented"
        int group_id FK "References GROUPS(group_id)"
        string user_id FK "References USERS(user_id)"
        datetime joined_at "Join timestamp"
    }

    COMMUNITY_QUESTIONS {
        int id PK "Auto-incremented"
        string user_id FK "References USERS(user_id)"
        string question "Question text"
        datetime timestamp "Post time"
    }

    COMMUNITY_ANSWERS {
        int id PK "Auto-incremented"
        string user_id FK "References USERS(user_id)"
        int question_id FK "References COMMUNITY_QUESTIONS(id)"
        string answer "Answer text"
        datetime timestamp "Post time"
    }

    CHALLENGES {
        int challenge_id PK "Auto-incremented"
        string issuer_id FK "References USERS(user_id)"
        string opponent_id FK "References USERS(user_id)"
        string section "Math or Reading and Writing"
        string domain "e.g., 'Algebra'"
        int num_questions "Number of questions"
        float issuer_theta "Issuer's score"
        float opponent_theta "Opponent's score (null until completed)"
        datetime created_at "Creation timestamp"
        datetime completed_at "Completion timestamp (null until done)"
    }
```

**Changes**:

* Added `fcm_token` to `USERS` for push notifications.
* Added `GROUPS`, `GROUP_MEMBERS`, `COMMUNITY_QUESTIONS`, `COMMUNITY_ANSWERS`, `CHALLENGES` for social features.

**`api/models.py` (Updated)**

```python
from sqlalchemy import Column, Integer, String, Float, JSON, DateTime, ForeignKey, Boolean
from sqlalchemy.ext.declarative import declarative_base
from datetime import datetime

Base = declarative_base()

class User(Base):
    __tablename__ = "users"
    user_id = Column(String, primary_key=True)
    username = Column(String, nullable=False)
    email = Column(String, unique=True)
    fcm_token = Column(String)  # Firebase Cloud Messaging token
    created_at = Column(DateTime, default=datetime.utcnow)

# Existing models (Question, Response, Result, UsedQuestion, StudyPlan, Points, LeaderboardOpt, Tutor, UserTutor, UserParent, TutorAssignment) unchanged

class Group(Base):
    __tablename__ = "groups"
    group_id = Column(Integer, primary_key=True, autoincrement=True)
    name = Column(String, nullable=False)
    creator_id = Column(String, ForeignKey("users.user_id"), nullable=False)
    created_at = Column(DateTime, default=datetime.utcnow)

class GroupMember(Base):
    __tablename__ = "group_members"
    id = Column(Integer, primary_key=True, autoincrement=True)
    group_id = Column(Integer, ForeignKey("groups.group_id"), nullable=False)
    user_id = Column(String, ForeignKey("users.user_id"), nullable=False)
    joined_at = Column(DateTime, default=datetime.utcnow)

class CommunityQuestion(Base):
    __tablename__ = "community_questions"
    id = Column(Integer, primary_key=True, autoincrement=True)
    user_id = Column(String, ForeignKey("users.user_id"), nullable=False)
    question = Column(String, nullable=False)
    timestamp = Column(DateTime, default=datetime.utcnow)

class CommunityAnswer(Base):
    __tablename__ = "community_answers"
    id = Column(Integer, primary_key=True, autoincrement=True)
    user_id = Column(String, ForeignKey("users.user_id"), nullable=False)
    question_id = Column(Integer, ForeignKey("community_questions.id"), nullable=False)
    answer = Column(String, nullable=False)
    timestamp = Column(DateTime, default=datetime.utcnow)

class Challenge(Base):
    __tablename__ = "challenges"
    challenge_id = Column(Integer, primary_key=True, autoincrement=True)
    issuer_id = Column(String, ForeignKey("users.user_id"), nullable=False)
    opponent_id = Column(String, ForeignKey("users.user_id"), nullable=False)
    section = Column(String, nullable=False)
    domain = Column(String, nullable=False)
    num_questions = Column(Integer, nullable=False)
    issuer_theta = Column(Float)
    opponent_theta = Column(Float)
    created_at = Column(DateTime, default=datetime.utcnow)
    completed_at = Column(DateTime)
```

**`api/utils.py` (Updated)**

```python
# Existing imports and functions unchanged

def send_notification(fcm_token: str, title: str, body: str):
    # Placeholder: Use Firebase Admin SDK in production
    from firebase_admin import messaging
    message = messaging.Message(
        notification=messaging.Notification(title=title, body=body),
        token=fcm_token
    )
    try:
        messaging.send(message)
    except Exception as e:
        print(f"Failed to send notification: {e}")

def notify_users(db: Session, user_ids: List[str], title: str, body: str):
    for user_id in user_ids:
        user = db.query(User).filter(User.user_id == user_id).first()
        if user and user.fcm_token:
            send_notification(user.fcm_token, title, body)
```

**`api/routes/community.py` (New Module)**

```python
from fastapi import APIRouter, HTTPException, Depends
from pydantic import BaseModel
from sqlalchemy.orm import Session
from api.utils import notify_users, award_points
from api.models import Group, GroupMember, CommunityQuestion, CommunityAnswer, Challenge
from database import get_db

router = APIRouter()

class GroupRequest(BaseModel):
    user_id: str
    name: str

class JoinGroupRequest(BaseModel):
    user_id: str
    group_id: int

class QuestionPost(BaseModel):
    user_id: str
    question: str

class AnswerPost(BaseModel):
    user_id: str
    question_id: int
    answer: str

class ChallengeRequest(BaseModel):
    issuer_id: str
    opponent_id: str
    section: str
    domain: str
    num_questions: int

@router.post("/group/create")
async def create_group(request: GroupRequest, db: Session = Depends(get_db)):
    group = Group(name=request.name, creator_id=request.user_id)
    db.add(group)
    db.flush()
    db.add(GroupMember(group_id=group.group_id, user_id=request.user_id))
    gamification = award_points(db, request.user_id, "group_create")  # Custom task type
    db.commit()
    notify_users(db, [request.user_id], "Group Created", f"You started '{request.name}'!")
    return {"group_id": group.group_id, "name": request.name, "gamification": gamification}

@router.post("/group/join")
async def join_group(request: JoinGroupRequest, db: Session = Depends(get_db)):
    if not db.query(Group).filter(Group.group_id == request.group_id).first():
        raise HTTPException(status_code=404, detail="Group not found")
    member = GroupMember(group_id=request.group_id, user_id=request.user_id)
    db.add(member)
    gamification = award_points(db, request.user_id, "group_join")
    db.commit()
    notify_users(db, [request.user_id], "Joined Group", f"You joined group {request.group_id}!")
    return {"group_id": request.group_id, "user_id": request.user_id, "gamification": gamification}

@router.get("/group/{group_id}")
async def get_group(group_id: int, db: Session = Depends(get_db)):
    group = db.query(Group).filter(Group.group_id == group_id).first()
    if not group:
        raise HTTPException(status_code=404, detail="Group not found")
    members = db.query(GroupMember).filter(GroupMember.group_id == group_id).all()
    return {"group_id": group_id, "name": group.name, "members": [m.user_id for m in members]}

@router.post("/questions")
async def post_question(request: QuestionPost, db: Session = Depends(get_db)):
    q = CommunityQuestion(user_id=request.user_id, question=request.question)
    db.add(q)
    gamification = award_points(db, request.user_id, "question_post")
    db.commit()
    notify_users(db, [request.user_id], "Question Posted", "Your question is live in the community!")
    return {"question_id": q.id, "question": q.question, "gamification": gamification}

@router.post("/answers")
async def post_answer(request: AnswerPost, db: Session = Depends(get_db)):
    if not db.query(CommunityQuestion).filter(CommunityQuestion.id == request.question_id).first():
        raise HTTPException(status_code=404, detail="Question not found")
    a = CommunityAnswer(user_id=request.user_id, question_id=request.question_id, answer=request.answer)
    db.add(a)
    gamification = award_points(db, request.user_id, "answer_post")
    db.commit()
    notify_users(db, [request.user_id], "Answer Posted", "Thanks for helping the community!")
    return {"answer_id": a.id, "answer": a.answer, "gamification": gamification}

@router.post("/challenge")
async def create_challenge(request: ChallengeRequest, db: Session = Depends(get_db)):
    question_bank = load_question_bank()
    used_ids = get_used_questions(db, request.issuer_id)
    proficiencies = get_latest_proficiencies(db, request.issuer_id)
    questions = select_proficiency_based_questions(request.section, question_bank, {request.domain: {skill: {} for skill in (RW_TEST_PLAN if request.section == "Reading and Writing" else MATH_TEST_PLAN)[request.domain].keys()}}, used_ids, proficiencies.get(request.section, {}), request.num_questions)
    challenge = Challenge(issuer_id=request.issuer_id, opponent_id=request.opponent_id, section=request.section, domain=request.domain, num_questions=request.num_questions)
    db.add(challenge)
    db.flush()
    practice_db[f"challenge_{challenge.challenge_id}"] = {"issuer_id": request.issuer_id, "questions": questions, "responses": []}
    gamification = award_points(db, request.issuer_id, "challenge_create")
    db.commit()
    notify_users(db, [request.opponent_id], "New Challenge!", f"{db.query(User).get(request.issuer_id).username} challenged you in {request.domain}!")
    return {"challenge_id": challenge.challenge_id, "questions": questions, "gamification": gamification}

@router.post("/challenge/respond/{challenge_id}")
async def respond_challenge(challenge_id: int, request: PracticeResponseRequest, db: Session = Depends(get_db)):
    challenge = db.query(Challenge).filter(Challenge.challenge_id == challenge_id).first()
    if not challenge or request.user_id != challenge.opponent_id:
        raise HTTPException(status_code=403, detail="Unauthorized or challenge not found")
    session = practice_db.get(f"challenge_{challenge_id}")
    if not session:
        raise HTTPException(status_code=404, detail="Challenge session not found")
    
    session["responses"] = [r.dict() for r in request.responses]
    irt = IRTSelector()
    responses = [(q, r["is_correct"]) for q in session["questions"] for r in session["responses"] if q["metadata"]["Question ID"] == r["question_id"]]
    theta = irt.update_theta(responses)
    challenge.opponent_theta = theta
    challenge.completed_at = datetime.utcnow()
    gamification = award_points(db, request.user_id, "challenge_complete")
    db.commit()
    notify_users(db, [challenge.issuer_id], "Challenge Completed!", f"{db.query(User).get(request.user_id).username} finished your challenge!")
    return {"challenge_id": challenge_id, "theta": theta, "gamification": gamification}
```

**`api/routes/users.py` (Updated for FCM Token)**

```python
@router.post("/update-fcm")
async def update_fcm_token(user_id: str, fcm_token: str, db: Session = Depends(get_db)):
    user = db.query(User).filter(User.user_id == user_id).first()
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    user.fcm_token = fcm_token
    db.commit()
    return {"user_id": user_id, "fcm_token": fcm_token}
```

***

#### Step 3: Flutter Mobile App Implementation

**Project Structure**

```
sat_prep_app/
├── lib/
│   ├── main.dart
│   ├── models/
│   │   ├── user.dart
│   │   ├── question.dart
│   │   ├── response.dart
│   ├── screens/
│   │   ├── dashboard.dart
│   │   ├── practice.dart
│   │   ├── community.dart
│   │   ├── notifications.dart
│   ├── services/
│   │   ├── api_service.dart
│   │   ├── db_service.dart
│   │   ├── notification_service.dart
│   ├── widgets/
│   │   ├── question_card.dart
│   │   ├── group_card.dart
```

**`pubspec.yaml`**

```yaml
name: sat_prep_app
dependencies:
  flutter:
    sdk: flutter
  http: ^0.13.5
  sqflite: ^2.0.0+4
  path_provider: ^2.0.11
  firebase_core: ^1.24.0
  firebase_messaging: ^13.0.4
  charts_flutter: ^0.12.0
```

**`lib/main.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'package:firebase_messaging/firebase_messaging.dart';
import 'screens/dashboard.dart';
import 'services/notification_service.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();
  runApp(SATPrepApp());
}

class SATPrepApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    NotificationService().init();
    return MaterialApp(
      title: 'SAT Prep',
      theme: ThemeData(primarySwatch: Colors.blue),
      home: DashboardScreen(),
    );
  }
}
```

**`lib/models/question.dart`**

```dart
class Question {
  final String id;
  final String test;
  final String domain;
  final String skill;
  final String difficulty;
  final Map<String, dynamic> content;

  Question({required this.id, required this.test, required this.domain, required this.skill, required this.difficulty, required this.content});

  factory Question.fromJson(Map<String, dynamic> json) {
    return Question(
      id: json['metadata']['Question ID'],
      test: json['metadata']['Test'],
      domain: json['metadata']['Domain'],
      skill: json['metadata']['Skill'],
      difficulty: json['metadata']['Difficulty'],
      content: json['content'],
    );
  }

  Map<String, dynamic> toJson() => {
    'metadata': {'Question ID': id, 'Test': test, 'Domain': domain, 'Skill': skill, 'Difficulty': difficulty},
    'content': content
  };
}
```

**`lib/services/api_service.dart`**

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;

class ApiService {
  static const String baseUrl = 'http://your-api-url.com'; // Replace with actual URL

  Future<Map<String, dynamic>> getProgress(String userId) async {
    final response = await http.get(Uri.parse('$baseUrl/progress/$userId'));
    return jsonDecode(response.body);
  }

  Future<Map<String, dynamic>> startPractice(String userId, String section, String domain, String skill, int questions) async {
    final response = await http.post(
      Uri.parse('$baseUrl/practice/start'),
      body: jsonEncode({'user_id': userId, 'section': section, 'domain': domain, 'skill': skill, 'questions': questions}),
      headers: {'Content-Type': 'application/json'},
    );
    return jsonDecode(response.body);
  }

  Future<void> submitPractice(String userId, String practiceId, List<Map<String, dynamic>> responses) async {
    await http.post(
      Uri.parse('$baseUrl/practice/submit'),
      body: jsonEncode({'user_id': userId, 'practice_id': practiceId, 'responses': responses}),
      headers: {'Content-Type': 'application/json'},
    );
  }

  // Add similar methods for community, challenges, etc.
}
```

**`lib/services/db_service.dart` (Offline Sync)**

```dart
import 'package:sqflite/sqflite.dart';
import 'package:path/path.dart';
import '../models/question.dart';

class DBService {
  static Database? _database;

  Future<Database> get database async {
    if (_database != null) return _database!;
    _database = await _initDB();
    return _database!;
  }

  Future<Database> _initDB() async {
    String path = join(await getDatabasesPath(), 'sat_prep.db');
    return await openDatabase(path, version: 1, onCreate: (db, version) async {
      await db.execute('CREATE TABLE practice (id TEXT PRIMARY KEY, user_id TEXT, questions TEXT, responses TEXT, synced INTEGER)');
      await db.execute('CREATE TABLE assignments (id TEXT PRIMARY KEY, user_id TEXT, content TEXT, synced INTEGER)');
    });
  }

  Future<void> cachePractice(String id, String userId, List<Question> questions) async {
    final db = await database;
    await db.insert('practice', {
      'id': id,
      'user_id': userId,
      'questions': jsonEncode(questions.map((q) => q.toJson()).toList()),
      'responses': jsonEncode([]),
      'synced': 0
    }, conflictAlgorithm: ConflictAlgorithm.replace);
  }

  Future<List<Map<String, dynamic>>> getOfflinePractice(String userId) async {
    final db = await database;
    return await db.query('practice', where: 'user_id = ? AND synced = 0', whereArgs: [userId]);
  }

  Future<void> saveResponses(String practiceId, List<Map<String, dynamic>> responses) async {
    final db = await database;
    await db.update('practice', {'responses': jsonEncode(responses), 'synced': 0}, where: 'id = ?', whereArgs: [practiceId]);
  }

  Future<void> syncResponses(String userId) async {
    final db = await database;
    final offline = await getOfflinePractice(userId);
    for (var item in offline) {
      await ApiService().submitPractice(userId, item['id'], jsonDecode(item['responses']));
      await db.update('practice', {'synced': 1}, where: 'id = ?', whereArgs: [item['id']]);
    }
  }
}
```

**`lib/services/notification_service.dart`**

```dart
import 'package:firebase_messaging/firebase_messaging.dart';

class NotificationService {
  final FirebaseMessaging _fcm = FirebaseMessaging.instance;

  Future<void> init() async {
    await _fcm.requestPermission();
    String? token = await _fcm.getToken();
    if (token != null) {
      await ApiService().updateFcmToken('user123', token); // Replace with actual user_id
    }

    FirebaseMessaging.onMessage.listen((RemoteMessage message) {
      print('Foreground message: ${message.notification?.title} - ${message.notification?.body}');
    });

    FirebaseMessaging.onBackgroundMessage(_backgroundHandler);
  }

  static Future<void> _backgroundHandler(RemoteMessage message) async {
    print('Background message: ${message.notification?.title}');
  }

  Future<void> scheduleDailyReminder(String userId) async {
    // Use flutter_local_notifications for scheduled reminders if needed
    notifyUsers(userId); // Trigger via backend
  }
}
```

**`lib/screens/dashboard.dart`**

```dart
import 'package:flutter/material.dart';
import '../services/api_service.dart';
import '../screens/practice.dart';
import '../screens/community.dart';

class DashboardScreen extends StatelessWidget {
  final ApiService api = ApiService();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('SAT Prep')),
      body: FutureBuilder(
        future: api.getProgress('user123'), // Replace with actual user_id
        builder: (context, snapshot) {
          if (!snapshot.hasData) return Center(child: CircularProgressIndicator());
          var data = snapshot.data as Map<String, dynamic>;
          return ListView(
            children: [
              Text('Points: ${data['gamification']?['points'] ?? 0}', style: TextStyle(fontSize: 20)),
              Text('Streak: ${data['gamification']?['streak'] ?? 0} days'),
              ElevatedButton(
                onPressed: () => Navigator.push(context, MaterialPageRoute(builder: (_) => PracticeScreen())),
                child: Text('Start Practice'),
              ),
              ElevatedButton(
                onPressed: () => Navigator.push(context, MaterialPageRoute(builder: (_) => CommunityScreen())),
                child: Text('Community'),
              ),
            ],
          );
        },
      ),
    );
  }
}
```

**`lib/screens/practice.dart`**

```dart
import 'package:flutter/material.dart';
import '../services/api_service.dart';
import '../services/db_service.dart';
import '../models/question.dart';
import '../widgets/question_card.dart';

class PracticeScreen extends StatefulWidget {
  @override
  _PracticeScreenState createState() => _PracticeScreenState();
}

class _PracticeScreenState extends State<PracticeScreen> {
  final ApiService api = ApiService();
  final DBService db = DBService();
  List<Question> questions = [];
  List<Map<String, dynamic>> responses = [];

  @override
  void initState() {
    super.initState();
    _loadPractice();
  }

  Future<void> _loadPractice() async {
    try {
      var data = await api.startPractice('user123', 'Math', 'Algebra', 'Linear Functions', 5);
      questions = (data['questions'] as List).map((q) => Question.fromJson(q)).toList();
      await db.cachePractice(data['practice_id'], 'user123', questions);
    } catch (e) {
      var offline = await db.getOfflinePractice('user123');
      if (offline.isNotEmpty) {
        questions = (jsonDecode(offline.first['questions']) as List).map((q) => Question.fromJson(q)).toList();
      }
    }
    setState(() {});
  }

  void _submitResponse(String questionId, String answer, bool isCorrect, double timeSpent) {
    responses.add({'question_id': questionId, 'answer': answer, 'is_correct': isCorrect, 'time_spent': timeSpent});
    if (responses.length == questions.length) {
      db.saveResponses('prac_<id>', responses); // Replace with actual practice_id
      db.syncResponses('user123');
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Practice')),
      body: questions.isEmpty
          ? Center(child: CircularProgressIndicator())
          : ListView.builder(
              itemCount: questions.length,
              itemBuilder: (context, index) => QuestionCard(
                question: questions[index],
                onSubmit: _submitResponse,
              ),
            ),
    );
  }
}
```

**`lib/screens/community.dart`**

```dart
import 'package:flutter/material.dart';
import '../services/api_service.dart';

class CommunityScreen extends StatelessWidget {
  final ApiService api = ApiService();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Community')),
      body: Column(
        children: [
          ElevatedButton(
            onPressed: () async {
              await api.post(Uri.parse('$baseUrl/community/group/create'), body: jsonEncode({'user_id': 'user123', 'name': 'Math Study Group'}));
              ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text('Group Created')));
            },
            child: Text('Create Study Group'),
          ),
          ElevatedButton(
            onPressed: () async {
              await api.post(Uri.parse('$baseUrl/community/questions'), body: jsonEncode({'user_id': 'user123', 'question': 'How to solve quadratic equations?'}));
              ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text('Question Posted')));
            },
            child: Text('Post Question'),
          ),
          ElevatedButton(
            onPressed: () async {
              await api.post(Uri.parse('$baseUrl/community/challenge'), body: jsonEncode({'issuer_id': 'user123', 'opponent_id': 'user456', 'section': 'Math', 'domain': 'Algebra', 'num_questions': 5}));
              ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text('Challenge Issued')));
            },
            child: Text('Issue Challenge'),
          ),
        ],
      ),
    );
  }
}
```

**`lib/widgets/question_card.dart`**

```dart
import 'package:flutter/material.dart';

class QuestionCard extends StatefulWidget {
  final Question question;
  final Function(String, String, bool, double) onSubmit;

  QuestionCard({required this.question, required this.onSubmit});

  @override
  _QuestionCardState createState() => _QuestionCardState();
}

class _QuestionCardState extends State<QuestionCard> {
  String? selectedAnswer;
  Stopwatch stopwatch = Stopwatch();

  @override
  void initState() {
    super.initState();
    stopwatch.start();
  }

  @override
  Widget build(BuildContext context) {
    return Card(
      child: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            Text(widget.question.content['text']),
            if (widget.question.content.containsKey('options'))
              ...widget.question.content['options'].map((opt) => RadioListTile(
                    title: Text(opt),
                    value: opt,
                    groupValue: selectedAnswer,
                    onChanged: (value) => setState(() => selectedAnswer = value),
                  )).toList()
            else
              TextField(
                onChanged: (value) => selectedAnswer = value,
                decoration: InputDecoration(labelText: 'Your Answer'),
              ),
            ElevatedButton(
              onPressed: () {
                stopwatch.stop();
                bool isCorrect = selectedAnswer == widget.question.content['correct_answer'];
                widget.onSubmit(widget.question.id, selectedAnswer ?? '', isCorrect, stopwatch.elapsedMilliseconds / 1000.0);
              },
              child: Text('Submit'),
            ),
          ],
        ),
      ),
    );
  }
}
```

***

#### Step 4: HOOK Model Integration

* **Trigger**: Push notifications (“Practice Algebra today!”), social prompts (“Join a group!”).
* **Action**: Start practice, post in Q\&A, accept challenges via mobile UI.
* **Variable Reward**: Points from practice, random group bonuses (e.g., 20% chance of 50 extra points), challenge wins.
* **Investment**: Join groups, build streaks, contribute to community.

***

#### Step 5: Testing

* **Offline Mode**: Cache practice → Disconnect → Submit responses → Reconnect → Verify sync.
* **Notifications**: Register FCM token → Trigger `/community/group/create` → Check notification.
* **Social**: Create group → Join → Post question → Challenge friend → Complete → Confirm points/theta.

***

#### Conclusion

This **Flutter Mobile App** with **Offline Mode** and **Social Features**:

* **Mobile Frontend**: Mirrors web UI with practice, community screens.
* **Offline Sync**: Caches and syncs via SQLite.
* **Push Notifications**: Engages users with Firebase.
* **Social**: Adds groups, Q\&A, challenges with HOOK-driven motivation.

As of March 26, 2025, the app now supports on-the-go prep with peer interaction, enhancing its utility for thousands of users. Next steps could include UI polish or additional features—let me know your priority!
