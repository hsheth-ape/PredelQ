# PredelQ - Current State

> Last Updated: 2025-12-30

## Project Status

| Aspect | Status |
|--------|--------|
| **Phase** | Bootstrap |
| **Sprint** | 0 - Setup |
| **Blocker** | None |

---

## Credentials & Endpoints

> **Note:** Actual secrets are stored in GitHub Secrets or provided at session start.
> GitHub PAT and Supabase keys should be provided when starting a Claude session.

### GitHub
| Key | Value |
|-----|-------|
| Repo | `hsheth-ape/PredelQ` |
| Username | `hsheth-ape` |
| PAT | *Provide at session start* |

### Supabase
| Key | Value |
|-----|-------|
| Management API Key | *Stored in GitHub Secrets: `SUPABASE_MANAGEMENT_API_KEY`* |
| Project URL | *Not created yet* |
| Project ID | *Not created yet* |

### Modal
| Key | Value |
|-----|-------|
| Bootstrap Gateway | `https://hsheth-ape--gibbon-bootstrap-run.modal.run` |

### Vercel
| Key | Value |
|-----|-------|
| URL | *Not deployed yet* |

---

## What Exists

### Infrastructure
- [x] GitHub repo created
- [x] GitHub secrets configured (SUPABASE_MANAGEMENT_API_KEY, MODAL_GATEWAY_URL)
- [ ] Supabase project
- [ ] Vercel deployment
- [ ] Domain configured

### Code
- [ ] Frontend scaffolding
- [ ] Backend API
- [ ] Database schema

---

## What\'s Next

1. Define what PredelQ is (requirements gathering)
2. Create Supabase project
3. Design initial schema
4. Set up Vercel deployment

---

## Open Questions

- What problem does PredelQ solve?
- Who is the target user?
- What\'s the MVP scope?
