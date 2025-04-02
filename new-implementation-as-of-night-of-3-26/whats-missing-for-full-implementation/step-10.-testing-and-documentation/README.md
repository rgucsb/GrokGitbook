# Step 10. Testing and Documentation

#### 10. Testing and Documentation

* **Current State**: No tests or detailed docs.
* **Missing**:
  * **Unit Tests**: `pytest` for backend, `Jest` for frontend.
  * **Integration Tests**: Full user flow testing.
  * **README**: Detailed setup, API docs, and usage.
* **Why Needed**: Ensures reliability and usability.

**Example Test: `tests/test_auth.py`**

```python
import pytest
from fastapi.testclient import TestClient
from backend.src.main import app

client = TestClient(app)

def test_signup():
    response = client.post("/auth/signup", json={"email": "test@example.com", "password": "pass", "study_hours": 10})
    assert response.status_code == 200
    assert "access_token" in response.json()
```
