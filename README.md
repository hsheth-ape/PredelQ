# Claude Project Templates v2

> Updated templates with improved document structure based on learnings from Gibbon project.

## What's New in v2

### Key Improvements

1. **Two-Document Sprint Pattern**
   - **Inventory** = Design document (architecture, schemas, ~400 lines)
   - **Plan** = Execution instructions (exact code, step-by-step, ~1600 lines)

2. **Session Log Lifecycle**
   - Create → Milestones → Complete → Review → Promote → Cleanup
   - Milestone logging tied to successful builds

3. **Supabase Migration Pattern**
   - Direct API execution (don't leave manual steps)
   - Verification queries included

4. **Planning Output Types**
   - Visual specs → UX Claude via SVG
   - Workflow specs → Execution Claude via Mermaid

5. **Project Bootstrap Pattern**
   - Shared Modal gateway for code execution
   - Automated GitHub + Supabase project creation
   - "Keys to all keys" — one endpoint runs anything

---

## Directory Structure

```
claude-templates-v2/
├── instructions/           # Claude Project instruction templates
│   ├── PLANNING.md        # Planning Claude instructions
│   ├── EXECUTION.md       # Execution Claude instructions
│   └── UX.md              # UX Claude instructions (SVG-first)
│
├── docs-templates/         # Project documentation templates
│   ├── WAYS_OF_WORKING.md
│   ├── CURRENT.md
│   ├── BACKLOG.md
│   ├── LESSONS.md
│   ├── REVIEW.md
│   ├── SCHEMA.md
│   ├── CHANGELOG.md
│   └── sessions/
│       └── README.md
│
├── sprint-templates/       # Sprint planning templates
│   ├── SPRINT_INVENTORY_TEMPLATE.md  # Design document
│   ├── SPRINT_PLAN_TEMPLATE.md       # Execution instructions
│   └── SESSION_LOG_TEMPLATE.md       # Session logging
│
├── toolbox/                # Reusable patterns and scripts
│   ├── BOOTSTRAP.md       # Project bootstrap (GitHub, Supabase, Modal)
│   ├── GITHUB_API.md      # GitHub API patterns
│   ├── MIGRATIONS.md      # Supabase migration patterns
│   └── PLANNING.md        # Planning methodology
│
└── README.md
```

---

## Quick Start

### 1. Create Claude Projects

Create three Claude Projects with these instruction files:

| Project | Instructions File |
|---------|-------------------|
| [Project] - Planning | `instructions/PLANNING.md` |
| [Project] - Execution | `instructions/EXECUTION.md` |
| [Project] - UX | `instructions/UX.md` |

### 2. Bootstrap New Project (Optional)

Use `toolbox/BOOTSTRAP.md` patterns to automate:
- GitHub repo creation
- Supabase project creation
- Modal gateway (shared, no setup needed)

### 3. Set Up Project Docs

Copy `docs-templates/` to your repo as `docs/[project]/`:

```
your-repo/
└── docs/
    └── your-project/
        ├── WAYS_OF_WORKING.md
        ├── CURRENT.md
        ├── BACKLOG.md
        ├── LESSONS.md
        ├── REVIEW.md
        ├── sessions/
        │   └── README.md
        └── toolbox/
```

### 4. Fill in Placeholders

Replace in all files:
- `[PROJECT]` → Your project name
- `[OWNER]` → GitHub username/org
- `[REPO]` → Repository name
- `[YOUR_GITHUB_TOKEN]` → Your PAT
- Credentials in CURRENT.md

---

## Sprint Workflow

### Planning Phase

```
1. Planning Claude creates SPRINT_N_INVENTORY.md
   - Architecture decisions
   - Schema designs
   - Code sketches
   - ~400 lines

2. Review with human

3. Planning Claude creates SPRINT_N_PLAN.md
   - Exact file paths
   - Complete code (copy-paste ready)
   - Step-by-step instructions
   - Build verification at each step
   - ~1600 lines
```

### Execution Phase

```
1. Execution Claude reads SPRINT_N_PLAN.md
2. Creates session log
3. Executes phase by phase:
   - Make change
   - npm run build
   - Log milestone
   - Push to branch
   - Verify Vercel
4. Opens PR when complete
```

### Review Phase

```
1. Planning Claude reviews PR
2. Checks against plan
3. Verifies build/deploy
4. Documents findings in REVIEW.md
5. Human approves merge
```

---

## Shared Infrastructure

### Modal Bootstrap Gateway

One deployed function serves all projects — no Modal keys needed:

```python
MODAL_BOOTSTRAP_URL = "[your-deployed-url]"

# Run any shell command
requests.post(MODAL_BOOTSTRAP_URL, json={
    "type": "shell",
    "command": "npm run build"
})

# Run any Python code
requests.post(MODAL_BOOTSTRAP_URL, json={
    "type": "python", 
    "code": "print('Hello')"
})
```

See `toolbox/BOOTSTRAP.md` for full details.

---

## Key Principles

### From Gibbon Learnings

1. **Build verification is mandatory** — `npm run build` after every change
2. **Milestone logging saves work** — Log after every successful build
3. **GitHub API > git CLI** — API works when git is blocked
4. **Two documents per sprint** — Inventory for design, Plan for execution
5. **Agents run migrations** — Don't leave manual steps
6. **Session logs are ephemeral** — Promote valuable content to permanent docs
7. **Modal bootstrap gateway** — One deploy, unlimited capability

---

## License

MIT
