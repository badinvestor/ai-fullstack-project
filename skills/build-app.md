---
name: build-app
description: >
  Orchestrates the full four-agent pipeline to generate a complete
  full-stack app. Runs design → frontend → backend → review agents
  in sequence, saving each output to a local file before passing it
  to the next agent. Produces design.md, frontend_output.md,
  backend_output.md, REVIEW.md, and a RUNBOOK.md.
args:
  - name: project_idea
    description: One-line description of the app to build (e.g. "a habit tracker")
    required: true
  - name: backend
    description: "Backend framework: node or python (default: node)"
    required: false
    default: node
---

<!-- ============================================================
     SKILL FILE: skills/build-app.md
     ---------------------------------------------------------------
     This skill chains all four agents automatically.
     Each agent's output is saved to a file before the next agent runs.
     This means you can re-run any individual agent without losing
     the work done by the others.

     FILE CHAIN:
       [your idea] → 01_design_agent → design.md
       design.md   → 02_frontend_agent → frontend_output.md
       design.md   → 03_backend_agent  → backend_output.md
       design.md + frontend_output.md + backend_output.md
                   → 04_review_agent  → REVIEW.md
       All of the above → RUNBOOK.md (written by this skill directly)

     HOW TO INSTALL THIS SKILL:
       claude skill install skills/build-app.md

     HOW TO INVOKE:
       /build-app "habit tracker" node
       /build-app "recipe box" python

     WHAT GETS CREATED:
       design.md            — technical spec (API, DB, components)
       frontend_output.md   — all React source files
       backend_output.md    — all server source files
       REVIEW.md            — code review report
       RUNBOOK.md           — step-by-step instructions to run your app

     IF A STEP FAILS:
       Each output file is saved before the next step starts.
       Re-run only the failed agent's prompt manually (see each
       prompt file's HOW TO RUN section) then re-run /build-app
       to pick up from where it left off.
============================================================ -->

## Step 1 — Design Agent
Run the design agent with the project idea and backend choice.
Save output to design.md. This file is the shared contract for all other agents.

```bash
# Mac / Linux
claude -p "$(sed \
  -e 's|{PROJECT_IDEA}|{{project_idea}}|g' \
  -e 's|{BACKEND}|{{backend}}|g' \
  prompts/01_design_agent.md)" > design.md

echo "✓ design.md saved ($(wc -l < design.md) lines)"
```

```powershell
# Windows PowerShell
$prompt = (Get-Content prompts\01_design_agent.md -Raw) `
  -replace '\{PROJECT_IDEA\}','{{project_idea}}' `
  -replace '\{BACKEND\}','{{backend}}'
claude -p $prompt | Out-File -Encoding utf8 design.md
Write-Host "✓ design.md saved"
```

Verify design.md contains all three required sections before continuing:
`grep "^## API Spec\|^## DB Schema\|^## Component Tree" design.md`

---

## Step 2 — Frontend Agent
Run the frontend agent using design.md as context.
Save output to frontend_output.md.

```bash
# Mac / Linux
claude -p "$(cat prompts/02_frontend_agent.md)" \
  --context "$(cat design.md)" \
  > frontend_output.md

echo "✓ frontend_output.md saved ($(wc -l < frontend_output.md) lines)"
echo "  Files inside: $(grep -c '// FILE:' frontend_output.md)"
```

```powershell
# Windows PowerShell
$prompt  = Get-Content prompts\02_frontend_agent.md -Raw
$context = Get-Content design.md -Raw
claude -p $prompt --context $context | Out-File -Encoding utf8 frontend_output.md
Write-Host "✓ frontend_output.md saved"
```

---

## Step 3 — Backend Agent
Run the backend agent using design.md as context.
Save output to backend_output.md.

```bash
# Mac / Linux
claude -p "$(sed 's|{BACKEND}|{{backend}}|g' prompts/03_backend_agent.md)" \
  --context "$(cat design.md)" \
  > backend_output.md

echo "✓ backend_output.md saved ($(wc -l < backend_output.md) lines)"
```

```powershell
# Windows PowerShell
$prompt  = (Get-Content prompts\03_backend_agent.md -Raw) `
  -replace '\{BACKEND\}','{{backend}}'
$context = Get-Content design.md -Raw
claude -p $prompt --context $context | Out-File -Encoding utf8 backend_output.md
Write-Host "✓ backend_output.md saved"
```

---

## Step 4 — Review Agent
Run the review agent using design.md + frontend_output.md + backend_output.md.
Save output to REVIEW.md.

```bash
# Mac / Linux
claude -p "$(cat prompts/04_review_agent.md)" \
  --context "$(printf '# DESIGN\n'; cat design.md; \
               printf '\n\n# FRONTEND\n'; cat frontend_output.md; \
               printf '\n\n# BACKEND\n'; cat backend_output.md)" \
  > REVIEW.md

echo "✓ REVIEW.md saved"
```

```powershell
# Windows PowerShell
$prompt  = Get-Content prompts\04_review_agent.md -Raw
$context = "# DESIGN`n" + (Get-Content design.md -Raw) + `
           "`n`n# FRONTEND`n" + (Get-Content frontend_output.md -Raw) + `
           "`n`n# BACKEND`n" + (Get-Content backend_output.md -Raw)
claude -p $prompt --context $context | Out-File -Encoding utf8 REVIEW.md
Write-Host "✓ REVIEW.md saved"
```

---

## Step 5 — Write RUNBOOK.md
Write a RUNBOOK.md that tells the student exactly how to run their app locally.
Use the project idea, backend choice, and file structure generated above.

The RUNBOOK.md must contain:

```markdown
# RUNBOOK — {{project_idea}}
Generated by /build-app on: [today's date]
Backend: {{backend}}

## Prerequisites
- [ ] Node.js v18+ installed: `node --version`
- [ ] Python 3.10+ installed (FastAPI only): `python --version`
- [ ] All output files present: design.md, frontend_output.md, backend_output.md, REVIEW.md

## Step 1 — Extract Generated Files
Run the extraction script to copy all code blocks into their proper directories:
  python3 scripts/extract_files.py frontend_output.md
  python3 scripts/extract_files.py backend_output.md

## Step 2 — Install Dependencies

Frontend (run in one terminal):
  cd frontend
  npm install

Backend — Node/Express (run in a second terminal):
  cd backend
  npm install

Backend — Python/FastAPI (run in a second terminal):
  cd backend
  pip install -r requirements.txt

## Step 3 — Start Both Servers

Terminal 1 (frontend):
  cd frontend
  npm run dev
  → App running at http://localhost:5173

Terminal 2 (backend — Node):
  cd backend
  node server.js
  → API running at http://localhost:3001

Terminal 2 (backend — Python):
  cd backend
  uvicorn main:app --reload --port 3001
  → API docs at http://localhost:3001/docs

## Step 4 — Open in Browser
  http://localhost:5173

## Resetting the Database
Delete the SQLite file and restart the backend to re-seed sample data:
  rm backend/data/app.db   (Mac/Linux)
  del backend\data\app.db  (Windows)
  [restart backend server]

## Files Reference
  design.md          — original spec (do not edit by hand)
  frontend_output.md — raw frontend agent output
  backend_output.md  — raw backend agent output
  REVIEW.md          — code review report (read before running)
```

---

## Step 6 — Summary
Print a summary of what was created.

```bash
echo ""
echo "========================================="
echo " /build-app complete"
echo "========================================="
echo " design.md          $(wc -l < design.md) lines"
echo " frontend_output.md $(wc -l < frontend_output.md) lines"
echo " backend_output.md  $(wc -l < backend_output.md) lines"
echo " REVIEW.md          $(wc -l < REVIEW.md) lines"
echo " RUNBOOK.md         written"
echo ""
echo " Next: run 'python3 scripts/extract_files.py frontend_output.md'"
echo "       then follow RUNBOOK.md to start your app."
echo "========================================="
```
