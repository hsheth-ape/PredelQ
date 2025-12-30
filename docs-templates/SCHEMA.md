# [Project] - Database Schema

> **Last Updated:** [DATE]
> **Database:** Supabase PostgreSQL

---

## Tables

### [table_name]

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | NO | gen_random_uuid() | Primary key |
| [column] | [TYPE] | [YES/NO] | [default] | [Description] |
| created_at | TIMESTAMPTZ | NO | NOW() | Creation timestamp |

**Indexes:**
- `idx_[table]_[column]` on [column]

**Foreign Keys:**
- `[column]` → `[other_table](id)`

**RLS:** Enabled
- SELECT: [policy description]
- INSERT: [policy description]

---

## Enums

### [enum_name]
```sql
CREATE TYPE [enum_name] AS ENUM ('value1', 'value2', 'value3');
```

---

## Functions

### [function_name]
```sql
CREATE FUNCTION [function_name]([params]) RETURNS [type] AS $$
  -- implementation
$$ LANGUAGE sql;
```

---

## Migration History

| Date | Migration | Description |
|------|-----------|-------------|
| [DATE] | 20250101_initial.sql | Initial schema |
| [DATE] | 20250102_add_field.sql | Added [field] to [table] |
