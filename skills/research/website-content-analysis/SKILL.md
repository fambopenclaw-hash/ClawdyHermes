---
name: website-content-analysis
description: "Navigate to a URL and produce a structured descriptive analysis of the website's content, structure, navigation, and offerings using the browser toolset."
version: 1.0.0
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [browser, research, analysis, web, content]
    related_skills: [dogfood, html-publishing]
---

# Website Content Analysis

## When to Use

Trigger when the user says something like:
- "Analyze this website" or "Analyze the content of this site"
- "What does this website offer?" / "What's on this site?"
- "Explore this URL and tell me about it"
- "Check out this domain and summarize it"
- "Can you access this website?" followed by a URL

This is for **informational analysis** — understanding what a site contains, its structure, purpose, and offerings.

Do NOT use this skill for:
- Bug hunting / QA testing → use **dogfood** instead
- Bug reports, finding broken elements, console errors
- Code review of a web app
- Fetching news or monitoring RSS → use **get-news** or **blogwatcher**

## Prerequisites

- Browser toolset available (`browser_navigate`, `browser_snapshot`, `browser_click`, `browser_back`, `browser_vision`, `browser_scroll`)
- A target URL from the user

## Workflow

### Phase 1: Initial Landing & Overview

1. **Navigate** to the URL:
   ```
   browser_navigate(url="https://example.com")
   ```

2. **Snapshot the page** to understand its structure:
   ```
   browser_snapshot()
   ```

3. **Analyze the page structure** by reading the snapshot:
   - Identify the **site name / brand** (from the page title, header, logo text)
   - Identify the **navigation menu** — what sections exist (About, Products, Blog, etc.)
   - Note the **page layout**: main content area, sidebars, footer
   - Identify any **search functionality, login, or interactive elements**

### Phase 2: Explore Key Pages

For each major navigation section or content area:

1. **Click major navigation links** to explore sub-pages:
   ```
   browser_click(ref="@eN")
   ```

2. **Snapshot each sub-page** to understand its content:
   ```
   browser_snapshot()
   ```

3. **Note the content type** on each page:
   - Is it an article/blog post? A product listing? A map/diagram? A form?
   - How is information organized? (lists, tables, embedded media, PDFs)
   - What is the approximate quantity of content? (number of posts, items, etc.)

4. **Go back** to the main page before exploring the next section:
   ```
   browser_back()
   ```

5. **Re-snapshot** after navigating back if needed (element refs may change).

### Phase 3: Deep Dive (as needed)

For particularly rich pages (e.g., a page with multiple embedded maps, downloads, or resources):

1. **Scroll** to reveal content below the fold:
   ```
   browser_scroll(direction="down")
   ```

2. **Take a screenshot with vision** if visual analysis would be valuable:
   ```
   browser_vision(question="Describe what's visible on this page")
   ```

3. **Check for downloadable content** — PDFs, images, data files

### Phase 4: Synthesize

Present the analysis in a structured format covering:

1. **🏛️ About the Site** — name, author/publisher, purpose, tagline
2. **🗂️ Content Structure** — what pages/sections exist, how they're organized
3. **📋 Content Inventory** — detailed breakdown of what's available (articles, maps, downloads, tools, etc.) with quantities and descriptions
4. **🔑 Key Topics/Tags** — themes, categories, subjects covered
5. **🧭 Navigation Overview** — how to move through the site
6. **📊 Overall Assessment** — what the site is useful for, who it's for, quality observations

### Phase 5: Maintenance Assessment

Users often follow up with "Is this site still active?" or "Is it well-maintained?" After the content analysis, proactively assess maintenance by:

1. **Scan for timestamps** on all content you encountered:
   - Blog/article publish dates
   - Copyright footer dates
   - "Last updated" markers
   - Comment dates

2. **Check for technical issues**:
   - Unrendered shortcodes or plugins (e.g., `[aps-social id="1"]`, `[contact-form]`)
   - Typos in navigation labels or headings
   - Broken image placeholders or 404 resources
   - Dead links in navigation or content
   - Outdated copyright (e.g., "©2024" when it's 2026+)

3. **Evaluate abandonment signals**:
   - **Active**: content published within the last 3 months
   - **Slow**: last update 3–12 months ago
   - **Dormant**: last update 1–3 years ago
   - **Abandoned**: no new content in 3+ years

4. **Synthesize a verdict** — e.g. "The site is functioning but abandoned. Useful as a historical reference archive, but not a living source of current information."

### Phase 6: Alternative Discovery

Users may also ask "What are the alternatives?" After phases 1-5, research comparable sites:

1. **Search for alternatives** using the browser (search engines often block automated queries, so be prepared for CAPTCHAs):
   ```
   browser_navigate(url="https://www.bing.com/search?q=alternatives+to+[SITE_NAME]+[TOPIC]")
   ```
   - Try multiple search engines if one is blocked (Bing → DuckDuckGo → Brave → Yahoo)
   - If all search engines are blocked, fall back to curl with a browser-like User-Agent to query DuckDuckGo's HTML endpoint:
     ```
     curl -sL -H "User-Agent: Mozilla/5.0 ..." "https://html.duckduckgo.com/html/?q=..." 
     ```

2. **When search engines block you** (common without residential proxies), rely on:
   - Known industry/sector knowledge of major players
   - Directly navigating to known competitor URLs to verify they're alive
   - Checking official/government sources related to the topic

3. **For each alternative, check**:
   - Is it actively updated? (scan dates)
   - Is it free or paid?
   - How does its content compare to the original site?
   - Does it have unique features the original lacks?

4. **Present alternatives in a structured comparison** (table format with columns for Source, Type, Content, Cost) and highlight the best replacement.

---

## Phase 7: Data Extraction from Dynamic Pages

When `browser_snapshot()` shows only "Loading...", spinners, or empty containers — the page is a **client-side rendered SPA** (Next.js, React, Vue, Svelte, Angular). The actual data loads asynchronously after the initial HTML. Use **async JavaScript injection via `browser_console`** to extract it.

### Prerequisites

- Browser toolset: `browser_navigate`, `browser_snapshot`, `browser_console`
- Target URL
- Basic JavaScript (async/await, DOM queries, fetch API)

### Phase 7a: Identify Dynamic Content

1. Navigate to the URL: `browser_navigate(url="...")`
2. Snapshot: `browser_snapshot()`
   - If you see actual content → use main Phase 1-5 workflow
   - If you see "Loading...", spinners, empty containers → continue to Phase 7b
3. Check for client-side framework clues:
   - `__NEXT_DATA__` script tag → Next.js
   - `data-reactroot` or `__REACT_QUERY_STATE__` → React SPA
   - `__NUXT__` → Nuxt/Vue
   - API endpoint URLs in page source

### Phase 7b: Extract Visible Text (Simple)

Wait for JS to finish rendering, then dump all page text:

```javascript
// Wait 5 seconds for async data, then log page text
browser_console(
  expression="(async () => { await new Promise(r => setTimeout(r, 5000)); console.log(document.body.innerText); })()"
)

// Read the output
browser_console()
```

**Timing**: Start with 5s. If data still missing, increase to 7-8s. For very heavy pages, try up to 10s.

### Phase 7c: Extract Specific Elements (Targeted)

For table/list data that's truncated by the full-page dump:

```javascript
// Query specific rendered DOM elements
browser_console(
  expression="(async () => { await new Promise(r => setTimeout(r, 5000)); const items = document.querySelectorAll('.model-row, tr, .list-item, [class*=\"rank\"]'); items.forEach(item => console.log(item.innerText.trim())); })()"
)
```

### Phase 7d: Fetch Internal API Directly

When you know (or can guess) the internal JSON API endpoint:

```javascript
// Same-origin fetch — works for internal APIs
browser_console(
  expression="(async () => { await new Promise(r => setTimeout(r, 2000)); const resp = await fetch('/api/v1/data', {headers: {'Accept': 'application/json'}}); const data = await resp.json(); console.log(JSON.stringify(data, null, 2)); })()"
)
```

**Note**: Only works for same-origin endpoints. CORS blocks cross-origin fetches.

### Phase 7e: Pagination and Infinite Scroll

If the page only shows a few items:

1. **Scroll** to trigger lazy-loading: `browser_scroll(direction="down")`
2. **Wait** for new data to render: 3s delay
3. **Extract** again using the console injection pattern

### Common Framework Patterns

| Framework | Clues | Extraction Strategy |
|-----------|-------|-------------------|
| **Next.js** | `__NEXT_DATA__`, `/api/*` routes | Console injection always works; also try `/_next/data/<build-id>/path.json` |
| **React SPA** (CRA, Vite) | `#root` div, no SSR | Console injection; internal `/api/*` if same origin |
| **Vue/Nuxt** | `__NUXT__` | May SSR some data; check snapshot first, fall back to console injection |

### Pitfalls

- **Vision analysis won't help** if the model doesn't support images — stick to console text extraction
- **CORS** prevents cross-origin `fetch()` — only same-origin internal APIs work
- **Auth-required APIs** return redirects to login pages, not data
- **Rate limiting**: Some sites block rapid repeated API calls
- **No `__NEXT_DATA__`** doesn't rule out Next.js — newer versions don't always embed it
- **Filter/tab state**: The default tab may not be the one you want — click the right tab first, then extract

### Worked Example: OpenRouter Rankings

The full extraction recipe is in `references/openrouter-rankings-extraction.md`. Summary:
1. Navigate to `https://openrouter.ai/rankings` (Next.js SPA)
2. Wait 5s for rendering instead of trusting the "Loading..." snapshot
3. Extract `document.body.innerText` via console injection
4. Result: top 10 LLM models ranked by weekly token usage (see reference for exact data)

---

## Scraping + Publishing Workflow

When the user wants to **extract live data AND publish it as a dashboard**, combine this skill with `html-publishing`:

1. **Extract data** using Phase 7 techniques above
2. **Format** the data as JSON or structured text
3. **Build an auto-refreshing dashboard** or static HTML report using `html-publishing` Section 3 (Live API Dashboards)
4. **Publish** to GitHub Pages using `html-publishing` Section 4

For recurring scraping jobs, set up a `cronjob`:
```python
cronjob(
    action='create',
    schedule='0 9 * * *',   # Daily at 9AM
    prompt="Scrape URL, extract data, commit to repo",
    toolsets=["browser", "terminal", "file"]
)
```

---

## Tips
