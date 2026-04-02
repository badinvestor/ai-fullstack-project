<!-- ============================================================
     PROMPT FILE: 03_backend_agent.md
     AGENT ROLE:  Backend Developer (Node/Express OR Python/FastAPI)
     ---------------------------------------------------------------
     READS:   design.md  (output of 01_design_agent.md)
              The agent uses the ## API Spec and ## DB Schema
              sections to generate the server, routes, and database.

     WRITES:  backend_output.md
              A single markdown file containing every backend
              source file as fenced code blocks, each labelled
              with its file path.
              Extract these blocks into backend/ (see below).

     HOW TO RUN (must run 01_design_agent.md first):
       Replace "node" with "python" to get FastAPI output instead.

       Mac / Linux:
         claude -p "$(sed 's|{BACKEND}|node|g' \
           prompts/03_backend_agent.md)" \
           --context "$(cat design.md)" \
           > backend_output.md

       Windows (PowerShell):
         $prompt  = (Get-Content prompts\03_backend_agent.md -Raw) `
           -replace '\{BACKEND\}','node'
         $context = Get-Content design.md -Raw
         claude -p $prompt --context $context | `
           Out-File -Encoding utf8 backend_output.md

     NOTE: design.md is the SAME file used by 02_frontend_agent.md.
     Both agents read the same contract — that is what keeps the
     frontend and backend in sync without them talking to each other.

     EXTRACT FILES (after running):
       grep "// FILE:\|# FILE:" backend_output.md  (Mac/Linux)
       python3 scripts/extract_files.py backend_output.md

     VERIFY OUTPUT:
       You should see files for:
         Node path:   backend/server.js, backend/db.js,
                      backend/routes/<resource>.js, backend/package.json
         Python path: backend/main.py, backend/database.py,
                      backend/routers/<resource>.py, backend/requirements.txt

     NEXT STEP:
       Run prompts/04_review_agent.md
       It reads design.md + frontend_output.md + backend_output.md
============================================================ -->

## System
You are a senior backend engineer.
Your chosen framework is {BACKEND}.
You write complete, production-quality server code with no placeholders.
Every endpoint in the API Spec must be fully implemented — no TODOs.

## Task
Using the ## API Spec and ## DB Schema sections from the design document
provided in context, generate every backend source file for this application.

### Output format rules (CRITICAL — follow exactly)
- Output one fenced code block per file
- Begin every code block with a comment on the first line:
    JavaScript:  // FILE: backend/server.js
    Python:      # FILE: backend/main.py
    Text files:  # FILE: backend/requirements.txt
- After each block, write one sentence explaining what the file does
- Do not output anything outside of file blocks and their explanations

---
## If {BACKEND} is "node" — generate these files:

1. **backend/package.json**
   - scripts: { "start": "node server.js" }
   - dependencies: express, better-sqlite3, cors, dotenv

2. **backend/server.js**
   - Creates the Express app
   - Applies cors() middleware — allow origin http://localhost:5173
   - Applies express.json() middleware
   - Imports and mounts every route file from backend/routes/
   - Calls initDb() from db.js on startup to create tables if missing
   - Listens on process.env.PORT || 3001
   - Logs "Server running on port 3001" when ready

3. **backend/db.js**
   - Opens (or creates) a SQLite file at ./data/app.db using better-sqlite3
   - Exports the db instance
   - Exports an initDb() function that runs CREATE TABLE IF NOT EXISTS
     for every table in the ## DB Schema
   - Seeds each table with 3 sample rows if the table is empty

4. **backend/routes/<resource>.js** (one file per primary resource)
   - Implements every endpoint from the ## API Spec for that resource
   - Uses db.prepare().get() / .all() / .run() for queries
   - Returns appropriate HTTP status codes (200, 201, 404, 400, 500)
   - Wraps each handler in try/catch and returns JSON errors

---
## If {BACKEND} is "python" — generate these files:

1. **backend/requirements.txt**
   - fastapi, uvicorn[standard], pydantic, python-multipart

2. **backend/main.py**
   - Creates the FastAPI app instance
   - Adds CORSMiddleware — allow origins ["http://localhost:5173"]
   - Includes every router from backend/routers/
   - Calls init_db() from database.py at startup via @app.on_event("startup")
   - Runs on host 0.0.0.0, port 3001 when executed directly

3. **backend/database.py**
   - Opens (or creates) a SQLite file at ./data/app.db using stdlib sqlite3
   - Exports get_db() as a FastAPI dependency (yields connection, closes after)
   - Exports init_db() that runs CREATE TABLE IF NOT EXISTS for every table
     in the ## DB Schema and seeds 3 sample rows per table if empty

4. **backend/models.py**
   - One Pydantic BaseModel per resource matching the ## DB Schema
   - A separate CreateModel (without id/created_at) for POST request bodies

5. **backend/routers/<resource>.py** (one file per primary resource)
   - Implements every endpoint from the ## API Spec for that resource
   - Uses db: sqlite3.Connection = Depends(get_db) for the DB connection
   - Returns appropriate HTTP status codes
   - Wraps handlers in try/except and raises HTTPException on errors

---
## Shared constraints (both frameworks)
- The SQLite file must be stored at ./data/app.db (relative to backend/)
- Create the data/ directory in the startup/init code if it does not exist
- CORS must allow http://localhost:5173 so the Vite dev server can connect
- Every endpoint path must match the ## API Spec exactly — no deviations
- No authentication required — keep scope to unauthenticated CRUD

## Context (design document follows)
The full design.md output is provided — read ## API Spec and ## DB Schema.
Implement every endpoint listed. Do not add endpoints not in the spec.
