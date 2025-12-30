# Supabase Migration Patterns

> Run database migrations directly via API. Don't leave manual steps.

## Why API Migrations?

Execution Claude has credentials and should run migrations directly. Leaving migrations as "manual steps" creates friction and risks forgotten migrations.

---

## Setup

```python
import requests

SUPABASE_PROJECT_ID = "your-project-id"  # From CURRENT.md
MANAGEMENT_TOKEN = "sbp_..."  # From CURRENT.md

headers = {
    "Authorization": f"Bearer {MANAGEMENT_TOKEN}",
    "Content-Type": "application/json"
}
```

---

## Run Migration

```python
def run_migration(sql):
    """Execute SQL against the database."""
    r = requests.post(
        f"https://api.supabase.com/v1/projects/{SUPABASE_PROJECT_ID}/database/query",
        headers=headers,
        json={"query": sql},
        verify=False  # Required due to proxy in some environments
    )
    return r.status_code, r.json()

# Example
status, result = run_migration("""
    ALTER TABLE users ADD COLUMN new_field TEXT;
""")
print(f"Status: {status}")  # 201 = success
```

---

## Migration Workflow

### 1. Write Migration File
```sql
-- supabase/migrations/20250101_description.sql

-- Description: Add new_field to users table
-- Sprint: 7
-- Author: Execution Claude

ALTER TABLE users ADD COLUMN new_field TEXT;

-- Verify: SELECT column_name FROM information_schema.columns 
--         WHERE table_name = 'users' AND column_name = 'new_field';
```

### 2. Run via API
```python
with open("supabase/migrations/20250101_description.sql") as f:
    sql = f.read()
status, result = run_migration(sql)
```

### 3. Verify
```python
verify_sql = """
SELECT column_name FROM information_schema.columns 
WHERE table_name = 'users' AND column_name = 'new_field';
"""
status, result = run_migration(verify_sql)
print(result)  # Should show the column
```

### 4. Commit Migration File
Migration file tracked in git for reproducibility.

### 5. Log Milestone
```markdown
### [HH:MM] Milestone: Migration applied
- **What:** Added new_field to users table
- **SQL:** supabase/migrations/20250101_description.sql
- **Status:** ✅ Verified via information_schema
- **Next:** [Next step]
```

---

## Common Migrations

### Add Column
```sql
ALTER TABLE table_name ADD COLUMN column_name TYPE;
ALTER TABLE table_name ADD COLUMN column_name TYPE DEFAULT 'value';
ALTER TABLE table_name ADD COLUMN column_name TYPE NOT NULL DEFAULT 'value';
```

### Add Index
```sql
CREATE INDEX idx_table_column ON table_name(column_name);
CREATE INDEX idx_table_columns ON table_name(col1, col2);
```

### Create Table
```sql
CREATE TABLE IF NOT EXISTS table_name (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  field1 TEXT NOT NULL,
  field2 INTEGER,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### Add Foreign Key
```sql
ALTER TABLE child_table 
ADD CONSTRAINT fk_parent 
FOREIGN KEY (parent_id) REFERENCES parent_table(id);
```

### Add RLS Policy
```sql
ALTER TABLE table_name ENABLE ROW LEVEL SECURITY;

CREATE POLICY "policy_name" ON table_name
  FOR SELECT USING (condition);
```

---

## Verification Queries

### Check Table Exists
```sql
SELECT table_name FROM information_schema.tables 
WHERE table_schema = 'public' AND table_name = 'your_table';
```

### Check Column Exists
```sql
SELECT column_name, data_type FROM information_schema.columns 
WHERE table_name = 'your_table' AND column_name = 'your_column';
```

### Check Index Exists
```sql
SELECT indexname FROM pg_indexes 
WHERE tablename = 'your_table' AND indexname = 'idx_name';
```

### Check Foreign Key Exists
```sql
SELECT constraint_name FROM information_schema.table_constraints 
WHERE table_name = 'your_table' AND constraint_type = 'FOREIGN KEY';
```

---

## Rollback Patterns

Always have rollback ready before running migration:

```sql
-- Migration
ALTER TABLE users ADD COLUMN new_field TEXT;

-- Rollback
ALTER TABLE users DROP COLUMN new_field;
```

```sql
-- Migration
CREATE TABLE new_table (...);

-- Rollback
DROP TABLE IF EXISTS new_table;
```

---

## Error Handling

| Error | Cause | Fix |
|-------|-------|-----|
| `relation already exists` | Table/index already created | Skip or use IF NOT EXISTS |
| `column already exists` | Column already added | Skip or use IF NOT EXISTS |
| `violates foreign key` | Data integrity issue | Fix data first |
| `permission denied` | Wrong credentials | Check MANAGEMENT_TOKEN |

---

## Best Practices

1. **Use IF NOT EXISTS** — Makes migrations idempotent
2. **Include verification query** — Confirm migration worked
3. **Write rollback first** — Know how to undo
4. **One migration per concept** — Don't bundle unrelated changes
5. **Track in git** — Commit migration files
6. **Log milestone** — Document what was applied
