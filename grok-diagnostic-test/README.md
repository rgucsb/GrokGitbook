# Grok: Diagnostic Test

Below is detailed documentation in Markdown format for the **SAT Diagnostic Test** and **Review Applications**. This is designed for an engineer to build both the frontend and backend, covering the purpose, code structure, data flow, API specifications, and integration details. It assumes the diagnostic test generates results that the review app uses, as implemented in the provided Python scripts.

***

## SAT Diagnostic Test and Review Application Documentation

### Overview

This documentation covers two Python applications designed for a smart SAT test prep system:

1. **SAT Diagnostic Test**: An adaptive test using a 3PL IRT model to assess a student’s Math and Reading/Writing skills, outputting results to `diagnostic_results.json`.
2. **SAT Review Application**: An interactive, AI-enhanced chat that reviews the diagnostic results, providing feedback and estimated SAT scores.

The goal is to integrate these into a web or mobile app with a frontend (user interface) and backend (server-side logic), enabling students to take the test and review their performance.

***

### SAT Diagnostic Test Application

#### Purpose

The diagnostic test generates a 90-minute adaptive SAT test (45 minutes Math, 45 minutes Reading/Writing, 35 questions each) to:

* Assess student ability (theta) using IRT.
* Calculate proficiency (1–7) for skills and domains.
* Save results for review.

#### Code Structure

* **File**: `diagnostic_test.py`
* **Dependencies**: `numpy`, `scipy`, `json`, `random`
* **Key Components**:
  * **Test Plans**: `MATH_TEST_PLAN`, `RW_TEST_PLAN` define question counts per skill and difficulty.
  * **QuestionBank**: Loads questions from `question_bank.json`.
  * **IRTSelector**: Implements 3PL IRT for adaptive question selection and theta updates.
  * **Functions**:
    * `select_adaptive_questions`: Picks 35 questions per section.
    * `calculate_skill_proficiencies`: Computes proficiency per skill.
    * `save_results`: Outputs `diagnostic_results.json`.

#### Data Flow

1. **Input**: `question_bank.json` (questions with metadata and IRT parameters).
2. **Process**:
   * Initialize `IRTSelector` with theta = 0.
   * Select questions adaptively, updating theta after each simulated response.
   * Calculate skill proficiencies from responses.
3. **Output**:
   * `sat_math_diagnostic.json`: Math questions.
   * `sat_rw_diagnostic.json`: Reading/Writing questions.
   * `diagnostic_results.json`: Results for review.

#### Output Format (`diagnostic_results.json`)

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
                "Advanced Math": {...},
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

### SAT Review Application

#### Purpose

The review app provides an interactive, AI-supported chat to:

* Interpret `diagnostic_results.json`.
* Estimate SAT scores (200–800 per section).
* Offer feedback on strengths (proficiency ≥ 5) and areas to improve (≤ 3).

#### Code Structure

* **File**: `review_app.py`
* **Dependencies**: `json`, `time`, `random`
* **Key Components**:
  * **load\_results**: Reads `diagnostic_results.json`.
  * **proficiency\_to\_sat\_score**: Maps proficiency to SAT scores.
  * **ai\_respond**: Simulates Grok-like AI responses (replaceable with xAI API).
  * **chat\_with\_student**: Manages the interactive chat flow.

#### Data Flow

1. **Input**: `diagnostic_results.json` from the diagnostic test.
2. **Process**:
   * Calculate section-level proficiency averages.
   * Map to SAT scores.
   * Generate AI-driven feedback via chat steps.
3. **Output**: Console-based chat (to be adapted for frontend).

#### Chat Flow

1. **Greeting**: Welcomes the student.
2. **Overall Performance**: Shows estimated SAT scores.
3. **Math Review**: Overview, domains, optional skills.
4. **Reading/Writing Review**: Same structure.
5. **Wrap-Up**: Summarizes strengths and improvements.

***

### Backend Design

#### Framework

* **Recommended**: Flask or FastAPI (Python-based REST API).
* **Database**: PostgreSQL or MongoDB for scalability.

#### API Endpoints

**Diagnostic Test API**

1. **Start Test**
   * **Endpoint**: `POST /diagnostic/start`
   * **Input**: `{ "user_id": "123", "section": "Math" }`
   * **Output**: First question JSON (from `question_bank.json`).
   * **Logic**: Initialize `IRTSelector`, pick question via `select_adaptive_questions`.
2. **Submit Response**
   * **Endpoint**: `POST /diagnostic/respond`
   * **Input**: `{ "user_id": "123", "question_id": "d9e83476", "answer": "B" }`
   * **Output**: Next question JSON or results if 35 questions completed.
   * **Logic**: Update theta, store response, select next question or finalize.
3. **Get Results**
   * **Endpoint**: `GET /diagnostic/results?user_id=123`
   * **Output**: `diagnostic_results.json` structure.
   * **Logic**: Return saved results after test completion.

**Review API**

1. **Start Review**
   * **Endpoint**: `POST /review/start`
   * **Input**: `{ "user_id": "123" }`
   * **Output**: `{ "message": "Hey there! I’m Grok...", "step": "greeting" }`
   * **Logic**: Load results, begin chat.
2. **Next Chat Step**
   * **Endpoint**: `POST /review/next`
   * **Input**: `{ "user_id": "123", "step": "greeting", "response": "yes" }`
   * **Output**: `{ "message": "Here’s the big picture...", "step": "overall" }`
   * **Logic**: Process response, advance chat using `ai_respond`.

#### Database Schema

* **Questions Table**:
  * `id` (string): Question ID
  * `test` (string): "Math" or "Reading and Writing"
  * `domain` (string)
  * `skill` (string)
  * `difficulty` (string)
  * `data` (JSON): Question details
  * `irt_params` (JSON): `{ "a": float, "b": float, "c": float }`
* **User Responses Table**:
  * `user_id` (string)
  * `question_id` (string)
  * `response` (boolean)
  * `timestamp` (datetime)
* **Results Table**:
  * `user_id` (string)
  * `results` (JSON): `diagnostic_results.json` structure
  * `timestamp` (datetime)

#### Backend Logic

* **Diagnostic**:
  * Replace `question_bank.json` with database queries.
  * Store responses and update theta in real-time.
  * Save final results to DB and file (optional).
* **Review**:
  * Fetch results from DB.
  * Use `ai_respond` (or xAI API) for chat responses.
  * Track chat state (e.g., Redis or session).

***

### Frontend Design

#### Framework

* **Recommended**: React (web) or Flutter (mobile).

#### UI Components

1. **Diagnostic Test Interface**:
   * **Start Screen**: Select section (Math or Reading/Writing).
   * **Question Display**: Show question, options, timer (45 min/section).
   * **Submit Button**: Send answer to backend.
   * **Progress Bar**: 35 questions total.
   * **End Screen**: "Test complete, review your results!"
2. **Review Interface**:
   * **Chat Window**: Display Grok’s messages, scrollable.
   * **Input Field**: Text box for “yes”/other responses.
   * **Next Button**: Advance chat manually (optional).
   * **Score Summary**: Visual (e.g., bar chart) for Math, R\&W, total SAT scores.

#### Data Flow

1. **Diagnostic**:
   * `GET /diagnostic/start` → Display first question.
   * `POST /diagnostic/respond` → Submit answer, get next question.
   * After 35 questions → Fetch results, prompt review.
2. **Review**:
   * `POST /review/start` → Show greeting.
   * `POST /review/next` → Send user response, display next message.

***

### Integration Details

#### Connecting Frontend and Backend

* **Diagnostic**:
  * Frontend calls `/diagnostic/start` to begin.
  * Loops through `/diagnostic/respond` for 35 questions.
  * Stores results via `/diagnostic/results`.
* **Review**:
  * Frontend calls `/review/start` with user ID.
  * Iterates `/review/next` based on user input.

#### AI Integration (xAI Grok)

*   **Backend**: Replace `ai_respond` with:

    ```python
    import requests
    def ai_respond(context, data):
        response = requests.post("https://api.xai.com/grok", json={"prompt": context, "data": data})
        return response.json()["response"]
    ```
* **Frontend**: Display AI responses as chat messages.

#### File vs. Database

* **Current**: Uses `diagnostic_results.json` as a bridge.
* **Recommended**: Store results in DB, remove file dependency:
  * Backend saves to DB after diagnostic.
  * Review fetches from DB via API.

***

### Calculations Explained

#### Theta

* **What**: Student ability (-3 to 3) via 3PL IRT.
* **Calculation**: Updated iteratively using MLE:  \
  \[  \
  \theta\_{\text{new\}} = \theta\_{\text{old\}} + \frac{\sum a\_i (r\_i - P\_i)}{\sum a\_i^2 (P\_i - c\_i) (1 - P\_i) / (1 - c\_i)}  \
  ]
* **Code**: `IRTSelector.update_theta`.

#### Proficiency

* **Skill**: Theta per skill → 1–7 (e.g., theta = 2.5 → 6).
* **Domain**: Average skill proficiencies (e.g., Algebra = 4.2).
* **Section**: Average domain proficiencies (e.g., Math = 3.875).
* **Code**: `calculate_skill_proficiencies`, `theta_to_proficiency`.

#### SAT Score

* **Section**: Proficiency → 200–800 (e.g., 3.875 → 490).
  * ( \text{Score} = 200 + (\text{Proficiency} - 1) \times 100 )
* **Total**: Math + Reading/Writing (e.g., 990).
* **Code**: `proficiency_to_sat_score`.

***

### Testing

#### Diagnostic Test

* **Unit Tests**:
  * Verify theta updates with mock responses.
  * Check 35 questions selected per section.
  * Validate `diagnostic_results.json` structure.
* **Integration**: Run with sample `question_bank.json`, confirm outputs.

#### Review App

* **Unit Tests**:
  * Test `proficiency_to_sat_score` with edge cases (1, 4, 7).
  * Mock `ai_respond` outputs for consistency.
* **Integration**: Run after diagnostic, test chat with “yes”/“no” inputs.

***

### Recommendations

* **Scalability**: Use a DB (e.g., PostgreSQL) for questions and results.
* **Real-Time**: Replace simulated responses with student input via frontend.
* **AI**: Integrate xAI Grok API for dynamic responses.
* **UI/UX**: Add visualizations (e.g., proficiency graphs) in review.

This documentation provides a blueprint for an engineer to build a full-stack SAT prep app. Let me know if you need more specifics!
