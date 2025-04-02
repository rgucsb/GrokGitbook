# Synthetic Data Creation Plan

Let’s devise a plan to **create synthetic data** to kickstart the training of a small Language Learning Model (LLM) for the **SAT Prep Suite**, allowing us to begin development before we have a large user base. Once users join the platform and real data accumulates, we’ll transition to training on that data to refine and scale the model. This approach ensures we can launch with a functional, cost-effective LLM as of March 26, 2025, and improve it over time. Below, I’ll outline how to generate synthetic data, its structure, and the transition plan.

***

#### Step 1: Why Synthetic Data?

* **Immediate Start**: With limited initial users (e.g., <1K responses), synthetic data provides the volume (50K+ pairs) needed to train a small LLM (e.g., 300M parameters).
* **Cost Efficiency**: Avoids early reliance on expensive APIs like GPT-4o while bootstrapping the model.
* **Control**: Ensures coverage of SAT domains, skills, and tasks (feedback, explanations, plan adjustments).
* **Bridge**: Acts as a placeholder until real user data (e.g., from `RESPONSES`, `QUESTIONS`) scales up.

***

#### Step 2: Plan to Create Synthetic Data

**Objectives**

* Generate \~50K text pairs covering:
  1. **Personalized Feedback**: Input = performance data, Output = conversational insights.
  2. **Question Explanations**: Input = SAT questions, Output = step-by-step rationales.
  3. **Study Plan Suggestions**: Input = plan + performance, Output = adjustments.
* Mimic SAT Prep Suite data structure (`RESPONSES`, `QUESTIONS`, `STUDY_PLANS`).
* Ensure diversity across Math, Reading/Writing, and all domains/skills.

**Data Creation Methods**

1. **Template-Based Generation**:
   * Use hand-crafted templates with placeholders filled by SAT-specific terms.
   * Source terms from SAT syllabi (e.g., College Board domains: Algebra, Geometry, Reading Comprehension).
2. **Rule-Based Augmentation**:
   * Start with a small seed dataset (e.g., 100 real or manually written examples) and vary phrasing, numbers, or skills.
3. **External Resources**:
   * Adapt open SAT prep content (e.g., Khan Academy, public-domain SAT books) into structured pairs.
4. **Minimal GPT-4o Use**:
   * Use GPT-4o sparingly to generate initial examples, then extrapolate programmatically.

**Synthetic Data Types and Examples**

**1. Personalized Feedback**

* **Input Format**: JSON-like performance stats + predictions.
* **Output Format**: 50-100 word conversational text.
* **Template**:
  * Input: `{"{section}": {"{domain}": {"{skill}": {"correct": {n}, "total": {m}, "avg_time": {t}}}}, "predictions": {"{section}": {"{domain}": {"{skill}": {"predicted_prof": {p}}}}}}`
  * Output: “Hey! You’ve got {n}/{m} right in {skill} ({percent}%)—{comment}. Your avg time of {t}s is {time\_comment}. You’re headed to a {p} proficiency, so {suggestion}!”
* **Generation**:
  * Variables: `section` (Math, Reading and Writing), `domain` (e.g., Algebra), `skill` (e.g., Linear Functions), `n` (0-10), `m` (1-10), `t` (20-120), `p` (1-7).
  * Comments: “solid work” (>70%), “room to grow” (<50%).
  * Time Comments: “super quick” (<40s), “a bit slow” (>60s).
  * Suggestions: “keep it up”, “try harder problems”, “focus on basics”.
* **Example**:
  * **Input**: `{"Math": {"Algebra": {"Linear Functions": {"correct": 7, "total": 10, "avg_time": 45}}}, "predictions": {"Math": {"Algebra": {"Linear Functions": {"predicted_prof": 5.2}}}}}}`
  * **Output**: “Hey! You’ve got 7/10 right in Linear Functions (70%)—solid work! Your avg time of 45s is decent, but let’s aim for 40s. You’re headed to a 5.2 proficiency, so try some trickier problems to push it higher!”
* **Volume**: 20K pairs (10 skills × 10 performance combos × 200 variations).

**2. Question Explanations**

* **Input Format**: Question text + options.
* **Output Format**: Step-by-step rationale (50-100 words).
* **Template**:
  * Input: “{question\_type}: {equation/text}. Options: \[{opt1}, {opt2}, {opt3}, {opt4}]”
  * Output: “To solve this {question\_type}, {step1}. Then, {step2}. So, the answer is {correct\_opt}.”
* **Generation**:
  * Types: “Solve for x” (Math), “Find the main idea” (Reading).
  * Math: Generate equations (e.g., ax + b = c) with a (1-5), b (1-10), c (1-20).
  * Reading: Use short passages (50 words) with 4-option questions.
  * Steps: Derived logically (e.g., subtract b, divide by a).
* **Example**:
  * **Input**: “Solve for x: 3x + 4 = 13. Options: \[2, 3, 4, 5]”
  * **Output**: “To solve this equation, subtract 4 from both sides: 3x = 9. Then, divide by 3: x = 3. So, the answer is 3.”
* **Volume**: 20K pairs (500 unique questions × 40 rephrasings).

**3. Study Plan Suggestions**

* **Input Format**: Plan JSON + performance snippet.
* **Output Format**: Short adjustment (20-50 words).
* **Template**:
  * Input: `{"plan": {"{phase}": [{"day": {d}, "type": "{type}", "skill": "{skill1}"}]}}, {"{section}": {"{domain}": {"{skill2}": {"correct": {n}, "total": {m}}}}}}`
  * Output: “Your {skill2} is at {percent}%—{comment}. Switch day {d} to practice {skill2} instead!”
* **Generation**:
  * Phases: “foundation”, “skill\_building”.
  * Skills: Rotate through SAT list (e.g., Triangles, Evidence-Based Analysis).
  * Performance: Vary `n`/`m` for accuracy (20-80%).
  * Comments: “needs work” (<50%), “almost there” (>70%).
* **Example**:
  * **Input**: `{"plan": {"skill_building": [{"day": 1, "type": "practice", "skill": "Linear Functions"}]}}, {"Math": {"Geometry": {"Triangles": {"correct": 3, "total": 10}}}}}}`
  * **Output**: “Your Triangles is at 30%—needs work. Switch day 1 to practice Triangles instead!”
* **Volume**: 10K pairs (50 skills × 20 plans × 10 performance levels).

**Tools and Process**

*   **Python Script**:

    ```python
    import random
    import json

    skills = ["Linear Functions", "Triangles", "Reading Comprehension"]
    domains = {"Math": ["Algebra", "Geometry"], "Reading and Writing": ["Reading Comprehension"]}

    def generate_feedback_pair():
        section = random.choice(["Math", "Reading and Writing"])
        domain = random.choice(domains[section])
        skill = random.choice(skills)
        correct, total = random.randint(0, 10), 10
        avg_time = random.randint(20, 120)
        pred_prof = round(random.uniform(1, 7), 1)
        input_data = {section: {domain: {skill: {"correct": correct, "total": total, "avg_time": avg_time}}}, "predictions": {section: {domain: {skill: {"predicted_prof": pred_prof}}}}}
        percent = correct / total * 100
        output = f"Hey! You’ve got {correct}/{total} right in {skill} ({percent:.0f}%)—{'solid work' if percent > 70 else 'room to grow'}. Your avg time of {avg_time}s is {'quick' if avg_time < 40 else 'slow' if avg_time > 60 else 'decent'}. You’re at {pred_prof}, so {'keep it up' if pred_prof > 5 else 'try harder problems'}!"
        return {"input": json.dumps(input_data), "output": output}

    # Generate 50K pairs
    with open("synthetic_sat_data.jsonl", "w") as f:
        for _ in range(50000):
            pair = generate_feedback_pair()  # Extend for explanations, plans
            f.write(json.dumps(pair) + "\n")
    ```
* **Volume Control**: Adjust loops for 20K feedback, 20K explanations, 10K plan pairs.
* **Validation**: Manually review 100 samples for coherence and SAT relevance.

***

#### Step 3: Transition to Real Data

**Phase 1: Initial Deployment with Synthetic Data**

* **Training**: Use 50K synthetic pairs to fine-tune a small LLM (e.g., TinyLLaMA).
* **Deployment**: Integrate into `api/utils.py` (e.g., `generate_feedback_with_custom_llm`).
* **Performance**: Expect basic but functional output (e.g., perplexity \~30, BLEU \~0.5 vs. GPT-4o).

**Phase 2: Collect Real Data**

* **Sources**: `RESPONSES`, `QUESTIONS`, `LESSONS`, `STUDY_PLANS` as users join.
* **Milestones**:
  * **1K Users**: \~10K responses → 5K real pairs.
  * **10K Users**: \~100K responses → 50K pairs.
* **Storage**: Append to a growing JSONL file (e.g., `real_sat_data.jsonl`).

**Phase 3: Retrain with Real Data**

* **Trigger**: When real data reaches 50K pairs (e.g., 10K users × 10 responses).
* **Process**:
  * Mix synthetic (25%) and real (75%) data initially to retain coverage.
  * Fine-tune existing model incrementally (1-2 epochs, \~$100 on A100).
* **Evaluation**: Compare to synthetic-only model (e.g., perplexity <20, human-rated quality).
* **Full Transition**: At 100K pairs (e.g., 20K users), use 100% real data.

**Phase 4: Continuous Improvement**

* **Frequency**: Retrain quarterly as data grows (e.g., 500K pairs at 100K users).
* **Automation**: Script to fetch new data, retrain, and deploy updated weights.
* **Fallback**: Keep GPT-4o as a backup during retraining downtimes.

***

#### Step 4: Benefits and Risks

**Benefits**

* **Fast Start**: 50K synthetic pairs in \~1 week vs. months for real data.
* **Cost Savings**: Avoids $100s in GPT-4o fees pre-launch.
* **Flexibility**: Synthetic data ensures all skills are covered early.

**Risks**

* **Quality**: Synthetic data may lack nuance (e.g., less natural phrasing).
* **Overfitting**: Model might memorize templates; mitigated by real data later.
* **Effort**: \~20 hours to design templates and scripts.

**Mitigation**

* **Seed with GPT-4o**: Generate 1K high-quality pairs ($10-$20) to guide templates.
* **Diversity**: Randomize phrasing and numbers extensively.
* **Validation**: Human review to catch errors before training.

***

#### Step 5: Timeline

* **Week 1**: Design templates, write generation script, create 50K pairs.
* **Week 2**: Train initial model on synthetic data (\~$500 on A100).
* **Week 3**: Integrate into suite, test with dummy users.
* **Post-Launch**: Collect real data, retrain at 50K pairs (\~2-3 months with 10K users).

***

#### Conclusion

This plan uses **template-based synthetic data** (50K pairs: 20K feedback, 20K explanations, 10K plan adjustments) to train an LLM immediately, leveraging SAT-specific terms and structures. Once users join, we’ll transition to real data from `RESPONSES`, `QUESTIONS`, etc., retraining at \~50K pairs for a robust, cost-effective solution by mid-2025. Next step: Start coding the generation script—ready to proceed?
