# The Study Plan

Let’s update the **Study Plan Creation Process** in the **SAT Smart Prep App** by **Learner Labs** to include a specific distribution of full-length tests across the three phases, as requested: **2 full-length tests in Phase 1 (Foundation Building)**, **2 in Phase 2 (Practice and Strategy)**, and **4 in Phase 3 (Final Review and Refinement)**, totaling **8 full-length tests**. This change ensures a structured progression of test practice throughout the preparation period, aligning with the updated phase durations (40% for Phase 1, 30% for Phase 2, 30% for Phase 3) and the integration of advanced analytics (performance trends, improvement rates, and time spent per question) from the previous update.

***

### Updated Study Plan Creation Process

#### Changes to Full-Length Test Distribution

* **Current Distribution**:
  * Total full-length tests: 6–8, dynamically scheduled across Phases 2 and 3.
  * Phase 1: 0 full-length tests.
  * Phase 2: 4–5 full-length tests.
  * Phase 3: 2–3 full-length tests.
* **New Distribution**:
  * Total full-length tests: 8 (fixed, as specified).
  * Phase 1: 2 full-length tests.
  * Phase 2: 2 full-length tests.
  * Phase 3: 4 full-length tests.
* **Impact**:
  * Introducing 2 full-length tests in Phase 1 allows users to experience the SAT format early in their preparation, helping them identify pacing and endurance issues.
  * Reducing the number of tests in Phase 2 to 2 focuses this phase more on strategy and targeted practice, while still providing test experience.
  * Increasing Phase 3 to 4 tests ensures a strong final review period, allowing users to simulate the test multiple times before the actual SAT.

#### Updated Flow and Logic

**Step 1: Collect User Inputs (Unchanged)**

* **Inputs**:
  * User preferences: `sat_test_date`, `study_hours_per_week`, `study_days_per_week`.
  * Diagnostic Test results: `theta` scores for each domain and subskill (from `Proficiencies`).

**Step 2: Calculate Study Plan Parameters with Updated Test Distribution**

* **Backend** (`frontend/backend/src/study_plan.py`):
  * **Logic**:
    1. **Determine Study Duration and Phases** (Unchanged):
       * Calculate total weeks until the test date.
       * Divide into three phases: Phase 1 (40%), Phase 2 (30%), Phase 3 (30%).
    2. **Calculate Total Study Sessions** (Unchanged):
       * Compute total sessions based on study hours and days.
       * Allocate sessions to each phase.
    3. **Fetch Latest Diagnostic Results and Analytics** (Unchanged):
       * Query the `Proficiencies` table for the most recent `theta` scores.
       * Fetch historical `theta` scores for performance trends and improvement rates.
       * Fetch `time_spent` data from the `Responses` table for pacing analysis.
    4. **Analyze Advanced Analytics** (Unchanged):
       * Calculate performance trends, improvement rates, and pacing issues.
    5. **Adjust Session Allocation Based on Analytics** (Unchanged):
       * Prioritize subskills with low `theta`, negative trends, slow improvement rates, and pacing issues.
    6. **Schedule 8 Full-Length Tests with Fixed Distribution** (Updated):
       * **Total Tests**: 8 (fixed).
       * **Distribution**:
         * Phase 1: 2 full-length tests.
         * Phase 2: 2 full-length tests.
         * Phase 3: 4 full-length tests.
       * **Dynamic Scheduling**:
         * Distribute the 2 tests in Phase 1 evenly across `phase_1_weeks`.
         * Distribute the 2 tests in Phase 2 evenly across `phase_2_weeks`.
         * Distribute the 4 tests in Phase 3 evenly across `phase_3_weeks`.
       * **Adjust Regular Sessions**:
         *   Subtract the number of full-length test sessions from the total sessions in each phase:

             ```python
             phase_1_regular_sessions = phase_1_sessions - 2
             phase_2_regular_sessions = phase_2_sessions - 2
             phase_3_regular_sessions = phase_3_sessions - 4
             ```

**Step 3: Generate Study Plan Tasks with Updated Test Distribution**

* **Backend**:
  * **Logic**:
    1. **Create Study Plan Record** (Unchanged):
       * Generate a `plan_id` and insert into `Study_Plans`.
    2. **Schedule Tasks for Each Phase** (Updated):
       * **Phase 1: Foundation Building**:
         * Schedule 2 full-length tests:
           * Distribute evenly across `phase_1_weeks` (e.g., one test every `phase_1_weeks/2` weeks).
           * Task: "Take a full-length SAT practice test" (49 questions, 72 minutes, 150 points).
         * Schedule `phase_1_regular_sessions` regular tasks:
           * Tasks: "Complete a 5-question session in \[subskill]", "Practice pacing for \[subskill]" (if pacing issues).
       * **Phase 2: Practice and Strategy**:
         * Schedule 2 full-length tests:
           * Distribute evenly across `phase_2_weeks`.
         * Schedule `phase_2_regular_sessions` regular tasks:
           * Tasks: "Complete a 10-question session in \[subskill]", "Review strategies for \[subskill]" (if slow improvement).
       * **Phase 3: Final Review and Refinement**:
         * Schedule 4 full-length tests:
           * Distribute evenly across `phase_3_weeks`.
         * Schedule `phase_3_regular_sessions` regular tasks:
           * Tasks: "Review incorrect answers from \[subskill] practice", "Review recent performance in \[subskill]" (if negative trends), "Review test-day checklist".
    3. **Insert Tasks into Database** (Unchanged):
       * Create `Study_Plan_Actions` records for each task.
    4. **Return Study Plan** (Unchanged):
       * Return the `plan_id` and `milestones`.

**Step 4: Regular Updates (Unchanged)**

* The `/study_plan/update/{user_id}` endpoint continues to update the Study Plan weekly or after practice tests, incorporating the latest `theta` scores and advanced analytics.

***

### Updated Code Implementation

#### Backend (`frontend/backend/src/study_plan.py`)

Update the `/study_plan/create` and `/study_plan/update` endpoints to include the new full-length test distribution.

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

class StudyPlanRequest(BaseModel):
    test_date: str
    study_hours: int
    study_days: int

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

        # Allocate regular sessions to subskills with advanced analytics
        reading_writing_subskills = set(p['skill'] for p in proficiencies if p['domain'] == 'Reading & Writing')
        math_subskills = set(p['skill'] for p in proficiencies if p['domain'] == 'Math')
        focus_reading_writing = [s for s in focus_subskills if s in reading_writing_subskills]
        focus_math = [s for s in focus_subskills if s in math_subskills]

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
                                "points": points
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
                                    "points": 50
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
                                "points": points
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
                                    "points": 50
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
                                    "points": points
                                })
                                cursor.execute("""
                                    INSERT INTO study_plan_actions (id, plan_id, task, action, due_date, points)
                                    VALUES (%s, %s, %s, %s, %s, %s);
                                """, (str(uuid.uuid4()), plan_id, task, "Complete", day_date.isoformat(), points))
                        for subskill in math_allocation:
                            if improvement_rates.get(subskill, 0) < 0.1:
                                task = f"植物 strategies for {subskill}"
                                points = 75
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
                        for subskill in reading_writing_allocation:
                            if reading_writing_allocation[subskill] > 0:
                                task = f"Complete a 10-question session in {subskill}"
                                points = 75
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
                                    "points": points
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
                                    "points": points
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
                                    "points": points
                                })
                                cursor.execute("""
                                    INSERT INTO study_plan_actions (id, plan_id, task, action, due_date, points)
                                    VALUES (%s, %s, %s, %s, %s, %s);
                                """, (str(uuid.uuid4()), plan_id, task, "Complete", day_date.isoformat(), points))
                                math_allocation[subskill] -= 1

        conn.commit()
        return {"plan_id": plan_id, "milestones": milestones}
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
                                "points": points
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
                                    "points": 50
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
                                "points": points
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
                                    "points": 50
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
                                    "points": points
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
                                    "points": points
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
                                    "points": points
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
                                    "points": points
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
                                    "points": points
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
                                    "points": points
                                })
                                cursor.execute("""
                                    INSERT INTO study_plan_actions (id, plan_id, task, action, due_date, points)
                                    VALUES (%s, %s, %s, %s, %s, %s);
                                """, (str(uuid.uuid4()), plan_id, task, "Complete", day_date.isoformat(), points))
                                math_allocation[subskill] -= 1

        conn.commit()
        return {"plan_id": plan_id, "milestones": milestones}
    except Exception as e:
        conn.rollback()
        raise HTTPException(status_code=500, detail=str(e))
    finally:
        cursor.close()
        conn.close()
```

***

### Summary of the Updated Study Plan Creation Process

The updated **Study Plan Creation Process** now includes the following enhancements:

1. **Fixed Full-Length Test Distribution**:
   * Total full-length tests: 8 (fixed).
   * Phase 1: Foundation Building (40% of time, 2 full-length tests).
   * Phase 2: Practice and Strategy (30% of time, 2 full-length tests).
   * Phase 3: Final Review and Refinement (30% of time, 4 full-length tests).
   * Tests are scheduled evenly within each phase, ensuring users experience the SAT format early (Phase 1), practice under test-like conditions (Phase 2), and refine their skills with more tests closer to the test date (Phase 3).
2. **Advanced Analytics Integration** (Unchanged):
   * The Study Plan continues to use performance trends, improvement rates, and time spent per question to allocate sessions, ensuring a data-driven approach.
3. **Regular Updates** (Unchanged):
   * The `/study_plan/update/{user_id}` endpoint updates the Study Plan weekly or after practice tests, incorporating the latest `theta` scores and analytics.

This updated structure ensures a balanced preparation timeline with a clear progression of full-length test practice, while maintaining the use of advanced analytics to personalize the Study Plan. Let me know if you’d like to further refine this process or add additional features!
