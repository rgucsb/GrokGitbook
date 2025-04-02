# Comprehensive Analytics

To enhance the **SAT Prep Suite** with **Comprehensive Study Analytics**, I’ll implement the suggested improvements: a detailed dashboard with pacing, accuracy by domain/skill, and improvement trends (using charts), targeted drill suggestions based on weaknesses, and exportable reports for tutors and students. This builds on the current basic skill predictions (1-7 scale) and integrates with the existing React-based web app (and optionally mobile, though dashboard focus is web-centric) as of March 26, 2025. I’ll update the backend to provide detailed analytics data, modify the web frontend to display charts and suggestions, and add export functionality. Below, I’ll detail the enhancements, provide updated code, and outline a testing plan.

***

### Comprehensive Study Analytics Enhancements

#### 1. Detailed Dashboard

* **Goal**: Show pacing (time per question), accuracy by domain/skill, and improvement trends with charts.
* **Implementation**: Add backend endpoint, use `react-chartjs-2` for visualizations.

**Backend**

* Update `routes/review.py`:

```python
from sqlalchemy import func
from ..models import Response, Result, Question
from datetime import timedelta

@router.get("/analytics/{user_id}")
async def get_analytics(user_id: str, db: Session = Depends(get_db)):
    # Pacing: Average time per question
    responses = db.query(Response).filter(Response.user_id == user_id).all()
    pacing = {}
    for r in responses:
        q = db.query(Question).filter(Question.question_id == r.question_id).first()
        if q.skill not in pacing:
            pacing[q.skill] = []
        pacing[q.skill].append(r.time_spent)
    pacing = {skill: sum(times) / len(times) for skill, times in pacing.items() if times}

    # Accuracy by domain/skill
    accuracy = {}
    for r in responses:
        q = db.query(Question).filter(Question.question_id == r.question_id).first()
        domain, skill = q.domain, q.skill
        if domain not in accuracy:
            accuracy[domain] = {"total": 0, "correct": 0, "skills": {}}
        if skill not in accuracy[domain]["skills"]:
            accuracy[domain]["skills"][skill] = {"total": 0, "correct": 0}
        accuracy[domain]["total"] += 1
        accuracy[domain]["skills"][skill]["total"] += 1
        if r.answer == q.correct_answer:
            accuracy[domain]["correct"] += 1
            accuracy[domain]["skills"][skill]["correct"] += 1
    accuracy = {
        domain: {
            "overall": (data["correct"] / data["total"]) * 100,
            "skills": {skill: (s["correct"] / s["total"]) * 100 for skill, s in data["skills"].items()}
        } for domain, data in accuracy.items()
    }

    # Improvement trends (last 30 days)
    trends = []
    for i in range(30, -1, -1):
        day = datetime.utcnow() - timedelta(days=i)
        day_responses = db.query(Response).filter(
            Response.user_id == user_id,
            Response.timestamp >= day,
            Response.timestamp < day + timedelta(days=1)
        ).all()
        total = len(day_responses)
        correct = sum(1 for r in day_responses if db.query(Question).filter(Question.question_id == r.question_id).first().correct_answer == r.answer)
        trends.append({"date": day.strftime("%Y-%m-%d"), "accuracy": (correct / total * 100) if total > 0 else 0})

    return {"pacing": pacing, "accuracy": accuracy, "trends": trends}
```

**Web (`web/src/`)**

* Update `api.js`:

```javascript
async getAnalytics(userId) {
  return (await api.get(`/review/analytics/${userId}`)).data;
},
```

* Create `dashboard/analytics.js`:

```javascript
import { useEffect, useState } from 'react';
import { Typography, Box } from '@mui/material';
import { Line, Bar } from 'react-chartjs-2';
import { Chart as ChartJS, CategoryScale, LinearScale, PointElement, LineElement, BarElement, Title, Tooltip, Legend } from 'chart.js';
import api from '../shared-utils/api';

ChartJS.register(CategoryScale, LinearScale, PointElement, LineElement, BarElement, Title, Tooltip, Legend);

export default function AnalyticsDashboard({ userId }) {
  const [analytics, setAnalytics] = useState(null);

  useEffect(() => {
    api.getAnalytics(userId).then(setAnalytics);
  }, [userId]);

  if (!analytics) return <Typography>Loading...</Typography>;

  const pacingData = {
    labels: Object.keys(analytics.pacing),
    datasets: [{
      label: 'Time per Question (s)',
      data: Object.values(analytics.pacing),
      backgroundColor: 'rgba(75, 192, 192, 0.5)',
    }],
  };

  const accuracyData = {
    labels: Object.keys(analytics.accuracy),
    datasets: [{
      label: 'Overall Accuracy (%)',
      data: Object.values(analytics.accuracy).map(d => d.overall),
      backgroundColor: 'rgba(153, 102, 255, 0.5)',
    }],
  };

  const trendData = {
    labels: analytics.trends.map(t => t.date),
    datasets: [{
      label: 'Daily Accuracy (%)',
      data: analytics.trends.map(t => t.accuracy),
      borderColor: 'rgba(255, 99, 132, 1)',
      fill: false,
    }],
  };

  return (
    <Box sx={{ p: 4 }}>
      <Typography variant="h4">Study Analytics</Typography>
      <Box sx={{ mt: 2 }}>
        <Typography variant="h6">Pacing</Typography>
        <Bar data={pacingData} options={{ responsive: true }} />
      </Box>
      <Box sx={{ mt: 2 }}>
        <Typography variant="h6">Accuracy by Domain</Typography>
        <Bar data={accuracyData} options={{ responsive: true }} />
        {Object.entries(analytics.accuracy).map(([domain, data]) => (
          <Box key={domain} sx={{ mt: 1 }}>
            <Typography>{domain} Skills:</Typography>
            {Object.entries(data.skills).map(([skill, acc]) => (
              <Typography key={skill}>{skill}: {acc.toFixed(2)}%</Typography>
            ))}
          </Box>
        ))}
      </Box>
      <Box sx={{ mt: 2 }}>
        <Typography variant="h6">Improvement Trends</Typography>
        <Line data={trendData} options={{ responsive: true }} />
      </Box>
    </Box>
  );
}
```

* Update `dashboard.js`:

```javascript
import AnalyticsDashboard from './analytics';
// ...
<Box sx={{ mt: 4 }}>
  <AnalyticsDashboard userId={await api.getUserId()} />
</Box>
```

***

#### 2. Targeted Drills

* **Goal**: Suggest drills based on weaknesses (e.g., “Focus on Reading: Evidence-Based Analysis”).
* **Implementation**: Analyze accuracy, recommend skills.

**Backend**

* Add to `review.py`:

```python
@router.get("/drills/{user_id}")
async def get_targeted_drills(user_id: str, db: Session = Depends(get_db)):
    analytics = await get_analytics(user_id, db)
    drills = []
    for domain, data in analytics["accuracy"].items():
        for skill, accuracy in data["skills"].items():
            if accuracy < 70:  # Threshold for weakness
                drills.append(f"Focus on {domain}: {skill}")
    return {"drills": drills[:3]}  # Top 3 weaknesses
```

**Web**

* Update `api.js`:

```javascript
async getDrills(userId) {
  return (await api.get(`/review/drills/${userId}`)).data.drills;
},
```

* Update `dashboard/analytics.js`:

```javascript
const [drills, setDrills] = useState([]);
useEffect(() => {
  api.getAnalytics(userId).then(setAnalytics);
  api.getDrills(userId).then(setDrills);
}, [userId]);
// ...
<Box sx={{ mt: 2 }}>
  <Typography variant="h6">Targeted Drills</Typography>
  {drills.map((drill, idx) => (
    <Typography key={idx}>{drill}</Typography>
  ))}
</Box>
```

***

#### 3. Exportable Reports

* **Goal**: Generate PDF reports for tutors/students.
* **Implementation**: Use `jsPDF` for web export.

**Web**

* Update `package.json`:

```json
"dependencies": {
  // ... existing ...
  "jspdf": "^2.5.1"
}
```

* Update `dashboard/analytics.js`:

```javascript
import { jsPDF } from 'jspdf';
// ...
const exportReport = () => {
  const doc = new jsPDF();
  doc.text("SAT Prep Study Analytics", 10, 10);
  doc.text(`Pacing: ${JSON.stringify(analytics.pacing)}`, 10, 20);
  doc.text(`Accuracy: ${JSON.stringify(analytics.accuracy)}`, 10, 30);
  doc.text(`Trends: ${JSON.stringify(analytics.trends.map(t => `${t.date}: ${t.accuracy}%`))}`, 10, 40);
  doc.text(`Drills: ${drills.join(', ')}`, 10, 50);
  doc.save(`analytics_${userId}.pdf`);
};
// ...
<Button variant="contained" onClick={exportReport} sx={{ mt: 2 }}>Export Report</Button>
```

***

### Updated Files

* **Backend**: `routes/review.py`.
* **Web**: `api.js`, `dashboard.js`, new `dashboard/analytics.js`, `package.json`.

### Testing Plan

#### Setup

* Install dependency: `cd web && npm install`.
* Start backend: `cd api && uvicorn main:app --reload`.
* Web: `cd web && npm run dev`.

#### Test Cases

1. **Detailed Dashboard**:
   * Submit practice responses → Dashboard shows pacing (bar), accuracy (bar), trends (line).
2. **Targeted Drills**:
   * Low accuracy in “Reading: Evidence-Based Analysis” → Suggested drill appears.
3. **Exportable Reports**:
   * Click “Export Report” → PDF downloads with pacing, accuracy, trends, drills.

#### Expected Results

* Charts render correctly with data.
* Drills match weak skills (<70% accuracy).
* PDF contains all analytics data.

***

### Notes

* **Mobile**: Analytics not added (web-focused dashboard); extend if needed.
* **Chart Styling**: Basic—enhance with custom colors/options.
* **Data**: Assumes populated `responses` table; seed more data if empty.

### Conclusion

* **Dashboard**: Detailed with pacing, accuracy, trends via charts.
* **Drills**: Targeted suggestions based on weaknesses.
* **Reports**: Exportable PDFs for sharing.

Test it out—let me know if you want mobile analytics or more chart features!
