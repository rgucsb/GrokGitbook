# Authentication

To add authentication to the **SAT Prep Suite**, I’ll integrate a simple yet secure authentication system using **JSON Web Tokens (JWT)**. This will ensure that both the mobile app (Flutter) and web app (Flutter Web) can securely interact with the backend (FastAPI), replacing hardcoded `user_id` values with authenticated user identities. The system will include user registration, login, and token-based authorization, applied across all API endpoints and frontend screens. Below, I’ll update the existing code package with authentication logic as of March 26, 2025.

***

## SAT Prep Suite: Authentication Integration Document

### Overview

* **Authentication Method**: JWT with username/password.
* **Components**:
  * Backend: User model updates, auth endpoints, token validation.
  * Frontend: Login screen, token storage, API call updates.
* **Flow**: Users register → Login → Receive JWT → Include token in API requests → Backend verifies token.

***

### Backend Updates

#### 1. `api/requirements.txt` (Updated)

```plaintext
fastapi==0.95.0
uvicorn==0.21.1
sqlalchemy==1.4.47
redis==4.5.4
requests==2.28.2
boto3==1.26.100
pytest==7.2.2
transformers==4.28.1
torch==2.0.0
python-jose[cryptography]==3.3.0  # For JWT
passlib[bcrypt]==1.7.4          # For password hashing
```

***

#### 2. `api/models.py` (Updated)

**Purpose**: Add password hash to `User` and adjust relationships.

```python
from sqlalchemy import Column, Integer, String, JSON, DateTime, ForeignKey, Boolean
from sqlalchemy.ext.declarative import declarative_base
from datetime import datetime

Base = declarative_base()

class User(Base):
    __tablename__ = "users"
    user_id = Column(String, primary_key=True)
    name = Column(String, nullable=False)
    email = Column(String, nullable=False, unique=True)
    password_hash = Column(String, nullable=False)  # Store hashed password
    tutor_id = Column(String, ForeignKey("tutors.tutor_id"), nullable=True)

# Other models (Tutor, Question, etc.) remain unchanged
```

***

#### 3. `api/utils.py` (Updated)

**Purpose**: Add JWT creation and verification functions.

```python
from sqlalchemy.orm import Session
from typing import Dict, List
import json
import redis
import requests
from transformers import AutoModelForCausalLM, AutoTokenizer
import torch
from jose import JWTError, jwt
from passlib.context import CryptContext
from datetime import datetime, timedelta
from fastapi import HTTPException, status

redis_client = redis.Redis(host='localhost', port=6379, db=0)

# JWT settings
SECRET_KEY = "your-secret-key"  # Replace with secure key in production
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

# Password hashing
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

# Load custom LLM
try:
    model = AutoModelForCausalLM.from_pretrained("./sat_llm_final")
    tokenizer = AutoTokenizer.from_pretrained("./sat_llm_final")
    USE_CUSTOM_LLM = True
except Exception:
    USE_CUSTOM_LLM = False
    print("Custom LLM not available, falling back to external API")

def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)

def get_password_hash(password: str) -> str:
    return pwd_context.hash(password)

def create_access_token(data: Dict, expires_delta: timedelta = None) -> str:
    to_encode = data.copy()
    expire = datetime.utcnow() + (expires_delta or timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES))
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

def decode_access_token(token: str) -> Dict:
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        return payload
    except JWTError:
        raise HTTPException(status_code=status.HTTP_401_UNAUTHORIZED, detail="Invalid token")

def get_current_user(token: str, db: Session) -> User:
    payload = decode_access_token(token)
    user_id = payload.get("sub")
    user = db.query(User).filter(User.user_id == user_id).first()
    if not user:
        raise HTTPException(status_code=status.HTTP_401_UNAUTHORIZED, detail="User not found")
    return user

# Existing functions (log_tutor_feedback, generate_feedback_with_custom_llm, etc.) unchanged
```

***

#### 4. `api/routes/auth.py` (New)

**Purpose**: Handle registration and login.

```python
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.orm import Session
from api.database import get_db
from api.models import User
from api.utils import get_password_hash, verify_password, create_access_token
from pydantic import BaseModel

router = APIRouter(prefix="/auth", tags=["auth"])

class UserCreate(BaseModel):
    user_id: str
    name: str
    email: str
    password: str

class UserLogin(BaseModel):
    email: str
    password: str

@router.post("/register")
async def register_user(user: UserCreate, db: Session = Depends(get_db)):
    if db.query(User).filter(User.email == user.email).first():
        raise HTTPException(status_code=400, detail="Email already registered")
    hashed_password = get_password_hash(user.password)
    db_user = User(user_id=user.user_id, name=user.name, email=user.email, password_hash=hashed_password)
    db.add(db_user)
    db.commit()
    access_token = create_access_token({"sub": user.user_id})
    return {"access_token": access_token, "token_type": "bearer"}

@router.post("/login")
async def login_user(user: UserLogin, db: Session = Depends(get_db)):
    db_user = db.query(User).filter(User.email == user.email).first()
    if not db_user or not verify_password(user.password, db_user.password_hash):
        raise HTTPException(status_code=401, detail="Invalid credentials")
    access_token = create_access_token({"sub": db_user.user_id})
    return {"access_token": access_token, "token_type": "bearer"}
```

***

#### 5. `api/main.py` (Updated)

**Purpose**: Add auth router and dependency.

```python
from fastapi import FastAPI, Depends
from api.routes import practice_module, study_plan, review, community, tutor, test
from api.routes.auth import router as auth_router
from api.utils import get_current_user
from api.database import get_db
from sqlalchemy.orm import Session

app = FastAPI(title="SAT Prep Suite API")

app.include_router(auth_router)
app.include_router(practice_module.router)
app.include_router(study_plan.router)
app.include_router(review.router)
app.include_router(community.router)
app.include_router(tutor.router)
app.include_router(test.router)

@app.get("/")
def read_root():
    return {"message": "Welcome to SAT Prep Suite API"}

# Dependency for protected routes
def get_current_user_dependency(token: str = Depends(oauth2_scheme), db: Session = Depends(get_db)):
    return get_current_user(token, db)
```

***

#### 6. Protect Existing Routes (Example Updates)

**Purpose**: Add authentication to all endpoints.

**`practice_module.py` (Updated)**

```python
from fastapi import APIRouter, Depends
from sqlalchemy.orm import Session
from api.database import get_db
from api.models import Response, FeedbackRating, User
from api.utils import generate_feedback_with_custom_llm, get_current_user
from typing import List, Dict

router = APIRouter(prefix="/practice", tags=["practice"])

@router.post("/start/{user_id}")
async def start_practice(user_id: str, current_user: User = Depends(get_current_user_dependency), db: Session = Depends(get_db)):
    if current_user.user_id != user_id:
        raise HTTPException(status_code=403, detail="Not authorized")
    questions = db.query(Question).filter(Question.test == "practice").limit(10).all()
    return {"user_id": user_id, "practice_id": f"prac_{user_id}_{time.time()}", "questions": [{"id": q.question_id, **q.content} for q in questions]}

@router.post("/submit/{practice_id}")
async def submit_practice(practice_id: str, responses: List[Dict], current_user: User = Depends(get_current_user_dependency), db: Session = Depends(get_db)):
    user_id = practice_id.split("_")[1]
    if current_user.user_id != user_id:
        raise HTTPException(status_code=403, detail="Not authorized")
    feedback = []
    for r in responses:
        resp = Response(user_id=user_id, question_id=r["question_id"], answer=r["answer"], is_correct=r.get("is_correct", False), time_spent=r.get("time_spent", 0))
        db.add(resp)
        fb = generate_feedback_with_custom_llm({"question_id": r["question_id"], "answer": r["answer"]}, {})
        feedback.append({"question_id": r["question_id"], "feedback": fb})
        db.add(FeedbackRating(user_id=user_id, feedback_text=fb, rating=0, performance_data={"question_id": r["question_id"], "is_correct": r["is_correct"]}))
    db.commit()
    return {"practice_id": practice_id, "feedback": feedback}
```

* **Note**: Apply similar `Depends(get_current_user_dependency)` and `user_id` checks to `study_plan.py`, `review.py`, `community.py`, `tutor.py`, `test.py`.

***

### Frontend Updates (Flutter)

#### 1. `mobile/pubspec.yaml` (Updated)

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
  shared_preferences: ^2.0.15  # For token storage
```

***

#### 2. `mobile/lib/services/api_service.dart` (Updated)

**Purpose**: Add token to API calls.

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;
import 'package:shared_preferences/shared_preferences.dart';

class ApiService {
  static const String baseUrl = 'http://localhost:8000';
  String? _token;

  Future<void> setToken(String token) async {
    _token = token;
    final prefs = await SharedPreferences.getInstance();
    await prefs.setString('access_token', token);
  }

  Future<String?> getToken() async {
    if (_token != null) return _token;
    final prefs = await SharedPreferences.getInstance();
    _token = prefs.getString('access_token');
    return _token;
  }

  Future<Map<String, dynamic>> _authenticatedRequest(String method, String endpoint, {Map<String, dynamic>? body}) async {
    final token = await getToken();
    if (token == null) throw Exception('Not authenticated');
    final headers = {'Authorization': 'Bearer $token', 'Content-Type': 'application/json'};
    final uri = Uri.parse('$baseUrl$endpoint');
    http.Response response;
    if (method == 'GET') {
      response = await http.get(uri, headers: headers);
    } else {
      response = await http.post(uri, headers: headers, body: jsonEncode(body));
    }
    if (response.statusCode == 401) throw Exception('Unauthorized');
    return jsonDecode(response.body);
  }

  Future<Map<String, dynamic>> register(String userId, String name, String email, String password) async {
    final response = await http.post(Uri.parse('$baseUrl/auth/register'), body: jsonEncode({
      'user_id': userId, 'name': name, 'email': email, 'password': password
    }));
    final data = jsonDecode(response.body);
    await setToken(data['access_token']);
    return data;
  }

  Future<Map<String, dynamic>> login(String email, String password) async {
    final response = await http.post(Uri.parse('$baseUrl/auth/login'), body: jsonEncode({
      'email': email, 'password': password
    }));
    final data = jsonDecode(response.body);
    await setToken(data['access_token']);
    return data;
  }

  Future<Map<String, dynamic>> startPractice(String userId) async {
    return _authenticatedRequest('POST', '/practice/start/$userId');
  }

  Future<Map<String, dynamic>> submitPractice(String practiceId, List<Map<String, dynamic>> responses) async {
    return _authenticatedRequest('POST', '/practice/submit/$practiceId', body: responses);
  }

  Future<Map<String, dynamic>> startTest(String userId, String testType) async {
    return _authenticatedRequest('POST', '/test/start/$userId?test_type=$testType');
  }

  Future<Map<String, dynamic>> submitTest(String testId, List<Map<String, dynamic>> responses) async {
    return _authenticatedRequest('POST', '/test/submit/$testId', body: responses);
  }

  Future<Map<String, dynamic>> getTutorStudentProgress(String tutorId) async {
    return _authenticatedRequest('GET', '/tutor/progress/$tutorId');
  }

  Future<void> submitTutorFeedback(String tutorId, String userId, String feedbackText, String contextType, {Map<String, dynamic>? performanceData}) async {
    await _authenticatedRequest('POST', '/tutor/feedback', body: {
      'tutor_id': tutorId, 'user_id': userId, 'feedback_text': feedbackText, 'context_type': contextType, 'performance_data': performanceData
    });
  }

  Future<Map<String, dynamic>> createStudyPlan(String userId, String testDate) async {
    return _authenticatedRequest('POST', '/study_plan/create/$userId?test_date=$testDate');
  }

  Future<Map<String, dynamic>> logStudyPlanAction(int planId, Map<String, dynamic> actionData) async {
    return _authenticatedRequest('POST', '/study_plan/action/$planId', body: actionData);
  }

  Future<Map<String, dynamic>> getSkillPredictions(String userId) async {
    return _authenticatedRequest('GET', '/review/skills/$userId');
  }

  Future<Map<String, dynamic>> postCommunityNote(String userId, Map<String, dynamic> noteData) async {
    return _authenticatedRequest('POST', '/community/note/$userId', body: noteData);
  }

  Future<Map<String, dynamic>> getUserNotes(String userId) async {
    return _authenticatedRequest('GET', '/community/notes/$userId');
  }
}
```

***

#### 3. `mobile/lib/screens/login.dart` (New)

**Purpose**: Handle user login/registration.

```dart
import 'package:flutter/material.dart';
import '../services/api_service.dart';

class LoginScreen extends StatefulWidget {
  @override
  _LoginScreenState createState() => _LoginScreenState();
}

class _LoginScreenState extends State<LoginScreen> {
  final ApiService api = ApiService();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  final _userIdController = TextEditingController();
  final _nameController = TextEditingController();
  bool _isLogin = true;

  void _submit() {
    if (_isLogin) {
      api.login(_emailController.text, _passwordController.text).then((_) {
        Navigator.pushReplacementNamed(context, '/dashboard');
      }).catchError((e) => ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text('Login failed: $e'))));
    } else {
      api.register(_userIdController.text, _nameController.text, _emailController.text, _passwordController.text).then((_) {
        Navigator.pushReplacementNamed(context, '/dashboard');
      }).catchError((e) => ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text('Registration failed: $e'))));
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(_isLogin ? 'Login' : 'Register')),
      body: Padding(
        padding: EdgeInsets.all(16.0),
        child: Column(
          children: [
            if (!_isLogin) ...[
              TextField(controller: _userIdController, decoration: InputDecoration(labelText: 'User ID')),
              TextField(controller: _nameController, decoration: InputDecoration(labelText: 'Name')),
            ],
            TextField(controller: _emailController, decoration: InputDecoration(labelText: 'Email')),
            TextField(controller: _passwordController, decoration: InputDecoration(labelText: 'Password'), obscureText: true),
            SizedBox(height: 20),
            ElevatedButton(onPressed: _submit, child: Text(_isLogin ? 'Login' : 'Register')),
            TextButton(
              onPressed: () => setState(() => _isLogin = !_isLogin),
              child: Text(_isLogin ? 'Need an account? Register' : 'Have an account? Login'),
            ),
          ],
        ),
      ),
    );
  }
}
```

***

#### 4. `mobile/lib/main.dart` (Updated)

**Purpose**: Add login as initial route.

```dart
import 'package:flutter/material.dart';
import 'screens/dashboard.dart';
import 'screens/practice.dart';
import 'screens/study_plan.dart';
import 'screens/community.dart';
import 'screens/tutor_dashboard.dart';
import 'screens/test.dart';
import 'screens/results.dart';
import 'screens/login.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'SAT Prep Suite',
      initialRoute: '/login',
      routes: {
        '/login': (_) => LoginScreen(),
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

***

#### 5. Update Existing Screens

**Purpose**: Remove hardcoded `user_id`, use token-based auth.

**`practice.dart` (Updated)**

```dart
class _PracticeScreenState extends State<PracticeScreen> {
  final ApiService api = ApiService();
  Map<String, dynamic>? session;
  String userId = ''; // Fetch from token

  @override
  void initState() {
    super.initState();
    api.getToken().then((token) {
      if (token != null) {
        api.decode_access_token(token).then((payload) => setState(() => userId = payload['sub']));
        api.startPractice(userId).then((data) => setState(() => session = data));
      }
    });
  }
  // Rest unchanged
}
```

* **Note**: Apply similar updates to `study_plan.dart`, `community.dart`, `test.dart`, `tutor_dashboard.dart`, fetching `userId` from token payload (`sub`).

***

### Integration Check

* **Backend**: All routes now require JWT via `Depends(get_current_user_dependency)`, validated against `users` table.
* **Frontend**:
  * Mobile/Web share `api_service.dart` → Token stored in `SharedPreferences`, added to headers.
  * Login screen → Sets token, navigates to dashboard.
  * Screens → Fetch `user_id` from token, ensuring consistency.
* **Data Flow**: Token persists across mobile/web sessions, ensuring seamless user experience.

### Next Steps

1. **Add to Package**: Include new/updated files.
2.  **Seed Data**: Update `seed_data.py` with hashed passwords:

    ```python
    db.add(User(user_id="user123", name="Alice", email="alice@example.com", password_hash=get_password_hash("password123"), tutor_id="tutor123"))
    ```
3. **Test**: Register/login via `/login`, verify protected endpoints work with token.
4. **Security**: Replace `SECRET_KEY` with a secure value in production (e.g., environment variable).

This adds robust authentication—any specific tweaks needed? Let me know!
