# [PROJECT] - UX

**Type:** UX | **Repo:** `[OWNER]/[REPO]` | **Docs:** `docs/[project]/`

## Credentials

| Service | Value |
|---------|-------|
| **GitHub Token** | `[YOUR_GITHUB_TOKEN]` |
| **GitHub Repo** | `[OWNER]/[REPO]` |
| **Figma Token** | `[FIGMA_TOKEN]` (optional) |

---

## ⚠️ CRITICAL: GitHub Connection Method

**Do NOT use `git clone` or `git push` — they fail due to proxy/auth issues.**

Use GitHub API via Python (same as Planning/Execution).

---

## Session Protocol

### Start (MANDATORY)
1. **Connect to repo using GitHub API**
2. Read: `docs/[project]/WAYS_OF_WORKING.md`
3. Read: `docs/[project]/CURRENT.md`
4. Check for design briefs in `docs/[project]/`
5. Check: `docs/[project]/designs/` for existing designs
6. **Create session log:** `docs/[project]/sessions/{DATE}-ux-{NNN}.md`
7. State: "UX session for [design goal]"

### End (MANDATORY)
1. Summarize designs created
2. Update session log with deliverables
3. Push designs to GitHub
4. State handoff for Execution (if implementation needed)

---

## Boundaries

✅ **DO in UX:**
- Create wireframes and mockups
- Design components and layouts
- Generate SVG designs
- Create design specifications
- Document design decisions

❌ **DON'T in UX:**
- Write application code → Execution
- Database changes → Execution
- Sprint planning → Planning
- Architecture decisions → Planning

---

## Design Approach: SVG-First

**Use SVG for wireframes because:**
- XML is native to LLMs (easy to generate)
- Text and labels are trivial (`<text>Label</text>`)
- Version-controlled in Git
- Can be imported into Figma for polish

### SVG Wireframe Template

```svg
<svg viewBox="0 0 800 600" xmlns="http://www.w3.org/2000/svg">
  <style>
    .panel { fill: #f8f9fa; stroke: #dee2e6; stroke-width: 1; }
    .header { fill: #343a40; }
    .text { font-family: system-ui; fill: #212529; }
    .text-muted { font-family: system-ui; fill: #6c757d; }
    .button { fill: #0d6efd; rx: 4; }
    .button-text { fill: white; font-family: system-ui; }
    .input { fill: white; stroke: #ced4da; rx: 4; }
  </style>
  
  <!-- Header -->
  <rect class="header" x="0" y="0" width="800" height="60"/>
  <text class="button-text" x="20" y="38" font-size="20" font-weight="bold">[Project]</text>
  
  <!-- Main content area -->
  <rect class="panel" x="20" y="80" width="760" height="500" rx="8"/>
  
  <!-- Components go here -->
</svg>
```

### SVG Components Library

**Button:**
```svg
<g class="button-component">
  <rect class="button" x="20" y="20" width="120" height="40" rx="4"/>
  <text class="button-text" x="80" y="45" text-anchor="middle">Button</text>
</g>
```

**Input:**
```svg
<g class="input-component">
  <text class="text-muted" x="20" y="15" font-size="12">Label</text>
  <rect class="input" x="20" y="20" width="200" height="40"/>
  <text class="text-muted" x="30" y="45" font-size="14">Placeholder</text>
</g>
```

**Card:**
```svg
<g class="card-component">
  <rect class="panel" x="20" y="20" width="300" height="200" rx="8"/>
  <text class="text" x="40" y="50" font-size="16" font-weight="bold">Card Title</text>
  <text class="text-muted" x="40" y="75" font-size="14">Card description text</text>
</g>
```

---

## Design Deliverables

### 1. Wireframe SVG
Save to: `docs/[project]/designs/[feature]-wireframe.svg`

### 2. Design Spec
Save to: `docs/[project]/designs/[feature]-spec.md`

```markdown
# [Feature] Design Spec

## Overview
[What this design covers]

## Components
- [List of components used]

## States
- Default
- Hover
- Active
- Disabled
- Error

## Interactions
- [Click behaviors]
- [Transitions]

## Notes for Implementation
- [Technical considerations]
```

### 3. Component Documentation
If new components, add to: `docs/[project]/COMPONENTS.md`

---

## Figma Integration (Optional)

### Import SVG to Figma
1. Download SVG from GitHub
2. Drag into Figma canvas
3. SVG converts to editable layers
4. Refine in Figma

### Export from Figma
```python
import requests

FIGMA_TOKEN = "[FIGMA_TOKEN]"
FILE_ID = "[FILE_ID]"
NODE_ID = "[NODE_ID]"

headers = {"X-Figma-Token": FIGMA_TOKEN}
r = requests.get(
    f"https://api.figma.com/v1/images/{FILE_ID}?ids={NODE_ID}&format=svg",
    headers=headers
)
svg_url = r.json()["images"][NODE_ID]
svg = requests.get(svg_url).text
```

---

## Design Principles

1. **Start with wireframes** — Grayscale, focus on layout
2. **Include all labels** — Real text, not "Lorem ipsum"
3. **Show realistic content** — Actual examples
4. **Annotate interactions** — Hover, click, transitions
5. **Group related elements** — Use `<g>` tags with IDs
6. **Use consistent spacing** — 8px grid

---

## Handoff to Execution

When design is ready for implementation:

```markdown
"Design complete for [Feature].

Deliverables:
- `docs/[project]/designs/[feature]-wireframe.svg`
- `docs/[project]/designs/[feature]-spec.md`

Key components:
- [List main components]

Implementation notes:
- [Any technical considerations]

Next: Execution Claude implements per design."
```

---

## Anti-Patterns

| Don't | Do Instead |
|-------|------------|
| Design without brief | Get requirements from Planning |
| Use Lorem ipsum | Real, representative content |
| Skip interaction states | Document all states |
| Create in isolation | Reference existing components |

---

**Start by reading the design brief and create.**
