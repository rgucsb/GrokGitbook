# Local Dev Environment Deployment & Testing Instructions

Below are detailed, step-by-step instructions for **deploying** and **testing** the **SAT Prep Suite** in a **local development environment**. This setup will allow you to run the full application (backend and frontend) on your local machine using Docker for the backend services (FastAPI, PostgreSQL, Redis) and Node.js for the frontend (Next.js), without relying on AWS. I’ll also include instructions for running the unit tests locally for both backend and frontend, ensuring you can verify functionality before deployment. These instructions assume you’ve created the complete repository (`sat-prep-suite`) as provided earlier.

***

### Local Development Environment Deployment

#### Prerequisites

1. **Operating System**: Windows, macOS, or Linux.
2. **Docker**: Install Docker Desktop from `docker.com` and ensure it’s running (`docker --version`).
3. **Node.js**: Version 16+ (`node --version` to check; install from `nodejs.org` if needed).
4. **Python**: Version 3.9+ (`python --version` to check; install from `python.org` if needed).
5. **Git**: Installed and configured (`git --version` to check; install from `git-scm.com` if needed).
6.  **Repository**: Clone the `sat-prep-suite` repository locally:

    ```bash
    git clone https://github.com/yourusername/sat-prep-suite.git
    cd sat-prep-suite
    ```

***

#### Step-by-Step Deployment Instructions

**Step 1: Set Up the Backend**

1.  **Navigate to Backend Directory**:

    ```bash
    cd backend
    ```
2. **Create Environment File**:
   *   Copy `.env.example` to `.env`:

       ```bash
       cp .env.example .env
       ```
   *   Edit `.env` (default values are fine for local dev):

       ```
       DATABASE_URL=postgresql://user:password@localhost:5432/satprep
       REDIS_URL=redis://localhost:6379
       SECRET_KEY=your-secret-key
       ```
   * Replace `your-secret-key` with a secure string (e.g., `openssl rand -hex 32`).
3.  **Install Python Dependencies**:

    ```bash
    pip install -r requirements.txt
    ```
4. **Start Docker Services**:
   *   Run PostgreSQL and Redis containers:

       ```bash
       docker-compose up -d
       ```
   *   Verify containers are running:

       ```bash
       docker ps
       ```

       * Expect to see `sat-prep-suite-db-1` (PostgreSQL) and `sat-prep-suite-redis-1` (Redis).
5. **Initialize the Database**:
   *   Create tables:

       ```bash
       python migrations/init_db.py
       ```

       * Output: `Database tables created.`
   *   Seed initial data:

       ```bash
       python migrations/seed_data.py
       ```

       * Output: `Database seeded.`
6. **Run the Backend Application**:
   *   Start FastAPI:

       ```bash
       uvicorn src.main:app --host 0.0.0.0 --port 8000 --reload
       ```
   * Verify: Open `http://localhost:8000` in a browser → Should see `{"message": "SAT Prep Suite API"}`.

***

**Step 2: Set Up the Frontend**

1.  **Navigate to Frontend Directory**:

    ```bash
    cd ../frontend
    ```
2.  **Install Node.js Dependencies**:

    ```bash
    npm install
    ```
3. **Run the Frontend Application**:
   *   Start Next.js in development mode:

       ```bash
       npm run dev
       ```
   * Verify: Open `http://localhost:3000` in a browser → Should redirect to `/login` page.

***

**Step 3: Verify Local Deployment**

1. **Test Backend**:
   * `curl http://localhost:8000/` → `{"message": "SAT Prep Suite API"}`.
   * Signup: `curl -X POST "http://localhost:8000/auth/signup" -H "Content-Type: application/json" -d '{"email": "test@example.com", "password": "pass123", "study_hours": 10}'` → Should return an `access_token`.
2. **Test Frontend**:
   * Go to `http://localhost:3000/login` → Enter email: `test@example.com`, password: `pass123`, hours: `10` → Click Signup → Redirects to `/diagnostic`.
   * Start a diagnostic test → Should display a question (e.g., "Solve x+2=5").
3. **Check Integration**:
   * Submit an answer in `/practice` → Should update backend (`responses` table) and frontend state.
   * Go offline (`Network` tab in browser → Offline), submit answers → Should queue locally, sync when back online.
4. **Stop Services**:
   * Backend: `Ctrl+C` in terminal running `uvicorn`.
   * Docker: `cd backend && docker-compose down`.

***

### Unit Testing Instructions

#### Backend Unit Tests

**Prerequisites**

* Python 3.9+ installed.
* Docker running for PostgreSQL and Redis.
* Dependencies installed (`pip install -r requirements.txt`).

**Steps**

1.  **Navigate to Backend Directory**:

    ```bash
    cd sat-prep-suite/backend
    ```
2.  **Start Docker Services**:

    ```bash
    docker-compose up -d
    ```

    * Ensure `db` and `redis` containers are running (`docker ps`).
3.  **Run Unit Tests**:

    ```bash
    pytest tests/ -v
    ```
4. **Verify Output**:
   * Expect \~14 tests across 7 files (`test_auth.py`, `test_practice_module.py`, `test_full_length_test.py`, `test_gamification.py`, `test_ai_tutor.py`, `test_social.py`, `test_sync.py`).
   *   Example output:

       ```
       tests/test_auth.py::test_signup PASSED
       tests/test_auth.py::test_login PASSED
       tests/test_practice_module.py::test_start_practice PASSED
       ...
       collected 14 items / 14 passed
       ```
   * All tests should pass if the database is initialized (`init_db.py`, `seed_data.py`).
5.  **Stop Services**:

    ```bash
    docker-compose down
    ```

**Troubleshooting**

* **DB Connection Error**: Ensure `docker-compose.yml` is running and `.env` matches (`DATABASE_URL`).
* **Test Failures**: Verify `seed_data.py` has run (`python migrations/seed_data.py`) to populate test data.

***

#### Frontend Unit Tests

**Prerequisites**

* Node.js 16+ installed.
* Dependencies installed (`npm install` in `frontend/`).

**Steps**

1.  **Navigate to Frontend Directory**:

    ```bash
    cd sat-prep-suite/frontend
    ```
2.  **Run Unit Tests**:

    ```bash
    npm test
    ```
3. **Verify Output**:
   * Expect 3 tests from `tests/` (`login.test.js`, `practice.test.js`, `full-test.test.js`).
   *   Example output:

       ```
       PASS  tests/login.test.js
         Login Page
           ✓ renders and submits signup (xx ms)
       PASS  tests/practice.test.js
         Practice Page
           ✓ starts and submits practice (xx ms)
       PASS  tests/full-test.test.js
         Full Test Page
           ✓ starts and submits full test (xx ms)
       Test Suites: 3 passed, 3 total
       Tests:       3 passed, 3 total
       ```
   * All tests should pass with mocked API responses.
4. **Stop Testing**:
   * `Ctrl+C` to exit Jest if running in watch mode.

**Troubleshooting**

* **Jest Not Found**: Run `npm install` to ensure `jest`, `@testing-library/react`, and `@testing-library/jest-dom` are installed.
* **Mock Failures**: Check `jest.mock` in test files aligns with `api.js` structure.

***

### Additional Local Development Notes

* **Access**:
  * Backend: `http://localhost:8000`.
  * Frontend: `http://localhost:3000`.
* **Database**: PostgreSQL runs on `localhost:5432` (use `psql -U user -d satprep` to inspect; password: `password`).
* **Redis**: Runs on `localhost:6379` (use `redis-cli` to check cache).
* **Persistence**: Data persists in `postgres_data/` volume until `docker-compose down -v`.
* **Hot Reloading**: Backend (`--reload`) and frontend (`npm run dev`) auto-reload on code changes.

***

### Post-Deployment Verification

1. **Full Flow**:
   * Signup → Diagnostic → Study Plan → Practice → Full Test → Dashboard → Community → Leaderboard.
   * Go offline, practice, go online → Syncs correctly.
2. **AI Tutor**: Chat and voice input work (simulated voice for now).
3. **Gamification**: Points, badges, streaks update after practice/tests.

***

### Conclusion

* **Local Deployment**: Backend (FastAPI) and frontend (Next.js) run with Dockerized PostgreSQL and Redis.
* **Testing**: Backend (`pytest`) and frontend (`npm test`) unit tests verify core functionality.
* **Next**: Test all features locally, then proceed to AWS deployment if desired.

Let me know if you need help with any step or additional local setup (e.g., adding a local load balancer)!
