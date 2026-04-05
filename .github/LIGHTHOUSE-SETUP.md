# Lighthouse Audit + Auto-Deploy Setup

## Overview

This repo now has **fully automated deployment and quality auditing**:
- ✅ Every push to `main` triggers Lighthouse audit
- ✅ Audit results stored as machine-readable JSON artifacts
- ✅ Site auto-deploys to GitHub Pages on every successful audit
- ✅ HTML and JSON reports available for inspection

## Prerequisites (GitHub Repo Settings)

### 1. GitHub Pages must be enabled

In your **GitHub repo** → **Settings** → **Pages**:

```
Source: Deploy from a branch
Branch: main
Folder: / (root)
```

- GitHub will auto-provision a Let's Encrypt SSL cert within ~10 minutes
- Custom domain (CNAME) will work once Pages is enabled

### 2. Workflow permissions

The workflow uses GitHub's built-in `GITHUB_TOKEN`. Ensure in **Settings** → **Actions** → **General**:

```
Workflow permissions: Read and write permissions ✓
Allow GitHub Actions to create and approve pull requests ✓
```

(These are usually enabled by default in public repos)

### 3. Branch protection (optional but recommended)

If you want to require Lighthouse audit to pass before merging:

**Settings** → **Branches** → **Add rule** → Branch name `main`:

```
✓ Require status checks to pass before merging
  → Select: "Lighthouse Audit" job
```

This prevents merging until the audit completes successfully.

---

## Workflow Trigger Conditions

### Automatic Triggers
- **Every push to main branch**
- Any direct push or merged pull request

### Manual Trigger
- Via GitHub Actions → "Deploy & Lighthouse Audit" → "Run workflow" button

---

## File Paths & Structure

### Artifact Output Paths (Inside Workflow)

```
.github/workflows/deploy-and-audit.yml    ← Main workflow definition
lighthouse-reports/
├── lighthouse-report.json                ← Machine-readable full report
├── lighthouse-report.html                ← Human-readable HTML report
└── summary.json                          ← Extracted metrics (categories + core web vitals)
```

### Static Site Root

The workflow serves from **`/`** (repo root), which includes:
- `index.html` — Main landing page
- `styles.css` — Stylesheet
- `setup.html`, `ui-lab.html` — Additional pages
- `CNAME` — Custom domain pointer
- Other assets

**Assumption:** All site files are at the repo root. If you later move site files to a subdirectory (e.g., `/public`), update the workflow:

```yaml
# In "Upload artifact" step, change:
path: '.'       # Current (root)
# To:
path: 'public'  # If site files move to /public
```

---

## Accessing Lighthouse Results

### Option 1: GitHub Actions Artifacts (Recommended)

After each workflow run:

1. Go to repo → **Actions** → "Deploy & Lighthouse Audit" workflow
2. Click the latest run
3. Scroll to "Artifacts" section
4. Download `lighthouse-report` (contains `.json` and `.html`)

**Retention:** Artifacts kept for 90 days by default.

### Option 2: Workflow Summary in GitHub UI

The workflow posts results directly to the Actions summary:

- Go to repo → **Actions** → Latest run
- Scroll to "Lighthouse Audit Results" step
- See live score breakdown in markdown table

### Option 3: Machine-Readable JSON

**Direct file access** (after workflow completes):

```bash
# Clone artifacts locally (requires GitHub CLI)
gh run download <RUN_ID> -n lighthouse-report

# Or access the summary.json directly
cat lighthouse-reports/summary.json
```

---

## JSON Report Schema (summary.json)

The `summary.json` file contains:

```json
{
  "timestamp": "2026-04-05T14:23:45Z",
  "commit": "a1b2c3d",
  "categories": {
    "performance": { "title": "Performance", "score": 0.95 },
    "accessibility": { "title": "Accessibility", "score": 0.92 },
    "best-practices": { "title": "Best Practices", "score": 0.89 },
    "seo": { "title": "SEO", "score": 1.0 },
    "pwa": { "title": "PWA", "score": 0.72 }
  },
  "metrics": {
    "firstContentfulPaint": 1234,
    "largestContentfulPaint": 3456,
    "cumulativeLayoutShift": 0.05,
    "interactive": 2100
  }
}
```

**How to parse programmatically:**

```bash
# Extract performance score
jq '.categories.performance.score' lighthouse-reports/summary.json

# Extract all scores as CSV
jq -r '.categories | to_entries[] | [.key, (.value.score * 100)] | @csv' lighthouse-reports/summary.json
```

---

## Workflow Steps Explained

### 1. **Audit Job**

Runs on every trigger:

1. Checks out code
2. Sets up Node.js 20
3. Installs `lighthouse` (npm global)
4. Starts local HTTP server on `localhost:8080`
5. Runs Lighthouse audit against local server
6. Generates JSON + HTML reports
7. Parses metrics and posts to GitHub Actions summary
8. Uploads reports as artifacts

**Duration:** ~2-3 minutes per run

### 2. **Deploy Job**

Runs AFTER audit succeeds:

1. Prepares artifact (entire repo as-is)
2. Configures GitHub Pages deployment
3. Deploys to `https://[owner].github.io/openclawforentrepreneurs.com/`
   - Or custom domain if CNAME is set: `https://openclawforentrepreneurs.com`

**Duration:** ~30-60 seconds

**Important:** GitHub Pages auto-rebuilds when the artifact is pushed. No additional build step needed for static sites.

---

## Lighthouse Thresholds (Customizable)

Currently, the audit runs and reports scores **without blocking on thresholds**.

To add automated fail-gates, edit the workflow:

```yaml
- name: Check Lighthouse scores
  run: |
    node -e "
      const fs = require('fs');
      const report = JSON.parse(fs.readFileSync('./lighthouse-reports/lighthouse-report.json'));
      const scores = report.categories;

      const minScore = 0.80;  // 80/100 minimum
      let passed = true;

      for (const [key, cat] of Object.entries(scores)) {
        if (cat.score < minScore) {
          console.error(\`❌ \${cat.title}: \${Math.round(cat.score * 100)} (required: 80)\`);
          passed = false;
        }
      }

      if (!passed) process.exit(1);
    "
```

This would **fail the workflow** if any category drops below 80/100, preventing deployment.

---

## Troubleshooting

### "Lighthouse audit fails with 'Cannot reach localhost:8080'"

**Cause:** Server startup is too slow or port 8080 is already in use.

**Fix:** In workflow, increase sleep duration:
```yaml
sleep 3    # Change to: sleep 5
```

### "GitHub Pages deployment shows 'The source is not available' "

**Cause:** GitHub Pages not enabled in repo settings yet.

**Fix:** Go to **Settings** → **Pages** → Enable as described in "Prerequisites" section.

### "Artifacts not being saved"

**Cause:** Workflow failed before `upload-artifact` step.

**Fix:** Check the full workflow log in Actions → Latest run → "Audit" job to see exact error.

### "Want to audit multiple pages?"

Currently audits only `/` (index.html). To audit multiple pages:

```yaml
- name: Run Lighthouse audit (multiple URLs)
  run: |
    # Create array of URLs to audit
    URLs=("/" "/setup.html" "/ui-lab.html")

    for URL in "${URLs[@]}"; do
      FILENAME=$(echo "$URL" | sed 's/[^a-zA-Z0-9]/-/g').json
      lighthouse "http://localhost:8080${URL}" \
        --output=json \
        --output-path="./lighthouse-reports/$FILENAME"
    done
```

---

## Next Steps

1. **Enable GitHub Pages** in repo settings (see Prerequisites)
2. **Push to main** — workflow will trigger automatically
3. **Check Actions tab** — watch the workflow run in real-time
4. **View results** — Artifacts tab after workflow completes
5. **(Optional) Set up branch protection** to require audit to pass before merging

---

## Cost & Limits

- **Lighthouse CLI:** Free (open source)
- **GitHub Actions:** Free for public repos (unlimited minutes)
- **GitHub Pages:** Free
- **Artifact storage:** 90 days retention (free tier)

**Total monthly cost:** $0

---

## Reference

- [Lighthouse CI Handbook](https://github.com/GoogleChrome/lighthouse-ci)
- [GitHub Actions Docs](https://docs.github.com/en/actions)
- [GitHub Pages Deployment](https://docs.github.com/en/pages)
