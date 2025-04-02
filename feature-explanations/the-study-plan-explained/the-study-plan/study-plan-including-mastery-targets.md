# Study plan including mastery targets

Let’s add **mastery targets** to the **Study Plan** in the **SAT Smart Prep App** by **Learner Labs**. Mastery targets will provide users with specific goals for each domain (Reading & Writing, Math) and subskill (e.g., Grammar, Algebra), expressed as a percentage on the 0 to 100% mastery scale introduced earlier. These targets will be integrated into the Study Plan to guide users toward achieving a desired level of proficiency by their SAT test date. The backend will continue to use `theta` scores for calculations, while the UI will display mastery percentages. I’ll update the relevant code to implement this feature, ensuring that the Study Plan includes mastery targets and adjusts task scheduling to help users meet these goals.

***

### Adding Mastery Targets to the Study Plan

#### Feature Overview

* **Mastery Targets**: Each domain and subskill will have a target mastery percentage (e.g., 80% for Reading & Writing, 75% for Algebra). These targets represent the desired proficiency level the user should aim to achieve by their SAT test date.
* **Dynamic Task Scheduling**: The Study Plan will use the difference between the user’s current mastery and their target mastery to allocate tasks. Subskills with a larger gap between current and target mastery will receive more practice sessions.
* **UI Integration**: The Study Plan UI will display the mastery targets alongside the user’s current mastery, providing a clear goal (e.g., "Reading & Writing: Current Mastery 60%, Target 80%").
* **Backend Logic**: The backend will calculate the `theta` equivalent of the target mastery (e.g., 80% mastery corresponds to a `theta` of 1.8) and use this to determine the number of practice sessions needed to close the gap.

#### Implementation Plan

1. **Define Mastery Targets**:
   * Set default target mastery percentages for domains and subskills (e.g., 80% for domains, 75% for subskills).
   * Allow users to customize these targets during onboarding or Study Plan adjustments.
2. **Update Study Plan Logic**:
   * Calculate the `theta` equivalent of the target mastery for each domain and subskill.
   * Adjust task allocation in the Study Plan based on the gap between the user’s current `theta` and the target `theta`.
3. **Update UI**:
   * Display mastery targets in the Study Plan UI alongside current mastery percentages.
   * Show progress toward targets (e.g., "You’re 75% of the way to your Algebra target of 75%").
4. **Backend Calculations**:
   * Use `theta` for all calculations, converting to mastery percentages only for display purposes.

***

### Updated Code

#### 1. Update the Study Plan Backend - `backend/src/study_plan.py`

We’ll modify the `/study_plan/create` and `/study_plan/update` endpoints to include mastery targets and adjust task allocation based on the gap between current and target mastery.

```python
from fastapi import APIRouter, HTTPException
from pydantic import BaseModel
from psycopg2.extras import RealDictCursor
import psycopg2
import uuid
from datetime import datetime, date, timedelta

router = APIRouter()

def get_db_connection():
    return psycopg2.connect(
        dbname="satprep",
        user="user",
        password="password",
        host="localhost",
        port="5432"
    )

# Helper function to convert mastery percentage (0 to 100%) to theta (-3 to 3)
def mastery_to_theta(mastery):
    return (mastery / 100) * 6 - 3  # Inverse of thetaToMastery

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
                    task = "Take a full-length SAT practice test"
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
                    task = "Take a full-length SAT practice test"
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
                    task = "Take a full-length SAT practice test"
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
                    task = "Take a full-length SAT practice test"
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
                    task = "Take a full-length SAT practice test"
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
                    task = "Take a full-length SAT practice test"
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

#### 2. Update the Study Plan Screen (Web) - `frontend/pages/study-plan.js`

We’ll update the Study Plan screen to display mastery targets and current mastery percentages for domains and subskills, along with progress toward those targets.

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

#### 3. Update the Study Plan Screen (Mobile) - `SATSmartPrepApp/src/screens/StudyPlanScreen.js`

The mobile app’s Study Plan screen will also be updated to display mastery targets and current mastery percentages.

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

#### 4. Update the Onboarding Screen (Web) - `frontend/pages/onboarding.js`

We’ll update the onboarding process to allow users to set mastery targets during the Study Plan step.

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
               
```
