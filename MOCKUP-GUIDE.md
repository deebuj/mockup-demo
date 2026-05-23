# From Idea to Live Page: Creating Mockups and Deploying to GitHub Pages

> **Presentation guide — ~30 minutes**
> Target audience: developers and non-developers who need to turn ideas into shareable, interactive prototypes quickly.

---

## Agenda

| # | Topic | Time |
|---|-------|------|
| 1 | Why mockups matter | 2 min |
| 2 | Mockup spectrum: sketches → production | 3 min |
| 3 | Starting with Claude Design (claude.ai/design) | 5 min |
| 4 | Graduating to a React app (Vite + pnpm) | 8 min |
| 5 | Hosting on GitHub Pages — two approaches | 7 min |
| 6 | Best practices and lessons learned | 5 min |

---

## 1 — Why Mockups Matter

A mockup replaces 10 meetings.

- Stakeholders can **see** what they're approving instead of imagining it
- Developers get concrete requirements before writing production code
- Feedback loops shrink from weeks to hours
- Cheap to throw away — real code is not

**Real example:** A data management tool started as a conversation. Attaching a screenshot of the existing spreadsheet + a short description of pain points produced a working prototype in one session.

---

## 2 — The Mockup Spectrum

```
Napkin sketch  →  Wireframe  →  Static HTML  →  React App  →  Production
     |                |               |               |               |
  5 minutes       30 minutes       1–2 hours      half-day        weeks
  No fidelity    Low fidelity    Medium fidelity  High fidelity   Shipped
```

Choose the **lowest fidelity that answers your question.**

- Validating layout? → Wireframe or static HTML
- Validating interactions (filter, sort, expand)? → React app
- Validating real data? → Wire in a real API

---

## 3 — Starting with Claude Design

**URL:** https://claude.ai/design

Released April 17, 2026 as a research preview (Anthropic Labs). Available to **Pro, Max, Team, and Enterprise** subscribers. Powered by Claude Opus 4.7.

### What Claude Design is

Claude Design is a dedicated visual design tool built into claude.ai. It is not just a chat that writes code — it gives you an interactive canvas where you describe what you want and refine the output through:

- **Conversation** — type follow-up changes just like a chat
- **Inline comments** — click directly on the canvas, annotate a specific element, and Claude applies the change
- **Custom "tweak" sliders** — Claude generates controls specific to your design (e.g., a "compactness" slider for a table layout)
- **Direct edits** — click and type to change text in place
- **Design system awareness** — if your team uploads a design system, every project auto-applies your brand colors, fonts, and components

### What it can produce

| Output | Use case |
|--------|----------|
| Interactive prototypes | Share a link; stakeholders can click around without any code review |
| Wireframes and mockups | PM sketches out a feature; designer refines it |
| Slide decks | Generates on-brand presentations (export as PPTX, PDF, or HTML) |
| One-pagers / microsites | Single-page landing pages or internal docs |

### Input options

- **Text prompt** — describe what you want from scratch
- **Upload files** — DOCX, PPTX, XLSX (paste your spreadsheet to define columns and sample data)
- **Screenshot / image** — upload a screenshot of the existing tool you want to replace; Claude matches the style
- **Web capture** — grab elements directly from your live website so the mockup looks like your real product
- **Point at your codebase** — Claude reads existing components and applies consistent patterns

### Handoff to Claude Code (the bridge to a real app)

When the design is approved, Claude Design packages a **handoff bundle** that you pass to Claude Code (or Cursor) with a single instruction. This is the workflow we used:

```
claude.ai/design  →  "looks good, build this"  →  Cursor / Claude Code  →  React app  →  GitHub Pages
```

### Effective prompt structure for Claude Design

```
Context:     What problem does this tool solve? Who are the users?
Data:        Upload your spreadsheet (XLSX) or paste sample rows
Columns:     List the fields explicitly — AI guesses otherwise
Actions:     Filter, sort, inline edit, export CSV, expand row
Constraints: "No pop-ups", "spreadsheet feel", "green/red status pills"
Reference:   Attach a screenshot of the current tool for style matching
```

### Example prompt

> "Create a mockup of a tool that lets team members manage [entity] records.
> They track [status field] with date and comments.
> There will be 1000s of records — need pagination and search by [key fields].
> The edit should happen inline — no pop-ups.
> [attached: screenshot of the existing spreadsheet]"

### Claude Design vs. Figma

Claude Design is fast for exploration; Figma is better for precision production work. Use them together:

- **Claude Design** → rapid first draft, stakeholder feedback, interactive prototype
- **Figma** → responsive constraints, component libraries, pixel-perfect handoff to a design team
- **Claude Code / Cursor** → production React app from either source

### Tips

- Upload the actual spreadsheet (XLSX) — real column names produce realistic output immediately
- Say "no pop-ups" or "no modals" explicitly; AI defaults to modals for edit flows
- Use inline comments instead of re-prompting for small tweaks — it's faster and more precise
- Ask for one feature at a time after the first scaffold; large prompts produce large diffs that are hard to review
- Export as HTML and open locally to verify before moving to a React app — sometimes the HTML is good enough to ship

---

## 4 — Graduating to a React App

Static HTML is fine for layout review. Switch to React when you need:

- Sorting / filtering / pagination with real state
- Expandable rows, modals, form validation
- Something you can share and have others **actually use**

### Bootstrap with Vite + pnpm

```bash
# Create the app
pnpm create vite@latest frontend -- --template react-ts
cd frontend
pnpm install

# Run locally
pnpm dev          # http://localhost:5173
pnpm build        # produces dist/
pnpm preview      # serves dist/ locally
```

### Recommended minimal stack for a mockup

| Concern | Package | Why |
|---------|---------|-----|
| Build | Vite | Fast, zero-config |
| Language | TypeScript (strict) | Catches errors early; AI output is type-safe |
| Styling | Plain CSS (custom properties) | No build complexity; easy to hand off |
| State | React `useState` / `useMemo` | No external library needed for a POC |
| Validation | Zod (optional) | If connecting to real APIs |

Avoid adding Redux, React Query, or a component library for a mockup — they slow iteration.

### Project structure that works

```
frontend/
├── src/
│   ├── App.tsx          ← all UI logic for a small POC; split later
│   ├── types.ts         ← shared TypeScript types
│   ├── index.css        ← global styles with CSS custom properties
│   ├── data/
│   │   ├── sampleData.ts       ← realistic fake data
│   │   ├── enrichData.ts       ← seeds related records
│   │   └── lookups.ts          ← static lookup lists
│   └── utils/
│       └── helpers.ts          ← pure helper functions
├── index.html
├── vite.config.ts
├── package.json
└── pnpm-lock.yaml
```

### Realistic sample data is non-negotiable

Stakeholders make bad decisions on "lorem ipsum" data. Invest 30 minutes seeding data that looks real:

- Use real city names, realistic email patterns, plausible dates
- Include edge cases: empty fields, long text, max-length strings
- Include both happy-path and failure cases so filters are testable

### Iteration workflow with AI

1. **Describe the change** in plain English (attach a screenshot of what you see vs. what you want)
2. **Review the diff** — don't apply blindly; AI occasionally removes things
3. **Check for TypeScript errors** (`pnpm build`) before moving to the next change
4. **Commit working states** often — easy rollback, shows progress

---

## 5 — Hosting on GitHub Pages

GitHub Pages is free for public repos. Two approaches exist; know which one applies.

---

### Approach A — Legacy (static files from a branch)

Best for: plain HTML/CSS files, no build step.

**Setup:**
1. Push your `index.html` to the `main` branch inside a `/docs` folder
2. Go to **Settings → Pages → Source** → select `main` branch, `/docs` folder
3. GitHub serves it at `https://<org>.github.io/<repo>/`

**Limitations:**
- No build step — raw files only
- All assets must use relative paths
- One site per repo (no subdirectory routing without manual effort)

**When to use:** Quick one-page HTML mockup, no JavaScript framework.

---

### Approach B — GitHub Actions (built artifact) ← What we did

Best for: React / Vite apps, any tool with a build step.

**The key insight:** GitHub Actions builds the app and uploads an artifact; the Pages service deploys *that artifact*, not files from a branch.

**Step 1 — Tell GitHub Pages to use Actions as the source**

Go to **Settings → Pages → Source** → select **"GitHub Actions"**

> ⚠️ If this is set to "Deploy from a branch", the workflow will run but nothing will appear — the deployment artifact is silently ignored.

**Step 2 — Set the Vite base path**

GitHub Pages serves project repos at `https://<org>.github.io/<repo>/`. Vite must know this or asset URLs will 404.

```ts
// vite.config.ts
export default defineConfig({
  plugins: [react()],
  base: '/<repo>/my-app/',   // replace with your repo and app path
})
```

**Step 3 — Structure the artifact to match the URL**

If you want the app at `.../my-app/`, the artifact must have a `my-app/` folder containing the built files. The artifact root maps to the Pages root.

**Step 4 — The workflow**

```yaml
# .github/workflows/deploy-my-app.yml
name: Deploy My App

on:
  push:
    branches: [main]
    paths:
      - 'my-app/frontend/**'
  workflow_dispatch:          # manual trigger button in GitHub UI

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4
        with:
          version: 9

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: pnpm
          cache-dependency-path: my-app/frontend/pnpm-lock.yaml

      - name: Install dependencies
        working-directory: my-app/frontend
        run: pnpm install --frozen-lockfile

      - name: Build
        working-directory: my-app/frontend
        run: pnpm run build

      # Wrap dist/ inside a subdirectory matching the URL path
      - name: Stage for Pages
        run: |
          mkdir -p _pages/my-app
          cp -r my-app/frontend/dist/. _pages/my-app/

      - uses: actions/upload-pages-artifact@v3
        with:
          path: _pages          # ← artifact root = Pages root

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

**Trigger it manually** (useful when only the workflow file changed):
```bash
gh workflow run deploy-my-app.yml --ref main
```

**Check run status:**
```bash
gh run list --workflow=deploy-my-app.yml --limit 5
```

---

### Troubleshooting GitHub Pages 404s (checklist)

| Symptom | Likely cause | Fix |
|---------|-------------|-----|
| Site loads but assets 404 | Wrong `base` in `vite.config.ts` | Set `base` to `/<repo>/<subpath>/` |
| Workflow succeeds but site 404s | Pages source still set to "branch" | Settings → Pages → Source → "GitHub Actions" |
| Sub-path 404s | Artifact root doesn't match URL | Wrap `dist/` in a matching subdirectory |
| Old content still showing | CDN propagation delay | Wait 1–2 minutes and hard-refresh |
| Workflow never triggers on push | `paths:` filter doesn't match changed file | Add the workflow file itself to `paths:` or use `workflow_dispatch` |

---

## 6 — Best Practices

### Design

- **Start ugly, get feedback early.** A polished mockup discourages honest feedback ("you've already done so much work").
- **Use real column names and realistic values** from day one — stakeholders anchor on what they see.
- **Show edge cases:** empty states ("No results found"), long text truncation, max-length inputs.
- **Match the mental model of users**, not yours. If they use Excel, make it look like a table.
- **One scroll, one page.** Avoid tabs and multi-page flows in a mockup unless the workflow explicitly requires it.

### Iteration

- **Commit after each working state.** `git commit -m "add venue filter"` takes 10 seconds and saves hours.
- **pnpm build before every share.** TypeScript errors caught locally don't embarrass you in a demo.
- **Don't refactor during mockup.** Messy `App.tsx` is fine. Clean it when it becomes a real app.
- **Screenshot every major milestone.** Stakeholders forget what they asked for; screenshots don't.

### Prompting AI effectively

- **One change per prompt.** "Add filter AND sort AND fix the modal" produces tangled diffs.
- **Attach a screenshot** when describing a visual problem — a picture is worth 500 tokens.
- **Specify what NOT to do.** "Don't change the CSS, only add the new column" prevents regressions.
- **Always run `pnpm build`** after AI changes — TypeScript catches what looks correct but isn't.

### GitHub Pages

- **Lock dependencies with a lockfile** (`pnpm-lock.yaml`). `--frozen-lockfile` in CI ensures what runs locally matches what deploys.
- **Use `workflow_dispatch`** on every deploy workflow — a manual trigger button is invaluable for redeployments.
- **`paths:` filters save CI minutes.** Only rebuild the app when its source files change.
- **Don't put secrets in the repo.** GitHub Pages is public; use `sample.env` with placeholder values.

### Handoff to production

When a mockup gets approved, the transition to production needs:
- Replace sample/mock data with a real API call
- Add authentication (the mockup has none)
- Replace `crypto.randomUUID()` IDs with server-generated ones
- Add error handling, loading states, and empty states for real failure modes
- Move business logic out of `App.tsx` into dedicated service/hook files

---

## Quick Reference Card

```bash
# Start a new mockup
pnpm create vite@latest frontend -- --template react-ts
cd frontend && pnpm install && pnpm dev

# Build and check for errors
pnpm build

# Deploy manually
gh workflow run deploy-my-app.yml --ref main

# Check deployment status
gh run list --workflow=deploy-my-app.yml --limit 3

# Check Pages config
gh api repos/<org>/<repo>/pages --jq '{build_type,status,html_url}'

# Switch Pages to Actions mode (do once per repo)
gh api --method PUT repos/<org>/<repo>/pages --field build_type=workflow
```

---

## Resources

- [Claude Design](https://claude.ai/design) — interactive prototype / mockup canvas (Anthropic Labs, requires Pro/Max/Team/Enterprise)
- [Anthropic announcement](https://www.anthropic.com/news/claude-design-anthropic-labs) — what Claude Design is and how it works
- [Vite docs](https://vitejs.dev/guide/) — fast React build tool
- [GitHub Pages docs](https://docs.github.com/en/pages) — hosting reference
- [GitHub Actions: deploy-pages](https://github.com/actions/deploy-pages) — official deploy action
- [pnpm](https://pnpm.io) — fast, disk-efficient package manager
