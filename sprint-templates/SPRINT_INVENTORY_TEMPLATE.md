# Sprint [N] Inventory: [Title]

> **Status:** Discovery | Planning | Ready | In Progress | Complete
> **Scope:** [One-line description]
> **Created:** [DATE]
> **Est. Effort:** ~[X]h

---

## Executive Summary

[2-3 sentences describing what this sprint accomplishes]

| Component | What It Does | Effort |
|-----------|--------------|--------|
| [Component 1] | [Description] | Xh |
| [Component 2] | [Description] | Xh |
| **Total** | | **~Xh** |

---

## Part 1: [Component Name]

### The Problem
[What gap or issue this addresses]

### Solution
[High-level approach]

### Schema Design

```sql
CREATE TABLE [table_name] (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  
  -- [Group 1]
  field1 TEXT NOT NULL,
  field2 INTEGER,
  
  -- [Group 2]
  field3 JSONB DEFAULT '{}',
  
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_[table]_[field] ON [table]([field]);
```

### Code Sketch

```typescript
// src/lib/[module]/service.ts

export interface [Type] {
  id: string;
  field1: string;
  // ...
}

export async function [functionName](params: {
  // ...
}): Promise<[Type]> {
  // Implementation sketch
  // NOT complete code - just structure
}
```

### Effort: Xh

| Task | Hours |
|------|-------|
| Database migration | 1h |
| Service implementation | 2h |
| API endpoint | 1h |
| Testing | 1h |

---

## Part 2: [Component Name]

[Repeat structure from Part 1]

---

## Sprint Phases

### Phase 1: [Name] (Xh)
- [ ] Task 1
- [ ] Task 2
- [ ] Task 3

### Phase 2: [Name] (Xh)
- [ ] Task 1
- [ ] Task 2

[Continue for all phases]

**Total: ~Xh**

---

## Success Criteria

After Sprint [N]:

1. **[Criterion 1]** — [Measurable outcome]
2. **[Criterion 2]** — [Measurable outcome]
3. **[Criterion 3]** — [Measurable outcome]

---

## Dependencies

| Dependency | Status |
|------------|--------|
| [Dependency 1] | ✅ Ready | 🟡 In Progress | ❌ Blocked |
| [Dependency 2] | ✅ | 🟡 | ❌ |

---

## Open Questions

| Question | Status | Decision |
|----------|--------|----------|
| [Question 1] | Open | |
| [Question 2] | Resolved | [Decision] |

---

## Risks

| Risk | Mitigation |
|------|------------|
| [Risk 1] | [Mitigation strategy] |
| [Risk 2] | [Mitigation strategy] |

---

## Out of Scope

- [Item 1] — Deferred to Sprint [N+1]
- [Item 2] — Not needed for MVP

---

*Ready for review. After approval, Planning Claude creates SPRINT_[N]_PLAN.md.*
