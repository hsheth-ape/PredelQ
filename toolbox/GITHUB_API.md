# GitHub API Patterns

> Reliable patterns for GitHub operations when git CLI is blocked.

## Why GitHub API?

In some environments (like Claude's), `git clone` and `git push` fail due to proxy/auth issues. The GitHub REST API works reliably via HTTPS.

---

## Setup

```python
import requests
import base64
import json

TOKEN = "your_github_pat"
REPO = "owner/repo"
HEADERS = {"Authorization": f"token {TOKEN}"}
```

---

## Read Operations

### Read File
```python
def read_file(path, branch="main"):
    url = f"https://api.github.com/repos/{REPO}/contents/{path}?ref={branch}"
    r = requests.get(url, headers=HEADERS)
    if r.status_code == 200:
        return base64.b64decode(r.json()["content"]).decode()
    return f"Error: {r.status_code} - {r.text}"
```

### List Directory
```python
def list_dir(path="", branch="main"):
    url = f"https://api.github.com/repos/{REPO}/contents/{path}?ref={branch}"
    r = requests.get(url, headers=HEADERS)
    if r.status_code == 200:
        return [f["name"] for f in r.json()]
    return f"Error: {r.status_code}"
```

### Download Repo (Tarball)
```bash
curl -L -H "Authorization: token $TOKEN" \
  "https://api.github.com/repos/$REPO/tarball/main" -o repo.tar.gz
tar -xzf repo.tar.gz
```

---

## Write Operations

### Create/Update File
```python
def write_file(path, content, message, branch="main"):
    url = f"https://api.github.com/repos/{REPO}/contents/{path}"
    
    # Get current SHA if file exists
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
```

### Delete File
```python
def delete_file(path, message, branch="main"):
    url = f"https://api.github.com/repos/{REPO}/contents/{path}"
    r = requests.get(url + f"?ref={branch}", headers=HEADERS)
    sha = r.json().get("sha")
    
    return requests.delete(url, headers=HEADERS, json={
        "message": message,
        "sha": sha,
        "branch": branch
    })
```

---

## Branch Operations

### Create Branch
```python
def create_branch(name, from_branch="main"):
    # Get SHA of source branch
    r = requests.get(
        f"https://api.github.com/repos/{REPO}/git/ref/heads/{from_branch}",
        headers=HEADERS
    )
    sha = r.json()["object"]["sha"]
    
    # Create new branch
    return requests.post(
        f"https://api.github.com/repos/{REPO}/git/refs",
        headers=HEADERS,
        json={"ref": f"refs/heads/{name}", "sha": sha}
    )
```

### List Branches
```python
def list_branches():
    r = requests.get(
        f"https://api.github.com/repos/{REPO}/branches",
        headers=HEADERS
    )
    return [b["name"] for b in r.json()]
```

---

## Pull Request Operations

### Create PR
```python
def create_pr(title, body, head, base="main"):
    return requests.post(
        f"https://api.github.com/repos/{REPO}/pulls",
        headers=HEADERS,
        json={
            "title": title,
            "body": body,
            "head": head,
            "base": base
        }
    )
```

### Get PR
```python
def get_pr(pr_number):
    r = requests.get(
        f"https://api.github.com/repos/{REPO}/pulls/{pr_number}",
        headers=HEADERS
    )
    return r.json()
```

### Merge PR
```python
def merge_pr(pr_number, merge_method="squash"):
    return requests.put(
        f"https://api.github.com/repos/{REPO}/pulls/{pr_number}/merge",
        headers=HEADERS,
        json={"merge_method": merge_method}
    )
```

---

## Common Issues

### 404 on Write
- Check branch exists
- Check file path is correct (case-sensitive)
- Verify token has write permissions

### 409 Conflict
- File was modified since you read it
- Re-read to get current SHA, then retry

### 422 Unprocessable
- Check content is valid base64
- Check commit message is not empty
- Check branch name is valid

---

## Best Practices

1. **Always get SHA before update** — Prevents conflicts
2. **Use branches for code** — Protect main
3. **Verify after write** — Read back to confirm
4. **Handle errors** — Check status codes
