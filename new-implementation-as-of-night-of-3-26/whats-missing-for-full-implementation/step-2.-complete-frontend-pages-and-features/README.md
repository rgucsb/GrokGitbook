# Step 2. Complete Frontend Pages and Features

#### 2. Complete Frontend Pages and Features

* **Current State**: Basic pages (`login.js`, `diagnostic.js`, `study-plan.js`) are provided, but others are incomplete or missing.
* **Missing/Incomplete**:
  * **`practice.js`**: Needs full UI for practice sessions (question display, submission).
  * **`full-test.js`**: Missing UI for full-length test execution.
  * **`dashboard.js`**: Missing detailed analytics visualization (charts, trends).
  * **`review.js`**: Missing review display logic.
  * **`tutor-parent.js`**: Missing tutor/parent dashboard.
  * **`community.js`**: Missing social features (forums, friend lists).
  * **`leaderboard.js`**: Missing gamification UI.
  * **Offline Support**: No caching logic in `api.js`.
  * **Themes/Animations**: No `theme.js` or `framer-motion` integration.
* **Why Needed**: A complete frontend is essential for user interaction across all features.

**Example Addition: `pages/practice.js`**

```javascript
import { useState, useEffect } from 'react';
import { api } from '../utils/api';

export default function Practice() {
  const [questions, setQuestions] = useState([]);
  const userId = 'alex123'; // Replace with auth context

  useEffect(() => {
    const fetchPractice = async () => {
      const res = await api.post(`/practice/start/${userId}`, { domain: 'Math' });
      setQuestions(res.data.questions);
    };
    fetchPractice();
  }, []);

  return (
    <div>
      <h1>Practice</h1>
      {questions.map((q) => (
        <div key={q.question_id}>
          <p>{q.text}</p>
          <button>Submit Answer</button>
        </div>
      ))}
    </div>
  );
}
```
