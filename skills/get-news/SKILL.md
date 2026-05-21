---
name: get-news
description: Fetches and summarizes recent news from Malaysia (local), global, Iran-US conflict, and technology/AI categories within the last 24 hours. Use when the user requests news updates or summaries in any of these domains. Provides a standardized approach to gathering timely news with consistent formatting and category-specific search strategies.
---

# Get News

## Overview

The get-news skill retrieves and summarizes the latest news from four key categories: Malaysia local news, global news, Iran-US war developments, and technology/AI news. It ensures fresh content (within 24 hours) and presents it in a clear, categorized format with detailed summaries.

## When to Use This Skill

Trigger the skill when the user requests news updates or summaries in any of these four domains.

## Capabilities

The skill supports fetching news for the following categories:

1. **Malaysia Local News**
   - Focus: National and regional events within Malaysia
   - RSS feed: `https://news.google.com/rss/search?q=Malaysia+news&hl=en-MY&gl=MY&ceid=MY:en`

2. **Global News**
   - Focus: Major world events and international developments
   - RSS feed: `https://news.google.com/rss/search?q=global+news&hl=en-US&gl=US&ceid=US:en`

3. **Iran-US War**
   - Focus: Latest developments in the conflict between Iran and the United States
   - RSS feed: `https://news.google.com/rss/search?q=Iran+US+conflict&hl=en-US&gl=US&ceid=US:en`

4. **Technology & AI**
   - Focus: Recent advancements, product releases, and research in technology and artificial intelligence
   - RSS feed: `https://news.google.com/rss/search?q=artificial+intelligence+AI+technology&hl=en-US&gl=US&ceid=US:en`

## How to Use

When the user requests news:

1. Identify which categories are needed. If none are specified, fetch all four.
2. Fetch RSS feeds in parallel using `terminal` with `curl`.
3. Parse RSS XML to extract titles (skip the first "Google News" item).
4. Limit results to `count` items per category (default: 5).
5. Generate a **lengthy, detailed summary (3-4 sentences)** for each article by extracting core information through deep analysis.
6. Format each result as:
   `{number}. {title}`
   `Summary: {detailed_summary}`

### Optional Parameters

- `count`: Number of results to fetch per category (default: 5)
- `specific_categories`: A list of categories to fetch (if user specifies). Options: `["malaysia", "global", "iran_us", "tech_ai"]`

## Implementation Notes

### Error Handling

| Scenario                          | Action                                                                 |
|-----------------------------------|------------------------------------------------------------------------|
| Summary generation fails          | Provide title; denote "Summary unavailable."                          |
