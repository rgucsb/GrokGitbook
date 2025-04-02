# Copy of 3-28-2025 Latest Directory Structure

Let’s create a Markdown file that contains the JSON format for the `Questions` table, as requested, along with the updated `package.json` files for the frontend and mobile app, the updated `requirements.txt` for the backend, and the instructions for testing and installation. This Markdown file will serve as a comprehensive document for the changes made to address the compatibility issues with Vite and the database, and it will include the JSON format for the `Questions` table. You can save this as a `.md` file and use it for documentation or to share with your team.

***

### `updated_files_and_json.md`

````markdown
# SAT Smart Prep App Updates and Questions Table JSON Format

This document contains the updated files and instructions for the **SAT Smart Prep App** by **Learner Labs**, addressing compatibility issues with Vite and the database. It also includes the JSON format for the `Questions` table as requested.

## JSON Format for the Questions Table

Below is the JSON representation of a single record in the `Questions` table, reflecting the latest schema with IRT 3PL parameters, metadata fields, and qualification status.

```json
{
  "id": "q123",
  "domain": "Reading & Writing",
  "skill": "Grammar",
  "question_text": "Which of the following is the correct form of the sentence?",
  "options": [
    "A) The dog run fast.",
    "B) The dog runs fast.",
    "C) The dog running fast.",
    "D) The dog runned fast."
  ],
  "correct_answer": "B) The dog runs fast.",
  "explanation": "The correct form uses the singular verb 'runs' to agree with the singular subject 'dog'.",
  "irt_a": 1.2,
  "irt_b": 0.5,
  "irt_c": 0.25,
  "has_image": false,
  "has_table": false,
  "is_mcq": true,
  "qualification_status": "qualified",
  "created_at": "2025-03-28T10:00:00Z"
}
````

#### Notes on the JSON Format

* **Fields**: Matches the latest `Questions` table schema, including IRT 3PL parameters (`irt_a`, `irt_b`, `irt_c`), metadata (`has_image`, `has_table`, `is_mcq`), and `qualification_status`.
* **Data Types**:
  * `options` is a JSON array (list of strings).
  * `irt_a`, `irt_b`, `irt_c` are floats.
  * `has_image`, `has_table`, `is_mcq` are booleans.
  * `created_at` is a timestamp in ISO 8601 format.
* **Example Values**: Reflects a sample record for a multiple-choice question in the "Reading & Writing" domain, with `irt_c` fixed at 0.25 for MCQs.

### Updated Node Modules for Vite Compatibility

To resolve issues with Vite and ensure compatibility with modern Node.js versions, the `package.json` files for the frontend and mobile app have been updated.

#### Updated `frontend/package.json`

```json
{
  "name": "sat-smart-prep-frontend",
  "private": true,
  "version": "0.0.1",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "lint": "eslint . --ext js,jsx --report-unused-disable-directives --max-warnings 0"
  },
  "dependencies": {
    "react": "^18.3.1",
    "react-dom": "^18.3.1",
    "axios": "^1.7.2"
  },
  "devDependencies": {
    "vite": "^6.2.4",
    "@vitejs/plugin-react": "^4.3.1",
    "eslint": "^8.57.0",
    "eslint-plugin-react": "^7.34.3",
    "eslint-plugin-react-hooks": "^4.6.2",
    "eslint-plugin-react-refresh": "^0.4.7"
  }
}
```

**Changes Made**

* **Vite**: Updated to `6.2.4`, the latest version as of March 23, 2025.
* **@vitejs/plugin-react**: Updated to `4.3.1`, ensuring React Fast Refresh and JSX support.
* **React and React-DOM**: Updated to `18.3.1`, the latest stable version.
* **Axios**: Updated to `1.7.2`, for API requests.
* **ESLint and Plugins**: Updated to the latest versions for linting compatibility.

#### Updated `SATSmartPrepApp/package.json`

```json
{
  "name": "sat-smart-prep-mobile",
  "version": "0.0.1",
  "private": true,
  "scripts": {
    "start": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^18.3.1",
    "react-dom": "^18.3.1",
    "react-native": "^0.74.3",
    "react-native-reanimated": "^3.14.0",
    "axios": "^1.7.2"
  },
  "devDependencies": {
    "vite": "^6.2.4",
    "@vitejs/plugin-react": "^4.3.1",
    "@react-native-community/cli": "^13.6.9",
    "@react-native-community/cli-platform-android": "^13.6.9",
    "@react-native-community/cli-platform-ios": "^13.6.9"
  }
}
```

**Changes Made**

* **Vite and @vitejs/plugin-react**: Updated to `6.2.4` and `4.3.1`.
* **React and React-DOM**: Updated to `18.3.1`.
* **React Native**: Updated to `0.74.3`, the latest stable version.
* **React Native Reanimated**: Updated to `3.14.0`, for animations.
* **Axios**: Updated to `1.7.2`.
* **React Native CLI**: Updated to `13.6.9`, ensuring compatibility with React Native 0.74.3.

### Updated Database Client for PostgreSQL Compatibility

The backend has been updated to use `psycopg3` (version 3.2.1) instead of `psycopg2`, ensuring compatibility with modern PostgreSQL versions (10 to 16).

#### Updated `backend/requirements.txt`

```plaintext
fastapi==0.111.0
psycopg==3.2.1
uvicorn==0.30.1
```

**Changes Made**

* Replaced `psycopg2` with `psycopg==3.2.1`.
* Updated `fastapi` to `0.111.0` and `uvicorn` to `0.30.1`, the latest versions as of April 2025.

#### Code Changes for `psycopg3`

All backend files (`diagnostic.py`, `practice.py`, `progress_monitoring.py`, `study_plan.py`) have been updated to use `psycopg` instead of `psycopg2`. For example, in `backend/src/diagnostic.py`:

**Before**

```python
import psycopg2
from psycopg2.extras import RealDictCursor

def get_db_connection():
    return psycopg2.connect(
        dbname="satprep",
        user="user",
        password="password",
        host="localhost",
        port="5432"
    )
```

**After**

```python
import psycopg
from psycopg.rows import dict_row

def get_db_connection():
    return psycopg.connect(
        dbname="satprep",
        user="user",
        password="password",
        host="localhost",
        port="5432",
        row_factory=dict_row  # Equivalent to RealDictCursor
    )
```

### Test and Install Instructions

#### 1. Ensure Node.js and Python Versions

*   **Node.js**: Ensure Node.js 20.x is installed. Check with:

    ```bash
    node -v
    ```

    If not installed, install Node.js 20.x:

    ```bash
    nvm install 20
    nvm use 20
    ```
*   **Python**: Ensure Python 3.9+ is installed. Check with:

    ```bash
    python --version
    ```

    If not installed, download and install Python 3.9+.

#### 2. Install Updated Node Modules

**For `frontend/`**

1.  Navigate to the `frontend/` directory:

    ```bash
    cd frontend
    ```
2.  Remove old dependencies:

    ```bash
    rm -rf node_modules package-lock.json
    ```
3. Update `package.json` with the content provided above.
4.  Install dependencies:

    ```bash
    npm install
    ```
5.  Start the development server to test:

    ```bash
    npm run dev
    ```

**For `SATSmartPrepApp/`**

1.  Navigate to the `SATSmartPrepApp/` directory:

    ```bash
    cd SATSmartPrepApp
    ```
2.  Remove old dependencies:

    ```bash
    rm -rf node_modules package-lock.json
    ```
3. Update `package.json` with the content provided above.
4.  Install dependencies:

    ```bash
    npm install
    ```
5.  Start the development server to test:

    ```bash
    npm start
    ```

#### 3. Install Updated Python Dependencies

1.  Navigate to the `backend/` directory:

    ```bash
    cd backend
    ```
2.  Remove old virtual environment (if any):

    ```bash
    rm -rf venv
    ```
3.  Create a new virtual environment:

    ```bash
    python -m venv venv
    source venv/bin/activate  # On Windows: venv\Scripts\activate
    ```
4. Update `requirements.txt` with the content provided above.
5.  Install dependencies:

    ```bash
    pip install -r requirements.txt
    ```
6.  Start the backend server to test:

    ```bash
    uvicorn main:app --reload
    ```

#### 4. Test Database Compatibility

*   Ensure the PostgreSQL server is running and supports a modern version (e.g., PostgreSQL 16). Check the version with:

    ```bash
    psql --version
    ```
* If the PostgreSQL version is older than 10, upgrade to at least PostgreSQL 14 or 16.
* Test the database connection by running a simple query from the backend (e.g., `SELECT * FROM users`).

#### 5. Resolve Common Issues

* **Node.js/Vite Errors**:
  * If Vite fails to start, run `npm install --force` to resolve peer dependency issues.
  * Ensure `type: "module"` is present in `package.json`.
* **Database Connection Errors**:
  * Verify the PostgreSQL server is running and credentials are correct.
  * Ensure the PostgreSQL server version is compatible (10 to 16).
* **Testing Failures**:
  * Ensure the database schema includes `Question_IRT_Updates` and `Qualified_Questions_Reports` tables. Run a migration script if needed.

````

---

## Instructions to Use the Markdown File

1. **Save the File**:
   - Copy the content above into a file named `updated_files_and_json.md`.
   - Save it in your project directory or a documentation folder.

2. **View the File**:
   - Open `updated_files_and_json.md` in a Markdown viewer (e.g., VS Code, GitHub, or any Markdown renderer) to see the formatted content, including the JSON format, updated `package.json` files, and installation instructions.

3. **Apply the Changes**:
   - Follow the instructions in the "Test and Install Instructions" section to update your project dependencies and test the setup.
   - Use the JSON format for the `Questions` table as needed for testing or documentation.

4. **Upload to GitHub** (Optional):
   - Add `updated_files_and_json.md` to your GitHub repository for documentation:
     ```bash
     git add updated_files_and_json.md
     git commit -m "Added updated files and JSON format for Questions table"
     git push origin main
     ```

---

## Verification
- **JSON Format Included**: The Markdown file includes the JSON format for the `Questions` table, matching the latest schema.
- **Updated Dependencies**: The file includes the updated `package.json` files for `frontend/` and `SATSmartPrepApp/`, as well as the updated `requirements.txt` for the backend.
- **Instructions Provided**: The file provides detailed steps to test and install the updated dependencies, addressing the compatibility issues with Vite and the database.

Let me know if you need further assistance with applying these changes or if you’d like to modify the Markdown file further!
````
