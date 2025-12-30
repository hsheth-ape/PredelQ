# [PROJECT] - Execution

**Type:** EXECUTION | **Repo:** `[OWNER]/[REPO]` | **Docs:** `docs/[project]/`

## Credentials

| Service | Value |
|---------|-------|
| **GitHub Token** | `[YOUR_GITHUB_TOKEN]` |
| **GitHub Repo** | `[OWNER]/[REPO]` |

*All other credentials (Supabase, Vercel, Modal) in CURRENT.md*

---

## ⚠️ CRITICAL: GitHub Connection Method

**Do NOT use `git clone` or `git push` — they fail due to proxy/auth issues.**

**Step 1: Download repo via tarball**
```bash
cd /tmp
curl -L -H "Authorization: token [YOUR_GITHUB_TOKEN]" \
  https://api.github.com/repos/[OWNER]/[REPO]/tarball/main -o repo.tar.gz
tar -xzf repo.tar.gz && mv [OWNER]-[REPO]-* repo && cd repo
npm install
npm run build  # Verify clean start
```

**Step 2: Use GitHub API for all file operations**
```python
import requests
import base64

TOKEN = "[YOUR_GITHUB_TOKEN]"
REPO = "[OWNER]/[REPO]"
HEADERS = {"Authorization": f"token {TOKEN}"}

def read_file(path, branch="main"):
    url = f"https://api.github.com/repos/{REPO}/contents/{path}?ref={branch}"
    r = requests.get(url, headers=HEADERS)
    if r.status_code == 200:
        return base64.b64decode(r.json()["content"]).decode()
    return f"Error: {r.status_code} - {r.text}"

def write_file(path, content, message, branch="main"):
    url = f"https://api.github.com/repos/{REPO}/contents/{path}"
    r = requests.get(url + f"?ref={branch}", headers=HEADERS)
    sha = r.json().get("sha") if r.status_code == 200 else None
    data = {
        "message": message,
        "content": base64.b64encode(content.encode()).decode(),
        "branch": branch
    }
    if sha:
        data["sha"] = sha
    return requests.put(url, headers=HEADERS, json=data)

def create_branch(name, from_branch="main"):
    r = requests.get(f"https://api.github.com/repos/{REPO}/git/ref/heads/{from_branch}", headers=HEADERS)
    sha = r.json()["object"]["sha"]
    return requests.post(
        f"https://api.github.com/repos/{REPO}/git/refs",
        headers=HEADERS,
        json={"ref": f"refs/heads/{name}", "sha": sha}
    )

def create_pr(title, body, head, base="main"):
    return requests.post(
        f"https://api.github.com/repos/{REPO}/pulls",
        headers=HEADERS,
        json={"title": title, "body": body, "head": head, "base": base}
    )
```

**If GitHub API or build fails, STOP and report the error.**

---

## Session Protocol

### Start (MANDATORY)
1. **Download repo** (tarball method above)
2. **Verify build:** `npm run build` must pass
3. Read: `docs/[project]/WAYS_OF_WORKING.md`
4. Read: `docs/[project]/CURRENT.md` (credentials)
5. Read: `docs/[project]/LESSONS.md` (recent entries)
6. **Find execution plan:** `docs/[project]/SPRINT_N_PLAN.md`
7. **Create session log:** `docs/[project]/sessions/{DATE}-execution-{sprint}.md`
8. **Create branch:** `sprint-N/feature-name`
9. State: "EXECUTION session for Sprint N"
10. **Begin execution per plan — follow phases in order**

### End (MANDATORY)
1. Verify all planned items complete
2. Run final build verification
3. **Open PR** to main
4. Update session log with final status
5. Push session log to branch
6. State: "PR ready for review at [URL]"

---

## ⚠️ CRITICAL: Build Verification

**Run `npm run build` after EVERY file change. No exceptions.**

```bash
npm run build  # Must pass before proceeding
```

**If build fails:**
1. FIX IT before continuing
2. Never stack changes on broken code
3. Never commit broken code

---

## ⚠️ CRITICAL: Milestone Logging

**Log a milestone after EVERY successful `npm run build`.** This is mandatory.

```markdown
### [HH:MM] Milestone: [Step name]
- **What:** [Files created/modified]
- **Build:** ✅ passed
- **Pushed:** ✅ branch | ❌ not yet
- **Next:** [Next step]
```

**Why:** If session dies mid-execution, milestones show exactly where it stopped.

---

## The Execution Cycle

```
1. Make change (locally in downloaded repo)
2. npm run build (must pass)
3. Log milestone in session log
4. Push to branch via GitHub API
5. Verify Vercel deployment (if applicable)
6. Next change
```

**Never skip the build step. Ever.**
**Never skip the milestone log. Ever.**

---

## ⚠️ CRITICAL: Database Migrations

**Execution Claude runs migrations directly. Do NOT leave as manual steps.**

### Supabase Management API

```python
import requests

SUPABASE_PROJECT_ID = "[PROJECT_ID]"  # From CURRENT.md
MANAGEMENT_TOKEN = "[TOKEN]"  # From CURRENT.md

headers = {
    "Authorization": f"Bearer {MANAGEMENT_TOKEN}",
    "Content-Type": "application/json"
}

def run_migration(sql):
    r = requests.post(
        f"https://api.supabase.com/v1/projects/{SUPABASE_PROJECT_ID}/database/query",
        headers=headers,
        json={"query": sql},
        verify=False  # Required due to proxy
    )
    return r.status_code, r.json()

# Example
status, result = run_migration("ALTER TABLE users ADD COLUMN new_field TEXT;")
print(f"Status: {status}")
```

### Migration Checklist
1. **Write migration file** → `supabase/migrations/YYYYMMDD_description.sql`
2. **Run via API** → Use script above
3. **Verify** → Query `information_schema` to confirm
4. **Commit migration file** → Track in git
5. **Log milestone** → Note migration was applied

---

## Boundaries

✅ **DO in Execution:**
- Write code per execution plan
- Run builds and tests
- Push to branches
- Open PRs
- Run database migrations
- Fix build errors

❌ **DON'T in Execution:**
- Change scope or requirements → Ask Planning
- Skip build verification → Never
- Commit to main directly → Use branches
- Make architectural decisions → Ask Planning
- Improvise when blocked → Report to Planning

**If plan is unclear or blocked:** STOP and report. Do not improvise.

---

## Git Workflow

| Change Type | Target | Why |
|-------------|--------|-----|
| Code changes | Branch → PR | Protect main |
| Schema changes | Branch → PR | High risk |
| Docs only | Main directly | Low risk |

**Branch naming:** `sprint-N/feature-name` or `fix/bug-description`

**Commit convention:**
```
type(scope): description

Types: feat, fix, docs, refactor, test, chore
```

---

## Phased Execution

Execution plans are divided into phases. Follow this pattern:

### Between Phases
1. Complete all steps in current phase
2. Verify build passes
3. Push all changes to branch
4. Log phase completion milestone
5. **Natural checkpoint** — Can pause here if needed
6. Begin next phase

### If Session Interrupted
- Milestones show exactly where you stopped
- Next session resumes from last milestone
- Don't repeat completed work

---

## PR Checklist

Before opening PR:
- [ ] All phases complete
- [ ] Build passes
- [ ] All migrations applied
- [ ] Session log updated
- [ ] Vercel deployment successful (if applicable)

```python
pr = create_pr(
    title="Sprint N: [Description]",
    body="""
## Summary
[What this PR does]

## Changes
- [List of changes]

## Testing
- [x] Build passes
- [x] Migrations applied
- [x] Deployment verified
    """,
    head="sprint-N/feature-name",
    base="main"
)
print(f"PR: {pr.json().get('html_url')}")
```

---

## Troubleshooting

### Build Fails
1. Read the full error message
2. Fix the specific error
3. Don't add more code on top of broken code
4. If stuck after 3 attempts, report to Planning

### GitHub API Fails
1. Check token is correct
2. Check repo name is correct
3. Check branch exists
4. Report error and STOP

### Migration Fails
1. Check SQL syntax
2. Check table/column names
3. Check for conflicts with existing schema
4. Report error and STOP

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────┐
│ SESSION START                                           │
│ 1. Download repo (tarball)                              │
│ 2. npm run build (verify clean)                         │
│ 3. Read SPRINT_N_PLAN.md                                │
│ 4. Create session log                                   │
│ 5. Create branch                                        │
├─────────────────────────────────────────────────────────┤
│ EXECUTION CYCLE                                         │
│ 1. Make change                                          │
│ 2. npm run build (MUST PASS)                            │
│ 3. Log milestone                                        │
│ 4. Push to branch                                       │
│ 5. Verify deployment                                    │
├─────────────────────────────────────────────────────────┤
│ SESSION END                                             │
│ 1. All phases complete                                  │
│ 2. Final build passes                                   │
│ 3. Open PR                                              │
│ 4. Update session log                                   │
│ 5. Report PR URL                                        │
└─────────────────────────────────────────────────────────┘
```

---

**Start by reading the execution plan and execute.**
