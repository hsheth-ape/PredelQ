# [Project] - Ways of Working

## Project Overview
- **Name:** [Project]
- **Repository:** [owner]/[repo]
- **Stack:** [Tech stack]
- **Created:** [Date]

## Session Types

| Type | Focus | Allowed | Not Allowed |
|------|-------|---------|-------------|
| **EXECUTION** | Code & Deploy | Code changes, schema changes, deployments, bug fixes | Planning, strategy, designs |
| **PLANNING** | Think & Review | Sprint planning, backlog, strategy, requirements, review | Code changes, designs, deployments |
| **UX** | Design | Wireframes, mockups, design specs | Code changes, deployments |

## Planning Outputs: Two Spec Types

| Spec Type | Format | Hands Off To |
|-----------|--------|--------------|
| **Visual/UI** | Design brief | UX Claude |
| **Workflow/Technical** | Execution plan + Mermaid | Execution Claude |

**Rule:** If Execution Claude needs to read it → Mermaid. If humans need visual iteration → UX Claude.

---

## ⚠️ CRITICAL: Build Verification

**Run `npm run build` after EVERY code change. No exceptions.**

### The Cycle

```
Make change → npm run build → Log milestone → Push → Verify deploy → Next change
```

**Never skip the build step. Ever.**

---

## ⚠️ CRITICAL: Milestone Logging (Execution)

**Log a milestone after EVERY successful `npm run build`.**

```markdown
### [HH:MM] Milestone: [Step name]
- **What:** [Files created/modified]
- **Build:** ✅ passed
- **Pushed:** ✅ branch | ❌ not yet
- **Next:** [Next step]
```

**Why:** If session fails mid-execution, milestones show exactly where it stopped.

---

## ⚠️ CRITICAL: Database Migrations (Execution)

**Execution Claude runs migrations directly. Do NOT leave as manual steps.**

```python
import requests

SUPABASE_PROJECT_ID = "[from CURRENT.md]"
MANAGEMENT_TOKEN = "[from CURRENT.md]"

headers = {
    "Authorization": f"Bearer {MANAGEMENT_TOKEN}",
    "Content-Type": "application/json"
}

def run_migration(sql):
    r = requests.post(
        f"https://api.supabase.com/v1/projects/{SUPABASE_PROJECT_ID}/database/query",
        headers=headers,
        json={"query": sql},
        verify=False
    )
    return r.status_code
```

---

## Session Protocol

### Every Session Start (MANDATORY)
1. Read WAYS_OF_WORKING.md (this file)
2. Read CURRENT.md (credentials, state)
3. Read LESSONS.md (recent entries)
4. Check `docs/predelq/sessions/` for recent logs
5. **Create session log file:** `docs/predelq/sessions/{DATE}-{type}-{NNN}.md`
6. State session type and goal
7. Ask "Where did we leave off?"

### Every Session End (MANDATORY)
1. Summarize accomplishments
2. Update session log with final status
3. Update CURRENT.md with new state
4. Fill "Promote to Project" section
5. State clear next steps
6. Confirm all changes pushed

---

## Session Log Lifecycle

```
1. Session starts    → Create log file from template
2. During session    → Add milestone entries (Execution: after each build)
3. Session ends      → Update status, fill outcomes
4. Review phase      → Analyze logs, mark items worth keeping
5. Planning promotes → Move items to LESSONS, BACKLOG, CURRENT
6. Cleanup           → Archive or delete old session logs
```

---

## Git Workflow

### Branch Naming

| Prefix | Use | Example |
|--------|-----|---------|
| `sprint-N/` | Sprint work | `sprint-3/github-tools` |
| `fix/` | Bug fixes | `fix/chat-scroll` |
| `experiment/` | Discovery | `experiment/modal-streaming` |

### When to Branch vs Main

| Change Type | Target | Why |
|-------------|--------|-----|
| Code changes | Branch → PR | Protect main |
| Schema changes | Branch → PR | High risk |
| Docs only | Main directly | Low risk |

**Rule:** If it can break the build → branch.

---

## Planning Hierarchy

| Layer | Question | Output |
|-------|----------|--------|
| **Discovery** | What's possible? | Insights, options |
| **Inventory** | What exists? | Feature catalog |
| **Strategy** | Where going? | Vision, phases |
| **Sprint Goals** | What to prove? | Success criteria |
| **Tasks** | How to execute? | Step-by-step plan |

### Rules
1. Discovery when uncertain
2. Inventory before sprints
3. Goals before tasks
4. Promote findings to docs

---

## Two-Document Sprint Pattern

| Document | Purpose | Lines |
|----------|---------|-------|
| SPRINT_N_INVENTORY.md | Design document | ~400 |
| SPRINT_N_PLAN.md | Execution instructions | ~1600 |

**Flow:**
1. Create Inventory
2. Review with human
3. Create Plan (after approval)
4. Hand to Execution

---

## Document Map

| Document | Purpose | Updated By |
|----------|---------|------------|
| CURRENT.md | Live state, credentials | All sessions |
| LESSONS.md | What we learned | All sessions |
| BACKLOG.md | Prioritized work | Planning |
| REVIEW.md | Review findings | Planning |
| SCHEMA.md | Database schema | Execution |
| CHANGELOG.md | Version history | Execution |
| sessions/*.md | Session logs | All sessions |

---

## Commit Convention

```
type(scope): description

Types: feat, fix, docs, refactor, test, chore
```

---

## Communication Style
- Be direct and specific
- Reference document sections
- Ask clarifying questions early
- Summarize decisions explicitly
