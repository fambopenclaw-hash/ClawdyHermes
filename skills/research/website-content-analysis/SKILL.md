---
name: website-content-analysis
description: "Navigate to a URL and produce a structured descriptive analysis of the website's content, structure, navigation, and offerings using the browser toolset."
version: 1.0.0
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [browser, research, analysis, web, content]
    related_skills: [dogfood]
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

## Tips

- **Start with the main page** — it usually tells you the most about the site's purpose
- **Follow navigation links** — the menu structure reveals what the site considers important
- **Go back between deep dives** — use `browser_back()` rather than re-navigating to save time
- **Check embedded content** — PDFs, iframes, image galleries, videos are important content indicators
- **Note dates on everything** — timestamps are the #1 indicator of whether a site is alive or dead
- **Check footer content** — copyright dates, contact info, and privacy/terms links give context about the publisher
- **Watch for unrendered shortcodes and typos** — they're telltale signs of neglected maintenance
- **When asked "what are the alternatives?"** — directly navigating to known competitors may be more reliable than search engines (which often block automated access)
