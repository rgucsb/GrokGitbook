# Step 8. Database Enhancements

#### 8. Database Enhancements

* **Current State**: Core tables are defined, but social, gamification, and AI tables are missing.
* **Missing**:
  * `tutor_interactions`, `tutor_actions`, `posts`, `comments`, `friends`, `badges`, `notifications`.
* **Why Needed**: Supports full feature set.

**Example Addition:**

```python
class TutorInteraction(Base):
    __tablename__ = "tutor_interactions"
    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(String, ForeignKey("users.user_id"))
    query = Column(String)
    response = Column(String)
    timestamp = Column(DateTime, default=func.now())
```
