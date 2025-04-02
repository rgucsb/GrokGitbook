# More/update COde 1

Below is an updated document that enhances the **SAT Prep Suite Code Package** by including the AI logic for `generate_feedback_with_custom_llm` in `utils.py` and providing full implementations for `study_plan.py`, `review.py`, and `community.py` in the backend. These additions flesh out the key functionality for study plan management, AI-driven review insights, and community features, aligning with the hybrid mobile/web app structure as of March 26, 2025. I’ll present this as a standalone document that supplements the previous code package, focusing solely on these updates for clarity.

***

## SAT Prep Suite: Enhanced Backend Logic Document

### Overview

This document provides the AI logic for `generate_feedback_with_custom_llm` and complete implementations for `study_plan.py`, `review.py`, and `community.py`. These enhancements enable personalized feedback, study plan adjustments, skill predictions, and community interactions, supporting both mobile (practice) and web (diagnostics/tests) experiences.

***

### Updated File: `api/utils.py`

**Purpose**: Includes the AI logic for generating feedback, transitioning from external APIs to a custom LLM.

```python
from sqlalchemy.orm import Session
from typing import Dict, List
import json
import redis
import requests  # For initial external API calls
from transformers import AutoModelForCausalLM, AutoTokenizer  # For custom LLM
import torch

redis_client = redis.Redis(host='localhost', port=6379, db=0)

# Load custom LLM (assumes model is trained and saved locally)
try:
    model = AutoModelForCausalLM.from_pretrained("./sat_llm_final")
    tokenizer = AutoTokenizer.from_pretrained("./sat_llm_final")
    USE_CUSTOM_LLM = True
except Exception:
    USE_CUSTOM_LLM = False
    print("Custom LLM not available, falling back to external API")

def log_tutor_feedback(db: Session, tutor_id: str, user_id: str, feedback_text: str, context_type: str, question_id: str = None, plan_id: int = None, performance_data: Dict = None):
    feedback = TutorFeedback(tutor_id=tutor_id, user_id=user_id, question_id=question_id, plan_id=plan_id, performance_data=performance_data, feedback_text=feedback_text, context_type=context_type)
    db.add(feedback)
    db.commit()

def generate_feedback_with_custom_llm(input_data: Dict, context: Dict) -> str:
    """
    Generates personalized feedback based on input data and context.
    Uses custom LLM if available, otherwise falls back to GPT-4o API.
    """
    # Prepare prompt based on input and context
    if "responses" in input_data:  # From test or practice
        correct = sum(1 for r in input_data["responses"] if r.get("is_correct", False))
        total = len(input_data["responses"])
        score = context.get("score", correct)
        prompt = f"Student answered {correct}/{total} questions correctly (score: {score}). Provide detailed feedback."
    elif "question_id" in input_data:  # Single question feedback
        prompt = f"Student answered question {input_data['question_id']} with '{input_data['answer']}'. Provide feedback."
    else:  # Generic feedback
        prompt = "Provide general encouragement based on recent performance."

    # Check Redis cache
    cache_key = f"feedback:{json.dumps(input_data)}:{json.dumps(context)}"
    cached = redis_client.get(cache_key)
    if cached:
        return cached.decode("utf-8")

    if USE_CUSTOM_LLM:
        # Use custom LLM
        inputs = tokenizer(prompt, return_tensors="pt", truncation=True, max_length=128)
        outputs = model.generate(**inputs, max_length=200, num_return_sequences=1, temperature=0.7)
        feedback = tokenizer.decode(outputs[0], skip_special_tokens=True)
    else:
        # Fallback to GPT-4o API (placeholder - requires API key)
        response = requests.post(
            "https://api.openai.com/v1/chat/completions",
            headers={"Authorization": "Bearer YOUR_API_KEY"},
            json={
                "model": "gpt-4o",
                "messages": [{"role": "user", "content": prompt}],
                "max_tokens": 150
            }
        )
        feedback = response.json()["choices"][0]["message"]["content"] if response.status_code == 200 else "Great job! Keep practicing."

    # Cache result
    redis_client.setex(cache_key, 3600, feedback)  # Cache for 1 hour
    return feedback

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

**Notes**:

* **AI Logic**: `generate_feedback_with_custom_llm` uses a custom LLM (e.g., DistilBERT-based, trained offline) if available, falling back to GPT-4o via API. It crafts prompts dynamically based on input (e.g., test responses, single answers) and caches results in Redis for performance.
* **Dependencies**: Requires `transformers` and `torch` for LLM, `requests` for API fallback—add to `requirements.txt`.

***

### New File: `api/routes/study_plan.py`

**Purpose**: Manages study plans, including creation, task actions, and AI-driven adjustments.

```python
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from api.database import get_db
from api.models import StudyPlan, StudyPlanAction, User
from api.utils import generate_feedback_with_custom_llm
from typing import Dict, List
from datetime import datetime, timedelta

router = APIRouter(prefix="/study_plan", tags=["study_plan"])

@router.post("/create/{user_id}")
async def create_study_plan(user_id: str, test_date: str, db: Session = Depends(get_db)):
    """Create a study plan based on test date."""
    if not db.query(User).filter(User.user_id == user_id).first():
        raise HTTPException(status_code=404, detail="User not found")
    
    # Simple plan: 5 weeks, daily tasks
    test_date_obj = datetime.strptime(test_date, "%Y-%m-%d")
    plan = {
        "weeks": [
            {"week": i+1, "tasks": [
                {"day": j+1, "skill": "Algebra", "task": "Practice 10 questions"}
                for j in range(7)
            ]} for i in range(5)
        ],
        "start_date": (test_date_obj - timedelta(weeks=5)).strftime("%Y-%m-%d")
    }
    study_plan = StudyPlan(user_id=user_id, plan=plan, test_date=test_date_obj)
    db.add(study_plan)
    db.commit()
    return {"user_id": user_id, "plan_id": study_plan.plan_id, "plan": plan}

@router.post("/action/{plan_id}")
async def log_study_plan_action(plan_id: int, action_data: Dict, db: Session = Depends(get_db)):
    """Log task completion or skip, adjust plan if needed."""
    plan = db.query(StudyPlan).filter(StudyPlan.plan_id == plan_id).first()
    if not plan:
        raise HTTPException(status_code=404, detail="Plan not found")
    
    action = StudyPlanAction(
        user_id=plan.user_id,
        plan_id=plan_id,
        task=action_data["task"],
        action=action_data["action"],  # "completed" or "skipped"
        performance_update=action_data.get("performance_update", {})
    )
    db.add(action)
    
    # Adjust plan based on performance (simplified logic)
    if action.action == "completed" and "accuracy" in action.performance_update:
        feedback = generate_feedback_with_custom_llm({"task": action.task, "performance": action.performance_update}, {})
        if action.performance_update["accuracy"] < 0.7:  # Low performance
            plan.plan["weeks"].append({"week": len(plan.plan["weeks"]) + 1, "tasks": [{"day": 1, "skill": action.task["skill"], "task": "Review 5 more questions"}]})
    
    db.commit()
    return {"plan_id": plan_id, "action": action.action, "feedback": feedback}
```

**Notes**:

* **Features**: Creates a 5-week plan, logs task actions, adjusts plan dynamically based on performance (e.g., adds review tasks if accuracy <70%).
* **Interaction**: Uses `generate_feedback_with_custom_llm` for task-specific feedback.

***

### New File: `api/routes/review.py`

**Purpose**: Provides AI-driven insights, including skill predictions and feedback.

```python
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from api.database import get_db
from api.models import Response, Test, Result
from api.utils import generate_feedback_with_custom_llm
from typing import Dict

router = APIRouter(prefix="/review", tags=["review"])

@router.get("/skills/{user_id}")
async def get_skill_predictions(user_id: str, db: Session = Depends(get_db)):
    """Predict skill proficiencies based on practice and test history."""
    responses = db.query(Response).filter(Response.user_id == user_id).all()
    tests = db.query(Test).filter(Test.user_id == user_id).all()
    
    if not responses and not tests:
        raise HTTPException(status_code=404, detail="No data for user")
    
    # Simple proficiency calculation (1-7 scale)
    skills = {}
    for r in responses:
        q = db.query(Question).filter(Question.question_id == r.question_id).first()
        skill = q.skill
        skills[skill] = skills.get(skill, {"correct": 0, "total": 0})
        skills[skill]["correct"] += 1 if r.is_correct else 0
        skills[skill]["total"] += 1
    
    for t in tests:
        for r in t.responses:
            q = db.query(Question).filter(Question.question_id == r["question_id"]).first()
            skill = q.skill
            skills[skill] = skills.get(skill, {"correct": 0, "total": 0})
            skills[skill]["correct"] += 1 if r.get("is_correct", False) else 0
            skills[skill]["total"] += 1
    
    proficiencies = {skill: min(7, max(1, round((data["correct"] / data["total"]) * 7))) for skill, data in skills.items()}
    feedback = generate_feedback_with_custom_llm({"skills": proficiencies}, {})
    result = Result(user_id=user_id, proficiencies=proficiencies)
    db.add(result)
    db.commit()
    return {"user_id": user_id, "proficiencies": proficiencies, "feedback": feedback}

@router.get("/next/{user_id}")
async def get_next_questions(user_id: str, db: Session = Depends(get_db)):
    """Suggest next questions based on weaknesses."""
    latest_result = db.query(Result).filter(Result.user_id == user_id).order_by(Result.completed_at.desc()).first()
    if not latest_result:
        questions = db.query(Question).filter(Question.test == "practice").limit(5).all()
    else:
        weak_skill = min(latest_result.proficiencies.items(), key=lambda x: x[1])[0]
        questions = db.query(Question).filter(Question.skill == weak_skill).limit(5).all()
    return {"user_id": user_id, "questions": [q.content for q in questions]}
```

**Notes**:

* **Features**: Predicts skill levels (1-7) from practice/tests, suggests next questions targeting weak areas.
* **Interaction**: Integrates with LLM for feedback, updates `RESULTS` table.

***

### New File: `api/routes/community.py`

**Purpose**: Handles community notes and AI responses.

```python
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from api.database import get_db
from api.models import UserNote, User
from api.utils import generate_feedback_with_custom_llm
from typing import Dict

router = APIRouter(prefix="/community", tags=["community"])

@router.post("/note/{user_id}")
async def post_note(user_id: str, note_data: Dict, db: Session = Depends(get_db)):
    """Post a note or question, get AI response."""
    if not db.query(User).filter(User.user_id == user_id).first():
        raise HTTPException(status_code=404, detail="User not found")
    
    note_text = note_data["note_text"]
    question_id = note_data.get("question_id")
    ai_response = generate_feedback_with_custom_llm({"note": note_text, "question_id": question_id}, {})
    note = UserNote(user_id=user_id, question_id=question_id, note_text=note_text, ai_response=ai_response)
    db.add(note)
    db.commit()
    return {"user_id": user_id, "note_id": note.id, "note_text": note_text, "ai_response": ai_response}

@router.get("/notes/{user_id}")
async def get_user_notes(user_id: str, db: Session = Depends(get_db)):
    """Retrieve user's community notes."""
    notes = db.query(UserNote).filter(UserNote.user_id == user_id).all()
    return {"user_id": user_id, "notes": [{"id": n.id, "note_text": n.note_text, "ai_response": n.ai_response} for n in notes]}
```

**Notes**:

* **Features**: Allows posting notes/questions with AI responses, retrieves user’s note history.
* **Interaction**: Uses LLM for responses, stores in `USER_NOTES`.

***

### Updated `api/requirements.txt`

```
fastapi==0.95.0
uvicorn==0.21.1
sqlalchemy==1.4.47
redis==4.5.4
requests==2.28.2
boto3==1.26.100
pytest==7.2.2
transformers==4.28.1
torch==2.0.0
```

***

### Integration Notes

* **AI Logic**: `generate_feedback_with_custom_llm` dynamically handles feedback for practice, tests, study plans, and community notes. Initially, use GPT-4o (replace `YOUR_API_KEY` with a real key) until the custom LLM is trained and deployed to `./sat_llm_final`.
*   **Frontend Updates**: Mobile screens (`study_plan.dart`, `community.dart`) and web screens (`results.dart`) need API calls to these new endpoints—add to `api_service.dart`:

    ```dart
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
    ```
* **Deployment**: Ensure EC2 has GPU support (e.g., g4dn.xlarge) for LLM inference once trained.

***

### Conclusion

This document adds critical AI logic and full backend routes for study plans, reviews, and community features, completing the SAT Prep Suite’s core functionality. Integrate these into the previous package, update the frontend, and test per the deployment plan. Ready to proceed with training the LLM or further refinements? Let me know!
