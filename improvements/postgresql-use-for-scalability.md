# PostgreSQL use for scalability

To enhance the **SAT Prep Suite** with a **Scalable Backend**, I’ll implement the suggested improvements: migrating from SQLite to **PostgreSQL** for better scalability and concurrent user support, and using **Docker** for containerized deployment with a provided `Dockerfile`. This addresses the current single-server SQLite setup, enabling the app to handle growth and rival LearnQ.ai’s institutional scalability as of March 26, 2025. I’ll update the backend configuration, provide the Dockerfile, adjust database connections, and outline a deployment/testing plan.

***

### Scalable Backend Enhancements

#### 1. Migrate to PostgreSQL

* **Goal**: Replace SQLite with PostgreSQL for better concurrency, scalability, and robustness.
* **Approach**: Update SQLAlchemy setup, adjust migrations, and ensure compatibility.

**Backend Updates**

* Update `requirements.txt`:

```
fastapi
uvicorn
sqlalchemy
psycopg2-binary  # PostgreSQL driver
python-jose[cryptography]
passlib[bcrypt]
redis
```

* Update `api/database.py`:

```python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os

DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://user:password@localhost:5432/satprep")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

* Update `migrations/init_db.py`:

```python
from api.database import Base, engine
from api.models import User, Question, PracticeSession, Response, Result, Badge, Proficiency

Base.metadata.drop_all(bind=engine)  # Reset DB
Base.metadata.create_all(bind=engine)
```

* **Notes**:
  * SQLite-specific features (e.g., `func.random()`) need adjustment:
    * Replace `func.random()` with `func.random()` (PostgreSQL-compatible via `RANDOM()`).
  * Update `seed_data.py` to run with PostgreSQL (no schema changes needed, just connection).

**Configuration**

* Create `.env` (not tracked in git):

```
DATABASE_URL=postgresql://user:password@localhost:5432/satprep
REDIS_URL=redis://localhost:6379/0
JWT_SECRET=your-secret-key
```

* Update `main.py` to load `.env`:

```python
from dotenv import load_dotenv
import os

load_dotenv()
# ... rest unchanged ...
```

***

#### 2. Dockerized Deployment

* **Goal**: Containerize the app for consistent, scalable deployment.
* **Approach**: Add a `Dockerfile` and `docker-compose.yml` for local development with PostgreSQL and Redis.

**Dockerfile**

* Create `api/Dockerfile`:

```dockerfile
FROM python:3.9

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**Docker Compose**

* Create `api/docker-compose.yml`:

```yaml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:password@db:5432/satprep
      - REDIS_URL=redis://redis:6379/0
      - JWT_SECRET=your-secret-key
    depends_on:
      - db
      - redis
    volumes:
      - .:/app

  db:
    image: postgres:13
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=satprep
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:6
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

**Adjustments**

* Update `.gitignore`:

```
.env
postgres_data/
redis_data/
```

***

#### 3. Ensure Scalability

* **Concurrency**: PostgreSQL handles multiple connections better than SQLite’s file-based locking.
* **Load Balancing**: Docker setup allows scaling with orchestration (e.g., Kubernetes) later.
* **Caching**: Redis remains for AI feedback/chat sessions, reducing DB load.

**Example Scaling (Future)**

* Add `nginx` as a reverse proxy in `docker-compose.yml` for load balancing multiple `app` instances.
* Deploy to AWS ECS or Kubernetes for horizontal scaling.

***

### Updated Files

* **Backend**: `requirements.txt`, `database.py`, `migrations/init_db.py`, `main.py`, `.env` (example), `.gitignore`.
* **New**: `Dockerfile`, `docker-compose.yml`.

### Deployment and Testing Plan

#### Setup

1. **Install Docker**: Ensure Docker and Docker Compose are installed (`docker --version`, `docker-compose --version`).
2.  **Build and Run**:

    ```bash
    cd api
    docker-compose up --build
    ```
3. **Initialize DB**:
   *   Open a terminal in the `app` container:

       ```bash
       docker-compose exec app bash
       python migrations/init_db.py
       python seed_data.py
       ```

#### Test Cases

1. **Database Migration**:
   * Run app → Tables (`users`, `questions`, etc.) created in PostgreSQL.
   * Seed data → Questions load without errors.
2. **Concurrent Users**:
   * Simulate 10 users submitting practice responses simultaneously (e.g., via `curl` or Postman) → No lock errors, all responses saved.
3. **Docker Deployment**:
   * Start with `docker-compose up` → App accessible at `http://localhost:8000`.
   * Stop (`docker-compose down`) and restart → Data persists in `postgres_data`.
4. **API Functionality**:
   * Login, start practice, submit response → Works as with SQLite.

#### Expected Results

* PostgreSQL handles concurrent writes (unlike SQLite’s single-write limitation).
* Docker containers start cleanly, app connects to DB/Redis.
* No data loss on restart with volume mounts.

***

### Notes

* **Performance**: PostgreSQL excels with >100 users; SQLite struggles beyond 10 concurrent writes.
* **Security**: Update `.env` with strong credentials for production.
* **Production**: Add `gunicorn` (`CMD ["gunicorn", "-w", "4", "-k", "uvicorn.workers.UvicornWorker", "main:app"]`) for multi-worker setup.

### Conclusion

* **PostgreSQL**: Scalable, concurrent-ready DB replaces SQLite.
* **Docker**: Containerized deployment simplifies scaling and consistency.
* **Impact**: Matches institutional-grade backends (e.g., LearnQ.ai).

Run `docker-compose up` and test—let me know if you want load balancing or cloud deployment next!
