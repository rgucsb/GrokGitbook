# Advanced AI Integrations

Let’s integrate the external AI models into the **SAT Prep Suite**, excluding the expensive Vision/OCR option (e.g., Google Cloud Vision) as requested. We’ll focus on **Large Language Models (LLMs)**, **Adaptive Learning Models**, **Predictive Analytics Models**, and **Speech Recognition/Synthesis Models**, leveraging their APIs to enhance personalization, adaptability, and accessibility. This builds on the existing backend (with gamification, analytics, tutor/parent integration, mobile app, content variety, and refined AI insights) and elevates the suite’s capabilities for thousands of students as of March 26, 2025.

***

#### Step 1: Integration Plan Recap

* **Models to Integrate**:
  1. **LLMs**: OpenAI GPT-4o (feedback, explanations).
  2. **Adaptive Learning**: Knewton (question selection).
  3. **Predictive Analytics**: IBM Watson (score prediction).
  4. **Speech Models**: Google Speech-to-Text, Amazon Polly (audio I/O).
* **Excluded**: Vision/OCR (cost concern).
* **Approach**: Use RESTful APIs, secure API keys via environment variables, and cache responses in Redis for efficiency.

***

#### Step 2: Backend Implementation

**Environment Setup**

*   Install dependencies:

    ```bash
    pip install requests redis boto3 scipy
    ```
*   `.env` file (securely stored):

    ```plaintext
    GPT_API_KEY=your_openai_key
    KNEWTON_API_KEY=your_knewton_key
    WATSON_API_KEY=your_watson_key
    GOOGLE_API_KEY=your_google_key
    AWS_ACCESS_KEY=your_aws_key
    AWS_SECRET_KEY=your_aws_secret
    REDIS_HOST=localhost
    REDIS_PORT=6379
    ```

**`api/utils.py` (Updated with External AI Integration)**

```python
import os
import json
import redis
import requests
import boto3
from typing import Dict, List
from sqlalchemy.orm import Session
from datetime import datetime
from scipy.stats import linregress
from api.models import Response, Result, StudyPlan, Lesson, Question

# Redis connection
redis_client = redis.Redis(host=os.getenv("REDIS_HOST"), port=int(os.getenv("REDIS_PORT")), decode_responses=True)

# API Keys
GPT_API_KEY = os.getenv("GPT_API_KEY")
KNEWTON_API_KEY = os.getenv("KNEWTON_API_KEY")
WATSON_API_KEY = os.getenv("WATSON_API_KEY")
GOOGLE_API_KEY = os.getenv("GOOGLE_API_KEY")
AWS_ACCESS_KEY = os.getenv("AWS_ACCESS_KEY")
AWS_SECRET_KEY = os.getenv("AWS_SECRET_KEY")

# AWS Polly client
polly_client = boto3.client("polly", aws_access_key_id=AWS_ACCESS_KEY, aws_secret_access_key=AWS_SECRET_KEY, region_name="us-east-1")

def analyze_response_patterns(db: Session, user_id: str) -> Dict:
    # Existing function unchanged (skill_history, response_patterns)
    # ... (see previous implementation)
    return {"skill_history": skill_history, "response_patterns": response_patterns}

def predict_skill_growth_with_watson(db: Session, user_id: str, skill_history: Dict) -> Dict:
    """Integrate Watson for enhanced skill growth prediction"""
    cache_key = f"watson_pred_{user_id}"
    cached = redis_client.get(cache_key)
    if cached:
        return json.loads(cached)
    
    flat_history = {}
    for section, domains in skill_history.items():
        for domain, skills in domains.items():
            for skill, history in skills.items():
                times, profs = zip(*history) if history else ([0], [3])
                days = [(t - times[0]).days for t in times]
                slope, _, r_value, _, _ = linregress(days, profs) if len(history) > 1 else (0, 0, 0.5, 0, 0)
                flat_history[f"{section}_{domain}_{skill}"] = {"slope": slope, "r_value": r_value, "latest_prof": profs[-1]}
    
    payload = {"features": flat_history, "target": "future_proficiency"}
    response = requests.post(
        "https://api.ibm.com/watson/predict",
        json=payload,
        headers={"Authorization": f"Bearer {WATSON_API_KEY}", "Content-Type": "application/json"}
    ).json()
    
    predictions = {"Math": {}, "Reading and Writing": {}}
    for key, pred in response["predictions"].items():
        section, domain, skill = key.split("_", 2)
        predictions[section][domain] = predictions[section].get(domain, {})
        predictions[section][domain][skill] = {
            "predicted_prof": min(7, max(1, pred["value"])),
            "confidence": pred["confidence"],
            "trend": "accelerating" if pred["value"] > flat_history[key]["latest_prof"] else "decelerating" if pred["value"] < flat_history[key]["latest_prof"] else "stable"
        }
    
    redis_client.setex(cache_key, 3600, json.dumps(predictions))  # Cache for 1 hour
    return predictions

def enhance_feedback_with_gpt(response_patterns: Dict, predictions: Dict) -> str:
    """Integrate GPT-4o for richer feedback"""
    cache_key = f"gpt_feedback_{hash(json.dumps(response_patterns))}_{hash(json.dumps(predictions))}"
    cached = redis_client.get(cache_key)
    if cached:
        return cached
    
    prompt = (
        "You’re a friendly SAT tutor. Analyze this student’s data: "
        f"Performance: {json.dumps(response_patterns)}. Predictions: {json.dumps(predictions)}. "
        "Write a 150-200 word conversational feedback message highlighting strengths, weaknesses, and actionable steps."
    )
    response = requests.post(
        "https://api.openai.com/v1/chat/completions",
        headers={"Authorization": f"Bearer {GPT_API_KEY}", "Content-Type": "application/json"},
        json={"model": "gpt-4o", "messages": [{"role": "user", "content": prompt}], "max_tokens": 250}
    ).json()
    
    feedback = response["choices"][0]["message"]["content"]
    redis_client.setex(cache_key, 86400, feedback)  # Cache for 24 hours
    return feedback

def select_questions_with_knewton(db: Session, user_id: str, section: str, domain: str, skill: str, num_questions: int) -> List[Dict]:
    """Integrate Knewton for adaptive question selection"""
    analysis = analyze_response_patterns(db, user_id)
    stats = analysis["response_patterns"][section].get(domain, {}).get(skill, {"correct": 0, "total": 0, "avg_time": []})
    payload = {
        "user_data": {
            "accuracy": stats["correct"] / stats["total"] if stats["total"] else 0,
            "avg_time": sum(stats["avg_time"]) / len(stats["avg_time"]) if stats["avg_time"] else 0,
            "attempts": stats["total"]
        },
        "section": section,
        "domain": domain,
        "skill": skill
    }
    response = requests.post(
        "https://api.knewton.com/v1/adapt",  # Hypothetical endpoint
        json=payload,
        headers={"Authorization": f"Bearer {KNEWTON_API_KEY}", "Content-Type": "application/json"}
    ).json()
    
    difficulty = response["recommended_difficulty"]  # e.g., "Medium"
    question_bank = load_question_bank()
    used_ids = get_used_questions(db, user_id)
    candidates = [q for q in question_bank if q["metadata"]["Test"] == section and 
                  q["metadata"]["Domain"] == domain and q["metadata"]["Skill"] == skill and 
                  q["metadata"]["Question ID"] not in used_ids and 
                  q["metadata"]["Difficulty"] == difficulty]
    
    selected = candidates[:num_questions]
    if len(selected) < num_questions:
        raise HTTPException(status_code=400, detail="Not enough questions")
    mark_questions_used(db, user_id, [q["metadata"]["Question ID"] for q in selected])
    return selected

def process_speech_input(audio_data: bytes) -> str:
    """Use Google Speech-to-Text for voice input"""
    response = requests.post(
        "https://speech.googleapis.com/v1/speech:recognize",
        headers={"Authorization": f"Bearer {GOOGLE_API_KEY}", "Content-Type": "application/json"},
        json={
            "config": {"encoding": "LINEAR16", "sampleRateHertz": 16000, "languageCode": "en-US"},
            "audio": {"content": audio_data.decode()}
        }
    ).json()
    return response["results"][0]["alternatives"][0]["transcript"] if "results" in response else ""

def generate_speech_explanation(text: str) -> bytes:
    """Use Amazon Polly for audio output"""
    cache_key = f"polly_{hash(text)}"
    cached = redis_client.get(cache_key)
    if cached:
        return bytes.fromhex(cached)
    
    response = polly_client.synthesize_speech(Text=text, OutputFormat="mp3", VoiceId="Joanna")
    audio = response["AudioStream"].read()
    redis_client.setex(cache_key, 86400, audio.hex())  # Cache for 24 hours
    return audio

def optimize_study_plan(db: Session, user_id: str, current_plan: StudyPlan) -> Dict:
    # Existing logic enhanced with Watson predictions
    analysis = analyze_response_patterns(db, user_id)
    predictions = predict_skill_growth_with_watson(db, user_id, analysis["skill_history"])
    response_patterns = analysis["response_patterns"]
    
    optimized_plan = current_plan.plan.copy()
    days_until_test = (current_plan.test_date - datetime.now()).days
    lagging_skills = []
    for section in ["Math", "Reading and Writing"]:
        for domain, skills in response_patterns[section].items():
            for skill, stats in skills.items():
                pred = predictions[section][domain][skill]
                if pred["predicted_prof"] < 4 or pred["trend"] == "decelerating":
                    lagging_skills.append((section, domain, skill))
    
    feedback = "Let’s tweak your plan based on your latest trends!" if lagging_skills else "You’re on a great path—minor adjustments ahead!"
    if lagging_skills:
        for phase in ["skill_building"]:
            for i, task in enumerate(optimized_plan[phase]):
                if task["type"] == "practice" and len(lagging_skills) > 0:
                    optimized_plan[phase][i] = {
                        "day": task["day"],
                        "type": "practice",
                        "section": lagging_skills[0][0],
                        "domain": lagging_skills[0][1],
                        "skill": lagging_skills[0][2],
                        "questions": 10
                    }
                    lagging_skills.pop(0)
    
    return {"optimized_plan": optimized_plan, "feedback": feedback}
```

**`api/routes/review.py` (Updated)**

```python
@router.get("/next/{user_id}")
async def get_next_steps(user_id: str, db: Session = Depends(get_db)):
    if not db.query(User).filter(User.user_id == user_id).first():
        raise HTTPException(status_code=404, detail="User not found")
    
    analysis = analyze_response_patterns(db, user_id)
    predictions = predict_skill_growth_with_watson(db, user_id, analysis["skill_history"])
    feedback = enhance_feedback_with_gpt(analysis["response_patterns"], predictions)
    return {"user_id": user_id, "skill_predictions": predictions, "custom_feedback": feedback}
```

**`api/routes/practice_module.py` (Updated)**

```python
@router.post("/start")
async def start_practice(request: PracticeRequest, db: Session = Depends(get_db)):
    practice_id = f"prac_{uuid.uuid4().hex[:8]}"
    questions = select_questions_with_knewton(db, request.user_id, request.section, request.domain, request.skill, request.questions)
    practice_db[practice_id] = {"user_id": request.user_id, "questions": questions, "responses": []}
    return {"practice_id": practice_id, "questions": questions, "time_limit": len(questions) * 3}

@router.post("/speech-input")
async def speech_input(user_id: str, file: UploadFile = File(...), db: Session = Depends(get_db)):
    audio_data = await file.read()
    text = process_speech_input(audio_data)
    feedback = enhance_feedback_with_gpt({"input": text}, {})  # Simple analysis for demo
    return {"user_id": user_id, "transcribed_text": text, "feedback": feedback}
```

**`api/routes/lessons.py` (Updated)**

```python
@router.get("/audio/{lesson_id}")
async def get_lesson_audio(lesson_id: int, db: Session = Depends(get_db)):
    lesson = db.query(Lesson).filter(Lesson.lesson_id == lesson_id).first()
    if not lesson:
        raise HTTPException(status_code=404, detail="Lesson not found")
    audio = generate_speech_explanation(lesson.content["text"])
    return StreamingResponse(io.BytesIO(audio), media_type="audio/mp3")
```

***

#### Step 3: Mobile App Updates (Flutter)

**`pubspec.yaml` (Updated)**

```yaml
dependencies:
  flutter:
    sdk: flutter
  http: ^0.13.5
  sqflite: ^2.0.0+4
  path_provider: ^2.0.11
  firebase_core: ^1.24.0
  firebase_messaging: ^13.0.4
  charts_flutter: ^0.12.0
  audio_session: ^0.1.10  # For audio playback
  record: ^4.4.0         # For recording audio
```

**`lib/services/api_service.dart` (Updated)**

```dart
Future<Map<String, dynamic>> getNextSteps(String userId) async {
  final response = await http.get(Uri.parse('$baseUrl/review/next/$userId'));
  return jsonDecode(response.body);
}

Future<Map<String, dynamic>> submitSpeechInput(String userId, Uint8List audioData) async {
  final response = await http.post(
    Uri.parse('$baseUrl/practice/speech-input'),
    body: jsonEncode({'user_id': userId, 'audio': base64Encode(audioData)}),
    headers: {'Content-Type': 'application/json'},
  );
  return jsonDecode(response.body);
}

Future<Uint8List> getLessonAudio(int lessonId) async {
  final response = await http.get(Uri.parse('$baseUrl/lessons/audio/$lessonId'));
  return response.bodyBytes;
}
```

**`lib/screens/practice.dart` (Updated with Speech Input)**

```dart
import 'package:record/record.dart';
import 'dart:typed_data';

class _PracticeScreenState extends State<PracticeScreen> {
  final Record _recorder = Record();
  bool _isRecording = false;

  // Existing code...

  Future<void> _recordAndSubmit() async {
    if (_isRecording) {
      final path = await _recorder.stop();
      final audioData = await File(path!).readAsBytes();
      final result = await api.submitSpeechInput('user123', audioData);
      ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text(result['feedback'])));
      _isRecording = false;
    } else {
      await _recorder.start();
      setState(() => _isRecording = true);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Practice')),
      body: Column(
        children: [
          Expanded(
            child: questions.isEmpty
                ? Center(child: CircularProgressIndicator())
                : ListView.builder(
                    itemCount: questions.length,
                    itemBuilder: (context, index) => QuestionCard(
                      question: questions[index],
                      onSubmit: (qId, answer, isCorrect, time) async {
                        _submitResponse(qId, answer, isCorrect, time);
                        var result = await api.submitPractice('user123', 'prac_<id>', responses);
                        if (index == questions.length - 1) {
                          _showFeedbackAndIntervention(result);
                        }
                      },
                    ),
                  ),
          ),
          ElevatedButton(
            onPressed: _recordAndSubmit,
            child: Text(_isRecording ? 'Stop Recording' : 'Ask via Voice'),
          ),
        ],
      ),
    );
  }
}
```

**`lib/screens/lessons.dart` (Updated with Audio)**

```dart
import 'package:audioplayers/audioplayers.dart';

class LessonsScreen extends StatefulWidget {
  @override
  _LessonsScreenState createState() => _LessonsScreenState();
}

class _LessonsScreenState extends State<LessonsScreen> {
  final ApiService api = ApiService();
  final AudioPlayer _player = AudioPlayer();

  Future<void> _playAudio(int lessonId) async {
    final audio = await api.getLessonAudio(lessonId);
    await _player.play(BytesSource(audio));
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Lessons')),
      body: FutureBuilder(
        future: api.getLessons(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) return Center(child: CircularProgressIndicator());
          var lessons = snapshot.data as List<Lesson>;
          return ListView.builder(
            itemCount: lessons.length,
            itemBuilder: (context, index) {
              var lesson = lessons[index];
              return ListTile(
                title: Text('${lesson.skill} (${lesson.duration} min)'),
                subtitle: Text(lesson.content['text'].substring(0, 50) + '...'),
                trailing: IconButton(
                  icon: Icon(Icons.volume_up),
                  onPressed: () => _playAudio(lesson.id),
                ),
                onTap: () {
                  showDialog(
                    context: context,
                    builder: (_) => AlertDialog(
                      title: Text(lesson.skill),
                      content: Column(
                        children: [
                          Text(lesson.content['text']),
                          if (lesson.content['video_url'] != null)
                            TextButton(
                              onPressed: () => launch(lesson.content['video_url']),
                              child: Text('Watch Video'),
                            ),
                        ],
                      ),
                      actions: [
                        TextButton(
                          onPressed: () async {
                            await api.completeLesson('user123', lesson.id);
                            Navigator.pop(context);
                          },
                          child: Text('Complete'),
                        ),
                      ],
                    ),
                  );
                },
              );
            },
          );
        },
      ),
    );
  }
}
```

**`lib/screens/dashboard.dart` (Updated)**

```dart
class DashboardScreen extends StatelessWidget {
  // Existing code...

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('SAT Prep')),
      body: FutureBuilder(
        future: Future.wait([api.getProgress('user123'), api.getNextSteps('user123')]),
        builder: (context, snapshot) {
          if (!snapshot.hasData) return Center(child: CircularProgressIndicator());
          var progress = snapshot.data![0] as Map<String, dynamic>;
          var nextSteps = snapshot.data![1] as Map<String, dynamic>;
          return ListView(
            children: [
              Text('Points: ${progress['gamification']?['points'] ?? 0}', style: TextStyle(fontSize: 20)),
              Text('Streak: ${progress['gamification']?['streak'] ?? 0} days'),
              Padding(
                padding: EdgeInsets.all(16),
                child: Text('Next Steps: ${nextSteps['custom_feedback']}', style: TextStyle(fontSize: 16)),
              ),
              ElevatedButton(
                onPressed: () async {
                  var optimized = await api.optimizeStudyPlan('user123');
                  ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text(optimized['feedback'])));
                },
                child: Text('Optimize Study Plan'),
              ),
              // Other buttons unchanged
            ],
          );
        },
      ),
    );
  }
}
```

***

#### Step 4: Testing

* **GPT-4o Feedback**: Submit 5 responses → Check `/review/next` for detailed 150-200 word feedback.
* **Knewton Question Selection**: Start practice with low accuracy → Verify easier questions → High accuracy → Verify harder questions.
* **Watson Prediction**: Analyze response history → Confirm `/review/next` includes refined predictions (e.g., “predicted\_prof”: 5.3, “confidence”: 0.88).
* **Speech I/O**: Record “How do I solve quadratics?” → Verify transcription and GPT feedback → Play lesson audio → Confirm playback.

***

#### Step 5: Sample Output

**`/review/next/user123`**

```json
{
  "user_id": "user123",
  "skill_predictions": {
    "Math": {
      "Algebra": {"predicted_prof": 5.3, "confidence": 0.88, "trend": "accelerating"}
    }
  },
  "custom_feedback": "Hey! You’re making awesome progress in Algebra—your predicted proficiency is 5.3 and climbing fast! You’re acing linear equations (80% accuracy), but quadratics trip you up sometimes. Try focusing on factoring techniques next—they’ll unlock higher scores. Your pacing is solid at 45s per question, so keep that rhythm. Let’s push for a 6 soon!"
}
```

**`/practice/start`**

```json
{
  "practice_id": "prac_xyz789",
  "questions": [{"metadata": {"Question ID": "q5", "Difficulty": "Medium", "Skill": "Linear Functions"}}],
  "time_limit": 15
}
```

**`/practice/speech-input`**

```json
{
  "user_id": "user123",
  "transcribed_text": "How do I solve quadratics?",
  "feedback": "Great question! To solve quadratics, start with factoring or the quadratic formula: x = [-b ± √(b²-4ac)]/(2a). For example, in x² + 5x + 6 = 0, factor to (x+2)(x+3) = 0, so x = -2 or -3. Practice a few to get comfy!"
}
```

**`/lessons/audio/1`**

* Returns MP3: “Here’s how to solve linear functions: isolate x by…”

***

#### Step 6: Cost Management

* **GPT-4o**: \~$0.005/1K tokens → \~$0.01 per feedback (cached for 24h).
* **Knewton**: Licensing TBD; assume $0.02 per adaptation.
* **Watson**: \~$0.03 per prediction (cached for 1h).
* **Speech APIs**: Google \~$0.006/min, Polly \~$0.004/min → Minimal cost for occasional use.
* **Mitigation**: Cache responses, limit free-tier API calls, offer premium subscription.

***

#### Conclusion

This integration adds:

* **GPT-4o**: Rich, conversational feedback and speech input responses.
* **Knewton**: Smarter, adaptive question selection.
* **Watson**: Precise skill growth predictions.
* **Speech Models**: Voice-based interaction and audio lessons.

Excluding Vision/OCR keeps costs manageable while delivering a highly intelligent, accessible SAT Prep Suite as of March 26, 2025. Next steps could include A/B testing these features or expanding to other tests (e.g., ACT)—let me know your priority!
