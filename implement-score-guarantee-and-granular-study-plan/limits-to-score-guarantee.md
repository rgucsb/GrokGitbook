# Limits to score guarantee

Let’s update the **Score Improvement Guarantee** for the **SAT Smart Prep App** by **Learner Labs** to limit eligibility to users who start with a score of 1250 or lower, based on either their initial diagnostic test or a previous SAT/PSAT score entered during onboarding. This change ensures the guarantee targets users who need the most improvement, aligning with the app’s mission to support students aiming for significant score gains while making the policy more sustainable for the company. I’ll update the user-facing conditions, backend logic, and re-evaluate Priya’s eligibility from the previous user journey to reflect this new restriction.

***

### Updated Conditions for the Score Improvement Guarantee

#### Updated User-Facing Conditions (Policy Screen/Page)

The policy is updated to include the new eligibility criterion for users starting with a score of 1250 or lower. This will be reflected in the Policy screen (mobile) and Policy page (web).

**Updated Score Improvement Guarantee Policy**

At **Learner Labs**, we are committed to helping you achieve your best SAT score with **SAT Smart Prep App**. We guarantee a 150-point score increase on your SAT practice tests if you meet the following conditions:

1. **Starting Score of 1250 or Lower**:
   * You must have a starting score of 1250 or lower, as determined by either:
     * Your first full-length SAT practice test (diagnostic test) within the app, or
     * Your previous SAT or PSAT score entered during onboarding.
   * If you do not provide a previous score, the app will use your first diagnostic test score to determine eligibility.
2. **Complete At Least Two Full-Length SAT Practice Tests**:
   * You must complete at least two full-length SAT practice tests within the app. These tests must be taken under timed conditions to accurately reflect your progress.
   * The first test establishes your baseline score, and the second (or later) test is used to measure your improvement.
3. **Follow the Recommended Study Plan for 30 Days**:
   * You must actively follow the app’s recommended study plan for at least 30 consecutive days after completing your first full-length test.
   * This includes completing at least 80% of the assigned tasks in your granular study plan (e.g., practice sessions, milestones) during this period.
4. **Achieve a 150-Point Increase**:
   * The guarantee applies to the difference between your first full-length SAT practice test score and your highest subsequent full-length SAT practice test score within the app.
   * If your score does not increase by at least 150 points, you are eligible for a refund.
5. **Submit a Refund Request Within 60 Days**:
   * You must submit a refund request within 60 days of your subscription start date (or the date of your first full-length test, whichever is later).
   * To request a refund, contact our support team at **support@learnerlabs.com** with your account details and a summary of your study activity.
6. **Subscription Requirement**:
   * The guarantee applies only to users with an active paid subscription (monthly or annual) to **SAT Smart Prep App**. Free trial users or users in promotional programs (e.g., College Board pilot) are not eligible unless they upgrade to a paid plan.

**Additional Notes**

* **Practice Test Scores Only**: This guarantee applies only to SAT practice test scores within the app and does not apply to official SAT scores reported by the College Board.
* **Refund Processing**: Refunds are processed within 14 business days of approval. The refund amount will be the full subscription fee paid, minus any applicable taxes or fees.
* **Fair Use**: Learner Labs reserves the right to deny refund requests if there is evidence of misuse, such as not following the study plan, skipping questions, or attempting to manipulate scores.

**How to Check Eligibility**

* Go to the **Dashboard** in the app and click "Check Score Guarantee" to see your score improvement and eligibility status.
* If eligible, follow the instructions to contact support for your refund.

**Update Policy Screen (Mobile)**

* **File**: `SATSmartPrepApp/src/screens/PolicyScreen.js`
*   **Updated Code**:

    ```javascript
    import React from 'react';
    import { View, Text, StyleSheet, ScrollView } from 'react-native';
    import Animated, { FadeIn, FadeOut } from 'react-native-reanimated';
    import { colors, typography } from '../styles';

    const PolicyScreen = () => {
      return (
        <Animated.View entering={FadeIn} exiting={FadeOut} style={styles.container}>
          <ScrollView>
            <Text style={typography.heading}>Score Improvement Guarantee</Text>
            <Text style={typography.body}>
              At Learner Labs, we are committed to helping you achieve your best SAT score with SAT Smart Prep App. We guarantee a 150-point score increase on your SAT practice tests if you meet the following conditions:
            </Text>
            <Text style={typography.body}>
              1. **Starting Score of 1250 or Lower**: You must have a starting score of 1250 or lower, as determined by either your first full-length SAT practice test (diagnostic test) within the app or your previous SAT or PSAT score entered during onboarding. If you do not provide a previous score, the app will use your first diagnostic test score to determine eligibility.
            </Text>
            <Text style={typography.body}>
              2. **Complete At Least Two Full-Length SAT Practice Tests**: You must complete at least two full-length SAT practice tests within the app. These tests must be taken under timed conditions to accurately reflect your progress. The first test establishes your baseline score, and the second (or later) test is used to measure your improvement.
            </Text>
            <Text style={typography.body}>
              3. **Follow the Recommended Study Plan for 30 Days**: You must actively follow the app’s recommended study plan for at least 30 consecutive days after completing your first full-length test. This includes completing at least 80% of the assigned tasks in your granular study plan (e.g., practice sessions, milestones) during this period.
            </Text>
            <Text style={typography.body}>
              4. **Achieve a 150-Point Increase**: The guarantee applies to the difference between your first full-length SAT practice test score and your highest subsequent full-length SAT practice test score within the app. If your score does not increase by at least 150 points, you are eligible for a refund.
            </Text>
            <Text style={typography.body}>
              5. **Submit a Refund Request Within 60 Days**: You must submit a refund request within 60 days of your subscription start date (or the date of your first full-length test, whichever is later). To request a refund, contact our support team at support@learnerlabs.com with your account details and a summary of your study activity.
            </Text>
            <Text style={typography.body}>
              6. **Subscription Requirement**: The guarantee applies only to users with an active paid subscription (monthly or annual) to SAT Smart Prep App. Free trial users or users in promotional programs (e.g., College Board pilot) are not eligible unless they upgrade to a paid plan.
            </Text>
            <Text style={typography.body}>
              **Additional Notes**:
              - **Practice Test Scores Only**: This guarantee applies only to SAT practice test scores within the app and does not apply to official SAT scores reported by the College Board.
              - **Refund Processing**: Refunds are processed within 14 business days of approval. The refund amount will be the full subscription fee paid, minus any applicable taxes or fees.
              - **Fair Use**: Learner Labs reserves the right to deny refund requests if there is evidence of misuse, such as not following the study plan, skipping questions, or attempting to manipulate scores.
            </Text>
            <Text style={typography.body}>
              **How to Check Eligibility**:
              Go to the Dashboard in the app and click "Check Score Guarantee" to see your score improvement and eligibility status. If eligible, follow the instructions to contact support for your refund.
            </Text>
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
    });

    export default PolicyScreen;
    ```

**Update Policy Page (Web)**

* **File**: `frontend/pages/policy.js`
*   **Updated Code**:

    ```javascript
    import React from 'react';
    import { useRouter } from 'next/router';
    import { colors, typography } from '../styles';

    export default function Policy() {
      const router = useRouter();

      return (
        <div className="policy">
          <h1 style={typography.heading}>Score Improvement Guarantee</h1>
          <p style={typography.body}>
            At Learner Labs, we are committed to helping you achieve your best SAT score with SAT Smart Prep App. We guarantee a 150-point score increase on your SAT practice tests if you meet the following conditions:
          </p>
          <p style={typography.body}>
            1. <strong>Starting Score of 1250 or Lower</strong>: You must have a starting score of 1250 or lower, as determined by either your first full-length SAT practice test (diagnostic test) within the app or your previous SAT or PSAT score entered during onboarding. If you do not provide a previous score, the app will use your first diagnostic test score to determine eligibility.
          </p>
          <p style={typography.body}>
            2. <strong>Complete At Least Two Full-Length SAT Practice Tests</strong>: You must complete at least two full-length SAT practice tests within the app. These tests must be taken under timed conditions to accurately reflect your progress. The first test establishes your baseline score, and the second (or later) test is used to measure your improvement.
          </p>
          <p style={typography.body}>
            3. <strong>Follow the Recommended Study Plan for 30 Days</strong>: You must actively follow the app’s recommended study plan for at least 30 consecutive days after completing your first full-length test. This includes completing at least 80% of the assigned tasks in your granular study plan (e.g., practice sessions, milestones) during this period.
          </p>
          <p style={typography.body}>
            4. <strong>Achieve a 150-Point Increase</strong>: The guarantee applies to the difference between your first full-length SAT practice test score and your highest subsequent full-length SAT practice test score within the app. If your score does not increase by at least 150 points, you are eligible for a refund.
          </p>
          <p style={typography.body}>
            5. <strong>Submit a Refund Request Within 60 Days</strong>: You must submit a refund request within 60 days of your subscription start date (or the date of your first full-length test, whichever is later). To request a refund, contact our support team at support@learnerlabs.com with your account details and a summary of your study activity.
          </p>
          <p style={typography.body}>
            6. <strong>Subscription Requirement</strong>: The guarantee applies only to users with an active paid subscription (monthly or annual) to SAT Smart Prep App. Free trial users or users in promotional programs (e.g., College Board pilot) are not eligible unless they upgrade to a paid plan.
          </p>
          <p style={typography.body}>
            <strong>Additional Notes</strong>:
            - <strong>Practice Test Scores Only</strong>: This guarantee applies only to SAT practice test scores within the app and does not apply to official SAT scores reported by the College Board.
            - <strong>Refund Processing</strong>: Refunds are processed within 14 business days of approval. The refund amount will be the full subscription fee paid, minus any applicable taxes or fees.
            - <strong>Fair Use</strong>: Learner Labs reserves the right to deny refund requests if there is evidence of misuse, such as not following the study plan, skipping questions, or attempting to manipulate scores.
          </p>
          <p style={typography.body}>
            <strong>How to Check Eligibility</strong>:
            Go to the Dashboard in the app and click "Check Score Guarantee" to see your score improvement and eligibility status. If eligible, follow the instructions to contact support for your refund.
          </p>

          <style jsx>{`
            .policy {
              max-width: 900px;
              margin: 50px auto;
              padding: 20px;
              text-align: center;
              background: ${colors.white};
              border-radius: 10px;
              box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
            }
          `}</style>
        </div>
      );
    }
    ```

***

#### Updated Internal Logic (Backend Implementation)

The backend logic in `frontend/backend/src/progress_monitoring.py` needs to be updated to check the starting score condition (1250 or lower). This involves retrieving the user’s previous SAT/PSAT score from the `users` table (entered during onboarding) or using the first full-length test score if no previous score is provided.

**Updated Backend Conditions and Logic**

1. **Starting Score of 1250 or Lower**:
   * **Logic**: The endpoint checks the `users` table for `sat_math_score` and `sat_reading_score` (entered during onboarding). If these are not provided, it uses the first full-length test score from the `practice_sessions` table. The starting score must be 1250 or lower to qualify.
   * **Implementation**:
     * Add a check for the starting score at the beginning of the `/progress_monitoring/guarantee/{user_id}` endpoint.
     * If the user has provided previous scores, calculate the total starting score as `sat_math_score + sat_reading_score`.
     * If no previous scores are provided, use the first full-length test score (`theta * 400 + 400`).
2. **Other Conditions**: The remaining conditions (two full-length tests, study plan completion, 150-point increase, 60-day window, subscription requirement) remain unchanged.

**Updated Backend Code**

Here’s the updated `/progress_monitoring/guarantee/{user_id}` endpoint with the new starting score condition:

```python
from fastapi import APIRouter, HTTPException
from psycopg2.extras import RealDictCursor
from datetime import datetime, timedelta
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

@router.get("/progress_monitoring/guarantee/{user_id}")
async def check_score_guarantee(user_id: str):
    conn = get_db_connection()
    cursor = conn.cursor(cursor_factory=RealDictCursor)
    try:
        # Check subscription status
        cursor.execute("SELECT subscription_status, sat_math_score, sat_reading_score FROM users WHERE user_id = %s", (user_id,))
        user = cursor.fetchone()
        if not user:
            raise HTTPException(status_code=404, detail="User not found")
        if user['subscription_status'] != 'active':
            return {"eligible": False, "message": "You must have an active paid subscription to be eligible for the guarantee."}

        # Check starting score (from previous SAT/PSAT or first full-length test)
        starting_score = None
        if user['sat_math_score'] is not None and user['sat_reading_score'] is not None:
            starting_score = user['sat_math_score'] + user['sat_reading_score']
        else:
            cursor.execute("""
                SELECT theta
                FROM practice_sessions
                WHERE user_id = %s AND test_type = 'SAT'
                ORDER BY timestamp
                LIMIT 1;
            """, (user_id,))
            first_session = cursor.fetchone()
            if first_session:
                starting_score = first_session['theta'] * 400 + 400
            else:
                return {"eligible": False, "message": "No starting score available. Complete at least one full-length test to determine eligibility."}

        if starting_score > 1250:
            return {"eligible": False, "message": "Your starting score is above 1250. The score improvement guarantee is only available for users starting at 1250 or lower."}

        # Check for at least two full-length tests
        cursor.execute("""
            SELECT theta, timestamp
            FROM practice_sessions
            WHERE user_id = %s AND test_type = 'SAT'
            ORDER BY timestamp;
        """, (user_id,))
        sessions = cursor.fetchall()
        if len(sessions) < 2:
            return {"eligible": False, "message": "Not enough test data to evaluate guarantee. Complete at least two full-length tests."}

        # Check if within 60 days of first test
        first_test_date = sessions[0]['timestamp']
        if (datetime.now() - first_test_date).days > 60:
            return {"eligible": False, "message": "Refund request period has expired. Requests must be submitted within 60 days of your first test."}

        # Check study plan completion rate
        cursor.execute("""
            SELECT COUNT(*) as total, COUNT(CASE WHEN completed IS NOT NULL THEN 1 END) as completed
            FROM study_plan_actions
            WHERE plan_id IN (SELECT plan_id FROM study_plans WHERE user_id = %s)
            AND due_date BETWEEN %s AND %s;
        """, (user_id, first_test_date, first_test_date + timedelta(days=30)))
        study_plan_stats = cursor.fetchone()
        completion_rate = study_plan_stats['completed'] / study_plan_stats['total'] if study_plan_stats['total'] > 0 else 0
        if completion_rate < 0.8:
            return {"eligible": False, "message": "You must complete at least 80% of your study plan tasks within 30 days of your first test."}

        # Check score improvement
        initial_score = sessions[0]['theta'] * 400 + 400
        latest_score = max(session['theta'] * 400 + 400 for session in sessions[1:])
        improvement = latest_score - initial_score
        if improvement < 150:
            return {
                "eligible": True,
                "improvement": improvement,
                "message": "You qualify for a refund! Contact support at support@learnerlabs.com."
            }
        return {
            "eligible": False,
            "improvement": improvement,
            "message": "Great job! You’ve improved by 150+ points."
        }
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
    finally:
        cursor.close()
        conn.close()
```

***

#### Re-Evaluation of Priya’s Eligibility

Let’s re-evaluate Priya’s eligibility for the Score Improvement Guarantee with the new starting score condition, using her user journey from the previous response.

* **Priya’s Profile**:
  * Scored 1350 on PSAT10 (entered during onboarding).
  * First full-length SAT practice test score: 1400 (Day 90).
  * Highest subsequent SAT practice test score: 1510 (Day 300).
  * Improvement: 1510 - 1400 = 110 points.
* **New Condition: Starting Score of 1250 or Lower**:
  * **Previous Score (PSAT10)**: 1350 (entered during onboarding).
  * **First Full-Length Test**: 1400 (used if no previous score is provided, but Priya provided her PSAT10 score).
  * **Evaluation**: Priya’s starting score is 1350 (from PSAT10), which is above 1250.
  * **Result**: Priya is **not eligible** for the guarantee due to her starting score being above 1250. The app displays: “Your starting score is above 1250. The score improvement guarantee is only available for users starting at 1250 or lower.”
* **Other Conditions** (for reference):
  * Two Full-Length Tests: Met (completed multiple full-length tests).
  * Study Plan for 30 Days: Met (completed 90% of tasks).
  * 150-Point Increase: Not Met (improvement: 110 points).
  * Within 60 Days: Not Met (\~210 days elapsed).
  * Active Subscription: Met.

**Updated Message in Priya’s Journey**

* **Original Message (Day 300)**: “You’re almost there! Keep going to meet the 150-point guarantee.”
* **Updated Message (Day 300)**: “Your starting score is above 1250. The score improvement guarantee is only available for users starting at 1250 or lower.” (Priya is informed early in her journey that she’s not eligible, so she focuses on her National Merit and SAT goals without expecting a refund.)

***

#### Example Application with a New User: Ethan

To further illustrate the updated guarantee, let’s consider a new user, Ethan, who starts with a lower score:

* **Ethan’s Profile**:
  * Scored 1100 on PSAT10 (Math: 540, Reading/Writing: 560, entered during onboarding).
  * First full-length SAT practice test score: 1080 (Day 1).
  * Highest subsequent SAT practice test score: 1200 (Day 45).
  * Improvement: 1200 - 1080 = 120 points.
* **Condition: Starting Score of 1250 or Lower**:
  * Starting Score (PSAT10): 1100 (below 1250). **Met**.
* **Other Conditions**:
  * Two Full-Length Tests: Met (completed 2 tests).
  * Study Plan for 30 Days: Met (completed 85% of tasks).
  * 150-Point Increase: Not Met (improvement: 120 points).
  * Within 60 Days: Met (Day 45 is within 60 days).
  * Active Subscription: Met.
* **Result**: Ethan is **eligible** for a refund because his starting score is 1100 (below 1250), and he meets all other conditions except the 150-point increase. The app displays: “You qualify for a refund! Contact support at support@learnerlabs.com.”

***

### Summary of Changes

* **New Condition**: The guarantee is now limited to users with a starting score of 1250 or lower, based on either their previous SAT/PSAT score (entered during onboarding) or their first full-length test score in the app.
* **Impact on Priya**: Priya, with a starting score of 1350, is no longer eligible for the guarantee, which aligns with the updated policy targeting users who need more significant improvement.
* **Impact on Ethan**: Ethan, with a starting score of 1100, qualifies for the guarantee and can claim a refund due to not meeting the 150-point improvement threshold.
* **Policy Update**: The user-facing policy clearly communicates the new starting score requirement, ensuring transparency and setting appropriate expectations.

This updated guarantee focuses the benefit on users who start with lower scores, making it more sustainable for **Learner Labs** while still providing a compelling incentive for the target audience. Let me know if you’d like to test this with more user scenarios or proceed with additional updates!
