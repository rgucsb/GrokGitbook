# Step 3.  Social and Community Features

#### 3. Social and Community Features

* **Current State**: Absent from both backend and frontend.
* **Missing**:
  * **Backend**: `social.py` with endpoints for posts, comments, friend requests.
  * **Frontend**: `community.js` for forum UI, friend management.
  * **Database**: Tables like `posts`, `comments`, `friends`.
* **Why Needed**: Social features enhance engagement and community learning.

**Example Addition: `src/social.py`**

```python
from fastapi import APIRouter, Depends
from sqlalchemy.orm import Session
from .database import get_db

router = APIRouter(prefix="/community", tags=["community"])

class PostCreate:
    content: str

@router.post("/posts/{user_id}")
async def create_post(user_id: str, post: PostCreate, db: Session = Depends(get_db)):
    db.execute("INSERT INTO posts (user_id, content) VALUES (:user_id, :content)", {"user_id": user_id, "content": post.content})
    db.commit()
    return {"message": "Post created"}
```

**Database Addition:**

```python
class Post(Base):
    __tablename__ = "posts"
    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(String, ForeignKey("users.user_id"))
    content = Column(String)
    timestamp = Column(DateTime, default=func.now())
```
