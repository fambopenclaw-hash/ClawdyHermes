# Live API Dashboard Template

## When to Use

Build a self-contained HTML dashboard that fetches live data from a REST API, renders it with real-time updates and animations, and deploys to GitHub Pages.

## Architecture

Single HTML file with embedded CSS and JS. Browser-side `fetch()` to a public API, `setInterval()` for auto-refresh, `requestAnimationFrame()` for number animations.

## Structure

```
Header: back link, title, live badge, status indicator (green/red dot)
  ↓
Summary bar: total, top item, last-updated timestamp
  ↓
Dashboard grid: responsive card grid (CSS grid, auto-fill minmax(260px, 1fr))
  └─ Each card: avatar icon, name, description, primary metric (animated),
       loading bar, secondary metric, rank badge, link
  ↓
Footer: data source attribution
```

## Design Tokens

```css
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
```

## Key JavaScript Functions

### animateCount() — smooth number transitions

```javascript
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
```

### fetchAll() — batch API calls

```javascript
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
```

### updateUI() — render fetched data

Updates status dot, summary bar (total count + top item + timestamp), and per-card metrics (animated count, loading bar width, secondary metrics, rank badges).

## Example: GitHub Star Tracker

Wrap in:
```html
<div class="header">...</div>
<div class="body">
  <div class="summary-bar">...</div>
  <div class="dashboard-grid" id="dashboardGrid"></div>
</div>
<div class="footer">...</div>
```

Items array config:
```javascript
const ITEMS = [
  { name: 'Repo Name', owner: 'owner', repo: 'repo', icon: '🔵', accent: 'accent-oc' },
  // ...
];
const API_BASE = 'https://api.github.com/repos';
const REFRESH_MS = 60000;
```

## Pitfalls

- **Rate limiting**: GitHub unauthenticated = 60 req/hr. Use `Authorization: Bearer <token>` for higher limits.
- **CORS**: Test with `curl` first. Some APIs don't support browser CORS.
- **Cold start**: For cached dashboards (GH Actions), seed placeholder `data.json` in the initial commit.
- **Number animation**: Avoid refresh intervals < 10s to prevent animation jumps.
- **File naming**: Use underscores or hyphens; avoid spaces.
