# Project Bootstrap Patterns

> Automate new project setup with shared gateways and API tokens.

## Overview

Three of four services can be fully automated. One shared Modal endpoint serves all projects.

| Service | Method | Shared? |
|---------|--------|---------|
| **GitHub** | API with PAT | ✅ One token creates all repos |
| **Supabase** | Management API | ✅ One token creates all projects |
| **Modal** | Shared bootstrap endpoint | ✅ One endpoint runs anything |
| **Vercel** | Dashboard (manual) | ❌ Connect repo once per project |

---

## Credentials (Shared)

```python
# GitHub - creates repos, manages code
GITHUB_PAT = "[YOUR_GITHUB_PAT]"
GITHUB_USER = "[YOUR_GITHUB_USER]"

# Supabase - creates projects, runs migrations
SUPABASE_MGMT_TOKEN = "[YOUR_SUPABASE_MANAGEMENT_TOKEN]"

# Modal - shared bootstrap gateway (no auth needed, URL is the key)
MODAL_BOOTSTRAP_URL = "[YOUR_MODAL_BOOTSTRAP_URL]"
# Example: https://username--appname-function-name.modal.run
```

**Note:** Get these from your project's CURRENT.md or from the respective service dashboards.

---

## 1. GitHub: Create Repository

```python
import requests

GITHUB_PAT = "[YOUR_GITHUB_PAT]"
HEADERS = {"Authorization": f"token {GITHUB_PAT}"}

def create_repo(name, description="", private=False):
    """Create a new GitHub repository."""
    r = requests.post(
        "https://api.github.com/user/repos",
        headers=HEADERS,
        json={
            "name": name,
            "description": description,
            "private": private,
            "auto_init": True
        }
    )
    if r.status_code == 201:
        return r.json()["html_url"]
    return f"Error: {r.status_code} - {r.text}"

# Usage
url = create_repo("my-new-project", "Built with Gibbon")
print(f"Created: {url}")
```

---

## 2. Supabase: Create Project

```python
import requests

SUPABASE_MGMT_TOKEN = "[YOUR_SUPABASE_MANAGEMENT_TOKEN]"

def create_supabase_project(name, org_id, region="us-east-1", db_pass=None):
    """Create a new Supabase project."""
    import secrets
    
    r = requests.post(
        "https://api.supabase.com/v1/projects",
        headers={
            "Authorization": f"Bearer {SUPABASE_MGMT_TOKEN}",
            "Content-Type": "application/json"
        },
        json={
            "name": name,
            "organization_id": org_id,
            "region": region,
            "db_pass": db_pass or secrets.token_urlsafe(16)
        }
    )
    if r.status_code == 201:
        data = r.json()
        return {
            "id": data["id"],
            "url": f"https://{data['id']}.supabase.co",
            "anon_key": data.get("anon_key"),
            "service_role_key": data.get("service_role_key")
        }
    return f"Error: {r.status_code} - {r.text}"

def get_org_id():
    """Get your Supabase organization ID."""
    r = requests.get(
        "https://api.supabase.com/v1/organizations",
        headers={"Authorization": f"Bearer {SUPABASE_MGMT_TOKEN}"}
    )
    return r.json()[0]["id"]  # First org

# Usage
org_id = get_org_id()
project = create_supabase_project("my-new-project", org_id)
print(f"Created: {project}")
```

---

## 3. Modal: Shared Bootstrap Gateway

**No setup required.** One endpoint handles all projects.

### The "Keys to All Keys" Pattern

A single Modal function accepts shell or Python code via HTTP, runs it in a Linux container, and returns results. No Modal keys needed — the URL is the key.

```python
import requests

MODAL_BOOTSTRAP_URL = "[YOUR_MODAL_BOOTSTRAP_URL]"

def run_shell(command):
    """Run shell command in Modal container."""
    r = requests.post(MODAL_BOOTSTRAP_URL, json={
        "type": "shell",
        "command": command
    })
    return r.json()

def run_python(code):
    """Run Python code in Modal container."""
    r = requests.post(MODAL_BOOTSTRAP_URL, json={
        "type": "python",
        "code": code
    })
    return r.json()

# Examples
result = run_shell("echo 'Hello from Modal'")
print(result)  # {"success": true, "stdout": "Hello from Modal\n", "stderr": ""}

result = run_shell("node --version")
print(result)

result = run_python("import sys; print(sys.version)")
print(result)
```

### Install Packages at Runtime

No need to redeploy — install anything on the fly:

```python
# Install Node.js and run npm
run_shell("apt-get update && apt-get install -y nodejs npm")
run_shell("npm --version")

# Install Python packages
run_shell("pip install pandas numpy")
run_python("import pandas as pd; print(pd.__version__)")
```

### Deploy Your Own Bootstrap (One-Time)

Deploy this Modal function once from your local machine:

```python
# bootstrap.py - deploy with: modal deploy bootstrap.py
import modal

app = modal.App("my-bootstrap")

@app.function(image=modal.Image.debian_slim().pip_install("requests"))
@modal.web_endpoint(method="POST")
def run(data: dict):
    import subprocess
    
    if data.get("type") == "shell":
        result = subprocess.run(
            data["command"], 
            shell=True, 
            capture_output=True, 
            text=True
        )
        return {
            "success": result.returncode == 0,
            "stdout": result.stdout,
            "stderr": result.stderr
        }
    
    elif data.get("type") == "python":
        import io, sys
        stdout = io.StringIO()
        sys.stdout = stdout
        try:
            exec(data["code"])
            return {"success": True, "stdout": stdout.getvalue(), "stderr": ""}
        except Exception as e:
            return {"success": False, "stdout": stdout.getvalue(), "stderr": str(e)}
        finally:
            sys.stdout = sys.__stdout__
    
    return {"error": "Invalid type. Use 'shell' or 'python'"}
```

After deploying, you get a URL like: `https://username--my-bootstrap-run.modal.run`

### Why This Works

| Benefit | Explanation |
|---------|-------------|
| **One deploy** | Bootstrap function deployed once, serves forever |
| **HTTP/REST** | Works from Claude (gRPC is blocked) |
| **No auth** | URL is the key |
| **Serverless** | ~$0.0005 per execution |
| **Shared** | Same endpoint for all projects |

---

## 4. Vercel: Manual Setup (One-Time)

Vercel requires manual GitHub integration per repo:

1. Go to https://vercel.com/new
2. Import your GitHub repo
3. Configure build settings (usually auto-detected)
4. Deploy

After initial setup, pushes to main auto-deploy.

### Optional: Vercel API Token

For API access (check deploy status, trigger builds):

1. Go to https://vercel.com/account/tokens
2. Create token
3. Use in API calls:

```python
VERCEL_TOKEN = "[YOUR_VERCEL_TOKEN]"

def get_deployments(project_name):
    r = requests.get(
        f"https://api.vercel.com/v6/deployments?projectId={project_name}",
        headers={"Authorization": f"Bearer {VERCEL_TOKEN}"}
    )
    return r.json()
```

---

## Full Bootstrap Script

```python
import requests
import secrets

# Credentials - fill these in from your CURRENT.md
GITHUB_PAT = "[YOUR_GITHUB_PAT]"
SUPABASE_MGMT_TOKEN = "[YOUR_SUPABASE_MANAGEMENT_TOKEN]"
MODAL_BOOTSTRAP_URL = "[YOUR_MODAL_BOOTSTRAP_URL]"

def bootstrap_project(name, description=""):
    """Create a new project with GitHub + Supabase."""
    
    print(f"🚀 Bootstrapping: {name}")
    
    # 1. Create GitHub repo
    print("  📦 Creating GitHub repo...")
    r = requests.post(
        "https://api.github.com/user/repos",
        headers={"Authorization": f"token {GITHUB_PAT}"},
        json={"name": name, "description": description, "auto_init": True}
    )
    github_url = r.json().get("html_url")
    print(f"     ✅ {github_url}")
    
    # 2. Get Supabase org
    r = requests.get(
        "https://api.supabase.com/v1/organizations",
        headers={"Authorization": f"Bearer {SUPABASE_MGMT_TOKEN}"}
    )
    org_id = r.json()[0]["id"]
    
    # 3. Create Supabase project
    print("  🗄️ Creating Supabase project...")
    r = requests.post(
        "https://api.supabase.com/v1/projects",
        headers={
            "Authorization": f"Bearer {SUPABASE_MGMT_TOKEN}",
            "Content-Type": "application/json"
        },
        json={
            "name": name,
            "organization_id": org_id,
            "region": "us-east-1",
            "db_pass": secrets.token_urlsafe(16)
        }
    )
    supabase_data = r.json()
    supabase_url = f"https://{supabase_data['id']}.supabase.co"
    print(f"     ✅ {supabase_url}")
    
    # 4. Modal - nothing to do
    print("  ⚡ Modal: Using shared bootstrap endpoint")
    print(f"     ✅ {MODAL_BOOTSTRAP_URL}")
    
    # 5. Vercel - manual
    print("  ▲ Vercel: Manual setup required")
    print(f"     → Go to https://vercel.com/new and import {github_url}")
    
    return {
        "github": github_url,
        "supabase_url": supabase_url,
        "supabase_id": supabase_data["id"],
        "modal": MODAL_BOOTSTRAP_URL
    }

# Usage
project = bootstrap_project("my-awesome-app", "Built with Claude")
print(f"\n✅ Project ready: {project}")
```

---

## Output: CURRENT.md Credentials Block

After bootstrapping, add to your project's CURRENT.md:

```markdown
## Credentials & Endpoints

### GitHub
| Key | Value |
|-----|-------|
| Repo | `[owner]/[project-name]` |
| PAT | `[your-pat]` |

### Supabase
| Key | Value |
|-----|-------|
| URL | `https://[project-id].supabase.co` |
| Project ID | `[project-id]` |
| Service Role Key | `[from dashboard]` |
| Management Token | `[your-token]` |

### Modal
| Key | Value |
|-----|-------|
| Bootstrap | `[your-bootstrap-url]` |

### Vercel
| Key | Value |
|-----|-------|
| URL | `https://[project].vercel.app` |
```

---

## Notes

### Why Modal Bootstrap Works
- **gRPC is blocked** in Claude's environment (can't run `modal deploy`)
- **HTTPS works** (can call deployed endpoints)
- **Solution:** Deploy bootstrap once from local machine, call forever via HTTP

### Security Considerations
- The Modal endpoint URL acts as a bearer token — don't share publicly
- GitHub PAT and Supabase token have broad permissions — keep secure
- For production, consider scoped tokens with minimal permissions

### Cost
- **GitHub:** Free for public repos
- **Supabase:** Free tier available
- **Modal:** ~$0.0005 per bootstrap call (serverless)
- **Vercel:** Free tier available
