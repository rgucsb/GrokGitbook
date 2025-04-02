# Step 4. Engagement and Gamification

#### 4. Engagement and Gamification

* **Current State**: `gamification.py` is referenced but not fully implemented.
* **Missing**:
  * **Backend**: Full badge system, streak tracking.
  * **Frontend**: `leaderboard.js`, badge display in `dashboard.js`.
  * **Database**: `badges` table.
* **Why Needed**: Drives user motivation and retention.

**Database Addition:**

```python
class Badge(Base):
    __tablename__ = "badges"
    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(String, ForeignKey("users.user_id"))
    name = Column(String)
    awarded_at = Column(DateTime, default=func.now())
```
