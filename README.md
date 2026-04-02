# Multi-Agent Full-Stack App Generator
### Course Materials — AI-Assisted Software Engineering

A two-session college course (2 × 1 hour) where Junior and Senior CS students build a complete full-stack web application using four collaborating Claude Code agents.

---

## What Students Build

Students pick a project idea, run four AI agents in sequence, and end up with a locally-running full-stack app pushed to their own GitHub repo — in two class sessions.

**The four-agent pipeline:**

```
[Your Idea]
    │
    ▼
01_design_agent   ──► design.md
                       (API spec + DB schema + component tree)
    │
    ├──────────────────────────────────────┐
    ▼                                      ▼
02_frontend_agent                  03_backend_agent
──► frontend_output.md             ──► backend_output.md
──► extract → frontend/            ──► extract → backend/
    │                                      │
    └──────────────┬───────────────────────┘
                   ▼
           04_review_agent
           ──► REVIEW.md
```

**Tech stack:** React + Vite · Node/Express or Python/FastAPI · SQLite

---

## Repository Contents

```
course-ai-agents/
├── prompts/
│   ├── 01_design_agent.md      # Architect — writes design.md
│   ├── 02_frontend_agent.md    # React dev — writes frontend_output.md
│   ├── 03_backend_agent.md     # API dev — writes backend_output.md
│   └── 04_review_agent.md      # Reviewer — writes REVIEW.md
├── scripts/
│   └── extract_files.py        # Splits output .md files into real source dirs
├── skills/
│   └── build-app.md            # /build-app skill — chains all four agents
├── SETUP.md                    # Student environment setup guide (Mac + Windows)
└── README.md                   # This file
```

---

## Quick Start (Students)

**Step 1 — Set up your environment**
Follow [SETUP.md](./SETUP.md) before your first class session.

**Step 2 — Install the skill**
```bash
claude skill install skills/build-app.md
```

**Step 3 — Pick a project idea**

| Category | Ideas |
|---|---|
| Productivity | Study Session Tracker · Personal Budget Manager · Habit Tracker |
| Developer Tools | Code Snippet Manager · Bug Tracker · API Request Tester |
| Campus / Social | Study Group Finder · Campus Event Board · Peer Tutoring Board |
| Fun / Creative | Movie/Book Wishlist · Recipe Box · Trivia Game Builder |

**Step 4 — Run the pipeline**

Option A — one command with the skill:
```bash
/build-app "habit tracker" node
```

Option B — run each agent manually (recommended for learning):
```bash
# Agent 01 — design
claude -p "$(sed -e 's|{PROJECT_IDEA}|Habit Tracker|g' -e 's|{BACKEND}|node|g' \
  prompts/01_design_agent.md)" > design.md

# Agent 02 — frontend
claude -p "$(cat prompts/02_frontend_agent.md)" \
  --context "$(cat design.md)" > frontend_output.md
python3 scripts/extract_files.py frontend_output.md

# Agent 03 — backend
claude -p "$(sed 's|{BACKEND}|node|g' prompts/03_backend_agent.md)" \
  --context "$(cat design.md)" > backend_output.md
python3 scripts/extract_files.py backend_output.md

# Agent 04 — review
claude -p "$(cat prompts/04_review_agent.md)" \
  --context "$(printf '# DESIGN\n'; cat design.md; \
               printf '\n\n# FRONTEND\n'; cat frontend_output.md; \
               printf '\n\n# BACKEND\n'; cat backend_output.md)" > REVIEW.md
```

**Step 5 — Run your app locally**
```bash
# Terminal 1 — frontend
cd frontend && npm install && npm run dev
# → http://localhost:5173

# Terminal 2 — backend (Node)
cd backend && npm install && node server.js
# → http://localhost:3001

# Terminal 2 — backend (Python)
cd backend && pip install -r requirements.txt
uvicorn main:app --reload --port 3001
# → http://localhost:3001/docs
```

**Step 6 — Push to GitHub**
```bash
git add design.md frontend_output.md backend_output.md REVIEW.md frontend/ backend/
git commit -m "feat: multi-agent generated full-stack app"
git push
```

---

## Output Files Reference

| File | Created by | Read by |
|---|---|---|
| `design.md` | Agent 01 | Agents 02, 03, 04 |
| `frontend_output.md` | Agent 02 | Agent 04, extract script |
| `backend_output.md` | Agent 03 | Agent 04, extract script |
| `REVIEW.md` | Agent 04 | You |
| `RUNBOOK.md` | `/build-app` skill | You |

---

## Windows Notes

All `$(cat ...)` commands use PowerShell equivalents. Each prompt file's `HOW TO RUN` section includes the PowerShell version. The extract script works identically on both platforms:
```powershell
python scripts\extract_files.py frontend_output.md
```

---

## Prerequisites

- Free [claude.ai](https://claude.ai) account
- Claude Code CLI installed and authenticated
- Node.js v18+
- Python 3.10+ (FastAPI track only)
- Git configured with your name and email

See [SETUP.md](./SETUP.md) for step-by-step installation instructions.

---

## For Instructors

Slide deck, lesson plan, and instructor prompts are maintained separately.
Contact the course team for access to the instructor materials repository.
