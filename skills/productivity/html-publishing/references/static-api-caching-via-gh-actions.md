# Static API Caching via GitHub Actions

Replace direct-browser API calls on GitHub Pages with a GitHub Action that periodically fetches data into a static JSON file.

## Problem

A static HTML page on GitHub Pages calls an external API (e.g., `api.github.com`) directly from the browser with `fetch()`. The API rate-limits unauthenticated requests (GitHub unauthenticated: 60 req/hr). You can't embed auth tokens in client-side JS without exposing them publicly. Result: **HTTP 403** errors after a few requests.

## Solution

Use a GitHub Action to fetch the data server-side (with auth) on a cron schedule, save it as a static JSON file in the repo, and have the HTML page read the JSON instead.

### Architecture

```
                          ┌──────────────────┐
                          │  External API     │
                          │  (e.g. GitHub)    │
                          └────────┬─────────┘
                                   │  Authenticated via
                                   │  GITHUB_TOKEN / secret
                                   ▼
┌──────────────────────────────────────────────┐
│  GitHub Action (cron: */15 * * * *)          │
│  - Fetches data via curl/API call            │
│  - Saves as star_data.json (or name.json)    │
│  - Commits & pushes to repo                  │
└──────────────────────┬───────────────────────┘
                       │  Served as static file
                       ▼
┌──────────────────────────────────────────────┐
│  GitHub Pages (static site)                   │
│  - HTML page fetches star_data.json           │
│  - No auth needed (same-origin or CORS-free)  │
│  - Zero API calls from browser                │
└──────────────────────────────────────────────┘
```

### GitHub Action Workflow Template

```yaml
name: Star Tracker  # Or whatever your data source is

on:
  schedule:
    - cron: '*/15 * * * *'   # Every 15 min
  workflow_dispatch:           # Manual trigger

jobs:
  fetch-and-update:
    runs-on: ubuntu-latest
    permissions:
      contents: write          # Needed to push the JSON file

    steps:
      - uses: actions/checkout@v4

      - name: Fetch data
        run: |
          # Define your data sources
          DATA='[]'
          
          # Example: fetch from GitHub API for multiple repos
          for REPO in "owner1/repo1" "owner2/repo2"; do
            RESP=$(curl -s "https://api.github.com/repos/${REPO}")
            # Extract fields you need using jq or python
            STARS=$(echo "$RESP" | jq '.stargazers_count')
            # ... build your JSON array
          done
          
          NOW=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
          echo "$DATA" | jq --arg now "$NOW" \
            '{ lastUpdated: $now, repos: . }' > data.json

      - name: Commit and push
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add data.json
          if git diff --cached --quiet; then
            echo "No changes"
          else
            git commit -m "chore: update cached data [skip ci]"
            git push
          fi
```

### Key Details

| Aspect | Critical detail |
|--------|----------------|
| **Schedule** | `*/15 * * * *` stays within GitHub Actions free tier (60k exec/min mo). Avoid `* * * * *` (every minute). |
| **Permissions** | Must set `permissions: contents: write` for the token to push. |
| **Commit user** | Use `github-actions[bot]` so commits are clearly automated. |
| **Git push** | `GITHUB_TOKEN` can push to the same repo by default. |
| **[skip ci]** | `[skip ci]` in commit message prevents re-triggering the workflow (avoid loops). |
| **Initial seed** | Commit a placeholder `data.json` in the initial push so the HTML has something to read before the Action runs the first time. |

### HTML Client-Side Changes

Replace direct API calls in your HTML/JS:

```javascript
// BEFORE (broken — hits rate limits)
const res = await fetch('https://api.github.com/repos/owner/repo');

// AFTER (reads cached static JSON)
const res = await fetch('data.json');
const json = await res.json();
const repoData = json.repos.find(r => r.owner === 'owner' && r.repo === 'repo');
```

Also add fallback states:
- Loading state while `data.json` is being fetched
- "No data" state if the Action hasn't run yet (initial deploy)
- Graceful degradation if `data.json` fails to load

### Pitfalls

- **Workflow rate limit:** Even with a PAT, GitHub API has authenticated limits (5,000 req/hr). Keep the cron interval reasonable (15 min = 4 req/hr × number of data sources).
- **Cold start:** On first deploy, `data.json` won't exist until the Action runs. Either seed it with placeholder data, or include a workflow_dispatch trigger to run it manually after deploy.
- **GITHUB_TOKEN scope:** The auto-generated token only has access to the current repo. If you need to fetch from a private repo or a different org, use a PAT stored as a repo secret instead.
- **File format:** The JSON schema must match what the HTML expects. Define the schema before writing either side. A mismatched schema produces silent failures (undefined values, empty cards).
- **Push timing:** If two workflow runs overlap, the second push may fail with a rejected error. The `[skip ci]` marker and the cron spacing (> 5 min) usually prevent this.
