---
name: live-api-dashboard
description: Build and deploy a live data-fetching HTML dashboard page to GitHub Pages — fetches from a REST API, renders real-time data with auto-refresh and animated counters.
version: 1.0.0
author: Hermes Agent
tags: [dashboard, live, api, html, github-pages, realtime, visualization, tracker]
related_skills: [publish-html-github-pages, claude-design]
---

# Live API Dashboard

Build a self-contained, single-page HTML dashboard that fetches live data from a REST API (e.g. GitHub API, weather API, crypto prices, etc.), renders it with real-time updates and animations, and deploys it to GitHub Pages.

## When to Use

Trigger this skill when:
- User asks for a "live tracker", "dashboard", "realtime stats", or "live monitor" page
- User wants to display data from a public REST API in a visual dashboard
- User asks for GitHub star trackers, repo comparisons, or similar live metrics
- User wants "auto-refreshing" or "live updating" HTML page
- Building a page where data changes over time and should animate on update

Do NOT use for:
- Static reports from markdown (→ use `markdown-to-html`)
- Design exploration/variants (→ use `sketch`)
- Architecture or cloud diagrams (→ use `architecture-diagram`)
- One-off landing pages or prototypes (→ use `claude-design`)

## Workflow

### 1. Identify the Data Sources

Determine what API endpoints to fetch from. Often these are public REST APIs. Example patterns:

- **GitHub stars**: `GET https://api.github.com/repos/{owner}/{repo}`
- **Crypto prices**: `GET https://api.coingecko.com/api/v3/simple/price`
- **Weather**: `GET https://api.open-meteo.com/v1/forecast`
- **Custom data**: Any public JSON API

Verify the API returns JSON with `curl` before building the dashboard:

```bash
curl -s "https://api.github.com/repos/{owner}/{repo}" | python3 -c "import json,sys; d=json.load(sys.stdin); print(d.get('stargazers_count', 'ERROR'))"
```

### 2. Locate the GitHub Pages Repo

Find the local clone:

```bash
find ~ -maxdepth 5 -type d \( -name "github.io" -o -name "*.github.io" \) 2>/dev/null
```

### 3. Build the Dashboard HTML

Create a **single self-contained HTML file** with:

#### Key Components

**Header area:**
- Back link to `index.html` (`<a class="back-link" href="index.html">← Back to Index</a>`)
- Title describing the dashboard
- Status indicator (live/error dot)

**Summary bar (optional):**
- Aggregate stats (total, average, top item)
- Last-updated timestamp

**Card/tile grid:**
- One card per data item
- Each card shows: name/icon, description, primary metric (stars, price, etc.), secondary metric (forks, volume, etc.)
- Rank badges (#1, #2, #3, #4) for ordered data

**Data fetching:**
- Use `fetch()` to call the API
- Auto-refresh via `setInterval(fetchData, 60000)` (60 seconds is a good default)
- Error handling: show error message on card, update status indicator

**Animations:**
- Animate number transitions with `requestAnimationFrame` (ease-out cubic)
- Animated loading bars proportional to the max value

#### HTML Template Structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Live Dashboard Title</title>
  <style>
    /* ── Variables ── */
    :root {
      --dark:   #0f1b2d;
      --blue:   #0032A0;
      --light:  #e8edf5;
      --mid:    #c5d0e6;
      --green:  #1a7a4a;
      --amber:  #d4820a;
      --red:    #c0392b;
      --purple: #6b21a8;
      --text:   #1a1a2e;
      --muted:  #5a6a7a;
      --oc:     #ff6b35;
      --card-bg: #fff;
    }

    * { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      font-family: 'Segoe UI', 'Calibri', sans-serif;
      background: var(--light);
      color: var(--text);
      min-height: 100vh;
    }

    /* Back link */
    .back-link {
      display: inline-flex; align-items: center; gap: 6px;
      color: var(--mid); text-decoration: none; font-size: 13px;
      font-weight: 600; padding: 6px 12px;
      background: rgba(255,255,255,0.08); border-radius: 6px;
      transition: background 0.2s, color 0.2s;
    }
    .back-link:hover {
      background: rgba(255,255,255,0.15); color: #fff;
    }

    /* Header */
    .header {
      background: var(--dark);
      background-image: linear-gradient(135deg, #0f1b2d 0%, #1a2f4e 100%);
      color: #fff; padding: 32px 40px 28px;
      border-bottom: 4px solid var(--oc);
    }
    .header-top {
      display: flex; align-items: center; justify-content: space-between;
      margin-bottom: 8px; flex-wrap: wrap; gap: 10px;
    }
    .header h1 { font-size: 24px; font-weight: 700; letter-spacing: -0.3px; }
    .badge {
      background: var(--oc); color: #fff; font-size: 11px; font-weight: 700;
      letter-spacing: 1.5px; text-transform: uppercase; padding: 4px 10px; border-radius: 4px;
    }
    .header-meta {
      font-size: 13px; color: var(--mid); margin-top: 6px;
      display: flex; align-items: center; gap: 20px; flex-wrap: wrap;
    }
    .status-dot {
      display: inline-block; width: 8px; height: 8px; border-radius: 50%;
      background: #4ade80; animation: pulse 2s infinite;
    }
    .status-dot.error { background: #f87171; animation: none; }
    @keyframes pulse { 0%, 100% { opacity: 1; } 50% { opacity: 0.4; } }

    .body { padding: 36px 40px 60px; max-width: 1100px; }

    /* Card grid */
    .dashboard-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(260px, 1fr));
      gap: 20px;
    }

    .data-card {
      background: var(--card-bg); border-radius: 14px;
      box-shadow: 0 1px 5px rgba(0,0,0,.08); overflow: hidden;
      transition: box-shadow 0.2s, transform 0.15s;
      border-top: 4px solid var(--blue);
    }
    .data-card:hover {
      box-shadow: 0 6px 20px rgba(0,0,0,.13); transform: translateY(-3px);
    }

    /* Accent colors per card */
    .data-card.accent-oc     { border-top-color: var(--oc); }
    .data-card.accent-green  { border-top-color: var(--green); }
    .data-card.accent-purple { border-top-color: var(--purple); }
    .data-card.accent-amber  { border-top-color: var(--amber); }
    .data-card.accent-red    { border-top-color: var(--red); }

    /* Card content */
    .card-head { padding: 20px 22px 14px; display: flex; align-items: flex-start; gap: 12px; }
    .card-avatar {
      width: 44px; height: 44px; border-radius: 10px;
      background: var(--dark); display: flex; align-items: center;
      justify-content: center; font-size: 20px; color: #fff; flex-shrink: 0;
    }
    .card-info { flex: 1; min-width: 0; }
    .card-name { font-size: 15px; font-weight: 700; color: var(--dark); }
    .card-subname { font-size: 11.5px; color: var(--muted); margin-top: 2px; }
    .card-desc {
      font-size: 12.5px; color: var(--muted); line-height: 1.5;
      padding: 0 22px 14px; min-height: 38px;
    }

    /* Metric area */
    .metric-area {
      padding: 14px 22px 18px; background: #f8faff;
      border-top: 1px solid #e8edf5; text-align: center;
    }
    .metric-value {
      font-size: 38px; font-weight: 800; color: var(--dark);
      letter-spacing: -1px; line-height: 1;
      display: flex; align-items: center; justify-content: center; gap: 8px;
    }
    .metric-icon { font-size: 30px; }
    .metric-label {
      font-size: 11.5px; color: var(--muted); text-transform: uppercase;
      letter-spacing: 1.2px; font-weight: 600; margin-top: 4px;
    }
    .metric-loading { font-size: 14px; color: var(--muted); animation: pulse 1.5s infinite; }
    .loading-track {
      height: 4px; background: #e8edf5; border-radius: 2px; overflow: hidden; margin-top: 12px;
    }
    .loading-fill {
      height: 100%; width: 0%;
      background: linear-gradient(90deg, var(--blue), var(--oc));
      border-radius: 2px; transition: width 1s ease;
    }

    .card-footer {
      padding: 12px 22px; display: flex; justify-content: space-between;
      align-items: center; border-top: 1px solid #e8edf5;
      font-size: 12px; color: var(--muted);
    }
    .card-footer a { color: var(--blue); text-decoration: none; font-weight: 600; }
    .card-footer a:hover { text-decoration: underline; }

    /* Summary bar */
    .summary-bar {
      display: flex; align-items: center; gap: 16px; flex-wrap: wrap;
      background: #fff; border-radius: 12px; padding: 16px 22px;
      box-shadow: 0 1px 5px rgba(0,0,0,.08); margin-bottom: 24px;
    }
    .summary-stat { display: flex; align-items: baseline; gap: 6px; }
    .summary-stat .num { font-size: 18px; font-weight: 800; color: var(--dark); }
    .summary-stat .label { font-size: 12px; color: var(--muted); }
    .summary-divider { width: 1px; height: 28px; background: var(--mid); }
    .last-updated { margin-left: auto; font-size: 11.5px; color: var(--muted); }

    /* Rank badges */
    .rank-badge {
      display: inline-flex; align-items: center; gap: 3px;
      font-size: 11px; font-weight: 700; padding: 2px 8px; border-radius: 20px;
    }
    .rank-1 { background: #fef3c7; color: #b45309; }
    .rank-2 { background: #e5e7eb; color: #6b7280; }
    .rank-3 { background: #fed7aa; color: #c2410c; }
    .rank-4 { background: #fce7f3; color: #be185d; }

    .footer {
      padding: 24px 40px; border-top: 1px solid var(--mid);
      font-size: 12px; color: var(--muted); text-align: center;
    }

    @media (max-width: 600px) {
      .header { padding: 24px 20px 20px; }
      .header h1 { font-size: 20px; }
      .body   { padding: 24px 20px 40px; }
      .dashboard-grid { grid-template-columns: 1fr; }
      .summary-bar { flex-direction: column; align-items: flex-start; }
      .last-updated { margin-left: 0; }
      .summary-divider { display: none; }
    }
  </style>
</head>
<body>

  <!-- Header -->
  <div class="header">
    <div class="header-top">
      <div style="display:flex;align-items:center;gap:12px;">
        <span class="badge">★ Live</span>
        <h1>Dashboard Title</h1>
      </div>
      <a class="back-link" href="index.html">← Back to Index</a>
    </div>
    <div class="header-meta">
      <span><span class="status-dot" id="statusDot"></span> <span id="statusText">Loading...</span></span>
      <span>Auto-refreshes every 60s</span>
    </div>
  </div>

  <!-- Main -->
  <div class="body">
    <div class="summary-bar" id="summaryBar">
      <div class="summary-stat">
        <span class="num" id="totalMetric">—</span>
        <span class="label">Total</span>
      </div>
      <div class="summary-divider"></div>
      <div class="summary-stat">
        <span class="num" id="topItem">—</span>
        <span class="label">Top</span>
      </div>
      <div class="last-updated" id="lastUpdated">—</div>
    </div>

    <div class="dashboard-grid" id="dashboardGrid"></div>
  </div>

  <div class="footer">
    Data from <a href="https://developer.github.com" style="color:var(--blue);">API</a>
  </div>

  <script>
    // ── Config ──
    const ITEMS = [
      { name: 'Item 1',     owner: 'owner', repo: 'repo1',       icon: '🔵', accent: 'accent-oc' },
      // ... add items here
    ];

    const API_BASE = 'https://api.github.com/repos';
    const REFRESH_MS = 60000;

    // ── Helpers ──
    function formatNumber(n) {
      if (n >= 1_000_000) return (n / 1_000_000).toFixed(1) + 'M';
      if (n >= 1_000) return (n / 1_000).toFixed(1) + 'K';
      return n.toString();
    }

    function fullNumber(n) {
      return n.toLocaleString();
    }

    function animateCount(el, target, suffix = '') {
      const current = parseInt(el.textContent.replace(/[^0-9]/g, '')) || 0;
      const duration = 1000;
      const start = performance.now();

      function step(now) {
        const progress = Math.min((now - start) / duration, 1);
        const eased = 1 - Math.pow(1 - progress, 3);
        const value = Math.round(current + (target - current) * eased);
        el.textContent = fullNumber(value) + suffix;
        if (progress < 1) requestAnimationFrame(step);
      }
      requestAnimationFrame(step);
    }

    // ── Build skeleton ──
    function buildSkeleton() {
      const grid = document.getElementById('dashboardGrid');
      grid.innerHTML = '';
      ITEMS.forEach(item => {
        const card = document.createElement('div');
        card.className = `data-card ${item.accent}`;
        card.id = `card-${item.owner}-${item.repo}`;
        card.innerHTML = `
          <div class="card-head">
            <div class="card-avatar">${item.icon}</div>
            <div class="card-info">
              <div class="card-name">${item.name}</div>
              <div class="card-subname">${item.owner}/${item.repo}</div>
            </div>
          </div>
          <div class="card-desc" id="desc-${item.owner}-${item.repo}">
            <span class="metric-loading">Loading...</span>
          </div>
          <div class="metric-area">
            <div class="metric-value">
              <span class="metric-icon">★</span>
              <span id="metric-${item.owner}-${item.repo}"><span class="metric-loading">—</span></span>
            </div>
            <div class="metric-label">Primary Metric</div>
            <div class="loading-track">
              <div class="loading-fill" id="bar-${item.owner}-${item.repo}"></div>
            </div>
          </div>
          <div class="card-footer">
            <span id="secondary-${item.owner}-${item.repo}">Loading...</span>
            <span id="rank-${item.owner}-${item.repo}"></span>
            <a href="https://github.com/${item.owner}/${item.repo}" target="_blank">View →</a>
          </div>
        `;
        grid.appendChild(card);
      });
    }

    // ── Fetch data ──
    async function fetchAll() {
      const results = [];
      for (const item of ITEMS) {
        try {
          const res = await fetch(`${API_BASE}/${item.owner}/${item.repo}`);
          if (!res.ok) throw new Error(`HTTP ${res.status}`);
          const data = await res.json();
          results.push({ ...item, data });
        } catch (e) {
          results.push({ ...item, error: e.message });
        }
      }
      return results;
    }

    // ── Update UI ──
    function updateUI(results) {
      const errors = results.filter(r => r.error);
      const statusDot = document.getElementById('statusDot');
      const statusText = document.getElementById('statusText');

      if (errors.length > 0) {
        statusDot.className = 'status-dot error';
        statusText.textContent = `${errors.length} error(s)`;
      } else {
        statusDot.className = 'status-dot';
        statusText.textContent = 'All live';
      }

      const valid = results.filter(r => r.data && r.data.stargazers_count !== undefined);
      const sorted = [...valid].sort((a, b) => b.data.stargazers_count - a.data.stargazers_count);

      // Summary
      if (valid.length > 0) {
        const total = valid.reduce((s, r) => s + r.data.stargazers_count, 0);
        animateCount(document.getElementById('totalMetric'), total);
        document.getElementById('topItem').textContent = sorted[0].name;
      }

      document.getElementById('lastUpdated').textContent =
        'Updated: ' + new Date().toLocaleTimeString();

      // Per-card
      results.forEach(r => {
        if (r.error) {
          document.getElementById(`metric-${r.owner}-${r.repo}`).innerHTML =
            `<span style="color:var(--red);font-size:18px;">⚠ Error</span>`;
          document.getElementById(`desc-${r.owner}-${r.repo}`).textContent = r.error;
          return;
        }
        const d = r.data;
        animateCount(document.getElementById(`metric-${r.owner}-${r.repo}`), d.stargazers_count);
        document.getElementById(`desc-${r.owner}-${r.repo}`).textContent =
          d.description || 'No description';

        const maxStars = sorted.length > 0 ? sorted[0].data.stargazers_count : 1;
        const pct = Math.min((d.stargazers_count / maxStars) * 100, 100);
        document.getElementById(`bar-${r.owner}-${r.repo}`).style.width = pct + '%';

        // Update secondary metric (e.g., forks)
        document.getElementById(`secondary-${r.owner}-${r.repo}`).textContent =
          '🍴 ' + fullNumber(d.forks_count) + ' forks';

        // Rank
        const rankIndex = sorted.findIndex(s => s.owner === r.owner && s.repo === r.repo);
        if (rankIndex >= 0) {
          const rank = rankIndex + 1;
          document.getElementById(`rank-${r.owner}-${r.repo}`).innerHTML =
            `<span class="rank-badge rank-${rank}">#${rank}</span>`;
        }
      });
    }

    // ── Init ──
    buildSkeleton();
    fetchAll().then(updateUI);
    setInterval(() => fetchAll().then(updateUI), REFRESH_MS);
  </script>
</body>
</html>
```

### 4. Verify the Page Works

After creating the file, verify it renders correctly. Check for:
- No console errors (CORS, API rate limits, 404s)
- Cards display data after loading
- Auto-refresh works (check status indicator)
- Responsive at mobile width

### 5. Deploy to GitHub Pages

After the HTML file is built, use the existing `publish-html-github-pages` skill to:
1. Copy the file to the GitHub Pages repo
2. Add a card/entry on the index page
3. Commit and push
4. Verify the deployment

> **Important**: Always update `index.html` with a card/link to the new dashboard page, matching the existing index page's layout style (card-grid or link-list).

### 6. Verify Deployment

Wait ~30 seconds for GitHub Pages to deploy, then verify:

```bash
curl -s -o /dev/null -w "%{http_code}" "https://{username}.github.io/{repo-name}/{filename}"
# Expected: 200
```

## Pitfalls

- **GitHub API rate limiting**: Unauthenticated requests are limited to 60/hour. For dashboards used heavily, recommend adding `Authorization: Bearer <token>` header in fetch calls.
- **CORS**: GitHub API supports CORS from browsers, but other APIs may not. Test with `curl` first if unsure.
- **Merge conflicts**: If the remote index.html has been updated since your last pull, you'll get merge conflicts. Resolve by rebasing: `git pull --rebase origin main`, fix conflicts, then push.
- **Deployment delay**: GitHub Pages takes ~20-30 seconds after push. Wait before claiming 404.
- **Number animation**: The `animateCount` function parses the current text content. If you change the number format mid-animation, the count may jump. Avoid rapid refreshes (< 10s interval).
- **File naming**: Use underscores or hyphens — avoid spaces in filenames for GitHub Pages.
