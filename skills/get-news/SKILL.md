---
name: get-news
description: Fetches and summarizes recent news from Malaysia (local), global, Iran-US conflict, and technology/AI categories within the last 24 hours. Use when the user requests news updates or summaries in any of these domains. Provides a standardized approach to gathering timely news with consistent formatting and category-specific search strategies.
---

# Get News

## Overview

The get-news skill retrieves and summarizes the latest news from four key categories: Malaysia local news, global news, Iran-US war developments, and technology/AI news. It ensures fresh content (within 24 hours) and presents it in a clear, categorized format.

## When to Use This Skill

Trigger the skill when the user:
- Asks for news updates or current events
- Requests summaries of recent happenings in Malaysia, globally, regarding the Iran-US conflict, or in technology/AI
- Wants to stay informed about developments in any of these four categories

## Capabilities

The skill supports fetching news for the following categories:

1. **Malaysia Local News**
   - Focus: National and regional events within Malaysia
   - RSS feed: `https://news.google.com/rss/search?q=Malaysia+news&hl=en-MY&gl=MY&ceid=MY:en`
   - Topics: "Malaysia politics", "Malaysia economy" (append to query as needed)
   - Freshness: last 24 hours (see Implementation Notes)

2. **Global News**
   - Focus: Major world events and international developments
   - RSS feed: `https://news.google.com/rss/search?q=global+news&hl=en-US&gl=US&ceid=US:en`
   - Freshness: last 24 hours (see Implementation Notes)

3. **Iran-US War**
   - Focus: Latest developments in the conflict between Iran and the United States
   - RSS feed: `https://news.google.com/rss/search?q=Iran+US+conflict&hl=en-US&gl=US&ceid=US:en`
   - Also try: "Iran US war", "Iran United States tensions"
   - Freshness: last 24 hours (see Implementation Notes)

4. **Technology & AI**
   - Focus: Recent advancements, product releases, and research in technology and artificial intelligence
   - RSS feed: `https://news.google.com/rss/search?q=artificial+intelligence+AI+technology&hl=en-US&gl=US&ceid=US:en`
   - Also try: "AI news", "technology news latest"
   - Freshness: last 24 hours (see Implementation Notes)

## How to Use

When the user requests news:

1. Identify which categories are needed. If none are specified, fetch all four.
2. Fetch RSS feeds in parallel using `terminal` with `curl` (see Implementation Notes).
3. Parse RSS XML to extract titles and URLs. Skip the first result (it's always "Google News").
4. Limit results to `count` items per category (default: 5).
5. Format each result as: `{number}. {title}` followed by a second line with the URL.
6. Present the compiled news summary under clear category headings with emoji.
7. If a category yields no results (excluding "Google News"), note: "No recent news found in this category."
8. If a category returns only "Google News" as the first item with nothing else, note that and continue.

### Optional Parameters

- `count`: Number of results to fetch per category (default: 5)
- `specific_categories`: A list of categories to fetch (if user specifies). Options: `["malaysia", "global", "iran_us", "tech_ai"]`

### Example Interactions

**User:** "Get me the latest news"
**Agent:** Fetches all four categories and presents a combined summary.

**User:** "What's happening in Malaysia?"
**Agent:** Fetches only Malaysia local news.

**User:** "Tell me about the Iran-US situation and any new AI developments."
**Agent:** Fetches Iran-US war and Technology & AI categories.

## Implementation Notes

### Hermes Tool Mapping (Critical)

The original skill referenced a `web_search` tool that does not exist in Hermes. Use the following approach:

**Primary: RSS via curl**
```
curl -s --max-time 10 "<RSS_FEED_URL>" | python3 -c "
import sys, re
content = sys.stdin.read()
titles = re.findall(r'<title>([^<]+)</title>', content)
urls = re.findall(r'<link>([^<]+)</link>', content)
for i, (t, u) in enumerate(zip(titles[1:CUTOFF], urls[1:CUTOFF])):
    print(f'{i+1}. {t}')
    print(f'   {u}')
"
```
Replace `CUTOFF` with `count + 1` (skip "Google News" at index 0).

**Alternative: Browser (if RSS fails or needs freshness filtering)**
1. Navigate to `https://news.google.com/search?q=<QUERY>&hl=en-US&gl=US&ceid=US:en`
2. Use `browser_snapshot()` or `browser_vision()` to extract results
3. Parse publish timestamps manually to filter to last 24h

### Freshness Filtering

Google News RSS does not expose publish dates in a machine-readable format. Known limitations:
- Results are ranked by relevance, not recency
- First 5-10 results are typically recent but not guaranteed within 24h
- For strict 24h enforcement: use Browser approach and parse timestamps from the page

### RSS Feed URL Construction

| Category    | Base URL                                                      | Query Param               |
|-------------|---------------------------------------------------------------|---------------------------|
| Malaysia    | `https://news.google.com/rss/search`                           | `q=Malaysia+news`          |
| Global      | `https://news.google.com/rss/search`                           | `q=global+news`            |
| Iran-US     | `https://news.google.com/rss/search`                           | `q=Iran+US+conflict`       |
| Tech/AI     | `https://news.google.com/rss/search`                           | `q=artificial+intelligence+AI+technology` |

Append `&hl=<locale>&gl=<region>&ceid=<region>:<locale>` for localization.

### Error Handling

| Scenario                          | Action                                                                 |
|-----------------------------------|------------------------------------------------------------------------|
| curl returns empty/error          | Retry once. If still fails, note "Could not fetch [Category] news."    |
| All results are "Google News"     | Note "No recent news found in this category."                          |
| Category yields 0 results         | Note "No recent news found in this category."                          |
| Network timeout                   | Retry once with `--max-time 15`. If still fails, skip category.        |
| XML parse fails                   | Fall back to Browser approach for that category.                        |

## Notes

- Fetch all categories in parallel for best performance.
- Do not hardcode article counts — respect the `count` parameter.
- Prefer broad queries to capture a range of news sources.
- If the user wants more depth on a specific topic, consider follow-up searches within that category.
- The skill does not currently support historical news beyond 24 hours (RSS limitation).
