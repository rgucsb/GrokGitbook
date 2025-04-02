# Integrated Diagnostic and Review Apps

I'll update the diagnostic test code to produce a `diagnostic_results.json` file with a structure compatible with the review app, and then update the review app to read and use this output file directly. Below are the two updated scripts.

***

#### Updated Diagnostic Test Code

This version ensures the diagnostic test outputs a `diagnostic_results.json` file with a clear structure: theta and proficiencies for each section, plus metadata.

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
        return [
            q for q in self.questions
            if q["metadata"]["Test"] == test
            and q["metadata"]["Domain"] == domain
            and q["metadata"]["Skill"] == skill
            and q["metadata"]["Question ID"] not in used_ids
        ]

class IRTSelector:
    def __init__(self, initial_theta: float = 0.0):
        self.theta = initial_theta
    
    def probability_correct(self, a: float, b: float, c: float) -> float:
        logit = a * (self.theta - b)
        return c + (1 - c) / (1 + np.exp(-logit))
    
    def information(self, a: float, b: float, c: float) -> float:
        p = self.probability_correct(a, b, c)
        q = 1 - p
        if p <= c or q == 0:
            return 0
        return (a**2 * (p - c)**2 * q) / ((1 - c)**2 * p)
    
    def update_theta(self, responses: List[Tuple[Dict, bool]]) -> float:
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
    if theta < -2.0: return 1
    elif theta < -1.0: return 2
    elif theta < 0.0: return 3
    elif theta < 1.0: return 4
    elif theta < 2.0: return 5
    elif theta < 3.0: return 6
    else: return 7

def calculate_skill_proficiencies(responses: List[Tuple[Dict, bool]], test_plan: Dict) -> Dict[str, Dict[str, int]]:
    skill_responses = {domain: {skill: [] for skill in skills} for domain, skills in test_plan.items()}
    for q, r in responses:
        domain = q["metadata"]["Domain"]
        skill = q["metadata"]["Skill"]
        skill_responses[domain][skill].append((q, r))
    
    irt = IRTSelector(initial_theta=0.0)
    proficiencies = {}
    for domain, skills in skill_responses.items():
        proficiencies[domain] = {}
        for skill, resp in skills.items():
            theta = irt.update_theta(resp) if resp else 0.0
            proficiencies[domain][skill] = theta_to_proficiency(theta) if resp else 3
    return proficiencies

def select_adaptive_questions(question_bank: QuestionBank, test_plan: Dict, test_type: str, irt_selector: IRTSelector) -> Tuple[List[Dict], List[Tuple[Dict, bool]]]:
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
        
        p_correct = irt_selector.probability_correct(
            question["irt_parameters"]["a"], question["irt_parameters"]["b"], question["irt_parameters"]["c"])
        response = random.random() < p_correct  # Simulated response
        responses.append((question, response))
        
        irt_selector.theta = irt_selector.update_theta(responses)
    
    if len(selected_questions) != 35:
        raise ValueError(f"Selected {len(selected_questions)} questions for {test_type}, expected 35")
    return selected_questions, responses

def save_test(questions: List[Dict], output_file: str):
    try:
        with open(output_file, 'w') as f:
            json.dump(questions, f, indent=4)
    except IOError as e:
        raise IOError(f"Failed to save test to '{output_file}': {e}")

def save_results(results: Dict, output_file: str = "diagnostic_results.json"):
    """Save diagnostic results in a format compatible with the review app."""
    try:
        with open(output_file, 'w') as f:
            json.dump(results, f, indent=4)
    except IOError as e:
        raise IOError(f"Failed to save results to '{output_file}': {e}")

def main():
    question_bank_file = "question_bank.json"
    
    try:
        qb = QuestionBank(question_bank_file)
        irt_selector_math = IRTSelector(initial_theta=0.0)
        irt_selector_rw = IRTSelector(initial_theta=0.0)
        
        # Math test
        math_questions, math_responses = select_adaptive_questions(qb, MATH_TEST_PLAN, "Math", irt_selector_math)
        math_proficiencies = calculate_skill_proficiencies(math_responses, MATH_TEST_PLAN)
        save_test(math_questions, "sat_math_diagnostic.json")
        
        # Reading and Writing test
        rw_questions, rw_responses = select_adaptive_questions(qb, RW_TEST_PLAN, "Reading and Writing", irt_selector_rw)
        rw_proficiencies = calculate_skill_proficiencies(rw_responses, RW_TEST_PLAN)
        save_test(rw_questions, "sat_rw_diagnostic.json")
        
        # Compile results for review app
        results = {
            "sections": {
                "Math": {
                    "theta": float(irt_selector_math.theta),  # Ensure JSON-serializable
                    "proficiencies": math_proficiencies
                },
                "Reading and Writing": {
                    "theta": float(irt_selector_rw.theta),
                    "proficiencies": rw_proficiencies
                }
            },
            "metadata": {
                "date": "2025-03-26",
                "version": "1.0"
            }
        }
        save_results(results)
        print(f"Diagnostic completed. Results saved to 'diagnostic_results.json'.")
    
    except (FileNotFoundError, ValueError, IOError) as e:
        print(f"Error: {e}")
    except Exception as e:
        print(f"Unexpected error: {e}")

if __name__ == "__main__":
    main()
```

**Key Changes:**

* **Output Structure**: The `save_results` function creates `diagnostic_results.json` with:
  * `sections`: "Math" and "Reading and Writing" with `theta` (float) and `proficiencies` (nested dict of domains and skills).
  * `metadata`: Date and version for tracking.
* **Serialization**: Ensures `theta` is a float for JSON compatibility.
* **Purpose**: Provides all necessary data for the review app to calculate scores and display feedback.

**Example `diagnostic_results.json`:**

```json
{
    "sections": {
        "Math": {
            "theta": 1.23,
            "proficiencies": {
                "Algebra": {
                    "Linear Equations in One Variable": 4,
                    "Linear Functions": 5,
                    ...
                },
                ...
            }
        },
        "Reading and Writing": {
            "theta": 0.87,
            "proficiencies": {
                "Information and Ideas": {
                    "Central Ideas and Details": 4,
                    "Inferences": 5,
                    ...
                },
                ...
            }
        }
    },
    "metadata": {
        "date": "2025-03-26",
        "version": "1.0"
    }
}
```

***

#### Updated Review App Code

This version reads the `diagnostic_results.json` file directly and uses its data for the interactive, AI-enhanced review.

```python
import json
import time
import random
from typing import Dict

def load_results(file_path: str = "diagnostic_results.json") -> Dict:
    """Load diagnostic results from the specified JSON file."""
    try:
        with open(file_path, 'r') as f:
            return json.load(f)
    except FileNotFoundError:
        print(f"Results file '{file_path}' not found. Please run the diagnostic test first.")
        raise
    except json.JSONDecodeError:
        print(f"Invalid JSON in '{file_path}'. Please ensure the diagnostic output is valid.")
        raise

def proficiency_to_sat_score(avg_proficiency: float, section: str) -> int:
    """Map average proficiency (1-7) to an SAT section score (200-800)."""
    base_score = 200
    max_score = 800
    score_range = max_score - base_score
    proficiency_range = 7 - 1
    score_per_prof = score_range / proficiency_range
    scaled_score = base_score + (avg_proficiency - 1) * score_per_prof
    return min(max(round(scaled_score / 10) * 10, base_score), max_score)

def ai_respond(context: str, data: Dict = None) -> str:
    """Simulate Grok-like AI response based on context and data."""
    positive_phrases = [
        "You’re doing awesome!", "That’s fantastic!", "You’ve got this!", 
        "I’m so proud of you!", "Way to go!", "You’re a rockstar!"
    ]
    encouragement_phrases = [
        "We’ll boost this up together!", "This is just a fun challenge!", 
        "I’ve got your back!", "Let’s turn this into a strength!"
    ]
    
    if "greeting" in context:
        return f"Hey there! I’m Grok, your SAT prep buddy. {random.choice(positive_phrases)} " \
               "Let’s dive into your diagnostic results together—ready?"
    
    elif "overall" in context:
        math_score = data["math_score"]
        rw_score = data["rw_score"]
        total = math_score + rw_score
        tone = "super impressive" if total > 1200 else "awesome" if total > 1000 else "a great start"
        return f"Here’s the big picture: your Math score is {math_score}/800, and Reading/Writing is {rw_score}/800. " \
               f"That’s {total}/1600—{tone}! Want to know how we got those numbers?"
    
    elif "math_overview" in context:
        math_score = data["math_score"]
        return f"Let’s talk Math! Your estimated score is {math_score}/800—{'great work' if math_score >= 500 else 'a solid start'}! " \
               f"It’s split into four areas. Want to check them out?"
    
    elif "domain" in context:
        domain, avg_prof, domain_score = data["domain"], data["avg_prof"], data["domain_score"]
        tone = "amazing" if avg_prof >= 5 else "solid" if avg_prof >= 4 else "a good foundation"
        extra = f"{random.choice(encouragement_phrases)}" if avg_prof < 4 else "Keep that momentum going!"
        return f"In {domain}, you averaged {avg_prof:.1f}/7, adding about {domain_score} points to your score. " \
               f"That’s {tone}! {extra}"
    
    elif "skill_strength" in context:
        skill, prof = data["skill"], data["prof"]
        return f"You crushed {skill} with a {prof}/7! {random.choice(positive_phrases)}"
    
    elif "skill_improve" in context:
        skill, prof = data["skill"], data["prof"]
        return f"{skill} was {prof}/7. {random.choice(encouragement_phrases)}"
    
    elif "rw_overview" in context:
        rw_score = data["rw_score"]
        return f"Now, Reading and Writing! Your score is {rw_score}/800—{'fantastic' if rw_score >= 500 else 'a great base'}! " \
               f"It’s got three areas—want to see them?"
    
    elif "wrap_up" in context:
        total_score = data["total_score"]
        strengths = data["strengths"]
        improvements = data["improvements"]
        tone = "outstanding" if total_score > 1200 else "awesome" if total_score > 1000 else "a super start"
        return f"You’re incredible! Your total SAT score is {total_score}/1600—{tone}! " \
               f"You’re shining in {strengths[0][0]} ({strengths[0][1]}/7) and {strengths[1][0]} ({strengths[1][1]}/7), " \
               f"and we’ll power up {improvements[0][0]} ({improvements[0][1]}/7) and {improvements[1][0]} ({improvements[1][1]}/7) together. " \
               "What’s next on your SAT prep adventure?"
    
    return "Hmm, let’s keep moving—want to talk about something specific?"

def chat_with_student(results: Dict):
    """Interactive chat with AI to review SAT diagnostic results."""
    def pause(seconds: int = 2):
        time.sleep(seconds)
    
    def get_input(prompt: str) -> str:
        return input(prompt).strip().lower()

    # Extract data from results
    math_profs = results["sections"]["Math"]["proficiencies"]
    rw_profs = results["sections"]["Reading and Writing"]["proficiencies"]
    math_avg_prof = sum(sum(skills.values()) / len(skills) for skills in math_profs.values()) / len(math_profs)
    rw_avg_prof = sum(sum(skills.values()) / len(skills) for skills in rw_profs.values()) / len(rw_profs)
    math_score = proficiency_to_sat_score(math_avg_prof, "Math")
    rw_score = proficiency_to_sat_score(rw_avg_prof, "Reading and Writing")

    # Step 1: Greeting
    print(ai_respond("greeting"))
    pause()
    if get_input(">> ") != "yes":
        print("No rush! Come back when you’re ready—just say 'yes'!")
        return

    # Step 2: Overall Performance
    print(ai_respond("overall", {"math_score": math_score, "rw_score": rw_score}))
    pause(3)
    if get_input(">> ") == "yes":
        math_theta = results["sections"]["Math"]["theta"]
        rw_theta = results["sections"]["Reading and Writing"]["theta"]
        print(f"Alright! Your Math theta was {math_theta:.2f}, and Reading/Writing was {rw_theta:.2f}. "
              "We turn those into proficiencies (1-7), then map them to SAT scores: 1 is 200, 4 is 500, 7 is 800. Cool, right?")
    else:
        print("No problem, let’s jump into the fun stuff!")
    pause()

    # Step 3: Math Overview
    print(ai_respond("math_overview", {"math_score": math_score}))
    pause(2)
    if get_input(">> ") != "yes":
        print("Okay, we’ll skip ahead. Just say 'yes' anytime to dive in!")
    else:
        # Step 4: Math Domains
        for domain, skills in math_profs.items():
            avg_prof = sum(skills.values()) / len(skills)
            domain_score = proficiency_to_sat_score(avg_prof, "Math") // 4
            print(ai_respond("domain", {"domain": domain, "avg_prof": avg_prof, "domain_score": domain_score}))
            pause(2)

        # Step 5: Math Skills (Optional)
        print("\nWant the nitty-gritty on your Math skills? Say 'yes'!")
        if get_input(">> ") == "yes":
            strengths = []
            improvements = []
            for domain, skills in math_profs.items():
                print(f"\n{domain} Skills:")
                pause()
                for skill, prof in skills.items():
                    print(f"- {skill}: {prof}/7")
                    if prof >= 5:
                        strengths.append((skill, prof))
                    elif prof <= 3:
                        improvements.append((skill, prof))
                    pause(1)
            
            print("\nYour Math Strengths:")
            pause()
            for skill, prof in strengths:
                print(ai_respond("skill_strength", {"skill": skill, "prof": prof}))
                pause(1)
            print("\nAreas to Boost:")
            pause()
            for skill, prof in improvements:
                print(ai_respond("skill_improve", {"skill": skill, "prof": prof}))
                pause(1)
        else:
            print("No worries, we’ll keep it chill for Math!")

    # Step 6: Reading and Writing Overview
    print(ai_respond("rw_overview", {"rw_score": rw_score}))
    pause(2)
    if get_input(">> ") != "yes":
        print("All good—let’s wrap up whenever you’re ready!")
    else:
        # Step 7: Reading and Writing Domains
        for domain, skills in rw_profs.items():
            avg_prof = sum(skills.values()) / len(skills)
            domain_score = proficiency_to_sat_score(avg_prof, "Reading and Writing") // 3
            print(ai_respond("domain", {"domain": domain, "avg_prof": avg_prof, "domain_score": domain_score}))
            pause(2)

        # Step 8: Reading and Writing Skills (Optional)
        print("\nWant to peek at your Reading and Writing skills? Say 'yes'!")
        if get_input(">> ") == "yes":
            strengths = []
            improvements = []
            for domain, skills in rw_profs.items():
                print(f"\n{domain} Skills:")
                pause()
                for skill, prof in skills.items():
                    print(f"- {skill}: {prof}/7")
                    if prof >= 5:
                        strengths.append((skill, prof))
                    elif prof <= 3:
                        improvements.append((skill, prof))
                    pause(1)
            
            print("\nYour Reading and Writing Strengths:")
            pause()
            for skill, prof in strengths:
                print(ai_respond("skill_strength", {"skill": skill, "prof": prof}))
                pause(1)
            print("\nAreas to Boost:")
            pause()
            for skill, prof in improvements:
                print(ai_respond("skill_improve", {"skill": skill, "prof": prof}))
                pause(1)
        else:
            print("Sweet, we’ll keep it light!")

    # Step 9: Wrap-Up
    strengths = [(s, p) for d in math_profs.values() for s, p in d.items() if p >= 5] + \
                [(s, p) for d in rw_profs.items() for s, p in d.items() if p >= 5]
    improvements = [(s, p) for d in math_profs.values() for s, p in d.items() if p <= 3] + \
                   [(s, p) for d in rw_profs.items() for s, p in d.items() if p <= 3]
    print(ai_respond("wrap_up", {"total_score": math_score + rw_score, "strengths": strengths[:2], "improvements": improvements[:2]}))
    pause(2)
    print("What do you want to tackle next? Just let me know!")

def main():
    try:
        results = load_results("diagnostic_results.json")
        chat_with_student(results)
    except (FileNotFoundError, ValueError) as e:
        print(f"Error: {e}")
    except Exception as e:
        print(f"Unexpected error: {e}")

if __name__ == "__main__":
    main()
```

**Key Changes:**

* **Input Source**: `load_results` now reads `diagnostic_results.json` directly, expecting the structure from the diagnostic test output. It raises an error if the file is missing or invalid, removing the fallback to sample data.
* **Data Access**: Uses `results["sections"]["Math"]` and `results["sections"]["Reading and Writing"]` to access theta and proficiencies, matching the diagnostic output.
* **Error Handling**: Provides clear feedback if the file isn’t found, encouraging the user to run the diagnostic first.

***

#### How to Use

1. **Run the Diagnostic Test**:
   * Save the diagnostic code as `diagnostic_test.py`.
   * Ensure a valid `question_bank.json` exists.
   * Run: `python diagnostic_test.py`
   * This generates `diagnostic_results.json`, `sat_math_diagnostic.json`, and `sat_rw_diagnostic.json`.
2. **Run the Review App**:
   * Save the review code as `review_app.py`.
   * Run: `python review_app.py`
   * The review app will read `diagnostic_results.json` and start the interactive chat.

**Example Workflow:**

* After running `diagnostic_test.py`, `diagnostic_results.json` is created.
* Running `review_app.py` will use this file to provide feedback based on the actual diagnostic results, not sample data.

***

#### Testing

* **Diagnostic Test**: Verify `diagnostic_results.json` is created with the expected structure (check theta and proficiencies for both sections).
* **Review App**: Run after the diagnostic to ensure it loads the file correctly. Test with “yes” and “no” inputs to confirm the chat flow and data accuracy (e.g., scores match proficiency averages).
* **Error Cases**: Delete `diagnostic_results.json` and run the review app to confirm it prompts to run the diagnostic first.

This ensures seamless integration between the diagnostic test and review app, using real data for a personalized experience! Let me know if you need further adjustments!
