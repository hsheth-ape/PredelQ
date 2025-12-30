# Sprint [N] Execution Plan: [Title]

> **Created:** [DATE]
> **Status:** Ready for Execution
> **Branch:** `sprint-[N]/[feature-name]`
> **Estimated:** ~[X] hours

---

## Pre-Execution Checklist

- [ ] Read `docs/[project]/WAYS_OF_WORKING.md`
- [ ] Read `docs/[project]/LESSONS.md`
- [ ] Read `docs/[project]/SPRINT_[N]_INVENTORY.md` (design reference)
- [ ] Verify on `main` branch
- [ ] Run `npm run build` - must pass
- [ ] Create branch: `git checkout -b sprint-[N]/[feature-name]`
- [ ] Create session log: `docs/[project]/sessions/[DATE]-execution-sprint[N].md`

---

## Phase 1: [Phase Name] ([X]h)

**Goal:** [One sentence describing phase outcome]

### Step 1.1: [Step Name]

**File:** `[path/to/file.ext]`

```[language]
[COMPLETE CODE - not a sketch]
[This should be copy-paste ready]
[Include all imports, types, implementations]
```

**Apply/Verify:**
```bash
# For migrations
python3 << 'PYEOF'
import requests
SUPABASE_PROJECT_ID = "[from CURRENT.md]"
MANAGEMENT_TOKEN = "[from CURRENT.md]"
headers = {"Authorization": f"Bearer {MANAGEMENT_TOKEN}", "Content-Type": "application/json"}
sql = """[THE SQL FROM ABOVE]"""
r = requests.post(f"https://api.supabase.com/v1/projects/{SUPABASE_PROJECT_ID}/database/query",
    headers=headers, json={"query": sql}, verify=False)
print(f"Status: {r.status_code}")
PYEOF

# Verify
SELECT table_name FROM information_schema.tables WHERE table_name = '[table]';
```

**Build:** `npm run build` - must pass

---

### Step 1.2: [Step Name]

**File:** `[path/to/file.ext]`

```[language]
[COMPLETE CODE]
```

**Build:** `npm run build` - must pass

---

## Phase 2: [Phase Name] ([X]h)

**Goal:** [One sentence]

### Step 2.1: [Step Name]

[Continue pattern...]

---

## Phase 3: [Phase Name] ([X]h)

[Continue pattern...]

---

## Final Verification

### Build Checklist

- [ ] All migrations applied
- [ ] `npm run build` passes
- [ ] Vercel deployment succeeds
- [ ] No TypeScript errors
- [ ] No console errors in browser

### Functional Checklist

- [ ] [Feature 1] works as expected
- [ ] [Feature 2] works as expected
- [ ] [Edge case] handled correctly

### Test Commands

```sql
-- Verify database state
SELECT * FROM [table] LIMIT 5;

-- Check indexes exist
SELECT indexname FROM pg_indexes WHERE tablename = '[table]';
```

```bash
# Test API endpoint
curl -X GET "https://[url]/api/[endpoint]" -H "Authorization: Bearer [token]"
```

---

## PR Checklist

- [ ] All phases complete
- [ ] Build passes
- [ ] Deployment successful
- [ ] Session log updated
- [ ] Create PR: `sprint-[N]/[feature]` → `main`

**PR Template:**

```markdown
## Sprint [N]: [Title]

### Summary
[What this PR accomplishes]

### Changes
- [Change 1]
- [Change 2]
- [Change 3]

### Database
- [x] Migration: `[migration_file.sql]`
- [x] Applied via Supabase API

### Testing
- [x] Build passes
- [x] [Feature 1] verified
- [x] [Feature 2] verified

### Screenshots
[If UI changes]
```

---

## Rollback Plan

If critical issues found after merge:

```bash
# Revert commit
git revert [commit-sha]

# Or reset to tag
git reset --hard v[previous-version]
```

```sql
-- Rollback migration (if needed)
DROP TABLE IF EXISTS [new_table];
ALTER TABLE [table] DROP COLUMN IF EXISTS [new_column];
```

---

*Ready for execution.*
