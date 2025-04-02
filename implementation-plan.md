---
coverY: 0
layout:
  cover:
    visible: false
    size: full
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Implementation Plan

Since we’ve been building the **SAT Prep Suite** incrementally through our discussions, and nothing has been tested yet, I’ll create a comprehensive, step-by-step plan to **implement, deploy, and test** the app based on all the components we’ve designed so far. This includes the backend (FastAPI with SQLite), mobile app (Flutter), and data collection features (student and tutor feedback) to train our small LLM. The plan assumes we’re starting from scratch with only the code provided in our conversations, aiming for a functional deployment and thorough testing by March 26, 2025. Here’s the detailed roadmap:

***

#### Step 1: Prerequisites and Setup

**Goal**: Establish the development environment and infrastructure.

1. **Install Dependencies**
   *   **Backend**:

       ```bash
       pip install fastapi uvicorn sqlalchemy sqlite redis requests boto3 scipy transformers torch
       ```
   * **Frontend (Flutter)**:
     * Install Flutter SDK (v3.0+): [Flutter Installation Guide](https://flutter.dev/docs/get-started/install)
     * Verify setup: `flutter doctor`
     *   Add dependencies to `pubspec.yaml`:

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
     * Run: `flutter pub get`
2. **Set Up Environment**
   *   Create `.env` file in backend root:

       ```plaintext
       REDIS_HOST=localhost
       REDIS_PORT=6379
       AWS_ACCESS_KEY=your_aws_key  # For Polly
       AWS_SECRET_KEY=your_aws_secret
       GOOGLE_API_KEY=your_google_key  # For Speech-to-Text
       ```
   * Install Redis locally: `sudo apt-get install redis-server` (Linux) or equivalent.
3.  **Directory Structure**

    ```
    sat_prep_suite/
    ├── api/                # Backend
    │   ├── main.py
    │   ├── models.py
    │   ├── utils.py
    │   ├── routes/
    │   │   ├── practice_module.py
    │   │   ├── study_plan.py
    │   │   ├── review.py
    │   │   ├── community.py
    │   │   └── tutor.py
    │   ├── database.py
    │   └── migrations/
    │       └── add_tutor_tables.py
    ├── mobile/             # Flutter app
    │   ├── lib/
    │   │   ├── main.dart
    │   │   ├── screens/
    │   │   │   ├── dashboard.dart
    │   │   │   ├── practice.dart
    │   │   │   ├── study_plan.dart
    │   │   │   ├── community.dart
    │   │   │   └── tutor_dashboard.dart
    │   │   └── services/
    │   │       └── api_service.dart
    │   └── pubspec.yaml
    ├── sat_data.jsonl      # Synthetic/real data for LLM
    └── README.md
    ```

***

#### Step 2: Implement Core Components

**Goal**: Build and integrate all backend and frontend pieces.

**Backend (FastAPI)**

1. **Database Initialization**
   *   Create `database.py`:

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
   * Run migration: `python api/migrations/add_tutor_tables.py`
2. **Core Routes**
   * Copy `main.py`, `models.py`, `utils.py`, and all `routes/*.py` from previous responses.
   *   Add `USERS` table to `models.py` (simplified):

       ```python
       class User(Base):
           __tablename__ = "users"
           user_id = Column(String, primary_key=True)
           name = Column(String, nullable=False)
           email = Column(String, nullable=False)
           tutor_id = Column(String, ForeignKey("tutors.tutor_id"), nullable=True)
       ```
3. **Seed Data**
   *   Create `seed_data.py`:

       ```python
       from sqlalchemy.orm import Session
       from api.models import Tutor, User
       from api.database import get_db

       def seed(db: Session):
           tutors = [
               Tutor(tutor_id="tutor123", name="Jane Doe", email="jane@example.com"),
               Tutor(tutor_id="tutor456", name="John Smith", email="john@example.com"),
           ]
           users = [
               User(user_id="user123", name="Alice", email="alice@example.com", tutor_id="tutor123"),
               User(user_id="user456", name="Bob", email="bob@example.com", tutor_id="tutor456"),
           ]
           db.add_all(tutors + users)
           db.commit()

       db = next(get_db())
       seed(db)
       ```
   * Run: `python seed_data.py`

**Frontend (Flutter)**

1. **API Service**
   * Copy `api_service.dart` from previous responses into `mobile/lib/services/`.
2. **Screens**
   * Copy `main.dart`, `dashboard.dart`, `practice.dart`, `study_plan.dart`, `community.dart`, and `tutor_dashboard.dart` into `mobile/lib/screens/`.
   * Ensure all routes are wired correctly in `main.dart`.
3. **Base URL**
   *   Update `api_service.dart` with a local/test base URL:

       ```dart
       const String baseUrl = 'http://localhost:8000';  # Adjust for deployment
       ```

***

#### Step 3: Deploy Locally

**Goal**: Run the app on a local machine for initial testing.

1. **Backend**
   * Start Redis: `redis-server`
   * Run FastAPI: `uvicorn api.main:app --reload --port 8000`
   * Verify: Open `http://localhost:8000/docs` to see Swagger UI.
2. **Frontend**
   * Run Flutter emulator: `flutter emulators --launch <emulator_id>`
   * Start app: `cd mobile && flutter run`
   * Verify: App launches on emulator with Dashboard screen.
3. **Synthetic Data**
   *   Generate 50K pairs (from previous plan):

       ```python
       # synthetic_data.py (simplified from earlier)
       import json, random
       skills = ["Linear Functions", "Triangles"]
       with open("sat_data.jsonl", "w") as f:
           for _ in range(50000):
               skill = random.choice(skills)
               correct = random.randint(0, 10)
               input_data = {"Math": {"Algebra": {skill: {"correct": correct, "total": 10}}}}
               output = f"You got {correct}/10 in {skill}—{'great' if correct > 7 else 'keep practicing'}!"
               f.write(json.dumps({"input": json.dumps(input_data), "output": output}) + "\n")
       ```
   * Run: `python synthetic_data.py`

***

#### Step 4: Test Core Functionality

**Goal**: Validate each feature works as expected locally.

1. **Setup Test Users**
   * Ensure `user123` (student) and `tutor123` (tutor) are in DB from seed.
2. **Test Plan**
   * **Dashboard**:
     * Navigate to `/dashboard` → Verify buttons for Practice, Study Plan, Community, Tutor.
     * Expected: All routes accessible.
   * **Practice**:
     * Start practice → Submit answer → Rate feedback (1-5 stars) → Ask for help.
     * Check `RESPONSES`, `FEEDBACK_RATINGS`, `HELP_REQUESTS` tables.
     * Expected: Entries logged (e.g., 1 feedback rating, 1 help request).
   * **Study Plan**:
     * View plan → Mark task as completed/skipped.
     * Check `STUDY_PLAN_ACTIONS` table.
     * Expected: Action logged with performance update.
   * **Community**:
     * Add note → Submit.
     * Check `USER_NOTES` table.
     * Expected: Note and AI response logged.
   * **Tutor Dashboard**:
     * Navigate to `/tutor` → View student progress → Submit feedback (e.g., “Great job on Algebra!”).
     * Check `TUTOR_FEEDBACK` table.
     * Expected: Feedback entry with `tutor_id=tutor123`, `user_id=user123`.
   * **Data Export**:
     * Run `export_llm_training_data` → Check `real_sat_data.jsonl`.
     * Expected: \~5-10 pairs from test interactions.
3. **Fix Issues**
   * Debug errors (e.g., missing DB tables, API 404s) → Adjust code as needed.

***

#### Step 5: Deploy to Production

**Goal**: Make the app publicly accessible.

1. **Backend (AWS EC2)**
   * Launch EC2 instance (e.g., t3.micro, Ubuntu 20.04):
     * Install dependencies: Python, Redis, SQLite.
     * Copy `api/` folder: `scp -r api/ ec2-user@<ec2-ip>:~/`.
     * Install Nginx: `sudo apt-get install nginx`.
     *   Configure Nginx:

         ```
         server {
             listen 80;
             server_name <your-domain>;
             location / {
                 proxy_pass http://localhost:8000;
                 proxy_set_header Host $host;
             }
         }
         ```
     * Run FastAPI: `uvicorn api.main:app --host 0.0.0.0 --port 8000 &`
     * Verify: `http://<ec2-ip>/docs`
2. **Frontend (Firebase Hosting)**
   * Build Flutter web: `cd mobile && flutter build web`
   * Initialize Firebase: `firebase init hosting`
   * Deploy: `firebase deploy`
   * Verify: Access app at Firebase URL (e.g., `https://sat-prep-suite.web.app`).
3. **Update API URL**
   * Edit `api_service.dart`: `const String baseUrl = 'http://<ec2-ip>';`
   * Rebuild and redeploy Flutter app.

***

#### Step 6: Test in Production

**Goal**: Ensure end-to-end functionality with real users.

1. **Test Users**
   * Register 2 test users (student, tutor) via app or DB insert.
2. **End-to-End Tests**
   * **Student Flow**:
     * Login as `user123` → Complete practice → Rate feedback → Ask help → Mark plan task → Add note.
     * Expected: All actions logged in respective tables.
   * **Tutor Flow**:
     * Login as `tutor123` → View `user123` progress → Submit feedback.
     * Expected: `TUTOR_FEEDBACK` entry created.
   * **Data Collection**:
     * Export data → Verify `real_sat_data.jsonl` has \~10-20 pairs.
   * **Performance**:
     * Test with 5 simultaneous users → Check latency (<2s per request).
3. **Bug Fixes**
   * Monitor logs (e.g., `tail -f /var/log/nginx/error.log`) → Fix 500s, timeouts.

***

#### Step 7: Train Initial LLM

**Goal**: Use synthetic + early real data to train the LLM.

1. **Setup**
   * Install Transformers: `pip install transformers datasets`
   * Rent GPU (e.g., AWS EC2 g4dn.xlarge, \~$0.50/hr).
2. **Train**
   *   Use `train_llm.py` (from earlier):

       ```python
       from transformers import AutoModelForCausalLM, AutoTokenizer, Trainer, TrainingArguments
       from datasets import load_dataset

       model_name = "distilbert-base-uncased"  # Smaller model for initial test
       model = AutoModelForCausalLM.from_pretrained(model_name)
       tokenizer = AutoTokenizer.from_pretrained(model_name)

       dataset = load_dataset("json", data_files="sat_data.jsonl")
       def tokenize(examples):
           return tokenizer(examples["input"], examples["output"], truncation=True, padding="max_length", max_length=128)

       tokenized_dataset = dataset.map(tokenize, batched=True)

       args = TrainingArguments(
           output_dir="./sat_llm",
           num_train_epochs=3,
           per_device_train_batch_size=16,
           evaluation_strategy="epoch",
           save_strategy="epoch",
       )

       trainer = Trainer(model=model, args=args, train_dataset=tokenized_dataset["train"])
       trainer.train()
       model.save_pretrained("./sat_llm_final")
       tokenizer.save_pretrained("./sat_llm_final")
       ```
   * Run: `python train_llm.py` (\~$10 for 20 hours on GPU).
3. **Integrate**
   * Update `utils.py` to use local LLM (as previously shown).
   * Deploy updated backend.

***

#### Step 8: Monitor and Scale

**Goal**: Prepare for user growth and LLM refinement.

1. **Monitoring**
   * Check DB growth: `SELECT COUNT(*) FROM TUTOR_FEEDBACK;` (weekly).
   * Log API usage: Add basic logging in `main.py`.
2. **Scaling**
   * At 1K users (\~5K pairs): Retrain LLM with real + synthetic data.
   * Upgrade EC2 to t3.medium if latency >2s.
3. **Next Steps**
   * Add user registration/login.
   * Expand question bank.
   * A/B test LLM vs. GPT-4o.

***

#### Timeline

* **Week 1**: Setup, implement backend (Steps 1-2).
* **Week 2**: Implement frontend, deploy locally (Steps 2-3).
* **Week 3**: Test locally, fix bugs (Step 4).
* **Week 4**: Deploy to production, test end-to-end (Steps 5-6).
* **Week 5**: Train initial LLM, integrate (Step 7).
* **Ongoing**: Monitor, scale (Step 8).

***

#### Conclusion

This plan takes the untested code from our discussions and turns it into a deployable, testable SAT Prep Suite. It starts with local setup, progresses to production deployment on AWS/Firebase, and culminates in an initial LLM trained on synthetic data, ready to collect real user/tutor data. Begin with Step 1—any questions before we start?
