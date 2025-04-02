# Github update pkg files 3-28-2025

Let’s create a package containing all files updated since midnight on March 28, 2025, for the **SAT Smart Prep App** by **Learner Labs**, so you can download them and upload them to GitHub. I’ll first list all the files that were modified or added since midnight, as identified in the previous summary, and then provide a structured package (in a format you can copy and use) with the contents of each file. This package will include all relevant files, ensuring none are missed.

***

### Step 1: List of Files Updated Since Midnight on March 28, 2025

From the previous summary, the following files were modified since midnight on March 28, 2025:

| **File Path**                                     | **Status** | **Description of Change**                                                                                                         | **Timestamp**       |
| ------------------------------------------------- | ---------- | --------------------------------------------------------------------------------------------------------------------------------- | ------------------- |
| `backend/src/progress_monitoring.py`              | Modified   | Updated `/progress_monitoring/proficiencies/{user_id}` endpoint to return `target_mastery` for Progress Monitoring.               | 2025-03-28 01:00:00 |
| `frontend/pages/dashboard.js`                     | Modified   | Updated to display mastery targets, current mastery percentages, and progress toward targets in Progress Monitoring.              | 2025-03-28 01:15:00 |
| `SATSmartPrepApp/src/screens/DashboardScreen.js`  | Modified   | Updated to display mastery targets, current mastery percentages, and progress toward targets in Progress Monitoring (mobile).     | 2025-03-28 01:30:00 |
| `backend/src/study_plan.py`                       | Modified   | Added mastery targets to Study Plan, updated task allocation based on mastery gaps, and used IRT 3PL model for full-length tests. | 2025-03-28 02:00:00 |
| `frontend/pages/study-plan.js`                    | Modified   | Updated to display mastery targets and current mastery percentages in the Study Plan UI.                                          | 2025-03-28 02:15:00 |
| `SATSmartPrepApp/src/screens/StudyPlanScreen.js`  | Modified   | Updated to display mastery targets and current mastery percentages in the Study Plan UI (mobile).                                 | 2025-03-28 02:30:00 |
| `frontend/pages/onboarding.js`                    | Modified   | Updated to allow users to set mastery targets during onboarding.                                                                  | 2025-03-28 02:45:00 |
| `SATSmartPrepApp/src/screens/OnboardingScreen.js` | Modified   | Updated to allow users to set mastery targets during onboarding (mobile).                                                         | 2025-03-28 03:00:00 |
| `backend/src/diagnostic.py`                       | Modified   | Updated to use IRT 3PL model for question selection in the Diagnostic Test, maintaining 27/22 distribution.                       | 2025-03-28 03:30:00 |
| `backend/src/practice.py`                         | Modified   | Updated to use IRT 3PL model for adaptive question selection in practice sessions, allowing "new" and "qualified" questions.      | 2025-03-28 03:45:00 |

#### Notes

* **Database Schema Changes**: The addition of the `Question_IRT_Updates` and `Qualified_Questions_Reports` tables would typically involve a database migration script (e.g., in `backend/migrations/`), but since no such file was explicitly modified in the provided interactions, it’s not included. In a real project, you would need to create a migration script to add these tables.
* **No New Files**: All changes were modifications to existing files, so no new files were added since midnight.

***

### Step 2: Create a Package of Updated Files

Below is a structured package containing the contents of all files updated since midnight on March 28, 2025. Each file’s content is provided in a format you can copy and save into your local repository, then upload to GitHub. I’ll present the files in a directory-like structure with their full contents.

#### Package Structure

```
SATSmartPrepApp_updated_files/
├── backend/
│   ├── src/
│   │   ├── diagnostic.py
│   │   ├── practice.py
│   │   ├── progress_monitoring.py
│   │   └── study_plan.py
├── frontend/
│   ├── pages/
│   │   ├── dashboard.js
│   │   ├── onboarding.js
│   │   └── study-plan.js
└── SATSmartPrepApp/
    ├── src/
    │   ├── screens/
    │   │   ├── DashboardScreen.js
    │   │   ├── OnboardingScreen.js
    │   │   └── StudyPlanScreen.js
```

#### File Contents

**1. `backend/src/diagnostic.py`**

```python
from fastapi import APIRouter, HTTPException
from psycopg2.extras import RealDictCursor
import psycopg2
import uuid
import math

router = APIRouter()

def get_db_connection():
    return psycopg2.connect(
        dbname="satprep",
        user="user",
        password="password",
        host="localhost",
        port="5432"
    )

# Helper function to calculate IRT 3PL probability
def irt_3pl_probability(theta, a, b, c):
    exponent = -a * (theta - b)
    return c + (1 - c) * (1 / (1 + math.exp(exponent)))

# Helper function to select questions using IRT 3PL
def select_questions_irt(conn, user_id, domain, num_questions, subskills, user_theta):
    cursor = conn.cursor(cursor_factory=RealDictCursor)
    selected_questions = []
    subskill_counts = {subskill: 0 for subskill in subskills}
    target_per_subskill = num_questions // len(subskills)  # Distribute questions evenly across subskills

    # Fetch all qualified questions for the domain
    cursor.execute("""
        SELECT id, domain, skill, irt_a, irt_b, irt_c
        FROM questions
        WHERE domain = %s AND qualification_status = 'qualified'
        ORDER BY RANDOM();
    """, (domain,))
    all_questions = cursor.fetchall()

    # Select questions based on IRT 3PL probability
    for question in all_questions:
        if len(selected_questions) >= num_questions:
            break
        subskill = question['skill']
        if subskill_counts[subskill] >= target_per_subskill and len(selected_questions) < num_questions:
            continue  # Ensure even distribution across subskills

        # Calculate the probability of a correct response given the user's theta
        a = question['irt_a'] or 1.0  # Default to 1.0 if not set
        b = question['irt_b'] or 0.0  # Default to 0.0 if not set
        c = question['irt_c'] or 0.25  # Default to 0.25 if not set
        probability = irt_3pl_probability(user_theta, a, b, c)

        # Select questions where the probability is close to 0.5 (appropriately challenging)
        if 0.4 <= probability <= 0.6:
            selected_questions.append(question)
            subskill_counts[subskill] += 1

    # If not enough questions are selected, fill the remaining slots with random qualified questions
    if len(selected_questions) < num_questions:
        remaining = num_questions - len(selected_questions)
        cursor.execute("""
            SELECT id, domain, skill, irt_a, irt_b, irt_c
            FROM questions
            WHERE domain = %s AND qualification_status = 'qualified'
            AND id NOT IN %s
            ORDER BY RANDOM()
            LIMIT %s;
        """, (domain, tuple(q['id'] for q in selected_questions), remaining))
        selected_questions.extend(cursor.fetchall())

    return selected_questions

@router.post("/diagnostic/start/{user_id}")
async def start_diagnostic(user_id: str):
    conn = get_db_connection()
    cursor = conn.cursor(cursor_factory=RealDictCursor)
    try:
        # Check if user exists
        cursor.execute("SELECT * FROM users WHERE user_id = %s", (user_id,))
        user = cursor.fetchone()
        if not user:
            raise HTTPException(status_code=404, detail="User not found")

        # Estimate user's initial theta (default to 0 if no prior data)
        cursor.execute("SELECT AVG(theta) as avg_theta FROM proficiencies WHERE user_id = %s", (user_id,))
        result = cursor.fetchone()
        user_theta = result['avg_theta'] if result['avg_theta'] is not None else 0.0

        # Define subskills for each domain
        reading_writing_subskills = ["Reading Comprehension", "Grammar", "Vocabulary"]
        math_subskills = ["Algebra", "Geometry", "Data Analysis"]

        # Select questions using IRT 3PL model
        reading_writing_questions = select_questions_irt(conn, user_id, "Reading & Writing", 27, reading_writing_subskills, user_theta)
        math_questions = select_questions_irt(conn, user_id, "Math", 22, math_subskills, user_theta)

        # Combine questions
        questions = reading_writing_questions + math_questions
        if len(questions) != 49:
            raise HTTPException(status_code=500, detail="Failed to select the required number of questions")

        # Create a diagnostic session
        session_id = str(uuid.uuid4())
        cursor.execute("""
            INSERT INTO diagnostic_sessions (session_id, user_id, questions)
            VALUES (%s, %s, %s)
            RETURNING session_id;
        """, (session_id, user_id, [q['id'] for q in questions]))
        conn.commit()

        return {
            "session_id": session_id,
            "questions": questions
        }
    except Exception as e:
        conn.rollback()
        raise HTTPException(status_code=500, detail=str(e))
    finally:
        cursor.close()
        conn.close()

@router.post("/diagnostic/submit/{session_id}")
async def submit_diagnostic(session_id: str, responses: list):
    conn = get_db_connection()
    cursor = conn.cursor(cursor_factory=RealDictCursor)
    try:
        # Fetch the diagnostic session
        cursor.execute("SELECT * FROM diagnostic_sessions WHERE session_id = %s", (session_id,))
        session = cursor.fetchone()
        if not session:
            raise HTTPException(status_code=404, detail="Diagnostic session not found")

        user_id = session['user_id']
        question_ids = session['questions']

        # Validate responses
        if len(responses) != len(question_ids):
            raise HTTPException(status_code=400, detail="Number of responses does not match number of questions")

        # Fetch questions to evaluate responses
        cursor.execute("SELECT id, correct_answer FROM questions WHERE id IN %s", (tuple(question_ids),))
        questions = {q['id']: q for q in cursor.fetchall()}

        # Process responses
        correct_count = {"Reading & Writing": 0, "Math": 0}
        total_count = {"Reading & Writing": 0, "Math": 0}
        for response in responses:
            question_id = response['question_id']
            user_answer = response['user_answer']
            time_spent = response['time_spent']

            if question_id not in questions:
                continue

            question = questions[question_id]
            is_correct = user_answer == question['correct_answer']
            cursor.execute("SELECT domain FROM questions WHERE id = %s", (question_id,))
            domain = cursor.fetchone()['domain']

            # Update counts
            total_count[domain] += 1
            if is_correct:
                correct_count[domain] += 1

            # Store response
            response_id = str(uuid.uuid4())
            cursor.execute("""
                INSERT INTO responses (id, user_id, question_id, user_answer, is_correct, time_spent, timestamp)
                VALUES (%s, %s, %s, %s, %s, %s, NOW());
            """, (response_id, user_id, question_id, user_answer, is_correct, time_spent))

        # Calculate theta scores (simplified for this example)
        reading_theta = (correct_count["Reading & Writing"] / total_count["Reading & Writing"]) * 3 - 1.5 if total_count["Reading & Writing"] > 0 else 0
        math_theta = (correct_count["Math"] / total_count["Math"]) * 3 - 1.5 if total_count["Math"] > 0 else 0
        overall_theta = (reading_theta + math_theta) / 2

        # Store proficiencies (simplified for this example)
        subskills = {
            "Reading & Writing": ["Reading Comprehension", "Grammar", "Vocabulary"],
            "Math": ["Algebra", "Geometry", "Data Analysis"]
        }
        for domain, skills in subskills.items():
            theta = reading_theta if domain == "Reading & Writing" else math_theta
            for skill in skills:
                cursor.execute("""
                    INSERT INTO proficiencies (id, user_id, domain, skill, theta, timestamp)
                    VALUES (%s, %s, %s, %s, %s, NOW());
                """, (str(uuid.uuid4()), user_id, domain, skill, theta))

        # Store overall score
        score = int(overall_theta * 400 + 400)
        cursor.execute("""
            INSERT INTO scores (id, user_id, score, theta, timestamp)
            VALUES (%s, %s, %s, %s, NOW());
        """, (str(uuid.uuid4()), user_id, score, overall_theta))

        conn.commit()
        return {
            "session_id": session_id,
            "theta": overall_theta,
            "reading_theta": reading_theta,
            "math_theta": math_theta,
            "score": score
        }
    except Exception as e:
        conn.rollback()
        raise HTTPException(status_code=500, detail=str(e))
    finally:
        cursor.close()
        conn.close()
```

**2. `backend/src/practice.py`**

```python
from fastapi import APIRouter, HTTPException
from psycopg2.extras import RealDictCursor
import psycopg2
import uuid
import math

router = APIRouter()

def get_db_connection():
    return psycopg2.connect(
        dbname="satprep",
        user="user",
        password="password",
        host="localhost",
        port="5432"
    )

# Helper function to calculate IRT 3PL probability
def irt_3pl_probability(theta, a, b, c):
    exponent = -a * (theta - b)
    return c + (1 - c) * (1 / (1 + math.exp(exponent)))

# Helper function to select questions for practice using IRT 3PL
def select_practice_questions_irt(conn, user_id, domain, subskill, num_questions, user_theta):
    cursor = conn.cursor(cursor_factory=RealDictCursor)
    selected_questions = []

    # Fetch all questions for the domain and subskill (both new and qualified)
    cursor.execute("""
        SELECT id, domain, skill, irt_a, irt_b, irt_c
        FROM questions
        WHERE domain = %s AND skill = %s
        ORDER BY RANDOM();
    """, (domain, subskill))
    all_questions = cursor.fetchall()

    # Select questions based on IRT 3PL probability
    for question in all_questions:
        if len(selected_questions) >= num_questions:
            break

        # Calculate the probability of a correct response given the user's theta
        a = question['irt_a'] or 1.0  # Default to 1.0 if not set
        b = question['irt_b'] or 0.0  # Default to 0.0 if not set
        c = question['irt_c'] or 0.25  # Default to 0.25 if not set
        probability = irt_3pl_probability(user_theta, a, b, c)

        # Select questions where the probability is close to 0.5 (appropriately challenging)
        if 0.4 <= probability <= 0.6:
            selected_questions.append(question)

    # If not enough questions are selected, fill the remaining slots with random questions
    if len(selected_questions) < num_questions:
        remaining = num_questions - len(selected_questions)
        cursor.execute("""
            SELECT id, domain, skill, irt_a, irt_b, irt_c
            FROM questions
            WHERE domain = %s AND skill = %s
            AND id NOT IN %s
            ORDER BY RANDOM()
            LIMIT %s;
        """, (domain, subskill, tuple(q['id'] for q in selected_questions), remaining))
        selected_questions.extend(cursor.fetchall())

    return selected_questions

@router.post("/practice/start/{user_id}")
async def start_practice(user_id: str, body: dict):
    conn = get_db_connection()
    cursor = conn.cursor(cursor_factory=RealDictCursor)
    try:
        domain = body.get('domain')
        subskill = body.get('subskill')
        num_questions = body.get('num_questions', 5)  # Default to 5 questions

        if not domain or not subskill:
            raise HTTPException(status_code=400, detail="Domain and subskill are required")

        # Fetch user's current theta for the subskill
        cursor.execute("""
            SELECT theta FROM proficiencies
            WHERE user_id = %s AND domain = %s AND skill = %s
            ORDER BY timestamp DESC LIMIT 1;
        """, (user_id, domain, subskill))
        result = cursor.fetchone()
        user_theta = result['theta'] if result else 0.0  # Default to 0 if no prior data

        # Select questions using IRT 3PL model
        questions = select_practice_questions_irt(conn, user_id, domain, subskill, num_questions, user_theta)

        if len(questions) != num_questions:
            raise HTTPException(status_code=500, detail="Failed to select the required number of questions")

        # Create a practice session
        session_id = str(uuid.uuid4())
        cursor.execute("""
            INSERT INTO practice_sessions (session_id, user_id, questions)
            VALUES (%s, %s, %s)
            RETURNING session_id;
        """, (session_id, user_id, [q['id'] for q in questions]))
        conn.commit()

        return {
            "session_id": session_id,
            "questions": questions
        }
    except Exception as e:
        conn.rollback()
        raise HTTPException(status_code=500, detail=str(e))
    finally:
        cursor.close()
        conn.close()

@router.post("/practice/submit/{session_id}")
async def submit_practice(session_id: str, responses: list):
    conn = get_db_connection()
    cursor = conn.cursor(cursor_factory=RealDictCursor)
    try:
        # Fetch the practice session
        cursor.execute("SELECT * FROM practice_sessions WHERE session_id = %s", (session_id,))
        session = cursor.fetchone()
        if not session:
            raise HTTPException(status_code=404, detail="Practice session not found")

        user_id = session['user_id']
        question_ids = session['questions']

        # Validate responses
        if len(responses) != len(question_ids):
            raise HTTPException(status_code=400, detail="Number of responses does not match number of questions")

        # Fetch questions to evaluate responses
        cursor.execute("SELECT id, correct_answer, domain, skill FROM questions WHERE id IN %s", (tuple(question_ids),))
        questions = {q['id']: q for q in cursor.fetchall()}

        # Process responses
        correct_count = 0
        total_count = len(responses)
        for response in responses:
            question_id = response['question_id']
            user_answer = response['user_answer']
            time_spent = response['time_spent']

            if question_id not in questions:
                continue

            question = questions[question_id]
            is_correct = user_answer == question['correct_answer']
            if is_correct:
                correct_count += 1

            # Store response
            response_id = str(uuid.uuid4())
            cursor.execute("""
                INSERT INTO responses (id, user_id, question_id, user_answer, is_correct, time_spent, timestamp)
                VALUES (%s, %s, %s, %s, %s, %s, NOW());
            """, (response_id, user_id, question_id, user_answer, is_correct, time_spent))

        # Update proficiency (simplified for this example)
        domain = questions[question_ids[0]]['domain']
        subskill = questions[question_ids[0]]['skill']
        theta = (correct_count / total_count) * 3 - 1.5  # Simplified theta calculation
        cursor.execute("""
            INSERT INTO proficiencies (id, user_id, domain, skill, theta, timestamp)
            VALUES (%s, %s, %s, %s, %s, NOW());
        """, (str(uuid.uuid4()), user_id, domain, subskill, theta))

        conn.commit()
        return {
            "session_id": session_id,
            "correct": correct_count,
            "total": total_count,
            "theta": theta
        }
    except Exception as e:
        conn.rollback()
        raise HTTPException(status_code=500, detail=str(e))
    finally:
        cursor.close()
        conn.close()

@router.get("/practice/review/{session_id}")
async def review_practice(session_id: str):
    conn = get_db_connection()
    cursor = conn.cursor(cursor_factory=RealDictCursor)
    try:
        # Fetch the practice session
        cursor.execute("SELECT * FROM practice_sessions WHERE session_id = %s", (session_id,))
        session = cursor.fetchone()
        if not session:
            raise HTTPException(status_code=404, detail="Practice session not found")

        question_ids = session['questions']

        # Fetch responses
        cursor.execute("SELECT * FROM responses WHERE question_id IN %s AND user_id = %s", (tuple(question_ids), session['user_id']))
        responses = cursor.fetchall()

        # Fetch questions
        cursor.execute("SELECT * FROM questions WHERE id IN %s", (tuple(question_ids),))
        questions = {q['id']: q for q in cursor.fetchall()}

        # Prepare review items
        review_items = []
        for response in responses:
            question = questions.get(response['question_id'], {})
            review_items.append({
                "question_text": question.get('question_text', ''),
                "user_answer": response['user_answer'],
                "correct_answer": question.get('correct_answer', ''),
                "is_correct": response['is_correct'],
                "explanation": question.get('explanation', ''),
                "time_spent": response['time_spent'],
                "domain": question.get('domain', ''),
                "skill": question.get('skill', '')
            })

        # Calculate summary
        summary = {
            "Reading & Writing": {"correct": 0, "total": 0, "theta": 0, "weakest_subskill": None},
            "Math": {"correct": 0, "total": 0, "theta": 0, "weakest_subskill": None}
        }
        subskill_correct = {}
        subskill_total = {}

        for item in review_items:
            domain = item['domain']
            skill = item['skill']
            summary[domain]["total"] += 1
            if item['is_correct']:
                summary[domain]["correct"] += 1
            if skill not in subskill_correct:
                subskill_correct[skill] = 0
                subskill_total[skill] = 0
            subskill_total[skill] += 1
            if item['is_correct']:
                subskill_correct[skill] += 1

        # Calculate theta (simplified)
        for domain in summary:
            if summary[domain]["total"] > 0:
                summary[domain]["theta"] = (summary[domain]["correct"] / summary[domain]["total"]) * 3 - 1.5
            else:
                summary[domain]["theta"] = 0

        # Identify weakest subskill
        weakest_subskill = None
        lowest_accuracy = float('inf')
        for skill in subskill_total:
            if subskill_total[skill] > 0:
                accuracy = subskill_correct[skill] / subskill_total[skill]
                if accuracy < lowest_accuracy:
                    lowest_accuracy = accuracy
                    weakest_subskill = skill
            domain = next((item['domain'] for item in review_items if item['skill'] == skill), None)
            if domain:
                summary[domain]["weakest_subskill"] = weakest_subskill

        return {
            "review_items": review_items,
            "summary": summary
        }
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
    finally:
        cursor.close()
        conn.close()
```

**3. `backend/src/progress_monitoring.py`**

```python
from fastapi import APIRouter, HTTPException
from psycopg2.extras import RealDictCursor
import psycopg2

router = APIRouter()

def get_db_connection():
    return psycopg2.connect(
        dbname="satprep",
        user="user",
        password="password",
        host="localhost",
        port="5432"
    )

@router.get("/progress_monitoring/proficiencies/{user_id}")
async def get_proficiencies(user_id: str):
    conn = get_db_connection()
    cursor = conn.cursor(cursor_factory=RealDictCursor)
    try:
        # Fetch proficiencies
        cursor.execute("SELECT domain, skill, theta FROM proficiencies WHERE user_id = %s", (user_id,))
        proficiencies = cursor.fetchall()

        # Fetch target mastery from the Study Plan
        cursor.execute("SELECT plan_id FROM study_plans WHERE user_id = %s LIMIT 1", (user_id,))
        plan = cursor.fetchone()
        target_mastery = {}
        if plan:
            # In a real implementation, target_mastery would be stored in the database.
            # For now, we'll assume it's passed in the Study Plan creation and fetch it from the request.
            # Since we don't have a direct way to store it in this example, we'll simulate fetching it.
            # In practice, you might store target_mastery in a separate table or column.
            # Here, we'll fetch it from the Study Plan creation logic (simulated).
            target_mastery = {
                "Reading & Writing": 80,
                "Math": 80,
                "Reading Comprehension": 75,
                "Grammar": 75,
                "Vocabulary": 75,
                "Algebra": 75,
                "Geometry": 75,
                "Data Analysis": 75
            }

        # Calculate current mastery percentages
        current_mastery = {}
        for prof in proficiencies:
            mastery = ((prof['theta'] + 3) / 6) * 100  # Convert theta to mastery percentage
            current_mastery[prof['skill']] = mastery
            if prof['domain'] not in current_mastery:
                current_mastery[prof['domain']] = mastery
            else:
                current_mastery[prof['domain']] = (current_mastery[prof['domain']] + mastery) / 2  # Average for domain

        return {
            "proficiencies": proficiencies,
            "current_mastery": current_mastery,
            "target_mastery": target_mastery
        }
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
    finally:
        cursor.close()
        conn.close()

@router.get("/progress_monitoring/scores/{user_id}")
async def get_scores(user_id: str):
    conn = get_db_connection()
    cursor = conn.cursor(cursor_factory=RealDictCursor)
    try:
        cursor.execute("SELECT score, theta, timestamp FROM scores WHERE user_id = %s ORDER BY timestamp DESC", (user_id,))
        scores = cursor.fetchall()
        return scores
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
    finally:
        cursor.close()
        conn.close()

@router.get("/progress_monitoring/guarantee/{user_id}")
async def check_guarantee(user_id: str):
    conn = get_db_connection()
    cursor = conn.cursor(cursor_factory=RealDictCursor)
    try:
        cursor.execute("SELECT score, timestamp FROM scores WHERE user_id = %s ORDER BY timestamp", (user_id,))
        scores = cursor.fetchall()
        cursor.execute("SELECT COUNT(*) as completed FROM study_plan_actions WHERE plan_id = (SELECT plan_id FROM study_plans WHERE user_id = %s) AND completed IS NOT NULL", (user_id,))
        completed_tasks = cursor.fetchone()['completed']
        cursor.execute("SELECT COUNT(*) as total FROM study_plan_actions WHERE plan_id = (SELECT plan_id FROM study_plans WHERE user_id = %s)", (user_id,))
        total_tasks = cursor.fetchone()['total']

        if not scores:
            return {"message": "No scores available to evaluate the guarantee."}

        completion_rate = (completed_tasks / total_tasks) * 100 if total_tasks > 0 else 0
        if completion_rate < 80:
            return {"message": "You need to complete at least 80% of your Study Plan tasks to qualify for the score guarantee."}

        initial_score = scores[0]['score']
        latest_score = scores[-1]['score']
        score_improvement = latest_score - initial_score

        if initial_score > 1250:
            return {"message": "The score guarantee applies only to users with an initial score of 1250 or below."}

        if score_improvement >= 150:
            return {"message": "Congratulations! You've achieved a score improvement of 150 points or more."}
        else:
            return {"message": f"Your score improved by {score_improvement} points. You need {150 - score_improvement} more points to meet the 150-point score guarantee."}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
    finally:
        cursor.close()
        conn.close()
```

**4. `backend/src/study_plan.py`**

```python
from fastapi import APIRouter, HTTPException
from pydantic import BaseModel
from psycopg2.extras import RealDictCursor
import psycopg2
import uuid
from datetime import datetime, date, timedelta
import math

router = APIRouter()

def get_db_connection():
    return psycopg2.connect(
        dbname="satprep",
        user="user",
        password="password",
        host="localhost",
        port="5432"
    )

# Helper function to calculate IRT 3PL probability
def irt_3pl_probability(theta, a, b, c):
    exponent = -a * (theta - b)
    return c + (1 - c) * (1 / (1 + math.exp(exponent)))

# Helper function to convert mastery percentage (0 to 100%) to theta (-3 to 3)
def mastery_to_theta(mastery):
    return (mastery / 100) * 6 - 3  # Inverse of thetaToMastery

# Helper function to select questions for full-length tests using IRT 3PL
def select_full_length_test_questions(conn, user_id, domain, num_questions, subskills, user_theta):
    cursor = conn.cursor(cursor_factory=RealDictCursor)
    selected_questions = []
    subskill_counts = {subskill: 0 for subskill in subskills}
    target_per_subskill = num_questions // len(subskills)  # Distribute questions evenly across subskills

    # Fetch all qualified questions for the domain
    cursor.execute("""
        SELECT id, domain, skill, irt_a, irt_b, irt_c
        FROM questions
        WHERE domain = %s AND qualification_status = 'qualified'
        ORDER BY RANDOM();
    """, (domain,))
    all_questions = cursor.fetchall()

    # Select questions based on IRT 3PL probability
    for question in all_questions:
        if len(selected_questions) >= num_questions:
            break
        subskill = question['skill']
        if subskill_counts[subskill] >= target_per_subskill and len(selected_questions) < num_questions:
            continue  # Ensure even distribution across subskills

        # Calculate the probability of a correct response given the user's theta
        a = question['irt_a'] or 1.0  # Default to 1.0 if not set
        b = question['irt_b'] or 0.0  # Default to 0.0 if not set
        c = question['irt_c'] or 0.25  # Default to 0.25 if not set
        probability = irt_3pl_probability(user_theta, a, b, c)

        # Select questions where the probability is close to 0.5 (appropriately challenging)
        if 0.4 <= probability <= 0.6:
            selected_questions.append(question)
            subskill_counts[subskill] += 1

    # If not enough questions are selected, fill the remaining slots with random qualified questions
    if len(selected_questions) < num_questions:
        remaining = num_questions - len(selected_questions)
        cursor.execute("""
            SELECT id, domain, skill, irt_a, irt_b, irt_c
            FROM questions
            WHERE domain = %s AND qualification_status = 'qualified'
            AND id NOT IN %s
            ORDER BY RANDOM()
            LIMIT %s;
        """, (domain, tuple(q['id'] for q in selected_questions) if selected_questions else ('',), remaining))
        selected_questions.extend(cursor.fetchall())

    return selected_questions

# Helper function to create a full-length test session
def create_full_length_test_session(conn, user_id, user_theta):
    reading_writing_subskills = ["Reading Comprehension", "Grammar", "Vocabulary"]
    math_subskills = ["Algebra", "Geometry", "Data Analysis"]

    # Select questions using IRT 3PL model
    reading_writing_questions = select_full_length_test_questions(conn, user_id, "Reading & Writing", 27, reading_writing_subskills, user_theta)
    math_questions = select_full_length_test_questions(conn, user_id, "Math", 22, math_subskills, user_theta)

    # Combine questions
    questions = reading_writing_questions + math_questions
    if len(questions) != 49:
        raise HTTPException(status_code=500, detail="Failed to select the required number of questions for full-length test")

    # Create a practice session for the full-length test
    session_id = str(uuid.uuid4())
    cursor = conn.cursor(cursor_factory=RealDictCursor)
    cursor.execute("""
        INSERT INTO practice_sessions (session_id, user_id, questions)
        VALUES (%s, %s, %s)
        RETURNING session_id;
    """, (session_id, user_id, [q['id'] for q in questions]))
    conn.commit()

    return session_id, questions

class StudyPlanRequest(BaseModel):
    test_date: str
    study_hours: int
    study_days: int
    target_mastery: dict  # e.g., {"Reading & Writing": 80, "Math": 80, "Grammar": 75, ...}

@router.post("/study_plan/create/{user_id}")
async def create_study_plan(user_id: str, request: StudyPlanRequest):
    conn = get_db_connection()
    cursor = conn.cursor(cursor_factory=RealDictCursor)
    try:
        # Parse test date
        test_date = datetime.fromisoformat(request.test_date.replace('Z', '+00:00')).date()
        today = date.today()
        weeks_until_test = max(1, (test_date - today).days // 7)  # At least 1 week

        # Divide into 3 phases (40%, 30%, 30%)
        phase_1_weeks = max(1, int(weeks_until_test * 0.4))  # 40%
        phase_2_weeks = max(1, int(weeks_until_test * 0.3))  # 30%
        phase_3_weeks = max(1, weeks_until_test - phase_1_weeks - phase_2_weeks)  # 30%

        # Calculate total study sessions
        total_study_hours = request.study_hours * weeks_until_test
        sessions_per_week = request.study_days
        total_sessions = sessions_per_week * weeks_until_test
        session_duration = request.study_hours / request.study_days  # Hours per session

        # Allocate sessions to each phase
        phase_1_sessions = total_sessions * phase_1_weeks // weeks_until_test
        phase_2_sessions = total_sessions * phase_2_weeks // weeks_until_test
        phase_3_sessions = total_sessions - phase_1_sessions - phase_2_sessions

        # Fetch latest diagnostic results and analytics
        cursor.execute("SELECT domain, skill, theta, timestamp FROM proficiencies WHERE user_id = %s ORDER BY timestamp DESC", (user_id,))
        proficiencies = cursor.fetchall()

        cursor.execute("SELECT domain, skill, theta, timestamp FROM proficiencies WHERE user_id = %s ORDER BY timestamp", (user_id,))
        historical_proficiencies = cursor.fetchall()

        cursor.execute("""
            SELECT q.domain, q.skill, AVG(r.time_spent) as avg_time_spent
            FROM responses r
            JOIN questions q ON r.question_id = q.id
            WHERE r.user_id = %s
            GROUP BY q.domain, q.skill;
        """, (user_id,))
        time_spent_data = cursor.fetchall()

        # Calculate advanced analytics
        trends = {}
        improvement_rates = {}
        pacing_issues = {}
        subskills = set(p['skill'] for p in proficiencies)

        for subskill in subskills:
            subskill_thetas = [p['theta'] for p in proficiencies if p['skill'] == subskill]
            if len(subskill_thetas) >= 2:
                trends[subskill] = subskill_thetas[-1] - subskill_thetas[-2]
            else:
                trends[subskill] = 0

        for subskill in subskills:
            subskill_thetas = [(p['theta'], p['timestamp']) for p in historical_proficiencies if p['skill'] == subskill]
            if len(subskill_thetas) >= 2:
                time_diff = (subskill_thetas[-1][1] - subskill_thetas[0][1]).days / 7
                theta_diff = subskill_thetas[-1][0] - subskill_thetas[0][0]
                improvement_rates[subskill] = theta_diff / time_diff if time_diff > 0 else 0
            else:
                improvement_rates[subskill] = 0

        for row in time_spent_data:
            subskill = row['skill']
            avg_time = row['avg_time_spent']
            pacing_issues[subskill] = avg_time > 60

        # Calculate average theta per domain
        reading_writing_thetas = [p['theta'] for p in proficiencies if p['domain'] == 'Reading & Writing']
        math_thetas = [p['theta'] for p in proficiencies if p['domain'] == 'Math']
        avg_reading_writing_theta = sum(reading_writing_thetas) / len(reading_writing_thetas) if reading_writing_thetas else 0
        avg_math_theta = sum(math_thetas) / len(math_thetas) if math_thetas else 0
        overall_theta = (avg_reading_writing_theta + avg_math_theta) / 2

        # Identify focus subskills (theta < 0.5)
        focus_subskills = [p['skill'] for p in proficiencies if p['theta'] < 0.5]

        # Allocate sessions to domains based on relative weakness
        total_theta = avg_reading_writing_theta + avg_math_theta
        reading_writing_weight = (total_theta - avg_reading_writing_theta) / total_theta if total_theta > 0 else 0.5
        math_weight = (total_theta - avg_math_theta) / total_theta if total_theta > 0 else 0.5

        phase_1_reading_writing = int(phase_1_sessions * reading_writing_weight)
        phase_1_math = phase_1_sessions - phase_1_reading_writing
        phase_2_reading_writing = int(phase_2_sessions * reading_writing_weight)
        phase_2_math = phase_2_sessions - phase_2_reading_writing
        phase_3_reading_writing = int(phase_3_sessions * reading_writing_weight)
        phase_3_math = phase_3_sessions - phase_3_reading_writing

        # Schedule 8 full-length tests with fixed distribution
        phase_1_tests = 2
        phase_2_tests = 2
        phase_3_tests = 4

        phase_1_regular_sessions = phase_1_sessions - phase_1_tests
        phase_2_regular_sessions = phase_2_sessions - phase_2_tests
        phase_3_regular_sessions = phase_3_sessions - phase_3_tests

        # Allocate regular sessions to subskills with advanced analytics and mastery targets
        reading_writing_subskills = set(p['skill'] for p in proficiencies if p['domain'] == 'Reading & Writing')
        math_subskills = set(p['skill'] for p in proficiencies if p['domain'] == 'Math')
        focus_reading_writing = [s for s in focus_subskills if s in reading_writing_subskills]
        focus_math = [s for s in focus_subskills if s in math_subskills]

        # Calculate current theta and target theta for each subskill
        subskill_thetas = {p['skill']: p['theta'] for p in proficiencies}
        target_thetas = {}
        for domain in ["Reading & Writing", "Math"]:
            target_thetas[domain] = mastery_to_theta(request.target_mastery.get(domain, 80))  # Default to 80% if not specified
        for subskill in subskills:
            target_thetas[subskill] = mastery_to_theta(request.target_mastery.get(subskill, 75))  # Default to 75% if not specified

        reading_writing_allocation = {}
        math_allocation = {}
        for subskill in reading_writing_subskills:
            weight = 2 if subskill in focus_reading_writing else 1
            if trends.get(subskill, 0) <= 0:
                weight += 1
            if improvement_rates.get(subskill, 0) < 0.1:
                weight += 1
            if pacing_issues.get(subskill, False):
                weight += 1
            if trends.get(subskill, 0) > 0.2:
                weight -= 1
            if improvement_rates.get(subskill, 0) > 0.2:
                weight -= 1
            # Adjust weight based on mastery gap
            current_theta = subskill_thetas.get(subskill, 0)
            target_theta = target_thetas.get(subskill, mastery_to_theta(75))
            mastery_gap = target_theta - current_theta
            if mastery_gap > 0:
                weight += int(mastery_gap * 2)  # Increase weight based on the gap
            reading_writing_allocation[subskill] = max(1, weight)
        for subskill in math_subskills:
            weight = 2 if subskill in focus_math else 1
            if trends.get(subskill, 0) <= 0:
                weight += 1
            if improvement_rates.get(subskill, 0) < 0.1:
                weight += 1
            if pacing_issues.get(subskill, False):
                weight += 1
            if trends.get(subskill, 0) > 0.2:
                weight -= 1
            if improvement_rates.get(subskill, 0) > 0.2:
                weight -= 1
            current_theta = subskill_thetas.get(subskill, 0)
            target_theta = target_thetas.get(subskill, mastery_to_theta(75))
            mastery_gap = target_theta - current_theta
            if mastery_gap > 0:
                weight += int(mastery_gap * 2)
            math_allocation[subskill] = max(1, weight)

        total_reading_writing_weight = sum(reading_writing_allocation.values())
        total_math_weight = sum(math_allocation.values())
        for phase in ['phase_1_regular', 'phase_2_regular', 'phase_3_regular']:
            phase_reading_writing_sessions = locals()[f'{phase}_reading_writing']
            phase_math_sessions = locals()[f'{phase}_math']
            for subskill in reading_writing_allocation:
                reading_writing_allocation[subskill] = int(phase_reading_writing_sessions * (reading_writing_allocation[subskill] / total_reading_writing_weight))
            for subskill in math_allocation:
                math_allocation[subskill] = int(phase_math_sessions * (math_allocation[subskill] / total_math_weight))

        # Create Study Plan record
        plan_id = str(uuid.uuid4())
        cursor.execute("""
            INSERT INTO study_plans (plan_id, user_id, test_date)
            VALUES (%s, %s, %s);
        """, (plan_id, user_id, request.test_date))

        # Schedule tasks for each phase
        milestones = []
        current_date = date.today()

        # Phase 1: Foundation Building
        test_days = []
        for week in range(phase_1_weeks):
            week_start = current_date + timedelta(days=week * 7)
            if len(test_days) < phase_1_tests:
                test_days.append(week_start + timedelta(days=0))

        for week in range(phase_1_weeks):
            week_start = current_date + timedelta(days=week * 7)
            for day in range(request.study_days):
                day_date = week_start + timedelta(days=day)
                if day_date in test_days:
                    # Create a full-length test session
                    session_id, questions = create_full_length_test_session(conn, user_id, overall_theta)
                    task = f"Take a full-length SAT practice test (Session ID: {session_id})"
                    points = 150
                    milestones.append({
                        "phase": "Phase 1: Foundation Building",
                        "task": task,
                        "action": "Complete",
                        "due_date": day_date.isoformat(),
                        "points": points
                    })
                    cursor.execute("""
                        INSERT INTO study_plan_actions (id, plan_id, task, action, due_date, points)
                        VALUES (%s, %s, %s, %s, %s, %s);
                    """, (str(uuid.uuid4()), plan_id, task, "Complete", day_date.isoformat(), points))
                else:
                    for subskill in reading_writing_allocation:
                        if reading_writing_allocation[subskill] > 0:
                            task = f"Complete a 5-question session in {subskill}"
                            points = 50
                            milestones.append({
                                "phase": "Phase 1: Foundation Building",
                                "task": task,
                                "action": "Complete",
                                "due_date": day_date.isoformat(),
                                "points": points,
                                "subskill": subskill,
                                "target_mastery": request.target_mastery.get(subskill, 75)
                            })
                            cursor.execute("""
                                INSERT INTO study_plan_actions (id, plan_id, task, action, due_date, points)
                                VALUES (%s, %s, %s, %s, %s, %s);
                            """, (str(uuid.uuid4()), plan_id, task, "Complete", day_date.isoformat(), points))
                            if pacing_issues.get(subskill, False):
                                task = f"Practice pacing for {subskill}"
                                milestones.append({
                                    "phase": "Phase 1: Foundation Building",
                                    "task": task,
                                    "action": "Complete",
                                    "due_date": day_date.isoformat(),
                                    "points": 50,
                                    "subskill": subskill,
                                    "target_mastery": request.target_mastery.get(subskill, 75)
                                })
                                cursor.execute("""
                                    INSERT INTO study_plan_actions (id, plan_id, task, action, due_date, points)
                                    VALUES (%s, %s, %s, %s, %s, %s);
                                """, (str(uuid.uuid4()), plan_id, task, "Complete", day_date.isoformat(), 50))
                            reading_writing_allocation[subskill] -= 1
                    for subskill in math_allocation:
                        if math_allocation[subskill] > 0:
                            task = f"Complete a 5-question session in {subskill}"
                            points = 50
                            milestones.append({
                                "phase": "Phase 1: Foundation Building",
                                "task": task,
                                "action": "Complete",
                                "due_date": day_date.isoformat(),
                                "points": points,
                                "subskill": subskill,
                                "target_mastery": request.target_mastery.get(subskill, 75)
                            })
                            cursor.execute("""
                                INSERT INTO study_plan_actions (id, plan_id, task, action, due_date, points)
                                VALUES (%s, %s, %s, %s, %s, %s);
                            """, (str(uuid.uuid4()), plan_id, task, "Complete", day_date.isoformat(), points))
                            if pacing_issues.get(subskill, False):
                                task = f"Practice pacing for {subskill}"
                                milestones.append({
                                    "phase": "Phase 1: Foundation Building",
                                    "task": task,
                                    "action": "Complete",
                                    "due_date": day_date.isoformat(),
                                    "points": 50,
                                    "subskill": subskill,
                                    "target_mastery": request.target_mastery.get(subskill, 75)
                                })
                                cursor.execute("""
                                    INSERT INTO study_plan_actions (id, plan_id, task, action, due_date, points)
                                    VALUES (%s, %s, %s, %s, %s, %s);
                                """, (str(uuid.uuid4()), plan_id, task, "Complete", day_date.isoformat(), 50))
                            math_allocation[subskill] -= 1

        # Phase 2: Practice and Strategy
        test_days = []
        for week in range(phase_2_weeks):
            week_start = current_date + timedelta(days=(week + phase_1_weeks) * 7)
            if len(test_days) < phase_2_tests:
                test_days.append(week_start + timedelta(days=0))

        for week in range(phase_2_weeks):
            week_start = current_date + timedelta(days=(week + phase_1_weeks) * 7)
            for day in range(request.study_days):
                day_date = week_start + timedelta(days=day)
                if day_date in test_days:
                    # Create a full-length test session
                    session_id, questions = create_full_length_test_session(conn, user_id, overall_theta)
                    task = f"Take a full-length SAT practice test (Session ID: {session_id})"
                    points = 150
                    milestones.append({
                        "phase": "Phase 2: Practice and Strategy",
                        "task": task,
                        "action": "Complete",
                        "due_date": day_date.isoformat(),
                        "points": points
                    })
                    cursor.execute("""
                        INSERT INTO study_plan_actions (id, plan_id, task, action, due_date, points)
                        VALUES (%s, %s, %s, %s, %s, %s);
                    """, (str(uuid.uuid4()), plan_id, task, "Complete", day_date.isoformat(), points))
                else:
                    if day % 2 == 0:
                        for subskill in reading_writing_allocation:
                            if improvement_rates.get(subskill, 0) < 0.1:
                                task = f"Review strategies for {subskill}"
                                points = 75
                                milestones.append({
                                    "phase": "Phase 2: Practice and Strategy",
                                    "task": task,
                                    "action": "Complete",
                                    "due_date": day_date.isoformat(),
                                    "points": points,
                                    "subskill": subskill,
                                    "target_mastery": request.target_mastery.get(subskill, 75)
                                })
                                cursor.execute("""
                                    INSERT INTO study_plan_actions (id, plan_id, task, action, due_date, points)
                                    VALUES (%s, %s, %s, %s, %s, %s);
                                """, (str(uuid.uuid4()), plan_id, task, "Complete", day_date.isoformat(), points))
                        for subskill in math_allocation:
                            if improvement_rates.get(subskill, 0) < 0.1:
                                task = f"Review strategies for {subskill}"
                                points = 75
                                milestones.append({
                                    "phase": "Phase 2: Practice and Strategy",
                                    "task": task,
                                    "action": "Complete",
                                    "due_date": day_date.isoformat(),
                                    "points": points,
                                    "subskill": subskill,
                                    "target_mastery": request.target_mastery.get(subskill, 75)
                                })
                                cursor.execute("""
                                    INSERT INTO study_plan_actions (id, plan_id, task, action, due_date, points)
                                    VALUES (%s, %s, %s, %s, %s, %s);
                                """, (str(uuid.uuid4()), plan_id, task, "Complete", day_date.isoformat(), points))
                    else:
                        for subskill in reading_writing_allocation:
                            if reading_writing_allocation[subskill] > 0:
                                task = f"Complete a 10-question session in {subskill}"
                                points = 75
                                milestones.append({
                                    "phase": "Phase 2: Practice and Strategy",
                                    "task": task,
                                    "action": "Complete",
                                    "due_date": day_date.isoformat(),
                                    "points": points,
                                    "subskill": subskill,
                                    "target_mastery": request.target_mastery.get(subskill, 75)
                                })
                                cursor.execute("""
                                    INSERT INTO study_plan_actions (id, plan_id, task, action, due_date, points)
                                    VALUES (%s, %s, %s, %s, %s, %s);
                                """, (str(uuid.uuid4()), plan_id, task, "Complete", day_date.isoformat(), points))
                                reading_writing_allocation[subskill] -= 1
                        for subskill in math_allocation:
                            if math_allocation[subskill] > 0:
                                task = f"Complete a 10-question session in {subskill}"
                                points = 75
                                milestones.append({
                                    "phase": "Phase 2: Practice and Strategy",
                                    "task": task,
                                    "action": "Complete",
                                    "due_date": day_date.isoformat(),
                                    "points": points,
                                    "subskill": subskill,
                                    "target_mastery": request.target_mastery.get(subskill, 75)
                                })
                                cursor.execute("""
                                    INSERT INTO study_plan_actions (id, plan_id, task, action, due_date, points)
                                    VALUES (%s, %s, %s, %s, %s, %s);
                                """, (str(uuid.uuid4()), plan_id, task, "Complete", day_date.isoformat(), points))
                                math_allocation[subskill] -= 1

        # Phase 3: Final Review and Refinement
        test_days = []
        for week in range(phase_3_weeks):
            week_start = current_date + timedelta(days=(week + phase_1_weeks + phase_2_weeks) * 7)
            if len(test_days) < phase_3_tests:
                test_days.append(week_start + timedelta(days=0))

        for week in range(phase_3_weeks):
            week_start = current_date + timedelta(days=(week + phase_1_weeks + phase_2_weeks) * 7)
            for day in range(request.study_days):
                day_date = week_start + timedelta(days=day)
                if day_date in test_days:
                    # Create a full-length test session
                    session_id, questions = create_full_length_test_session(conn, user_id, overall_theta)
                    task = f"Take a full-length SAT practice test (Session ID: {session_id})"
                    points = 150
                    milestones.append({
                        "phase": "Phase 3: Final Review and Refinement",
                        "task": task,
                        "action": "Complete",
                        "due_date": day_date.isoformat(),
                        "points": points
                    })
                    cursor.execute("""
                        INSERT INTO study_plan_actions (id, plan_id, task, action, due_date, points)
                        VALUES (%s, %s, %s, %s, %s, %s);
                    """, (str(uuid.uuid4()), plan_id, task, "Complete", day_date.isoformat(), points))
                else:
                    if day % 2 == 0:
                        task = "Review test-day checklist"
                        points = 50
                        milestones.append({
                            "phase": "Phase 3: Final Review and Refinement",
                            "task": task,
                            "action": "Complete",
                            "due_date": day_date.isoformat(),
                            "points": points
                        })
                        cursor.execute("""
                            INSERT INTO study_plan_actions (id, plan_id, task, action, due_date, points)
                            VALUES (%s, %s, %s, %s, %s, %s);
                        """, (str(uuid.uuid4()), plan_id, task, "Complete", day_date.isoformat(), points))
                    else:
                        for subskill in reading_writing_allocation:
                            if reading_writing_allocation[subskill] > 0:
                                task = f"Review incorrect answers from {subskill} practice"
                                if trends.get(subskill, 0) <= 0:
                                    task = f"Review recent performance in {subskill}"
                                points = 50
                                milestones.append({
                                    "phase": "Phase 3: Final Review and Refinement",
                                    "task": task,
                                    "action": "Complete",
                                    "due_date": day_date.isoformat(),
                                    "points": points,
                                    "subskill": subskill,
                                    "target_mastery": request.target_mastery.get(subskill, 75)
                                })
                                cursor.execute("""
                                    INSERT INTO study_plan_actions (id, plan_id, task, action, due_date, points)
                                    VALUES (%s, %s, %s, %s, %s, %s);
                                """, (str(uuid.uuid4()), plan_id, task, "Complete", day_date.isoformat(), points))
                                reading_writing_allocation[subskill] -= 1
                        for subskill in math_allocation:
                            if math_allocation[subskill] > 0:
                                task = f"Review incorrect answers from {subskill} practice"
                                if trends.get(subskill, 0) <= 0:
                                    task = f"Review recent performance in {subskill}"
                                points = 50
                                milestones.append({
                                    "phase": "Phase 3: Final Review and Refinement",
                                    "task": task,
                                    "action": "Complete",
                                    "due_date": day_date.isoformat(),
                                    "points": points,
                                    "subskill": subskill,
                                    "target_mastery": request.target_mastery.get(subskill, 75)
                                })
                                cursor.execute("""
                                    INSERT INTO study_plan_actions (id, plan_id, task, action, due_date, points)
                                    VALUES (%s, %s, %s, %s, %s, %s);
                                """, (str(uuid.uuid4()), plan_id, task, "Complete", day_date.isoformat(), points))
                                math_allocation[subskill] -= 1

        conn.commit()
        return {
            "plan_id": plan_id,
            "milestones": milestones,
            "target_mastery": request.target_mastery,
            "current_mastery": {
                "Reading & Writing": (avg_reading_writing_theta + 3) / 6 * 100,
                "Math": (avg_math_theta + 3) / 6 * 100,
                **{p['skill']: (p['theta'] + 3) / 6 * 100 for p in proficiencies}
            }
        }
    except Exception as e:
        conn.rollback()
        raise HTTPException(status_code=500, detail=str(e))
    finally:
        cursor.close()
        conn.close()

@router.post("/study_plan/update/{user_id}")
async def update_study_plan(user_id: str, request: StudyPlanRequest):
    conn = get_db_connection()
    cursor = conn.cursor(cursor_factory=RealDictCursor)
    try:
        # Fetch the current Study Plan
        cursor.execute("SELECT plan_id, test_date FROM study_plans WHERE user_id = %s LIMIT 1", (user_id,))
        current_plan = cursor.fetchone()
        if not current_plan:
            raise HTTPException(status_code=404, detail="Study Plan not found")

        plan_id = current_plan['plan_id']
        test_date = datetime.fromisoformat(current_plan['test_date'].replace('Z', '+00:00')).date()
        today = date.today()
        weeks_until_test = max(1, (test_date - today).days // 7)

        # Fetch completed tasks
        cursor.execute("SELECT * FROM study_plan_actions WHERE plan_id = %s AND completed IS NOT NULL", (plan_id,))
        completed_tasks = cursor.fetchall()

        # Recalculate Study Plan parameters
        phase_1_weeks = max(1, int(weeks_until_test * 0.4))
        phase_2_weeks = max(1, int(weeks_until_test * 0.3))
        phase_3_weeks = max(1, weeks_until_test - phase_1_weeks - phase_2_weeks)

        total_study_hours = request.study_hours * weeks_until_test
        sessions_per_week = request.study_days
        total_sessions = sessions_per_week * weeks_until_test
        session_duration = request.study_hours / request.study_days

        phase_1_sessions = total_sessions * phase_1_weeks // weeks_until_test
        phase_2_sessions = total_sessions * phase_2_weeks // weeks_until_test
        phase_3_sessions = total_sessions - phase_1_sessions - phase_2_sessions

        # Fetch latest diagnostic results and analytics
        cursor.execute("SELECT domain, skill, theta, timestamp FROM proficiencies WHERE user_id = %s ORDER BY timestamp DESC", (user_id,))
        proficiencies = cursor.fetchall()

        cursor.execute("SELECT domain, skill, theta, timestamp FROM proficiencies WHERE user_id = %s ORDER BY timestamp", (user_id,))
        historical_proficiencies = cursor.fetchall()

        cursor.execute("""
            SELECT q.domain, q.skill, AVG(r.time_spent) as avg_time_spent
            FROM responses r
            JOIN questions q ON r.question_id = q.id
            WHERE r.user_id = %s
            GROUP BY q.domain, q.skill;
        """, (user_id,))
        time_spent_data = cursor.fetchall()

        # Calculate advanced analytics
        trends = {}
        improvement_rates = {}
        pacing_issues = {}
        subskills = set(p['skill'] for p in proficiencies)

        for subskill in subskills:
            subskill_thetas = [p['theta'] for p in proficiencies if p['skill'] == subskill]
            if len(subskill_thetas) >= 2:
                trends[subskill] = subskill_thetas[-1] - subskill_thetas[-2]
            else:
                trends[subskill] = 0

        for subskill in subskills:
            subskill_thetas = [(p['theta'], p['timestamp']) for p in historical_proficiencies if p['skill'] == subskill]
            if len(subskill_thetas) >= 2:
                time_diff = (subskill_thetas[-1][1] - subskill_thetas[0][1]).days / 7
                theta_diff = subskill_thetas[-1][0] - subskill_thetas[0][0]
                improvement_rates[subskill] = theta_diff / time_diff if time_diff > 0 else 0
            else:
                improvement_rates[subskill] = 0

        for row in time_spent_data:
            subskill = row['skill']
            avg_time = row['avg_time_spent']
            pacing_issues[subskill] = avg_time > 60

        reading_writing_thetas = [p['theta'] for p in proficiencies if p['domain'] == 'Reading & Writing']
        math_thetas = [p['theta'] for p in proficiencies if p['domain'] == 'Math']
        avg_reading_writing_theta = sum(reading_writing_thetas) / len(reading_writing_thetas) if reading_writing_thetas else 0
        avg_math_theta = sum(math_thetas) / len(math_thetas) if math_thetas else 0
        overall_theta = (avg_reading_writing_theta + avg_math_theta) / 2

        focus_subskills = [p['skill'] for p in proficiencies if p['theta'] < 0.5]

        total_theta = avg_reading_writing_theta + avg_math_theta
        reading_writing_weight = (total_theta - avg_reading_writing_theta) / total_theta if total_theta > 0 else 0.5
        math_weight = (total_theta - avg_math_theta) / total_theta if total_theta > 0 else 0.5

        phase_1_reading_writing = int(phase_1_sessions * reading_writing_weight)
        phase_1_math = phase_1_sessions - phase_1_reading_writing
        phase_2_reading_writing = int(phase_2_sessions * reading_writing_weight)
        phase_2_math = phase_2_sessions - phase_2_reading_writing
        phase_3_reading_writing = int(phase_3_sessions * reading_writing_weight)
        phase_3_math = phase_3_sessions - phase_3_reading_writing

        phase_1_tests = 2
        phase_2_tests = 2
        phase_3_tests = 4

        phase_1_regular_sessions = phase_1_sessions - phase_1_tests
        phase_2_regular_sessions = phase_2_sessions - phase_2_tests
        phase_3_regular_sessions = phase_3_sessions - phase_3_tests

        reading_writing_subskills = set(p['skill'] for p in proficiencies if p['domain'] == 'Reading & Writing')
        math_subskills = set(p['skill'] for p in proficiencies if p['domain'] == 'Math')
        focus_reading_writing = [s for s in focus_subskills if s in reading_writing_subskills]
        focus_math = [s for s in focus_subskills if s in math_subskills]

        subskill_thetas = {p['skill']: p['theta'] for p in proficiencies}
        target_thetas = {}
        for domain in ["Reading & Writing", "Math"]:
            target_thetas[domain] = mastery_to_theta(request.target_mastery.get(domain, 80))
        for subskill in subskills:
            target_thetas[subskill] = mastery_to_theta(request.target_mastery.get(subskill, 75))

        reading_writing_allocation = {}
        math_allocation = {}
        for subskill in reading_writing_subskills:
            weight = 2 if subskill in focus_reading_writing else 1
            if trends.get(subskill, 0) <= 0:
                weight += 1
            if improvement_rates.get(subskill, 0) < 0.1:
                weight += 1
            if pacing_issues.get(subskill, False):
                weight += 1
            if trends.get(subskill, 0) > 0.2:
                weight -= 1
            if improvement_rates.get(subskill, 0) > 0.2:
                weight -= 1
            current_theta = subskill_thetas.get(subskill, 0)
            target_theta = target_thetas.get(subskill, mastery_to_theta(75))
            mastery_gap = target_theta - current_theta
            if mastery_gap > 0:
                weight += int(mastery_gap * 2)
            reading_writing_allocation[subskill] = max(1, weight)
        for subskill in math_subskills:
            weight = 2 if subskill in focus_math else 1
            if trends.get(subskill, 0) <= 0:
                weight += 1
            if improvement_rates.get(subskill, 0) < 0.1:
                weight += 1
            if pacing_issues.get(subskill, False):
                weight += 1
            if trends.get(subskill, 0) > 0.2:
                weight -= 1
            if improvement_rates.get(subskill, 0) > 0.2:
                weight -= 1
            current_theta = subskill_thetas.get(subskill, 0)
            target_theta = target_thetas.get(subskill, mastery_to_theta(75))
            mastery_gap = target_theta - current_theta
            if mastery_gap > 0:
                weight += int(mastery_gap * 2)
            math_allocation[subskill] = max(1, weight)

        total_reading_writing_weight = sum(reading_writing_allocation.values())
        total_math_weight = sum(math_allocation.values())
        for phase in ['phase_1_regular', 'phase_2_regular', 'phase_3_regular']:
            phase_reading_writing_sessions = locals()[f'{phase}_reading_writing']
            phase_math_sessions = locals()[f'{phase}_math']
            for subskill in reading_writing_allocation:
                reading_writing_allocation[subskill] = int(phase_reading_writing_sessions * (reading_writing_allocation[subskill] / total_reading_writing_weight))
            for subskill in math_allocation:
                math_allocation[subskill] = int(phase_math_sessions * (math_allocation[subskill] / total_math_weight))

        # Delete uncompleted tasks
        cursor.execute("DELETE FROM study_plan_actions WHERE plan_id = %s AND completed IS NULL", (plan_id,))

        # Schedule new tasks
        milestones = completed_tasks
        current_date = date.today()

        # Phase 1: Foundation Building
        test_days = []
        for week in range(phase_1_weeks):
            week_start = current_date + timedelta(days=week * 7)
            if len(test_days) < phase_1_tests:
                test_days.append(week_start + timedelta(days=0))

        for week in range(phase_1_weeks):
            week_start = current_date + timedelta(days=week * 7)
            for day in range(request.study_days):
                day_date = week_start + timedelta(days=day)
                if day_date in test_days:
                    # Create a full-length test session
                    session_id, questions = create_full_length_test_session(conn, user_id, overall_theta)
                    task = f"Take a full-length SAT practice test (Session ID: {session_id})"
                    points = 150
                    milestones.append({
                        "phase": "Phase 1: Foundation Building",
                        "task": task,
                        "action": "Complete",
                        "due_date": day_date.isoformat(),
                        "points": points
                    })
                    cursor.execute("""
                        INSERT INTO study_plan_actions (id, plan_id, task, action, due_date, points)
                        VALUES (%s, %s, %s, %s, %s, %s);
                    """, (str(uuid.uuid4()), plan_id, task, "Complete", day_date.isoformat(), points))
                else:
                    for subskill in reading_writing_allocation:
                        if reading_writing_allocation[subskill] > 0:
                            task = f"Complete a 5-question session in {subskill}"
                            points = 50
                            milestones.append({
                                "phase": "Phase 1: Foundation Building",
                                "task": task,
                                "action": "Complete",
                                "due_date": day_date.isoformat(),
                                "points": points,
                                "subskill": subskill,
                                "target_mastery": request.target_mastery.get(subskill, 75)
                            })
                            cursor.execute("""
                                INSERT INTO study_plan_actions (id, plan_id, task, action, due_date, points)
                                VALUES (%s, %s, %s, %s, %s, %s);
                            """, (str(uuid.uuid4()), plan_id, task, "Complete", day_date.isoformat(), points))
                            if pacing_issues.get(subskill, False):
                                task = f"Practice pacing for {subskill}"
                                milestones.append({
                                    "phase": "Phase 1: Foundation Building",
                                    "task": task,
                                    "action": "Complete",
                                    "due_date": day_date.isoformat(),
                                    "points": 50,
                                    "subskill": subskill,
                                    "target_mastery": request.target_mastery.get(subskill, 75)
                                })
                                cursor.execute("""
                                    INSERT INTO study_plan_actions (id, plan_id, task, action, due_date, points)
                                    VALUES (%s, %s, %s, %s, %s, %s);
                                """, (str(uuid.uuid4()), plan_id, task, "Complete", day_date.isoformat(), 50))
                            reading_writing_allocation[subskill] -= 1
                    for subskill in math_allocation:
                        if math_allocation[subskill] > 0:
                            task = f"Complete a 5-question session in {subskill}"
                            points = 50
                            milestones.append({
                                "phase": "Phase 1: Foundation Building",
                                "task": task,
                                "action": "Complete",
                                "due_date": day_date.isoformat(),
                                "points": points,
                                "subskill": subskill,
                                "target_mastery": request.target_mastery.get(subskill, 75)
                            })
                            cursor.execute("""
                                INSERT INTO study_plan_actions (id, plan_id, task, action, due_date, points)
                                VALUES (%s, %s, %s, %s, %s, %s);
                            """, (str(uuid.uuid4()), plan_id, task, "Complete", day_date.isoformat(), points))
                            if pacing_issues.get(subskill, False):
                                task = f"Practice pacing for {subskill}"
                                milestones.append({
                                    "phase": "Phase 1: Foundation Building",
                                    "task": task,
                                    "action": "Complete",
                                    "due_date": day_date.isoformat(),
                                    "points": 50,
                                    "subskill": subskill,
                                    "target_mastery": request.target_mastery.get(subskill, 75)
                                })
                                cursor.execute("""
                                    INSERT INTO study_plan_actions (id, plan_id, task, action, due_date, points)
                                    VALUES (%s, %s, %s, %s, %s, %s);
                                """, (str(uuid.uuid4()), plan_id, task, "Complete", day_date.isoformat(), 50))
                            math_allocation[subskill] -= 1

        # Phase 2: Practice and Strategy
        test_days = []
        for week in range(phase_2_weeks):
            week_start = current_date + timedelta(days=(week + phase_1_weeks) * 7)
            if len(test_days) < phase_2_tests:
                test_days.append(week_start + timedelta(days=0))

        for week in range(phase_2_weeks):
            week_start = current_date + timedelta(days=(week + phase_1_weeks) * 7)
            for day in range(request.study_days):
                day_date = week_start + timedelta(days=day)
                if day_date in test_days:
                    # Create a full-length test session
                    session_id, questions = create_full_length_test_session(conn, user_id, overall_theta)
                    task = f"Take a full-length SAT practice test (Session ID: {session_id})"
                    points = 150
                    milestones.append({
                        "phase": "Phase 2: Practice and Strategy",
                        "task": task,
                        "action": "Complete",
                        "due_date": day_date.isoformat(),
                        "points": points
                    })
                    cursor.execute("""
                        INSERT INTO study_plan_actions (id, plan_id, task, action, due_date, points)
                        VALUES (%s, %s, %s, %s, %s, %s);
                    """, (str(uuid.uuid4()), plan_id, task, "Complete", day_date.isoformat(), points))
                else:
                    if day % 2 == 0:
                        for subskill in reading_writing_allocation:
                            if improvement_rates.get(subskill, 0) < 0.1:
                                task = f"Review strategies for {subskill}"
                                points = 75
                                milestones.append({
                                    "phase": "Phase 2: Practice and Strategy",
                                    "task": task,
                                    "action": "Complete",
                                    "due_date": day_date.isoformat(),
                                    "points": points,
                                    "subskill": subskill,
                                    "target_mastery": request.target_mastery.get(subskill, 75)
                                })
                                cursor.execute("""
                                    INSERT INTO study_plan_actions (id, plan_id, task, action, due_date, points)
                                    VALUES (%s, %s, %s, %s, %s, %s);
                                """, (str(uuid.uuid4()), plan_id, task, "Complete", day_date.isoformat(), points))
                        for subskill in math_allocation:
                            if improvement_rates.get(subskill, 0) < 0.1:
                                task = f"Review strategies for {subskill}"
                                points = 75
                                milestones.append({
                                    "phase": "Phase 2: Practice and Strategy",
                                    "task": task,
                                    "action": "Complete",
                                    "due_date": day_date.isoformat(),
                                    "points": points,
                                    "subskill": subskill,
                                    "target_mastery": request.target_mastery.get(subskill, 75)
                                })
                                cursor.execute("""
                                    INSERT INTO study_plan_actions (id, plan_id, task, action, due_date, points)
                                    VALUES (%s, %s, %s, %s, %s, %s);
                                """, (str(uuid.uuid4()), plan_id, task, "Complete", day_date.isoformat(), points))
                    else:
                        for subskill in reading_writing_allocation:
                            if reading_writing_allocation[subskill] > 0:
                                task = f"Complete a 10-question session in {subskill}"
                                points = 75
                                milestones.append({
                                    "phase": "Phase 2: Practice and Strategy",
                                    "task": task,
                                    "action": "Complete",
                                    "due_date": day_date.isoformat(),
                                    "points": points,
                                    "subskill": subskill,
                                    "target_mastery": request.target_mastery.get(subskill, 75)
                                })
                                cursor.execute("""
                                    INSERT INTO study_plan_actions (id, plan_id, task, action, due_date, points)
                                    VALUES (%s, %s, %s, %s, %s, %s);
                                """, (str(uuid.uuid4()), plan_id, task, "Complete", day_date.isoformat(), points))
                                reading_writing_allocation[subskill] -= 1
                        for subskill in math_allocation:
                            if math_allocation[subskill] > 0:
                                task = f"Complete a 10-question session in {subskill}"
                                points = 75
                                milestones.append({
                                    "phase": "Phase 2: Practice and Strategy",
                                    "task": task,
                                    "action": "Complete",
                                    "due_date": day_date.isoformat(),
                                    "points": points,
                                    "subskill": subskill,
                                    "target_mastery": request.target_mastery.get(subskill, 75)
                                })
                                cursor.execute("""
                                    INSERT INTO study_plan_actions (id, plan_id, task, action, due_date, points)
                                    VALUES (%s, %s, %s, %s, %s, %s);
                                """, (str(uuid.uuid4()), plan_id, task, "Complete", day_date.isoformat(), points))
                                math_allocation[subskill] -= 1

        # Phase 3: Final Review and Refinement
        test_days = []
        for week in range(phase_3_weeks):
            week_start = current_date + timedelta(days=(week + phase_1_weeks + phase_2_weeks) * 7)
            if len(test_days) < phase_3_tests:
                test_days.append(week_start + timedelta(days=0))

        for week in range(phase_3_weeks):
            week_start = current_date + timedelta(days=(week + phase_1_weeks + phase_2_weeks) * 7)
            for day in range(request.study_days):
                day_date = week_start + timedelta(days=day)
                if day_date in test_days:
                    # Create a full-length test session
                    session_id, questions = create_full_length_test_session(conn, user_id, overall_theta)
                    task = f"Take a full-length SAT practice test (Session ID: {session_id})"
                    points = 150
                    milestones.append({
                        "phase": "Phase 3: Final Review and Refinement",
                        "task": task,
                        "action": "Complete",
                        "due_date": day_date.isoformat(),
                        "points": points
                    })
                    cursor.execute("""
                        INSERT INTO study_plan_actions (id, plan_id, task, action, due_date, points)
                        VALUES (%s, %s, %s, %s, %s, %s);
                    """, (str(uuid.uuid4()), plan_id, task, "Complete", day_date.isoformat(), points))
                else:
                    if day % 2 == 0:
                        task = "Review test-day checklist"
                        points = 50
                        milestones.append({
                            "phase": "Phase 3: Final Review and Refinement",
                            "task": task,
                            "action": "Complete",
                            "due_date": day_date.isoformat(),
                            "points": points
                        })
                        cursor.execute("""
                            INSERT INTO study_plan_actions (id, plan_id, task, action, due_date, points)
                            VALUES (%s, %s, %s, %s, %s, %s);
                        """, (str(uuid.uuid4()), plan_id, task, "Complete", day_date.isoformat(), points))
                    else:
                        for subskill in reading_writing_allocation:
                            if reading_writing_allocation[subskill] > 0:
                                task = f"Review incorrect answers from {subskill} practice"
                                if trends.get(subskill, 0) <= 0:
                                    task = f"Review recent performance in {subskill}"
                                points = 50
                                milestones.append({
                                    "phase": "Phase 3: Final Review and Refinement",
                                    "task": task,
                                    "action": "Complete",
                                    "due_date": day_date.isoformat(),
                                    "points": points,
                                    "subskill": subskill,
                                    "target_mastery": request.target_mastery.get(subskill, 75)
                                })
                                cursor.execute("""
                                    INSERT INTO study_plan_actions (id, plan_id, task, action, due_date, points)
                                    VALUES (%s, %s, %s, %s, %s, %s);
                                """, (str(uuid.uuid4()), plan_id, task, "Complete", day_date.isoformat(), points))
                                reading_writing_allocation[subskill] -= 1
                        for subskill in math_allocation:
                            if math_allocation[subskill] > 0:
                                task = f"Review incorrect answers from {subskill} practice"
                                if trends.get(subskill, 0) <= 0:
                                    task = f"Review recent performance in {subskill}"
                                points = 50
                                milestones.append({
                                    "phase": "Phase 3: Final Review and Refinement",
                                    "task": task,
                                    "action": "Complete",
                                    "due_date": day_date.isoformat(),
                                    "points": points,
                                    "subskill": subskill,
                                    "target_mastery": request.target_mastery.get(subskill, 75)
                                })
                                cursor.execute("""
                                    INSERT INTO study_plan_actions (id, plan_id, task, action, due_date, points)
                                    VALUES (%s, %s, %s, %s, %s, %s);
                                """, (str(uuid.uuid4()), plan_id, task, "Complete", day_date.isoformat(), points))
                                math_allocation[subskill] -= 1

        conn.commit()
        return {
            "plan_id": plan_id,
            "milestones": milestones,
            "target_mastery": request.target_mastery,
            "current_mastery": {
                "Reading & Writing": (avg_reading_writing_theta + 3) / 6 * 100,
                "Math": (avg_math_theta + 3) / 6 * 100,
                **{p['skill']: (p['theta'] + 3) / 6 * 100 for p in proficiencies}
            }
        }
    except Exception as e:
        conn.rollback()
        raise HTTPException(status_code=500, detail=str(e))
    finally:
        cursor.close()
        conn.close()
```



Apologies for the interruption! Let’s continue creating the package of all files updated since midnight on March 28, 2025, for the **SAT Smart Prep App** by **Learner Labs**. I was in the process of providing the contents of the updated files, and I got interrupted while providing the content of `backend/src/study_plan.py`. I’ll resume from there and ensure all remaining files are included in the package, so you can copy them and upload them to GitHub.

***

### Resuming the Package Creation

**5. `frontend/pages/dashboard.js`**

```javascript
import React, { useState, useEffect } from 'react';
import { useRouter } from 'next/router';
import { api } from '../utils/api';
import { colors, typography, commonStyles } from '../styles';

// Helper function to convert theta (-3 to 3) to mastery percentage (0 to 100%)
const thetaToMastery = (theta) => {
    return Math.round(((theta + 3) / 6) * 100); // Linear transformation
};

export default function Dashboard({ diagnosticResults }) {
    const [user, setUser] = useState(null);
    const [challenges, setChallenges] = useState([]);
    const [achievements, setAchievements] = useState([]);
    const [proficiencies, setProficiencies] = useState([]);
    const [scoreHistory, setScoreHistory] = useState([]);
    const [feedback, setFeedback] = useState('');
    const [targetMastery, setTargetMastery] = useState({});
    const router = useRouter();
    const userId = localStorage.getItem('user_id');

    useEffect(() => {
        if (!userId) {
            router.push('/login');
        }
        api.get(`/gamification/coins/${userId}`).then(res => setUser({ coins: res.data.coins }));
        api.get(`/challenges/${userId}`).then(res => setChallenges(res.data));
        api.get(`/progress_monitoring/proficiencies/${userId}`).then(res => {
            setProficiencies(res.data.proficiencies);
            setTargetMastery(res.data.target_mastery);
        });
        api.get(`/progress_monitoring/scores/${userId}`).then(res => setScoreHistory(res.data));
        setAchievements([
            { id: 1, name: "Complete 100 Questions", target: 100, progress: 75, completed: false },
            { id: 2, name: "5-Day Streak", target: 5, progress: 3, completed: false }
        ]);
    }, [userId, router]);

    const submitFeedback = async () => {
        if (!feedback) return;
        await api.post('/feedback', { user_id: userId, content: feedback });
        alert('Thank you for your feedback!');
        setFeedback('');
    };

    const checkGuarantee = async () => {
        const res = await api.get(`/progress_monitoring/guarantee/${userId}`);
        alert(res.data.message);
    };

    const updateStudyPlan = async () => {
        try {
            const res = await api.post(`/study_plan/update/${userId}`, {
                test_date: userData.sat_test_date,
                study_hours: userData.study_hours_per_week,
                study_days: userData.study_days_per_week
            });
            alert('Study Plan updated successfully!');
            router.push('/study-plan');
        } catch (error) {
            alert('Failed to update Study Plan: ' + error.message);
        }
    };

    // Calculate overall mastery from diagnostic results
    const overallMastery = diagnosticResults ? thetaToMastery(diagnosticResults.theta) : 0;
    const estimatedScore = diagnosticResults ? (diagnosticResults.theta * 400 + 400) : 0;

    // Calculate domain-level mastery
    const readingMastery = diagnosticResults ? thetaToMastery(diagnosticResults.reading_theta || 0) : 0;
    const mathMastery = diagnosticResults ? thetaToMastery(diagnosticResults.math_theta || 0) : 0;

    // Calculate progress toward targets
    const readingProgress = targetMastery["Reading & Writing"] ? Math.min(100, (readingMastery / targetMastery["Reading & Writing"]) * 100) : 0;
    const mathProgress = targetMastery["Math"] ? Math.min(100, (mathMastery / targetMastery["Math"]) * 100) : 0;

    return (
        <div className="dashboard">
            {diagnosticResults ? (
                <div className="diagnostic-results">
                    <h2 style={typography.heading}>Your Diagnostic Results</h2>
                    <p style={typography.body}>Overall Mastery: {overallMastery}%</p>
                    <p style={typography.body}>Estimated SAT Score: {estimatedScore}</p>
                    <h3 style={typography.heading}>Math</h3>
                    <p style={typography.body}>Math Mastery: {mathMastery}% (Target: {targetMastery["Math"] || 80}%)</p>
                    <p style={typography.body}>Progress Toward Target: {Math.round(mathProgress)}%</p>
                    {mathMastery >= (targetMastery["Math"] || 80) && <p style={typography.body}>Target Achieved! 🎉</p>}
                    <p style={typography.body}>Strong Areas: Algebra (Mastery: {thetaToMastery(diagnosticResults.math_theta || 0)}%)</p>
                    <p style={typography.body}>Weak Areas: Geometry (Mastery: {thetaToMastery((diagnosticResults.math_theta || 0) - 0.5)}%)</p>
                    <h3 style={typography.heading}>Reading & Writing</h3>
                    <p style={typography.body}>Reading & Writing Mastery: {readingMastery}% (Target: {targetMastery["Reading & Writing"] || 80}%)</p>
                    <p style={typography.body}>Progress Toward Target: {Math.round(readingProgress)}%</p>
                    {readingMastery >= (targetMastery["Reading & Writing"] || 80) && <p style={typography.body}>Target Achieved! 🎉</p>}
                    <p style={typography.body}>Strong Areas: Reading Comprehension (Mastery: {thetaToMastery(diagnosticResults.reading_theta || 0)}%)</p>
                    <p style={typography.body}>Weak Areas: Grammar (Mastery: {thetaToMastery((diagnosticResults.reading_theta || 0) - 0.5)}%)</p>
                    <p style={typography.body}>Key Recommendation: Focus on Geometry and Grammar to improve your mastery.</p>
                    <button style={commonStyles.button} onClick={() => router.push(`/review?sessionId=${diagnosticResults.session_id}`)}>
                        <span style={commonStyles.buttonText}>Review Diagnostic Test</span>
                    </button>
                </div>
            ) : (
                <>
                    {user && <p style={typography.body}>Coins: {user.coins} 🪙</p>}
                    <div className="score-history">
                        <h2 style={typography.heading}>Score History</h2>
                        {scoreHistory.map((entry, idx) => (
                            <p key={idx} style={typography.body}>
                                {new Date(entry.timestamp).toLocaleDateString()}: {entry.score} (Mastery: {thetaToMastery(entry.theta)}%)
                            </p>
                        ))}
                        <button style={commonStyles.button} onClick={checkGuarantee}>
                            <span style={commonStyles.buttonText}>Check Score Guarantee</span>
                        </button>
                        <button style={commonStyles.button} onClick={() => router.push('/policy')}>
                            <span style={commonStyles.buttonText}>View Guarantee Policy</span>
                        </button>
                        <button style={commonStyles.button} onClick={updateStudyPlan}>
                            <span style={commonStyles.buttonText}>Update Study Plan</span>
                        </button>
                    </div>
                    <div className="daily-challenges">
                        <h2 style={typography.heading}>Daily Challenges</h2>
                        {challenges.map(challenge => (
                            <div key={challenge.id} style={commonStyles.card}>
                                <p style={typography.body}>
                                    {challenge.challenge_type === 'daily_questions' ? "Answer 10 Questions Correctly" : "Complete a 5-Question Session in Under 5 Minutes"}
                                </p>
                                <div className="progress-bar">
                                    <div style={{ width: `${(challenge.progress / challenge.target) * 100}%` }}></div>
                                    <span className="progress-text">{challenge.progress}/{challenge.target}</span>
                                </div>
                                {challenge.completed && <p style={typography.body}>Completed! 🎉</p>}
                            </div>
                        ))}
                    </div>
                    <div className="achievements">
                        <h2 style={typography.heading}>Achievements</h2>
                        {achievements.map(achievement => (
                            <div key={achievement.id} style={commonStyles.card}>
                                <p style={typography.body}>{achievement.name}</p>
                                <div className="progress-bar">
                                    <div style={{ width: `${(achievement.progress / achievement.target) * 100}%` }}></div>
                                    <span className="progress-text">{achievement.progress}/{achievement.target}</span>
                                </div>
                                {achievement.completed && <p style={typography.body}>Earned! 🏆</p>}
                            </div>
                        ))}
                    </div>
                    <div className="proficiencies">
                        <h2 style={typography.heading}>Your Progress</h2>
                        {proficiencies.map(prof => {
                            const currentMastery = thetaToMastery(prof.theta);
                            const target = targetMastery[prof.skill] || 75;
                            const progress = Math.min(100, (currentMastery / target) * 100);
                            return (
                                <div key={prof.id} style={commonStyles.card}>
                                    <p style={typography.body}>
                                        {prof.domain} - {prof.skill}: Mastery {currentMastery}% (Target: {target}%)
                                    </p>
                                    <p style={typography.body}>Progress Toward Target: {Math.round(progress)}%</p>
                                    {currentMastery >= target && <p style={typography.body}>Target Achieved! 🎉</p>}
                                </div>
                            );
                        })}
                    </div>
                    <div className="feedback">
                        <h2 style={typography.heading}>Feedback</h2>
                        <textarea
                            value={feedback}
                            onChange={(e) => setFeedback(e.target.value)}
                            placeholder="Share your feedback..."
                            className="feedback-input"
                        />
                        <button style={commonStyles.button} onClick={submitFeedback}>
                            <span style={commonStyles.buttonText}>Submit</span>
                        </button>
                    </div>
                </>
            )}

            <style jsx>{`
                .dashboard {
                    max-width: 900px;
                    margin: 50px auto;
                    text-align: center;
                }
                .diagnostic-results {
                    margin-bottom: 20px;
                }
                .score-history {
                    margin: 20px 0;
                }
                .daily-challenges {
                    margin: 20px 0;
                }
                .achievements {
                    margin: 20px 0;
                }
                .proficiencies {
                    margin: 20px 0;
                }
                .progress-bar {
                    background: #f0f0f0;
                    border-radius: 5px;
                    height: 20px;
                    position: relative;
                }
                .progress-bar div {
                    background: ${colors.primary};
                    height: 100%;
                    border-radius: 5px;
                }
                .progress-text {
                    position: absolute;
                    right: 10px;
                    top: 50%;
                    transform: translateY(-50%);
                    color: ${colors.text};
                }
                .feedback {
                    margin-top: 20px;
                }
                .feedback-input {
                    width: 100%;
                    height: 100px;
                    margin-bottom: 10px;
                    padding: 10px;
                    border: 1px solid ${colors.gray};
                    border-radius: 5px;
                }
            `}</style>
        </div>
    );
}
```

**6. `frontend/pages/onboarding.js`**

```javascript
import React, { useState, useEffect } from 'react';
import { useRouter } from 'next/router';
import { api } from '../utils/api';
import StudyPlan from './study-plan';
import { colors, typography, commonStyles } from '../styles';

const Onboarding = () => {
    const [step, setStep] = useState(1);
    const [userData, setUserData] = useState({
        user_id: null,
        name: '',
        email: '',
        sat_test_date: '',
        study_hours_per_week: 1,
        study_days_per_week: 1,
        preferred_study_time: 'Morning',
        target_mastery: {
            "Reading & Writing": 80,
            "Math": 80,
            "Reading Comprehension": 75,
            "Grammar": 75,
            "Vocabulary": 75,
            "Algebra": 75,
            "Geometry": 75,
            "Data Analysis": 75
        }
    });
    const [diagnosticResults, setDiagnosticResults] = useState(null);
    const router = useRouter();

    useEffect(() => {
        const userId = localStorage.getItem('user_id');
        if (userId) {
            setUserData(prev => ({ ...prev, user_id: userId }));
        }
    }, []);

    const nextStep = () => setStep(step + 1);
    const prevStep = () => setStep(step - 1);

    const handleInputChange = (key, value) => {
        setUserData(prev => ({ ...prev, [key]: value }));
    };

    const handleTargetMasteryChange = (key, value) => {
        setUserData(prev => ({
            ...prev,
            target_mastery: {
                ...prev.target_mastery,
                [key]: parseInt(value)
            }
        }));
    };

    const handleLogin = async () => {
        try {
            const res = await api.post('/auth/login', { email: userData.email });
            localStorage.setItem('user_id', res.data.user_id);
            setUserData(prev => ({ ...prev, user_id: res.data.user_id }));
            nextStep();
        } catch (error) {
            alert('Login failed: ' + error.message);
        }
    };

    const handleDiagnosticComplete = (results) => {
        setDiagnosticResults(results);
        nextStep();
    };

    const handleFinish = () => {
        router.push('/dashboard');
    };

    const WelcomeStep = () => (
        <div className="step">
            <h2 style={typography.heading}>Welcome to SAT Smart Prep!</h2>
            <p style={typography.body}>Let’s get started with your personalized SAT prep journey.</p>
            <button style={commonStyles.button} onClick={nextStep}>
                <span style={commonStyles.buttonText}>Next</span>
            </button>
        </div>
    );

    const LoginStep = () => (
        <div className="step">
            <h2 style={typography.heading}>Login</h2>
            <input
                type="email"
                placeholder="Email"
                value={userData.email}
                onChange={(e) => handleInputChange('email', e.target.value)}
                style={commonStyles.input}
            />
            <button style={commonStyles.button} onClick={handleLogin}>
                <span style={commonStyles.buttonText}>Login</span>
            </button>
        </div>
    );

    const NameStep = () => (
        <div className="step">
            <h2 style={typography.heading}>What’s Your Name?</h2>
            <input
                type="text"
                placeholder="Name"
                value={userData.name}
                onChange={(e) => handleInputChange('name', e.target.value)}
                style={commonStyles.input}
            />
            <div className="buttons">
                <button style={commonStyles.button} onClick={prevStep}>
                    <span style={commonStyles.buttonText}>Back</span>
                </button>
                <button style={commonStyles.button} onClick={nextStep} disabled={!userData.name}>
                    <span style={commonStyles.buttonText}>Next</span>
                </button>
            </div>
        </div>
    );

    const TestDateStep = () => (
        <div className="step">
            <h2 style={typography.heading}>When Are You Taking the SAT?</h2>
            <input
                type="date"
                value={userData.sat_test_date}
                onChange={(e) => handleInputChange('sat_test_date', e.target.value)}
                style={commonStyles.input}
            />
            <div className="buttons">
                <button style={commonStyles.button} onClick={prevStep}>
                    <span style={commonStyles.buttonText}>Back</span>
                </button>
                <button style={commonStyles.button} onClick={nextStep} disabled={!userData.sat_test_date}>
                    <span style={commonStyles.buttonText}>Next</span>
                </button>
            </div>
        </div>
    );

    const StudyPreferencesStep = () => (
        <div className="step">
            <h2 style={typography.heading}>Study Preferences</h2>
            <div className="form-group">
                <label style={typography.body}>Study Hours Per Week ({userData.study_hours_per_week} hours)</label>
                <input
                    type="range"
                    min="1"
                    max="20"
                    value={userData.study_hours_per_week}
                    onChange={(e) => handleInputChange('study_hours_per_week', parseInt(e.target.value))}
                    style={commonStyles.input}
                />
            </div>
            <div className="form-group">
                <label style={typography.body}>Study Days Per Week</label>
                <select
                    value={userData.study_days_per_week}
                    onChange={(e) => handleInputChange('study_days_per_week', parseInt(e.target.value))}
                    style={commonStyles.input}
                >
                    {[1, 2, 3, 4, 5, 6, 7].map(day => (
                        <option key={day} value={day}>{day}</option>
                    ))}
                </select>
            </div>
            <div className="form-group">
                <label style={typography.body}>Preferred Study Time</label>
                <select
                    value={userData.preferred_study_time}
                    onChange={(e) => handleInputChange('preferred_study_time', e.target.value)}
                    style={commonStyles.input}
                >
                    <option value="Morning">Morning</option>
                    <option value="Afternoon">Afternoon</option>
                    <option value="Evening">Evening</option>
                </select>
            </div>
            <h3 style={typography.heading}>Set Mastery Targets</h3>
            {Object.keys(userData.target_mastery).map(key => (
                <div key={key} className="form-group">
                    <label style={typography.body}>{key} Target Mastery ({userData.target_mastery[key]}%)</label>
                    <input
                        type="range"
                        min="50"
                        max="100"
                        value={userData.target_mastery[key]}
                        onChange={(e) => handleTargetMasteryChange(key, e.target.value)}
                        style={commonStyles.input}
                    />
                </div>
            ))}
            <div className="buttons">
                <button style={commonStyles.button} onClick={prevStep}>
                    <span style={commonStyles.buttonText}>Back</span>
                </button>
                <button style={commonStyles.button} onClick={nextStep}>
                    <span style={commonStyles.buttonText}>Next</span>
                </button>
            </div>
        </div>
    );

    const DiagnosticStep = () => (
        <div className="step">
            <h2 style={typography.heading}>Take a Diagnostic Test</h2>
            <p style={typography.body}>Complete a 49-question diagnostic test to assess your current level.</p>
            <button style={commonStyles.button} onClick={() => router.push('/diagnostic')}>
                <span style={commonStyles.buttonText}>Start Diagnostic Test</span>
            </button>
            <div className="buttons">
                <button style={commonStyles.button} onClick={prevStep}>
                    <span style={commonStyles.buttonText}>Back</span>
                </button>
            </div>
        </div>
    );

    const StudyPlanStep = () => (
        <div className="step">
            <h2 style={typography.heading}>Your Study Plan</h2>
            <StudyPlan userData={userData} />
            <div className="buttons">
                <button style={commonStyles.button} onClick={prevStep}>
                    <span style={commonStyles.buttonText}>Back</span>
                </button>
                <button style={commonStyles.button} onClick={handleFinish}>
                    <span style={commonStyles.buttonText}>Finish</span>
                </button>
            </div>
        </div>
    );

    return (
        <div className="onboarding">
            {step === 1 && <WelcomeStep />}
            {step === 2 && <LoginStep />}
            {step === 3 && <NameStep />}
            {step === 4 && <TestDateStep />}
            {step === 5 && <StudyPreferencesStep />}
            {step === 6 && <DiagnosticStep />}
            {step === 7 && <StudyPlanStep />}

            <style jsx>{`
                .onboarding {
                    max-width: 600px;
                    margin: 50px auto;
                    text-align: center;
                }
                .step {
                    padding: 20px;
                }
                .buttons {
                    display: flex;
                    justify-content: space-between;
                    margin-top: 20px;
                }
                .form-group {
                    margin-bottom: 20px;
                }
                .form-group label {
                    display: block;
                    margin-bottom: 5px;
                    font-weight: 500;
                }
                .form-group input,
                .form-group select {
                    width: 100%;
                    padding: 8px;
                    border: 1px solid ${colors.gray};
                    border-radius: 5px;
                }
            `}</style>
        </div>
    );
};

export default Onboarding;
```

**7. `frontend/pages/study-plan.js`**

```javascript
import React, { useState, useEffect } from 'react';
import { useRouter } from 'next/router';
import { api } from '../utils/api';
import { colors, typography, commonStyles } from '../styles';

// Helper function to convert theta (-3 to 3) to mastery percentage (0 to 100%)
const thetaToMastery = (theta) => {
    return Math.round(((theta + 3) / 6) * 100); // Linear transformation
};

export default function StudyPlan({ userData }) {
    const [studyPlan, setStudyPlan] = useState(null);
    const [editMode, setEditMode] = useState(false);
    const [studyHours, setStudyHours] = useState(userData?.study_hours_per_week || 1);
    const [studyDays, setStudyDays] = useState(userData?.study_days_per_week || 1);
    const [targetMastery, setTargetMastery] = useState({
        "Reading & Writing": 80,
        "Math": 80,
        "Reading Comprehension": 75,
        "Grammar": 75,
        "Vocabulary": 75,
        "Algebra": 75,
        "Geometry": 75,
        "Data Analysis": 75
    });
    const router = useRouter();
    const userId = localStorage.getItem('user_id');

    useEffect(() => {
        if (!userId) {
            router.push('/login');
        }
        fetchStudyPlan();
    }, [userId, router]);

    const fetchStudyPlan = async () => {
        try {
            const res = await api.get(`/study_plan/${userId}`);
            setStudyPlan(res.data);
            if (res.data.target_mastery) {
                setTargetMastery(res.data.target_mastery);
            }
        } catch (error) {
            if (error.status === 404) {
                const res = await api.post(`/study_plan/create/${userId}`, {
                    test_date: userData?.sat_test_date || new Date().toISOString(),
                    study_hours: userData?.study_hours_per_week || 1,
                    study_days: userData?.study_days_per_week || 1,
                    target_mastery: targetMastery
                });
                setStudyPlan(res.data);
                setTargetMastery(res.data.target_mastery);
            } else {
                alert('Failed to load study plan: ' + error.message);
            }
        }
    };

    const updateStudyPlan = async () => {
        const res = await api.post(`/study_plan/create/${userId}`, {
            test_date: userData.sat_test_date,
            study_hours: studyHours,
            study_days: studyDays,
            target_mastery: targetMastery
        });
        setStudyPlan(res.data);
        setTargetMastery(res.data.target_mastery);
        setEditMode(false);
    };

    const groupByPhase = (actions) => {
        const phases = {
            "Phase 1: Foundation Building": [],
            "Phase 2: Practice and Strategy": [],
            "Phase 3: Final Review and Refinement": []
        };
        actions.forEach(action => {
            phases[action.phase].push(action);
        });
        return phases;
    };

    const handleTargetMasteryChange = (key, value) => {
        setTargetMastery(prev => ({
            ...prev,
            [key]: parseInt(value)
        }));
    };

    return (
        <div className="study-plan">
            {studyPlan ? (
                <>
                    <h1 style={typography.heading}>Your Study Plan</h1>
                    <p style={typography.body}>Test Date: {new Date(studyPlan.test_date).toLocaleDateString()}</p>
                    <h2 style={typography.heading}>Mastery Targets</h2>
                    <div style={commonStyles.card}>
                        <h3 style={typography.heading}>Domains</h3>
                        <p style={typography.body}>
                            Reading & Writing: Current Mastery {Math.round(studyPlan.current_mastery["Reading & Writing"])}%, Target {studyPlan.target_mastery["Reading & Writing"]}%
                        </p>
                        <p style={typography.body}>
                            Math: Current Mastery {Math.round(studyPlan.current_mastery["Math"])}%, Target {studyPlan.target_mastery["Math"]}%
                        </p>
                        <h3 style={typography.heading}>Subskills</h3>
                        {Object.keys(studyPlan.current_mastery).filter(key => key !== "Reading & Writing" && key !== "Math").map(subskill => (
                            <p key={subskill} style={typography.body}>
                                {subskill}: Current Mastery {Math.round(studyPlan.current_mastery[subskill])}%, Target {studyPlan.target_mastery[subskill]}%
                                {studyPlan.current_mastery[subskill] >= studyPlan.target_mastery[subskill] && " (Target Achieved!)"}
                            </p>
                        ))}
                    </div>
                    <h2 style={typography.heading}>Tasks</h2>
                    {Object.entries(groupByPhase(studyPlan.actions)).map(([phase, actions]) => (
                        <div key={phase} className="phase">
                            <h3 style={typography.heading}>{phase}</h3>
                            {actions.map(action => (
                                <div key={action.id} style={commonStyles.card}>
                                    <p style={typography.body}>{action.task}</p>
                                    {action.subskill && (
                                        <p style={typography.body}>
                                            Target Mastery for {action.subskill}: {action.target_mastery}%
                                        </p>
                                    )}
                                    <p style={typography.body}>Due: {new Date(action.due_date).toLocaleDateString()}</p>
                                    <p style={typography.body}>Points: {action.points}</p>
                                    {action.completed ? (
                                        <p style={typography.body}>Completed on {new Date(action.completed).toLocaleDateString()}</p>
                                    ) : (
                                        <p style={typography.body}>Not yet completed</p>
                                    )}
                                </div>
                            ))}
                        </div>
                    ))}
                    <button style={commonStyles.button} onClick={() => setEditMode(true)}>
                        <span style={commonStyles.buttonText}>Adjust Study Plan</span>
                    </button>
                    {editMode && (
                        <div className="edit-plan">
                            <div className="form-group">
                                <label style={typography.body}>Study Hours Per Week ({studyHours} hours)</label>
                                <input
                                    type="range"
                                    min="1"
                                    max="20"
                                    value={studyHours}
                                    onChange={(e) => setStudyHours(parseInt(e.target.value))}
                                    className="slider"
                                />
                            </div>
                            <div className="form-group">
                                <label style={typography.body}>Study Days Per Week</label>
                                <select
                                    value={studyDays}
                                    onChange={(e) => setStudyDays(parseInt(e.target.value))}
                                    className="input"
                                >
                                    {[1, 2, 3, 4, 5, 6, 7].map(day => (
                                        <option key={day} value={day}>{day}</option>
                                    ))}
                                </select>
                            </div>
                            <h3 style={typography.heading}>Set Mastery Targets</h3>
                            {Object.keys(targetMastery).map(key => (
                                <div key={key} className="form-group">
                                    <label style={typography.body}>{key} Target Mastery ({targetMastery[key]}%)</label>
                                    <input
                                        type="range"
                                        min="50"
                                        max="100"
                                        value={targetMastery[key]}
                                        onChange={(e) => handleTargetMasteryChange(key, e.target.value)}
                                        className="slider"
                                    />
                                </div>
                            ))}
                            <button style={commonStyles.button} onClick={updateStudyPlan}>
                                <span style={commonStyles.buttonText}>Save Changes</span>
                            </button>
                        </div>
                    )}
                </>
            ) : (
                <p style={typography.body}>Loading your study plan...</p>
            )}

            <style jsx>{`
                .study-plan {
                    max-width: 900px;
                    margin: 50px auto;
                    text-align: center;
                }
                .phase {
                    margin-bottom: 30px;
                }
                .form-group {
                    margin-bottom: 20px;
                }
                .form-group label {
                    display: block;
                    margin-bottom: 5px;
                    font-weight: 500;
                }
                .form-group input,
                .form-group select {
                    width: 100%;
                    padding: 8px;
                    border: 1px solid ${colors.gray};
                    border-radius: 5px;
                }
                .slider {
                    width: 100%;
                }
            `}</style>
        </div>
    );
}
```

**8. `SATSmartPrepApp/src/screens/DashboardScreen.js`**

```javascript
import React, { useState, useEffect } from 'react';
import { View, Text, TextInput, TouchableOpacity, StyleSheet, Alert, ScrollView } from 'react-native';
import Animated, { FadeIn, FadeOut } from 'react-native-reanimated';
import { api } from '../utils/api';
import { colors, typography, commonStyles } from '../styles';

// Helper function to convert theta (-3 to 3) to mastery percentage (0 to 100%)
const thetaToMastery = (theta) => {
    return Math.round(((theta + 3) / 6) * 100); // Linear transformation
};

export default function DashboardScreen({ diagnosticResults, navigation }) {
    const [user, setUser] = useState(null);
    const [challenges, setChallenges] = useState([]);
    const [achievements, setAchievements] = useState([]);
    const [proficiencies, setProficiencies] = useState([]);
    const [scoreHistory, setScoreHistory] = useState([]);
    const [feedback, setFeedback] = useState('');
    const [targetMastery, setTargetMastery] = useState({});
    const userId = localStorage.getItem('user_id');

    useEffect(() => {
        api.get(`/gamification/coins/${userId}`).then(res => setUser({ coins: res.data.coins }));
        api.get(`/challenges/${userId}`).then(res => setChallenges(res.data));
        api.get(`/progress_monitoring/proficiencies/${userId}`).then(res => {
            setProficiencies(res.data.proficiencies);
            setTargetMastery(res.data.target_mastery);
        });
        api.get(`/progress_monitoring/scores/${userId}`).then(res => setScoreHistory(res.data));
        setAchievements([
            { id: 1, name: "Complete 100 Questions", target: 100, progress: 75, completed: false },
            { id: 2, name: "5-Day Streak", target: 5, progress: 3, completed: false }
        ]);
    }, []);

    const submitFeedback = async () => {
        if (!feedback) return;
        await api.post('/feedback', { user_id: userId, content: feedback });
        Alert.alert('Thank you for your feedback!');
        setFeedback('');
    };

    const checkGuarantee = async () => {
        const res = await api.get(`/progress_monitoring/guarantee/${userId}`);
        Alert.alert('Score Guarantee', res.data.message);
    };

    const updateStudyPlan = async () => {
        try {
            const res = await api.post(`/study_plan/update/${userId}`, {
                test_date: userData.sat_test_date,
                study_hours: userData.study_hours_per_week,
                study_days: userData.study_days_per_week
            });
            Alert.alert('Success', 'Study Plan updated successfully!');
            navigation.navigate('StudyPlan');
        } catch (error) {
            Alert.alert('Error', 'Failed to update Study Plan: ' + error.message);
        }
    };

    // Calculate overall mastery from diagnostic results
    const overallMastery = diagnosticResults ? thetaToMastery(diagnosticResults.theta) : 0;
    const estimatedScore = diagnosticResults ? (diagnosticResults.theta * 400 + 400) : 0;

    // Calculate domain-level mastery
    const readingMastery = diagnosticResults ? thetaToMastery(diagnosticResults.reading_theta || 0) : 0;
    const mathMastery = diagnosticResults ? thetaToMastery(diagnosticResults.math_theta || 0) : 0;

    // Calculate progress toward targets
    const readingProgress = targetMastery["Reading & Writing"] ? Math.min(100, (readingMastery / targetMastery["Reading & Writing"]) * 100) : 0;
    const mathProgress = targetMastery["Math"] ? Math.min(100, (mathMastery / targetMastery["Math"]) * 100) : 0;

    return (
        <Animated.View entering={FadeIn} exiting={FadeOut} style={styles.container}>
            <ScrollView>
                {diagnosticResults ? (
                    <View style={styles.diagnosticResults}>
                        <Text style={typography.heading}>Your Diagnostic Results</Text>
                        <Text style={typography.body}>Overall Mastery: {overallMastery}%</Text>
                        <Text style={typography.body}>Estimated SAT Score: {estimatedScore}</Text>
                        <Text style={typography.heading}>Math</Text>
                        <Text style={typography.body}>Math Mastery: {mathMastery}% (Target: {targetMastery["Math"] || 80}%)</Text>
                        <Text style={typography.body}>Progress Toward Target: {Math.round(mathProgress)}%</Text>
                        {mathMastery >= (targetMastery["Math"] || 80) && <Text style={typography.body}>Target Achieved! 🎉</Text>}
                        <Text style={typography.body}>Strong Areas: Algebra (Mastery: {thetaToMastery(diagnosticResults.math_theta || 0)}%)</Text>
                        <Text style={typography.body}>Weak Areas: Geometry (Mastery: {thetaToMastery((diagnosticResults.math_theta || 0) - 0.5)}%)</Text>
                        <Text style={typography.heading}>Reading & Writing</Text>
                        <Text style={typography.body}>Reading & Writing Mastery: {readingMastery}% (Target: {targetMastery["Reading & Writing"] || 80}%)</Text>
                        <Text style={typography.body}>Progress Toward Target: {Math.round(readingProgress)}%</Text>
                        {readingMastery >= (targetMastery["Reading & Writing"] || 80) && <Text style={typography.body}>Target Achieved! 🎉</Text>}
                        <Text style={typography.body}>Strong Areas: Reading Comprehension (Mastery: {thetaToMastery(diagnosticResults.reading_theta || 0)}%)</Text>
                        <Text style={typography.body}>Weak Areas: Grammar (Mastery: {thetaToMastery((diagnosticResults.reading_theta || 0) - 0.5)}%)</Text>
                        <Text style={typography.body}>Key Recommendation: Focus on Geometry and Grammar to improve your mastery.</Text>
                        <TouchableOpacity
                            style={commonStyles.button}
                            onPress={() => navigation.navigate('Review', { sessionId: diagnosticResults.session_id })}
                        >
                            <Text style={commonStyles.buttonText}>Review Diagnostic Test</Text>
                        </TouchableOpacity>
                    </View>
                ) : (
                    <>
                        {user && <Text style={typography.body}>Coins: {user.coins} 🪙</Text>}
                        <View style={styles.scoreHistory}>
                            <Text style={typography.heading}>Score History</Text>
                            {scoreHistory.map((entry, idx) => (
                                <Text key={idx} style={typography.body}>
                                    {new Date(entry.timestamp).toLocaleDateString()}: {entry.score} (Mastery: {thetaToMastery(entry.theta)}%)
                                </Text>
                            ))}
                            <TouchableOpacity style={commonStyles.button} onPress={checkGuarantee}>
                                <Text style={commonStyles.buttonText}>Check Score Guarantee</Text>
                            </TouchableOpacity>
                            <TouchableOpacity style={commonStyles.button} onPress={() => navigation.navigate('Policy')}>
                                <Text style={commonStyles.buttonText}>View Guarantee Policy</Text>
                            </TouchableOpacity>
                            <TouchableOpacity style={commonStyles.button} onPress={updateStudyPlan}>
                                <Text style={commonStyles.buttonText}>Update Study Plan</Text>
                            </TouchableOpacity>
                        </View>
                        <View style={styles.dailyChallenges}>
                            <Text style={typography.heading}>Daily Challenges</Text>
                            {challenges.map(challenge => (
                                <View key={challenge.id} style={commonStyles.card}>
                                    <Text style={typography.body}>
                                        {challenge.challenge_type === 'daily_questions' ? "Answer 10 Questions Correctly" : "Complete a 5-Question Session in Under 5 Minutes"}
                                    </Text>
                                    <View style={styles.progressBar}>
                                        <View style={[styles.progressFill, { width: `${(challenge.progress / challenge.target) * 100}%` }]} />
                                        <Text style={styles.progressText}>{challenge.progress}/{challenge.target}</Text>
                                    </View>
                                    {challenge.completed && <Text style={typography.body}>Completed! 🎉</Text>}
                                </View>
                            ))}
                        </View>
                        <View style={styles.achievements}>
                            <Text style={typography.heading}>Achievements</Text>
                            {achievements.map(achievement => (
                                <View key={achievement.id} style={commonStyles.card}>
                                    <Text style={typography.body}>{achievement.name}</Text>
                                    <View style={styles.progressBar}>
                                        <View style={[styles.progressFill, { width: `${(achievement.progress / achievement.target) * 100}%` }]} />
                                        <Text style={styles.progressText}>{achievement.progress}/{achievement.target}</Text>
                                    </View>
                                    {achievement.completed && <Text style={typography.body}>Earned! 🏆</Text>}
                                </View>
                            ))}
                        </View>
                        <View style={styles.proficiencies}>
                            <Text style={typography.heading}>Your Progress</Text>
                            {proficiencies.map(prof => {
                                const currentMastery = thetaToMastery(prof.theta);
                                const target = targetMastery[prof.skill] || 75;
                                const progress = Math.min(100, (currentMastery / target) * 100);
                                return (
                                    <View key={prof.id} style={commonStyles.card}>
                                        <Text style={typography.body}>
                                            {prof.domain} - {prof.skill}: Mastery {currentMastery}% (Target: {target}%)
                                        </Text>
                                        <Text style={typography.body}>Progress Toward Target: {Math.round(progress)}%</Text>
                                        {currentMastery >= target && <Text style={typography.body}>Target Achieved! 🎉</Text>}
                                    </View>
                                );
                            })}
                        </View>
                        <View style={styles.feedback}>
                            <Text style={typography.heading}>Feedback</Text>
                            <TextInput
                                style={styles.feedbackInput}
                                value={feedback}
                                onChangeText={setFeedback}
                                placeholder="Share your feedback..."
                                multiline
                            />
                            <TouchableOpacity style={commonStyles.button} onPress={submitFeedback}>
                                <Text style={commonStyles.buttonText}>Submit</Text>
                            </TouchableOpacity>
                        </View>
                    </>
                )}
            </ScrollView>
        </Animated.View>
    );
};

const styles = StyleSheet.create({
    container: {
        flex: 1,
        padding: 20,
        backgroundColor: colors.white,
    },
    diagnosticResults: {
        marginBottom: 20,
    },
    scoreHistory: {
        marginVertical: 20,
    },
    dailyChallenges: {
        marginVertical: 20,
    },
    achievements: {
        marginVertical: 20,
    },
    proficiencies: {
        marginVertical: 20,
    },
    progressBar: {
        backgroundColor: '#f0f0f0',
        borderRadius: 5,
        height: 20,
        position: 'relative',
    },
    progressFill: {
        backgroundColor: colors.primary,
        height: '100%',
        borderRadius: 5,
    },
    progressText: {
        position: 'absolute',
        right: 10,
        top: '50%',
        transform: [{ translateY: -10 }],
        color: colors.text,
    },
    feedback: {
        marginTop: 20,
    },
    feedbackInput: {
        width: '100%',
        height: 100,
        marginBottom: 10,
        padding: 10,
        borderWidth: 1,
        borderColor: colors.gray,
        borderRadius: 5,
    },
});
```

**9. `SATSmartPrepApp/src/screens/OnboardingScreen.js`**

```javascript
import React, { useState, useEffect } from 'react';
import { View, Text, TextInput, Picker, TouchableOpacity, StyleSheet, Alert } from 'react-native';
import Animated, { FadeIn, FadeOut } from 'react-native-reanimated';
import { api } from '../utils/api';
import { colors, typography, commonStyles } from '../styles';

const OnboardingScreen = ({ navigation }) => {
    const [step, setStep] = useState(1);
    const [userData, setUserData] = useState({
        user_id: null,
        name: '',
        email: '',
        sat_test_date: '',
        study_hours_per_week: 1,
        study_days_per_week: 1,
        preferred_study_time: 'Morning',
        target_mastery: {
            "Reading & Writing": 80,
            "Math": 80,
            "Reading Comprehension": 75,
            "Grammar": 75,
            "Vocabulary": 75,
            "Algebra": 75,
            "Geometry": 75,
            "Data Analysis": 75
        }
    });

    useEffect(() => {
        const userId = localStorage.getItem('user_id');
        if (userId) {
            setUserData(prev => ({ ...prev, user_id: userId }));
        }
    }, []);

    const nextStep = () => setStep(step + 1);
    const prevStep = () => setStep(step - 1);

    const handleInputChange = (key, value) => {
        setUserData(prev => ({ ...prev, [key]: value }));
    };

    const handleTargetMasteryChange = (key, value) => {
        setUserData(prev => ({
            ...prev,
            target_mastery: {
                ...prev.target_mastery,
                [key]: parseInt(value)
            }
        }));
    };

    const handleLogin = async () => {
        try {
            const res = await api.post('/auth/login', { email: userData.email });
            localStorage.setItem('user_id', res.data.user_id);
            setUserData(prev => ({ ...prev, user_id: res.data.user_id }));
            nextStep();
        } catch (error) {
            Alert.alert('Login Failed', error.message);
        }
    };

    const handleFinish = () => {
        navigation.navigate('Dashboard');
    };

    const WelcomeStep = () => (
        <View style={styles.step}>
            <Text style={typography.heading}>Welcome to SAT Smart Prep!</Text>
            <Text style={typography.body}>Let’s get started with your personalized SAT prep journey.</Text>
            <TouchableOpacity style={commonStyles.button} onPress={nextStep}>
                <Text style={commonStyles.buttonText}>Next</Text>
            </TouchableOpacity>
        </View>
    );

    const LoginStep = () => (
        <View style={styles.step}>
            <Text style={typography.heading}>Login</Text>
            <TextInput
                style={styles.input}
                placeholder="Email"
                value={userData.email}
                onChangeText={(value) => handleInputChange('email', value)}
                keyboardType="email-address"
            />
            <TouchableOpacity style={commonStyles.button} onPress={handleLogin}>
                <Text style={commonStyles.buttonText}>Login</Text>
            </TouchableOpacity>
        </View>
    );

    const NameStep = () => (
        <View style={styles.step}>
            <Text style={typography.heading}>What’s Your Name?</Text>
            <TextInput
                style={styles.input}
                placeholder="Name"
                value={userData.name}
                onChangeText={(value) => handleInputChange('name', value)}
            />
            <View style={styles.buttonRow}>
                <TouchableOpacity style={commonStyles.button} onPress={prevStep}>
                    <Text style={commonStyles.buttonText}>Back</Text>
                </TouchableOpacity>
                <TouchableOpacity
                    style={[commonStyles.button, !userData.name && styles.disabledButton]}
                    onPress={nextStep}
                    disabled={!userData.name}
                >
                    <Text style={commonStyles.buttonText}>Next</Text>
                </TouchableOpacity>
            </View>
        </View>
    );

    const TestDateStep = () => (
        <View style={styles.step}>
            <Text style={typography.heading}>When Are You Taking the SAT?</Text>
            <TextInput
                style={styles.input}
                placeholder="YYYY-MM-DD"
                value={userData.sat_test_date}
                onChangeText={(value) => handleInputChange('sat_test_date', value)}
            />
            <View style={styles.buttonRow}>
                <TouchableOpacity style={commonStyles.button} onPress={prevStep}>
                    <Text style={commonStyles.buttonText}>Back</Text>
                </TouchableOpacity>
                <TouchableOpacity
                    style={[commonStyles.button, !userData.sat_test_date && styles.disabledButton]}
                    onPress={nextStep}
                    disabled={!userData.sat_test_date}
                >
                    <Text style={commonStyles.buttonText}>Next</Text>
                </TouchableOpacity>
            </View>
        </View>
    );

    const StudyPreferencesStep = () => (
        <View style={styles.step}>
            <Text style={typography.heading}>Study Preferences</Text>
            <View style={styles.formGroup}>
                <Text style={typography.body}>Study Hours Per Week ({userData.study_hours_per_week} hours)</Text>
                <TextInput
                    style={styles.input}
                    value={userData.study_hours_per_week.toString()}
                    onChangeText={(value) => handleInputChange('study_hours_per_week', parseInt(value))}
                    keyboardType="numeric"
                />
            </View>
            <View style={styles.formGroup}>
                <Text style={typography.body}>Study Days Per Week</Text>
                <Picker
                    selectedValue={userData.study_days_per_week}
                    onValueChange={(value) => handleInputChange('study_days_per_week', value)}
                    style={styles.input}
                >
                    {[1, 2, 3, 4, 5, 6, 7].map(day => (
                        <Picker.Item key={day} label={day.toString()} value={day} />
                    ))}
                </Picker>
            </View>
            <View style={styles.formGroup}>
                <Text style={typography.body}>Preferred Study Time</Text>
                <Picker
                    selectedValue={userData.preferred_study_time}
                    onValueChange={(value) => handleInputChange('preferred_study_time', value)}
                    style={styles.input}
                >
                    <Picker.Item label="Morning" value="Morning" />
                    <Picker.Item label="Afternoon" value="Afternoon" />
                    <Picker.Item label="Evening" value="Evening" />
                </Picker>
            </View>
            <Text style={typography.heading}>Set Mastery Targets</Text>
            {Object.keys(userData.target_mastery).map(key => (
                <View key={key} style={styles.formGroup}>
                    <Text style={typography.body}>{key} Target Mastery ({userData.target_mastery[key]}%)</Text>
                    <TextInput
                        style={styles.input}
                        value={userData.target_mastery[key].toString()}
                        onChangeText={(value) => handleTargetMasteryChange(key, value)}
                        keyboardType="numeric"
                    />
                </View>
            ))}
            <View style={styles.buttonRow}>
                <TouchableOpacity style={commonStyles.button} onPress={prevStep}>
                    <Text style={commonStyles.buttonText}>Back</Text>
                </TouchableOpacity>
                <TouchableOpacity style={commonStyles.button} onPress={nextStep}>
                    <Text style={commonStyles.buttonText}>Next</Text>
                </TouchableOpacity>
            </View>
        </View>
    );

    const DiagnosticStep = () => (
        <View style={styles.step}>
            <Text style={typography.heading}>Take a Diagnostic Test</Text>
            <Text style={typography.body}>Complete a 49-question diagnostic test to assess your current level.</Text>
            <TouchableOpacity style={commonStyles.button} onPress={() => navigation.navigate('Diagnostic')}>
                <Text style={commonStyles.buttonText}>Start Diagnostic Test</Text>
            </TouchableOpacity>
            <View style={styles.buttonRow}>
                <TouchableOpacity style={commonStyles.button} onPress={prevStep}>
                    <Text style={commonStyles.buttonText}>Back</Text>
                </TouchableOpacity>
            </View>
        </View>
    );

    const StudyPlanStep = () => (
        <View style={styles.step}>
            <Text style={typography.heading}>Your Study Plan</Text>
            <Text style={typography.body}>Your study plan will be created based on your preferences.</Text>
            <View style={styles.buttonRow}>
                <TouchableOpacity style={commonStyles.button} onPress={prevStep}>
                    <Text style={commonStyles.buttonText}>Back</Text>
                </TouchableOpacity>
                <TouchableOpacity style={commonStyles.button} onPress={handleFinish}>
                    <Text style={commonStyles.buttonText}>Finish</Text>
                </TouchableOpacity>
            </View>
        </View>
    );

    return (
        <Animated.View entering={FadeIn} exiting={FadeOut} style={styles.container}>
            {step === 1 && <WelcomeStep />}
            {step === 2 && <LoginStep />}
            {step === 3 && <NameStep />}
            {step === 4 && <TestDateStep />}
            {step === 5 && <StudyPreferencesStep />}
            {step === 6 && <DiagnosticStep />}
            {step === 7 && <StudyPlanStep />}
        </Animated.View>
    );
};

const styles = StyleSheet.create({
    container: {
        flex: 1,
        padding: 20,
        backgroundColor: colors.white,
        alignItems: 'center',
    },
    step: {
        width: '100%',
        alignItems: 'center',
    },
    formGroup: {
        marginBottom: 20,
        width: '100%',
    },
    input: {
        width: '100%',
        padding: 8,
        borderWidth: 1,
        borderColor: colors.gray,
        borderRadius: 5,
    },
    buttonRow: {
        flexDirection: 'row',
        justifyContent: 'space-between',
        width: '100%',
        marginTop: 20,
    },
    disabledButton: {
        backgroundColor: colors.gray,
    },
});

export default OnboardingScreen;
```

**10. `SATSmartPrepApp/src/screens/StudyPlanScreen.js`**

```javascript
import React, { useState, useEffect } from 'react';
import { View, Text, TextInput, Picker, TouchableOpacity, StyleSheet, Alert, ScrollView } from 'react-native';
import Animated, { FadeIn, FadeOut } from 'react-native-reanimated';
import { api } from '../utils/api';
import { colors, typography, commonStyles } from '../styles';

// Helper function to convert theta (-3 to 3) to mastery percentage (0 to 100%)
const thetaToMastery = (theta) => {
    return Math.round(((theta + 3) / 6) * 100); // Linear transformation
};

const StudyPlanScreen = ({ userData }) => {
    const [studyPlan, setStudyPlan] = useState(null);
    const [editMode, setEditMode] = useState(false);
    const [studyHours, setStudyHours] = useState(userData?.study_hours_per_week || 1);
    const [studyDays, setStudyDays] = useState(userData?.study_days_per_week || 1);
    const [targetMastery, setTargetMastery] = useState({
        "Reading & Writing": 80,
        "Math": 80,
        "Reading Comprehension": 75,
        "Grammar": 75,
        "Vocabulary": 75,
        "Algebra": 75,
        "Geometry": 75,
        "Data Analysis": 75
    });
    const userId = localStorage.getItem('user_id');

    useEffect(() => {
        fetchStudyPlan();
    }, []);

    const fetchStudyPlan = async () => {
        try {
            const res = await api.get(`/study_plan/${userId}`);
            setStudyPlan(res.data);
            if (res.data.target_mastery) {
                setTargetMastery(res.data.target_mastery);
            }
        } catch (error) {
            if (error.status === 404) {
                const res = await api.post(`/study_plan/create/${userId}`, {
                    test_date: userData?.sat_test_date || new Date().toISOString(),
                    study_hours: userData?.study_hours_per_week || 1,
                    study_days: userData?.study_days_per_week || 1,
                    target_mastery: targetMastery
                });
                setStudyPlan(res.data);
                setTargetMastery(res.data.target_mastery);
            } else {
                Alert.alert('Error', 'Failed to load study plan: ' + error.message);
            }
        }
    };

    const updateStudyPlan = async () => {
        const res = await api.post(`/study_plan/create/${userId}`, {
            test_date: userData.sat_test_date,
            study_hours: studyHours,
            study_days: studyDays,
            target_mastery: targetMastery
        });
        setStudyPlan(res.data);
        setTargetMastery(res.data.target_mastery);
        setEditMode(false);
    };

    const groupByPhase = (actions) => {
        const phases = {
            "Phase 1: Foundation Building": [],
            "Phase 2: Practice and Strategy": [],
            "Phase 3: Final Review and Refinement": []
        };
        actions.forEach(action => {
            phases[action.phase].push(action);
        });
        return phases;
    };

    const handleTargetMasteryChange = (key, value) => {
        setTargetMastery(prev => ({
            ...prev,
            [key]: parseInt(value)
        }));
    };

    return (
        <Animated.View entering={FadeIn} exiting={FadeOut} style={styles.container}>
            <ScrollView>
                {studyPlan ? (
                    <>
                        <Text style={typography.heading}>Your Study Plan</Text>
                        <Text style={typography.body}>Test Date: {new Date(studyPlan.test_date).toLocaleDateString()}</Text>
                        <Text style={typography.heading}>Mastery Targets</Text>
                        <View style={commonStyles.card}>
                            <Text style={typography.heading}>Domains</Text>
                            <Text style={typography.body}>
                                Reading & Writing: Current Mastery {Math.round(studyPlan.current_mastery["Reading & Writing"])}%, Target {studyPlan.target_mastery["Reading & Writing"]}%
                            </Text>
                            <Text style={typography.body}>
                                Math: Current Mastery {Math.round(studyPlan.current_mastery["Math"])}%, Target {studyPlan.target_mastery["Math"]}%
                            </Text>
                            <Text style={typography.heading}>Subskills</Text>
                            {Object.keys(studyPlan.current_mastery).filter(key => key !== "Reading & Writing" && key !== "Math").map(subskill => (
                                <Text key={subskill} style={typography.body}>
                                    {subskill}: Current Mastery {Math.round(studyPlan.current_mastery[subskill])}%, Target {studyPlan.target_mastery[subskill]}%
                                    {studyPlan.current_mastery[subskill] >= studyPlan.target_mastery[subskill] && " (Target Achieved!)"}
                                </Text>
                            ))}
                        </View>
                        <Text style={typography.heading}>Tasks</Text>
                        {Object.entries(groupByPhase(studyPlan.actions)).map(([phase, actions]) => (
                            <View key={phase} style={styles.phase}>
                                <Text style={typography.heading}>{phase}</Text>
                                {actions.map(action => (
                                    <View key={action.id} style={commonStyles.card}>
                                        <Text style={typography.body}>{action.task}</Text>
                                        {action.subskill && (
                                            <Text style={typography.body}>
                                                Target Mastery for {action.subskill}: {action.target_mastery}%
                                            </Text>
                                        )}
                                        <Text style={typography.body}>Due: {new Date(action.due_date).toLocaleDateString()}</Text>
                                        <Text style={typography.body}>Points: {action.points}</Text>
                                        {action.completed ? (
                                            <Text style={typography.body}>Completed on {new Date(action.completed).toLocaleDateString()}</Text>
                                        ) : (
                                            <Text style={typography.body}>Not yet completed</Text>
                                        )}
                                    </View>
                                ))}
                            </View>
                        ))}
                        <TouchableOpacity style={commonStyles.button} onPress={() => setEditMode(true)}>
                            <Text style={commonStyles.buttonText}>Adjust Study Plan</Text>
                        </TouchableOpacity>
                        {editMode && (
                            <View style={styles.editPlan}>
                                <View style={styles.formGroup}>
                                    <Text style={typography.body}>Study Hours Per Week ({studyHours} hours)</Text>
                                    <TextInput
                                        style={styles.input}
                                        value={studyHours.toString()}
                                        onChangeText={(value) => setStudyHours(parseInt(value))}
                                        keyboardType="numeric"
                                    />
                                </View>
                                <View style={styles.formGroup}>
                                    <Text style={typography.body}>Study Days Per Week</Text>
                                    <Picker
                                        selectedValue={studyDays}
                                        onValueChange={(value) => setStudyDays(value)}
                                        style={styles.input}
                                    >
                                        {[1, 2, 3, 4, 5, 6, 7].map(day => (
                                            <Picker.Item key={day} label={day.toString()} value={day} />
                                        ))}
                                    </Picker>
                                </View>
                                <Text style={typography.heading}>Set Mastery Targets</Text>
                                {Object.keys(targetMastery).map(key => (
                                    <View key={key} style={styles.formGroup}>
                                        <Text style={typography.body}>{key} Target Mastery ({targetMastery[key]}%)</Text>
                                        <TextInput
                                            style={styles.input}
                                            value={targetMastery[key].toString()}
                                            onChangeText={(value) => handleTargetMasteryChange(key, value)}
                                            keyboardType="numeric"
                                        />
                                    </View>
                                ))}
                                <TouchableOpacity style={commonStyles.button} onPress={updateStudyPlan}>
                                    <Text style={commonStyles.buttonText}>Save Changes</Text>
                                </TouchableOpacity>
                            </View>
                        )}
                    </>
                ) : (
                    <Text style={typography.body}>Loading your study plan...</Text>
                )}
            </ScrollView>
        </Animated.View>
    );
};

const styles = StyleSheet.create({
    container: {
        flex: 1,
        padding: 20,
        backgroundColor: colors.white,
        alignItems: 'center',
    },
    phase: {
        marginBottom: 30,
    },
    formGroup: {
        marginBottom: 20,
        width: '100%',
    },
    input: {
        width: '100%',
        padding: 8,
        borderWidth: 1,
        borderColor: colors.gray,
        borderRadius: 5,
    },
    editPlan: {
        marginTop: 20,
        width: '100%',
    },
});
```

***

### Step 3: Instructions to Download and Upload to GitHub

#### 1. Create the Directory Structure Locally

* Create a directory named `SATSmartPrepApp_updated_files` on your local machine.
* Inside this directory, create the following subdirectories:
  * `backend/src/`
  * `frontend/pages/`
  * `SATSmartPrepApp/src/screens/`

#### 2. Copy the Files

* Copy each file’s content into the corresponding file in the directory structure:
  * `backend/src/diagnostic.py`
  * `backend/src/practice.py`
  * `backend/src/progress_monitoring.py`
  * `backend/src/study_plan.py`
  * `frontend/pages/dashboard.js`
  * `frontend/pages/onboarding.js`
  * `frontend/pages/study-plan.js`
  * `SATSmartPrepApp/src/screens/DashboardScreen.js`
  * `SATSmartPrepApp/src/screens/OnboardingScreen.js`
  * `SATSmartPrepApp/src/screens/StudyPlanScreen.js`

#### 3. Upload to GitHub

* **Navigate to Your Repository**:
  * Go to your GitHub repository where the SAT Smart Prep App is hosted.
* **Create a New Branch** (optional but recommended):
  *   Run the following commands in your local repository:

      ```bash
      git checkout -b updates-since-midnight-2025-03-28
      ```
* **Copy the Updated Files**:
  * Copy the files from `SATSmartPrepApp_updated_files` into your local repository, overwriting the existing files in the corresponding directories.
* **Stage and Commit the Changes**:
  *   Run the following commands:

      ```bash
      git add .
      git commit -m "Updated files since midnight on 2025-03-28: Added mastery targets, IRT 3PL model, and related features"
      ```
* **Push to GitHub**:
  *   Push the changes to your GitHub repository:

      ```bash
      git push origin updates-since-midnight-2025-03-28
      ```
* **Create a Pull Request** (if applicable):
  * Go to your GitHub repository in the browser, and create a pull request from the `updates-since-midnight-2025-03-28` branch to your main branch.

***

### Verification

* **All Files Included**: The package includes all 10 files updated since midnight on March 28, 2025, as listed in the table.
* **Contents Complete**: Each file’s content is fully provided, including the completed `study_plan.py` file that was previously interrupted.
* **Instructions Provided**: Clear steps are given to copy the files locally and upload them to GitHub.

Let me know if you need further assistance with uploading to GitHub or if you’d like to make additional changes!
