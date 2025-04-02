# AWS Deployment & Testing Instructions

Below are detailed, step-by-step instructions for **deploying** the **SAT Prep Suite** to AWS ECS (Elastic Container Service) using the provided repository, followed by instructions for running **unit tests** for both the backend and frontend. These steps assume you’ve uploaded the complete repository (`sat-prep-suite`) to GitHub as outlined previously and are starting with a fresh AWS setup. I’ll include prerequisites, AWS configuration, GitHub Actions setup, and local testing commands to ensure a smooth deployment and verification process.

***

### Deployment to AWS ECS

#### Prerequisites

1. **AWS Account**: Sign up at `aws.amazon.com` if you don’t have one.
2. **AWS CLI**: Install via `pip install awscli` or download from AWS, then configure with `aws configure` (requires Access Key ID, Secret Access Key, region: `us-east-1`).
3. **Docker**: Install Docker Desktop (`docker.com`) and ensure it’s running.
4. **Git**: Installed and configured (`git --version` to check).
5. **Node.js**: Version 16+ (`node --version` to check).
6. **Python**: Version 3.9+ (`python --version` to check).
7. **GitHub Account**: Repository `sat-prep-suite` pushed to `https://github.com/yourusername/sat-prep-suite`.

***

#### Step-by-Step Deployment Instructions

**Step 1: Set Up AWS Resources**

1. **Create an ECR Repository for Backend**:
   * Log in to AWS Console → Navigate to **Elastic Container Registry (ECR)**.
   * Click **Create repository** → Name: `sat-prep-suite-backend` → Private → Create.
   * Note the URI (e.g., `AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/sat-prep-suite-backend`).
2. **Create an ECR Repository for Frontend**:
   * Repeat the above → Name: `sat-prep-suite-frontend` → Private → Create.
   * Note the URI (e.g., `AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/sat-prep-suite-frontend`).
3. **Set Up RDS (PostgreSQL)**:
   * Go to **RDS** → **Create database**.
   * Engine: PostgreSQL → Version: 14.x.
   * Template: Free Tier → DB instance identifier: `satprep-db`.
   * Master username: `user` → Master password: `password`.
   * Instance size: `db.t3.micro` → Storage: 20 GB → Publicly accessible: Yes.
   * VPC: Default → Create database.
   * After creation, note the endpoint (e.g., `satprep-db.xxx.us-east-1.rds.amazonaws.com`).
4. **Set Up ElastiCache (Redis)**:
   * Go to **ElastiCache** → **Redis** → **Create cluster**.
   * Cluster mode: Disabled → Name: `satprep-redis`.
   * Node type: `cache.t3.micro` → Number of replicas: 0.
   * VPC: Default → Subnet group: Default → Create.
   * Note the primary endpoint (e.g., `satprep-redis.xxx.cache.amazonaws.com:6379`).
5. **Create an ECS Cluster**:
   * Go to **ECS** → **Clusters** → **Create cluster**.
   * Name: `sat-prep-cluster` → Infrastructure: AWS Fargate → Create.
   * Wait for the cluster to be active.

***

**Step 2: Define ECS Task Definitions**

1. **Backend Task Definition**:
   * Go to **ECS** → **Task Definitions** → **Create new task definition**.
   * Name: `sat-prep-backend-task` → Infrastructure: Fargate.
   * Operating system: Linux/x86\_64 → CPU: 1 vCPU → Memory: 2 GB.
   * Container:
     * Name: `backend` → Image: `<AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/sat-prep-suite-backend:latest`.
     * Port mappings: 8000 TCP.
     * Environment variables:
       * `DATABASE_URL`: `postgresql://user:password@satprep-db.xxx.us-east-1.rds.amazonaws.com:5432/satprep`.
       * `REDIS_URL`: `redis://satprep-redis.xxx.cache.amazonaws.com:6379`.
       * `SECRET_KEY`: `your-secret-key`.
   * Save and create.
2. **Frontend Task Definition**:
   * Repeat → Name: `sat-prep-frontend-task` → CPU: 0.5 vCPU → Memory: 1 GB.
   * Container:
     * Name: `frontend` → Image: `<AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/sat-prep-suite-frontend:latest`.
     * Port mappings: 3000 TCP.
   * Save and create.

***

**Step 3: Configure ECS Service**

1. **Create Backend Service**:
   * Go to **ECS** → **Clusters** → `sat-prep-cluster` → **Services** → **Create**.
   * Launch type: Fargate → Task definition: `sat-prep-backend-task`.
   * Service name: `sat-prep-backend-service` → Number of tasks: 1.
   * VPC: Default → Subnets: Select all → Security group: Create new (allow inbound 8000 TCP).
   * Load balancer: None (for simplicity; add ALB later if needed) → Create.
2. **Create Frontend Service**:
   * Repeat → Task definition: `sat-prep-frontend-task`.
   * Service name: `sat-prep-frontend-service` → Number of tasks: 1.
   * Security group: Allow inbound 3000 TCP → Create.

***

**Step 4: Set Up GitHub Secrets**

1. **Generate AWS Credentials**:
   * AWS Console → IAM → Users → Add user → Programmatic access → Attach `AmazonECS_FullAccess`, `AmazonEC2ContainerRegistryFullAccess` → Create → Note Access Key ID, Secret Access Key.
2. **Add Secrets to GitHub**:
   * Go to `https://github.com/yourusername/sat-prep-suite` → Settings → Secrets and Variables → Actions → New repository secret.
   * Add:
     * `AWS_ACCESS_KEY_ID`: `<your-access-key-id>`
     * `AWS_SECRET_ACCESS_KEY`: `<your-secret-access-key>`
     * `AWS_ACCOUNT_ID`: `<your-aws-account-id>` (12-digit number from AWS Console).

***

**Step 5: Push to GitHub and Trigger Deployment**

1.  **Local Setup**:

    ```bash
    mkdir sat-prep-suite
    cd sat-prep-suite
    git init
    ```
2. **Copy Files**:
   * Copy all backend and frontend files from previous responses into their respective directories (`backend/`, `frontend/`, root files).
3.  **Commit and Push**:

    ```bash
    git add .
    git commit -m "Initial deployment of SAT Prep Suite"
    git remote add origin https://github.com/yourusername/sat-prep-suite.git
    git branch -M main
    git push -u origin main
    ```
4. **Verify Deployment**:
   * Go to GitHub → Actions → Check the `Deploy to AWS ECS` workflow.
   * Ensure it completes successfully (builds and pushes Docker images to ECR, deploys to ECS).

***

**Step 6: Verify Deployment**

1. **Get Public IPs**:
   * ECS → `sat-prep-cluster` → Tasks → Note the public IPs of running tasks (e.g., `backend: 54.123.45.67`, `frontend: 54.123.45.68`).
2. **Test Backend**:
   * `curl http://54.123.45.67:8000/` → Should return `{"message": "SAT Prep Suite API"}`.
3. **Test Frontend**:
   * Open `http://54.123.45.68:3000` in a browser → Should redirect to login page.
4. **Initialize DB**:
   * Locally: `cd backend && docker-compose up -d db && python migrations/init_db.py && python migrations/seed_data.py`.
   * Update RDS with seeded data if needed (e.g., via `pgAdmin`).

***

### Unit Testing Instructions

#### Backend Unit Tests

**Prerequisites**

* Python 3.9+ installed.
* Docker running for PostgreSQL and Redis.

**Steps**

1.  **Navigate to Backend**:

    ```bash
    cd sat-prep-suite/backend
    ```
2.  **Install Dependencies**:

    ```bash
    pip install -r requirements.txt
    ```
3.  **Start Docker Services**:

    ```bash
    docker-compose up -d
    ```
4.  **Run Tests**:

    ```bash
    pytest tests/ -v
    ```
5. **Verify Output**:
   * Expect \~14 tests (2 per file: `test_auth.py`, `test_practice_module.py`, etc.).
   * Example: `test_auth.py::test_signup PASSED`, `test_practice_module.py::test_start_practice PASSED`.
   * All should pass if DB and Redis are running.

**Troubleshooting**

* **DB Connection Error**: Ensure `docker-compose.yml` is running (`docker ps` shows `db` and `redis` containers).
* **Test Failures**: Check seeded data in `seed_data.py` matches test expectations.

***

#### Frontend Unit Tests

**Prerequisites**

* Node.js 16+ installed.

**Steps**

1.  **Navigate to Frontend**:

    ```bash
    cd sat-prep-suite/frontend
    ```
2.  **Install Dependencies**:

    ```bash
    npm install
    ```
3.  **Run Tests**:

    ```bash
    npm test
    ```
4. **Verify Output**:
   * Expect 3 tests (`login.test.js`, `practice.test.js`, `full-test.test.js`).
   * Example: `PASS tests/login.test.js`, `Login Page renders and submits signup (xx ms)`.
   * All should pass with mocked API responses.

**Troubleshooting**

* **Jest Errors**: Ensure `jest`, `@testing-library/react`, and `@testing-library/jest-dom` are installed (`npm install`).
* **Mock Failures**: Verify `jest.mock` in test files matches `api.js` structure.

***

### Post-Deployment Notes

* **Access**: Frontend at `http://<frontend-ip>:3000`, backend at `http://<backend-ip>:8000`.
* **Scaling**: Current setup supports \~1,000 users; add load balancer (ALB) and multi-node ECS for 2,000+ (Item 9).
* **Cost**: \~$365.86/month for 1,000 users (ECS: $36.04, RDS: $12.41, Redis: $12.41, AI: $180).
* **Security**: Update `.env` with secure `SECRET_KEY`, hash passwords in production.

***

### Conclusion

* **Deployment**: Follow the AWS ECS steps to deploy the app, with GitHub Actions automating the process.
* **Testing**: Run backend (`pytest`) and frontend (`npm test`) unit tests to verify functionality.
* **Next**: Monitor deployment logs in ECS, test endpoints, and refine as needed.

Let me know if you need help with any step or additional configurations (e.g., ALB setup)!
