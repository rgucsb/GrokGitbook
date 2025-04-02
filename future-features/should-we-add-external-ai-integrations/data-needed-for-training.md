# Data Needed for Training

To train a small Language Learning Model (LLM) for the **SAT Prep Suite**, we need specific, structured data that reflects the SAT preparation domain and supports the model’s intended tasks: generating personalized feedback, explaining questions, and optimizing study plans. The data must be drawn from the suite’s existing database and supplemented as needed. Below, I’ll outline the **exact data** required, its sources within the suite, and how it will be used, ensuring we align with the system’s capabilities as of March 26, 2025.

***

#### Step 1: Data Requirements Overview

**Model Tasks**

1. **Personalized Feedback**: Analyze response patterns and predictions to provide conversational insights (e.g., “Your Algebra is improving—focus on word problems!”).
2. **Question Explanations**: Generate clear, tutor-like rationales for SAT questions.
3. **Study Plan Suggestions**: Offer concise, actionable adjustments (e.g., “Shift time to Geometry”).

**Data Characteristics**

* **Format**: Text-based input-output pairs or sequences.
* **Volume**: Minimum 50K examples for initial training; scales with student growth (e.g., 100K+ at 10K students).
* **Quality**: Clean, SAT-specific, and contextually rich.
* **Diversity**: Covers Math, Reading/Writing, all domains/skills, and varied student performance levels.

***

#### Step 2: Exact Data Types and Sources

**1. Response Patterns and Feedback Data**

* **Source**: `RESPONSES` and `RESULTS` tables.
* **Fields**:
  * `user_id`: Links responses to a student’s history.
  * `question_id`: Ties to specific questions.
  * `answer`: Student’s submitted answer.
  * `is_correct`: Boolean for accuracy.
  * `time_spent`: Time taken (seconds).
  * `timestamp`: Response submission time.
  * `proficiencies` (from `RESULTS`): Skill proficiency scores (e.g., {"Algebra": {"Linear Functions": 5\}}).
* **Format**: JSON-like summaries with feedback prompts.
* **Example**:
  * **Input**: `{"Math": {"Algebra": {"Linear Functions": {"correct": 8, "total": 10, "avg_time": 45}}}, "predictions": {"Math": {"Algebra": {"Linear Functions": {"predicted_prof": 5.2}}}}`
  * **Output**: “Hey! You’re rocking Linear Functions with 80% accuracy—nice work! Your predicted proficiency is 5.2, so you’re on a roll. Let’s push it further with some trickier problems!”
* **Volume**: \~10K responses from current users → \~5K feedback pairs after aggregation.
* **Use**: Trains the model to generate nuanced, conversational feedback.

**2. Question Content and Rationales**

* **Source**: `QUESTIONS` table.
* **Fields**:
  * `question_id`: Unique identifier.
  * `test`: "Math" or "Reading and Writing".
  * `domain`: e.g., "Algebra", "Reading Comprehension".
  * `skill`: e.g., "Linear Functions", "Evidence-Based Analysis".
  * `content`: JSON with `text`, `options`, `correct_answer`, `rationale`, and `external_links`.
* **Format**: Question-explanation pairs.
* **Example**:
  * **Input**: “Solve for x: 2x + 3 = 7. Options: \[1, 2, 3, 4]”
  * **Output**: “To solve 2x + 3 = 7, subtract 3 from both sides: 2x = 4. Then divide by 2: x = 2. The correct answer is 2.”
* **Volume**: \~1K questions → \~1K pairs (assuming each has a rationale).
* **Use**: Teaches the model to explain SAT questions clearly and concisely.

**3. Lesson Content**

* **Source**: `LESSONS` table.
* **Fields**:
  * `lesson_id`: Unique identifier.
  * `section`: "Math" or "Reading and Writing".
  * `domain`: e.g., "Algebra".
  * `skill`: e.g., "Linear Functions".
  * `content`: JSON with `text` and `video_url` (focus on text).
  * `duration`: Estimated time (minutes).
* **Format**: Lesson text as input, summarized explanation as output.
* **Example**:
  * **Input**: “Linear functions follow y = mx + b, where m is slope and b is y-intercept. Example: y = 2x + 3.”
  * **Output**: “Linear functions use y = mx + b. Here, m (slope) shows steepness, and b is where it crosses the y-axis. For y = 2x + 3, slope is 2, intercept is 3.”
* **Volume**: \~100 lessons → \~100 pairs (assuming initial content).
* **Use**: Enables the model to generate foundational explanations.

**4. Study Plan Adjustments**

* **Source**: `STUDY_PLANS` and `RESPONSES`/`RESULTS` for context.
* **Fields**:
  * `plan`: JSON with daily tasks (e.g., {"skill\_building": \[{"day": 1, "type": "practice", "skill": "Triangles"}]})
  * `user_id`: Links to performance data.
  * Response/proficiency data (as above).
* **Format**: Current plan + performance → adjustment suggestion.
* **Example**:
  * **Input**: `{"plan": {"skill_building": [{"day": 1, "type": "practice", "skill": "Linear Functions"}]}}, {"Math": {"Geometry": {"Triangles": {"correct": 2, "total": 5}}}}`
  * **Output**: “Your Triangles accuracy is only 40%—let’s shift day 1 to practice Triangles instead!”
* **Volume**: \~1K plans × adjustments → \~1K pairs.
* **Use**: Trains the model to suggest targeted plan tweaks.

**5. Synthetic Data (Optional)**

* **Source**: Generated from templates and existing data.
* **Method**:
  * Rephrase rationales (e.g., “Subtract 3, then divide” → “Take 3 away, then halve it”).
  * Create mock feedback (e.g., “You’re slow on Geometry” → “Geometry’s taking you longer—speed up!”).
* **Example**:
  * **Input**: “Accuracy: 60%, Time: 50s, Skill: Algebra”
  * **Output**: “You’re at 60% on Algebra—decent! But 50s is a bit pokey—try aiming for 40s.”
* **Volume**: \~10K synthetic pairs to boost dataset size.
* **Use**: Augments limited real data for robustness.

***

#### Step 3: Data Extraction from Suite

**SQL Queries**

1.  **Response Patterns**:

    ```sql
    SELECT 
        r.user_id,
        r.question_id,
        r.answer,
        r.is_correct,
        r.time_spent,
        r.timestamp,
        res.proficiencies
    FROM RESPONSES r
    LEFT JOIN RESULTS res ON r.test_id = res.test_id
    WHERE r.timestamp > '2024-01-01'  -- Recent data
    ```

    * Aggregate by user/skill for feedback input.
2.  **Questions**:

    ```sql
    SELECT 
        question_id,
        test,
        domain,
        skill,
        content->>'text' AS question_text,
        content->>'correct_answer' AS correct_answer,
        content->>'rationale' AS rationale
    FROM QUESTIONS
    ```

    * Extract content for explanation pairs.
3.  **Lessons**:

    ```sql
    SELECT 
        lesson_id,
        section,
        domain,
        skill,
        content->>'text' AS lesson_text
    FROM LESSONS
    ```

    * Use text for lesson summarization.
4.  **Study Plans**:

    ```sql
    SELECT 
        sp.plan,
        r.user_id,
        r.is_correct,
        r.question_id,
        q.domain,
        q.skill
    FROM STUDY_PLANS sp
    JOIN RESPONSES r ON sp.user_id = r.user_id
    JOIN QUESTIONS q ON r.question_id = q.question_id
    ```

    * Combine with performance for adjustment pairs.

**Processing**

*   **Script**: Python script to fetch, clean, and pair data.

    ```python
    import json
    from sqlalchemy import create_engine

    engine = create_engine("sqlite:///sat_prep.db")  # Adjust for your DB
    with engine.connect() as conn:
        responses = conn.execute("SELECT * FROM RESPONSES JOIN RESULTS ...").fetchall()
        questions = conn.execute("SELECT * FROM QUESTIONS").fetchall()
        lessons = conn.execute("SELECT * FROM LESSONS").fetchall()
        plans = conn.execute("SELECT * FROM STUDY_PLANS JOIN RESPONSES ...").fetchall()

    # Example: Save to JSONL
    with open("sat_data.jsonl", "w") as f:
        for r in responses:
            f.write(json.dumps({"input": r["proficiencies"], "output": "Custom feedback..."}) + "\n")
    ```

***

#### Step 4: Data Volume and Scaling

**Current Estimate**

* **Responses**: 10K students × 10 responses = 100K rows → \~5K aggregated feedback pairs.
* **Questions**: 1K questions → 1K explanation pairs.
* **Lessons**: 100 lessons → 100 pairs.
* **Plans**: 1K plans → 1K adjustment pairs.
* **Synthetic**: 10K pairs.
* **Total**: \~17K pairs initially.

**Scaling with Students**

* **10K Students**: 100K responses → \~50K pairs (with synthetic data).
* **100K Students**: 1M responses → \~500K pairs.
* **Growth**: Retrain quarterly as data doubles.

**Minimum Viable Dataset**

* **50K pairs**: Sufficient for fine-tuning a 300M-parameter model (e.g., TinyLLaMA) with decent performance (perplexity <20).

***

#### Step 5: Data Quality and Gaps

**Quality Checks**

* **Completeness**: Ensure `rationale` exists in `QUESTIONS.content`; fill missing with manual input or templates.
* **Consistency**: Standardize feedback tone (e.g., friendly, concise) via post-processing.
* **Relevance**: Filter out-of-domain data (e.g., non-SAT responses).

**Gaps and Mitigation**

* **Limited Lessons**: Augment with SAT textbooks or Khan Academy-style text (manually curated).
* **Sparse Feedback**: Use existing GPT-4o outputs as pseudo-labels for initial training.
* **Bias**: Balance data across skills (e.g., Algebra vs. Geometry) by oversampling weak areas.

***

#### Conclusion

The **exact data** for training includes:

1. **Response Patterns** (`RESPONSES`, `RESULTS`): 5K+ feedback pairs.
2. **Question Rationales** (`QUESTIONS`): 1K+ explanation pairs.
3. **Lesson Content** (`LESSONS`): 100+ pairs.
4. **Study Plan Adjustments** (`STUDY_PLANS`): 1K+ pairs.
5. **Synthetic Data**: 10K+ pairs to boost volume.

This leverages the suite’s existing tables (`RESPONSES`, `QUESTIONS`, `LESSONS`, `STUDY_PLANS`), starting with \~17K pairs and scaling to 50K+ as needed. With this, we can train a small LLM to replace GPT-4o, minimizing costs effectively. Next step: Start data extraction and preprocessing—ready to proceed?
