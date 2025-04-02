# Item 9. Configuration and Scalability

#### 9. Configuration and Scalability

* **Current State**: Basic `Dockerfile` and `docker-compose.yml`, no Redis or load balancing.
* **Missing**:
  * **Redis**: For caching chat and streaks.
  * **Nginx**: Load balancing for scalability.
  * **Environment**: Full `.env` setup (e.g., Redis URL, AI API keys).
* **Why Needed**: Ensures performance and scalability.

**Updated `docker-compose.yml`:**

```yaml
version: '3.8'
services:
  app:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:password@db:5432/satprep
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis
  db:
    image: postgres:14
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=satprep
    ports:
      - "5432:5432"
  redis:
    image: redis:6
    ports:
      - "6379:6379"
```
