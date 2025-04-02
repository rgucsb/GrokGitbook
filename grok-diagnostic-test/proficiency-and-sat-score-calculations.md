# Proficiency and SAT Score calculations

Below is an explanation of **theta**, how it’s calculated, how proficiency is determined for skills, domains, and sections (Math or Reading and Writing), and how the SAT score is estimated in the provided code. This is tailored to the context of the adaptive SAT diagnostic test and review app.

***

#### What is Theta?

**Theta (θ)** is a parameter in **Item Response Theory (IRT)** that represents a student’s **latent ability** or skill level on a continuous scale, typically ranging from about -3 to 3 in standardized IRT models. In the context of this SAT diagnostic:

* A higher theta (e.g., 2.0) indicates greater ability.
* A lower theta (e.g., -1.0) indicates lower ability.
* Theta = 0 is average ability relative to the question difficulty.

Theta is used to:

* Adaptively select questions that match the student’s ability (via the 3PL IRT model).
* Estimate how well a student performs across skills, domains, and sections.

***

#### How is Theta Calculated?

Theta is calculated using the **3-parameter logistic (3PL) IRT model**, which considers three item parameters for each question:

1. **a (Discrimination)**: How well the question distinguishes between students of different abilities (higher `a` = steeper curve).
2. **b (Difficulty)**: The ability level (theta) at which a student has a 50% chance of answering correctly (after accounting for guessing).
3. **c (Guessing)**: The probability of guessing the answer correctly (typically 0.25 for 4-choice questions).

**Probability of Correct Response**

The 3PL model calculates the probability ( P(\theta) ) that a student with ability ( \theta ) answers a question correctly:\
\[\
P(\theta) = c + \frac{1 - c}{1 + e^{-a(\theta - b)\}}\
]

* If ( \theta = b ), the probability is halfway between ( c ) and 1.
* As ( \theta ) increases beyond ( b ), ( P(\theta) ) approaches 1.
* As ( \theta ) decreases below ( b ), ( P(\theta) ) approaches ( c ).

**Updating Theta**

Theta is updated iteratively using **maximum likelihood estimation (MLE)** after each response:

1. **Initial Theta**: Starts at 0.0 (average ability).
2. **Response Data**: For each question answered (correct = 1, incorrect = 0), the code uses:
   * ( a, b, c ) from the question’s `irt_parameters`.
   * The student’s response (simulated in the code as `random.random() < P(θ)`).
3. **Iteration**:
   * Compute the difference between observed responses (1 or 0) and expected probabilities ( P(\theta) ).
   * Adjust theta using the formula:     \
     \[     \
     \theta\_{\text{new\}} = \theta\_{\text{old\}} + \frac{\sum a\_i (r\_i - P\_i)}{\sum a\_i^2 (P\_i - c\_i) (1 - P\_i) / (1 - c\_i)}     \
     ]
     * ( r\_i ): Response (1 or 0).
     * ( P\_i ): Probability of correct response for question ( i ).
   * Repeat for up to 10 iterations or until convergence.
4. **Result**: The final theta reflects the student’s ability based on all responses.

In the code (`IRTSelector.update_theta`):

* It processes all responses to update theta globally for each section (Math or Reading/Writing).
* For skill-specific theta, it’s recalculated using only responses for that skill.

***

#### How is Proficiency Calculated?

Proficiency is a discrete score (1–7) derived from theta to make it more interpretable for students. It’s calculated at three levels: **skills**, **domains**, and **sections**.

**Skill-Level Proficiency**

* **Process**:
  1. Group responses by skill (e.g., "Linear Functions") in `calculate_skill_proficiencies`.
  2. Calculate a skill-specific theta using `IRTSelector.update_theta` with only that skill’s responses.
  3. Map theta to proficiency using `theta_to_proficiency`:
     * ( \theta < -2.0 ): 1
     * ( -2.0 \leq \theta < -1.0 ): 2
     * ( -1.0 \leq \theta < 0.0 ): 3
     * ( 0.0 \leq \theta < 1.0 ): 4
     * ( 1.0 \leq \theta < 2.0 ): 5
     * ( 2.0 \leq \theta < 3.0 ): 6
     * ( \theta \geq 3.0 ): 7
  4. If no responses exist for a skill, default to 3 (average).
* **Example**: If a student answers 3 questions on "Nonlinear Functions" and gets a skill theta of 2.5, their proficiency is 6.

**Domain-Level Proficiency**

* **Process**:
  1. Average the proficiency scores of all skills within a domain (e.g., Algebra).
  2. Computed in the review app as `sum(skills.values()) / len(skills)` for each domain.
* **Example**: Algebra skills = \[4, 5, 4, 5, 3], domain proficiency = (4 + 5 + 4 + 5 + 3) / 5 = 4.2.

**Section-Level Proficiency (Math or Reading/Writing)**

* **Process**:
  1. Average the proficiency scores across all domains in the section.
  2. Computed in the review app as:
     * Math: Average of 4 domain proficiencies.
     * Reading/Writing: Average of 3 domain proficiencies.
* **Example**: Math domain proficiencies = \[4.2, 4.3, 3.2, 3.8], section proficiency = (4.2 + 4.3 + 3.2 + 3.8) / 4 ≈ 3.875.

***

#### How is the SAT Score Calculated?

The SAT score (200–800 per section, 400–1600 total) is estimated from the section-level proficiency in the review app (`proficiency_to_sat_score`).

**Calculation Process**

1. **Section Proficiency**: Use the average proficiency for the section (Math or Reading/Writing).
2. **Mapping to SAT Scale**:
   * SAT scores range from 200 (minimum) to 800 (maximum) per section.
   * Proficiency ranges from 1 to 7.
   * Linear interpolation:
     * ( \text{Score} = 200 + (\text{Proficiency} - 1) \times \frac{800 - 200}{7 - 1} )
     * ( \text{Score} = 200 + (\text{Proficiency} - 1) \times 100 )
   * Round to the nearest 10 and clamp between 200 and 800.
3. **Examples**:
   * Proficiency = 1: ( 200 + (1 - 1) \times 100 = 200 )
   * Proficiency = 4: ( 200 + (4 - 1) \times 100 = 500 )
   * Proficiency = 7: ( 200 + (7 - 1) \times 100 = 800 )
   * Proficiency = 3.875: ( 200 + (3.875 - 1) \times 100 = 487.5 ), rounded to 490.

**Section Scores**

* **Math Score**: Based on Math section proficiency (e.g., 3.875 → 490).
* **Reading/Writing Score**: Based on Reading/Writing section proficiency (e.g., 4.0 → 500).
* **Total Score**: Sum of Math and Reading/Writing scores (e.g., 490 + 500 = 990).

**Simplification Note**

* This is a **simplified model**:
  * Real SAT scoring uses raw scores (number correct) and a test-specific conversion table, adjusted for difficulty and form.
  * The code approximates this by mapping proficiency to a linear scale, assuming a consistent relationship between ability and score.

***

#### Summary in Context of the Code

1. **Theta Calculation**:
   * Starts at 0.0, updated after each response in `select_adaptive_questions` using `IRTSelector.update_theta`.
   * Final section theta (e.g., Math: 1.23) is saved in `diagnostic_results.json`.
2. **Proficiency Calculation**:
   * **Skill**: Theta per skill → 1–7 (e.g., Nonlinear Functions theta = 2.5 → 6).
   * **Domain**: Average of skill proficiencies (e.g., Algebra = 4.2).
   * **Section**: Average of domain proficiencies (e.g., Math = 3.875).
3. **SAT Score Calculation**:
   * Section proficiency → 200–800 scale (e.g., Math 3.875 → 490, Reading/Writing 4.0 → 500).
   * Total = Math + Reading/Writing (e.g., 990).

**Example from Code:**

* **Math Theta**: 1.23 → Section Proficiency ≈ 3.875 → SAT Math Score = 490.
* **Reading/Writing Theta**: 0.87 → Section Proficiency ≈ 4.0 → SAT R\&W Score = 500.
* **Total SAT Score**: 490 + 500 = 990.

This system provides a student-friendly way to interpret diagnostic performance and estimate SAT scores, though it’s an approximation compared to official scoring. Let me know if you’d like deeper details or a specific example!
