# Step 7. Offline Mode

#### 7. Offline Mode

* **Current State**: `sync.py` is missing.
* **Missing**:
  * **Backend**: `/sync` endpoint for data synchronization.
  * **Frontend**: Caching in `api.js` using localStorage or IndexedDB.
* **Why Needed**: Ensures usability without internet.

**Example Addition: `src/sync.py`**

```python
from fastapi import APIRouter, Depends
from sqlalchemy.orm import Session
from .database import get_db, Response

router = APIRouter(prefix="/sync", tags=["sync"])

@router.post("/sync/{user_id}")
async def sync_data(user_id: str, offline_data: list, db: Session = Depends(get_db)):
    for item in offline_data:
        db.add(Response(user_id=user_id, question_id=item["question_id"], answer=item["answer"], time_spent=item["time_spent"]))
    db.commit()
    return {"message": "Data synced"}
```
