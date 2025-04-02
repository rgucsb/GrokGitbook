# Code for Diagnostic with IRT

Below is an updated Python script that adds proficiency estimation for each skill (on a 1–7 scale) at the end of the adaptive SAT diagnostic test. The proficiency is calculated based on student responses using the 3PL IRT model, where ability (`theta`) is estimated per skill and mapped to a 7-point scale. The script retains the previous adaptive 3PL IRT framework, error handling, and validation, and outputs skill proficiencies alongside the test results.

***

#### Python Code with Skill Proficiency Output

```python
import json
import random
import numpy as np
from typing import List, Dict, Tuple
from scipy.stats import norm

# Test plans (35 questions each)
MATH_TEST_PLAN = {
    "Algebra": {
        "Linear Equations in One Variable": {"Easy": 1, "Medium": 1, "Hard": 0},
        "Linear Functions": {"Easy": 1, "Medium": 2, "Hard": 1},
        "Linear Equations in Two Variables": {"Easy": 1, "Medium": 1, "Hard": 1},
        "Systems of Two Linear Equations in Two Variables": {"Easy": 1, "Medium": 1, "Hard": 1},
        "Linear Inequalities in One or Two Variables": {"Easy": 0, "Medium": 1, "Hard": 1}
    },
    "Advanced Math": {
        "Nonlinear Functions": {"Easy": 2, "Medium": 2, "Hard": 2},
        "Nonlinear Equations in One Variable and Systems of Equations": {"Easy": 1, "Medium": 1, "Hard": 1},
        "Equivalent Expressions": {"Easy": 0, "Medium": 1, "Hard": 1}
    },
    "Problem-Solving and Data Analysis": {
        "Ratios, Rates, Proportional Relationships, and Units": {"Easy": 1, "Medium": 1, "Hard": 0},
        "Percentages": {"Easy": 0, "Medium": 1, "Hard": 1},
        "One-Variable Data: Distributions and Measures of Center and Spread": {"Easy": 0, "Medium": 0, "Hard": 1},
        "Two-Variable Data: Models and Scatterplots": {"Easy": 0, "Medium": 1, "Hard": 0},
        "Probability and Conditional Probability": {"Easy": 0, "Medium": 0, "Hard": 1},
        "Inference from Sample Statistics and Margin of Error": {"Easy": 0, "Medium": 1, "Hard": 0},
        "Evaluating Statistical Claims": {"Easy": 1, "Medium": 0, "Hard": 0}
    },
    "Geometry and Trigonometry": {
        "Area and Volume": {"Easy": 1, "Medium": 1, "Hard": 0},
        "Lines, Angles, and Triangles": {"Easy": 0, "Medium": 0, "Hard": 1},
        "Right Triangles and Trigonometry": {"Easy": 0, "Medium": 1, "Hard": 0},
        "Circles": {"Easy": 0, "Medium": 0, "Hard": 1}
    }
}

RW_TEST_PLAN = {
    "Information and Ideas": {
        "Central Ideas and Details": {"Easy": 2, "Medium": 2, "Hard": 1},
        "Inferences": {"Easy": 1, "Medium": 2, "Hard": 2},
        "Command of Evidence": {"Easy": 1, "Medium": 2, "Hard": 1}
    },
    "Craft and Structure": {
        "Words in Context": {"Easy": 2, "Medium": 2, "Hard": 1},
        "Text Structure and Purpose": {"Easy": 1, "Medium": 2, "Hard": 1},
        "Cross-Text Connections": {"Easy": 1, "Medium": 1, "Hard": 2}
    },
    "Expression of Ideas": {
        "Rhetorical Synthesis": {"Easy": 1, "Medium": 2, "Hard": 1},
        "Transitions": {"Easy": 1, "Medium": 2, "Hard": 1}
    }
}

class QuestionBank:
    def __init__(self, filepath: str):
        """Load the question bank from a JSON file."""
        try:
            with open(filepath, 'r') as f:
                self.questions = json.load(f)
            if not self.questions:
                raise ValueError("Question bank is empty.")
        except FileNotFoundError:
            raise FileNotFoundError(f"Question bank file '{filepath}' not found.")
        except json.JSONDecodeError:
            raise ValueError(f"Invalid JSON in '{filepath}'.")
    
    def filter_questions(self, test: str, domain: str, skill: str, used_ids: set) -> List[Dict]:
        """Filter questions by test, domain, skill, excluding used questions."""
        return [
            q for q in self.questions
            if q["metadata"]["Test"] == test
            and q["metadata"]["Domain"] == domain
            and q["metadata"]["Skill"] == skill
            and q["metadata"]["Question ID"] not in used_ids
        ]

class IRTSelector:
    def __init__(self, initial_theta: float = 0.0):
        """Initialize IRT with an initial ability estimate (theta)."""
        self.theta = initial_theta
    
    def probability_correct(self, a: float, b: float, c: float) -> float:
        """Calculate probability of a correct response using 3PL IRT model."""
        logit = a * (self.theta - b)
        return c + (1 - c) / (1 + np.exp(-logit))
    
    def information(self, a: float, b: float, c: float) -> float:
        """Calculate item information for a question (3PL)."""
        p = self.probability_correct(a, b, c)
        q = 1 - p
        if p <= c or q == 0:
            return 0
        return (a**2 * (p - c)**2 * q) / ((1 - c)**2 * p)
    
    def update_theta(self, responses: List[Tuple[Dict, bool]]) -> float:
        """Update ability estimate using maximum likelihood for 3PL."""
        if not responses:
            return self.theta
        theta_new = self.theta
        for _ in range(10):
            numerator = 0
            denominator = 0
            for q, r in responses:
                a = q["irt_parameters"]["a"]
                b = q["irt_parameters"]["b"]
                c = q["irt_parameters"]["c"]
                p = self.probability_correct(a, b, c)
                if p <= c or 1 - p == 0:
                    continue
                numerator += a * (int(r) - p)
                denominator += (a**2 * (p - c) * (1 - p)) / (1 - c)
            if denominator == 0:
                break
            theta_new = theta_new + numerator / denominator
        return theta_new if not np.isnan(theta_new) else self.theta

def theta_to_proficiency(theta: float) -> int:
    """Map theta to a 1-7 proficiency scale."""
    # Assuming theta ranges from -3 to 3 (standard IRT range), map to 1-7
    if theta < -2.0:
        return 1
    elif theta < -1.0:
        return 2
    elif theta < 0.0:
        return 3
    elif theta < 1.0:
        return 4
    elif theta < 2.0:
        return 5
    elif theta < 3.0:
        return 6
    else:
        return 7

def calculate_skill_proficiencies(responses: List[Tuple[Dict, bool]], test_plan: Dict) -> Dict[str, Dict[str, int]]:
    """Calculate proficiency (1-7) for each skill based on responses."""
    skill_responses = {}
    for domain, skills in test_plan.items():
        skill_responses[domain] = {skill: [] for skill in skills}
    
    # Group responses by skill
    for q, r in responses:
        domain = q["metadata"]["Domain"]
        skill = q["metadata"]["Skill"]
        skill_responses[domain][skill].append((q, r))
    
    # Estimate theta and proficiency per skill
    irt = IRTSelector(initial_theta=0.0)
    proficiencies = {}
    for domain, skills in skill_responses.items():
        proficiencies[domain] = {}
        for skill, resp in skills.items():
            if resp:  # Only calculate if there are responses
                theta = irt.update_theta(resp)
                proficiencies[domain][skill] = theta_to_proficiency(theta)
            else:
                proficiencies[domain][skill] = 3  # Default to average if no questions
    
    return proficiencies

def select_adaptive_questions(question_bank: QuestionBank, test_plan: Dict, test_type: str, irt_selector: IRTSelector) -> Tuple[List[Dict], List[Tuple[Dict, bool]]]:
    """Select 35 questions adaptively using 3PL IRT and track responses."""
    selected_questions = []
    responses = []
    used_ids = set()
    skill_counts = {domain: {skill: 0 for skill in skills} for domain, skills in test_plan.items()}
    difficulty_targets = {"Easy": 11, "Medium": 14, "Hard": 10}
    difficulty_counts = {"Easy": 0, "Medium": 0, "Hard": 0}

    while len(selected_questions) < 35:
        available_skills = [
            (domain, skill)
            for domain, skills in test_plan.items()
            for skill in skills
            if sum(test_plan[domain][skill].values()) > skill_counts[domain][skill]
        ]
        if not available_skills:
            raise ValueError(f"No available skills left to meet test plan for {test_type}")

        domain, skill = random.choice(available_skills)
        candidates = question_bank.filter_questions(test_type, domain, skill, used_ids)
        if not candidates:
            raise ValueError(f"No unused questions for {test_type} - {domain} - {skill}")

        valid_difficulties = [
            d for d in ["Easy", "Medium", "Hard"]
            if (test_plan[domain][skill][d] > 0 and
                difficulty_counts[d] < difficulty_targets[d] and
                skill_counts[domain][skill] < sum(test_plan[domain][skill].values()))
        ]
        if not valid_difficulties:
            continue
        candidates = [q for q in candidates if q["metadata"]["Difficulty"] in valid_difficulties]

        candidates.sort(key=lambda q: irt_selector.information(
            q["irt_parameters"]["a"], q["irt_parameters"]["b"], q["irt_parameters"]["c"]), reverse=True)
        question = candidates[0]
        
        selected_questions.append(question)
        used_ids.add(question["metadata"]["Question ID"])
        skill_counts[domain][skill] += 1
        difficulty_counts[question["metadata"]["Difficulty"]] += 1
        
        # Simulate response (replace with real input in practice)
        p_correct = irt_selector.probability_correct(
            question["irt_parameters"]["a"], question["irt_parameters"]["b"], question["irt_parameters"]["c"])
        response = random.random() < p_correct
        responses.append((question, response))
        
        irt_selector.theta = irt_selector.update_theta(responses)
    
    if len(selected_questions) != 35:
        raise ValueError(f"Selected {len(selected_questions)} questions for {test_type}, expected 35")
    for diff, count in difficulty_counts.items():
        if abs(count - difficulty_targets[diff]) > 2:
            print(f"Warning: {test_type} has {count} {diff} questions, expected ~{difficulty_targets[diff]}")
    
    return selected_questions, responses

def save_test(questions: List[Dict], output_file: str):
    """Save the selected questions to a JSON file."""
    try:
        with open(output_file, 'w') as f:
            json.dump(questions, f, indent=4)
    except IOError as e:
        raise IOError(f"Failed to save test to '{output_file}': {e}")

def main():
    question_bank_file = "question_bank.json"
    
    try:
        qb = QuestionBank(question_bank_file)
        irt_selector_math = IRTSelector(initial_theta=0.0)
        irt_selector_rw = IRTSelector(initial_theta=0.0)
        
        # Math test
        math_questions, math_responses = select_adaptive_questions(qb, MATH_TEST_PLAN, "Math", irt_selector_math)
        save_test(math_questions, "sat_math_diagnostic.json")
        math_proficiencies = calculate_skill_proficiencies(math_responses, MATH_TEST_PLAN)
        print(f"Math test: {len(math_questions)} questions, Final theta: {irt_selector_math.theta:.2f}")
        print("Math Skill Proficiencies (1-7):")
        for domain, skills in math_proficiencies.items():
            print(f"  {domain}:")
            for skill, proficiency in skills.items():
                print(f"    {skill}: {proficiency}")
        
        # Reading and Writing test
        rw_questions, rw_responses = select_adaptive_questions(qb, RW_TEST_PLAN, "Reading and Writing", irt_selector_rw)
        save_test(rw_questions, "sat_rw_diagnostic.json")
        rw_proficiencies = calculate_skill_proficiencies(rw_responses, RW_TEST_PLAN)
        print(f"R&W test: {len(rw_questions)} questions, Final theta: {irt_selector_rw.theta:.2f}")
        print("Reading and Writing Skill Proficiencies (1-7):")
        for domain, skills in rw_proficiencies.items():
            print(f"  {domain}:")
            for skill, proficiency in skills.items():
                print(f"    {skill}: {proficiency}")
    
    except (FileNotFoundError, ValueError, IOError) as e:
        print(f"Error: {e}")
    except Exception as e:
        print(f"Unexpected error: {e}")

if __name__ == "__main__":
    main()
```

***

#### Key Additions

1. **Skill Proficiency Calculation**:
   * **calculate\_skill\_proficiencies**: Groups responses by skill, estimates `theta` per skill using the 3PL IRT model, and maps it to a 1–7 scale.
   * **theta\_to\_proficiency**: Maps `theta` (typically -3 to 3 in IRT) to 1–7:
     * < -2.0: 1 (very low)
     * -2.0 to -1.0: 2
     * -1.0 to 0.0: 3
     * 0.0 to 1.0: 4 (average)
     * 1.0 to 2.0: 5
     * 2.0 to 3.0: 6
     * ≥ 3.0: 7 (very high)
   * Default proficiency is 3 (average) for skills with no questions.
2. **Updated Adaptive Selection**:
   * `select_adaptive_questions`: Now returns both questions and responses, enabling proficiency calculation.
   * Responses are simulated for demo purposes but can be replaced with real student input.
3. **Output**:
   * Prints overall `theta` and per-skill proficiencies for Math and Reading/Writing sections in a readable format.

***

#### Example Output

Assuming simulated responses:

```
Math test: 35 questions, Final theta: 1.23
Math Skill Proficiencies (1-7):
  Algebra:
    Linear Equations in One Variable: 4
    Linear Functions: 5
    Linear Equations in Two Variables: 4
    Systems of Two Linear Equations in Two Variables: 5
    Linear Inequalities in One or Two Variables: 3
  Advanced Math:
    Nonlinear Functions: 6
    Nonlinear Equations in One Variable and Systems of Equations: 4
    Equivalent Expressions: 3
  ...

R&W test: 35 questions, Final theta: 0.87
Reading and Writing Skill Proficiencies (1-7):
  Information and Ideas:
    Central Ideas and Details: 4
    Inferences: 5
    Command of Evidence: 4
  Craft and Structure:
    Words in Context: 5
    Text Structure and Purpose: 4
    Cross-Text Connections: 3
  ...
```

***

#### Usage

1. **Prepare the Bank**: Use a `question_bank.json` with sufficient questions (e.g., 100+ per section).
2.  **Run**: Execute the script:

    ```bash
    python script.py
    ```
3. **Output**: Saves question files (`sat_math_diagnostic.json`, `sat_rw_diagnostic.json`) and prints proficiencies.

***

#### Notes

*   **Real Responses**: Replace the simulation in `select_adaptive_questions` with actual student answers:

    ```python
    response = True if student_answer == question["correct_answer"] else False
    ```
* **Precision**: Skills with fewer questions (e.g., 1–2) have less reliable `theta` estimates. For better accuracy, increase questions per skill or use domain-level `theta` as a fallback.
* **Scale Tuning**: Adjust `theta_to_proficiency` thresholds if your `theta` range differs (e.g., based on your question bank’s `b` values).

Let me know if you’d like to refine the proficiency scale or add more features!
