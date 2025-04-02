# Step 6. Data Collection for AI Training

#### 6. Data Collection for AI Training

* **Current State**: `ai_data/interactions.json` is referenced but not populated.
* **Missing**:
  * **Backend**: Logic to log all user interactions (questions, responses, AI chats) into `ai_data/`.
  * **Format**: Structured JSON for training (e.g., `{ "user_id": "alex123", "query": "Solve x+2=5", "response": "x=3" }`).
* **Why Needed**: Essential for training a custom AI model.

**Example Addition:**

```python
import json
from pathlib import Path

def log_interaction(user_id: str, query: str, response: str):
    data = {"user_id": user_id, "query": query, "response": response, "timestamp": datetime.utcnow().isoformat()}
    file_path = Path("ai_data/interactions.json")
    if file_path.exists():
        with open(file_path, "r+") as f:
            interactions = json.load(f)
            interactions.append(data)
            f.seek(0)
            json.dump(interactions, f)
    else:
        with open(file_path, "w") as f:
            json.dump([data], f)
```
