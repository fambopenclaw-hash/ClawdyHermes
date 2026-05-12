---
name: github-alternatives-discovery
description: "Discover alternative/similar open-source GitHub projects for a given tool, library, or platform. Search by topic, description, keywords, and star ranking."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [GitHub, Discovery, Alternatives, Research, Open-Source]
---

# GitHub Alternatives Discovery

Find competing or similar open-source projects on GitHub for any given tool, library, platform, or framework.

## When to Use

- User asks: "What are alternatives to X?" / "Find projects similar to Y"
- User asks: "List top projects like Z" / "GitHub repos similar to X"
- User asks: "What else is in this space?" / "Competitors to X on GitHub"
- Any request to discover, compare, or rank open-source projects in a category
- Researching the landscape of an open-source ecosystem

## Procedure

### 1. Understand the Reference Project

First, fetch the reference project's details from the GitHub API:

```bash
curl -s "https://api.github.com/repos/owner/repo" | python3 -c "
import json, sys
d = json.load(sys.stdin)
print('Stars:', d.get('stargazers_count'))
print('Description:', d.get('description'))
print('Topics:', d.get('topics'))
print('Language:', d.get('language'))
print('License:', d.get('license', {}).get('spdx_id', 'N/A') if d.get('license') else 'N/A')
"
```

Extract key features: description keywords, topics, language, category (coding agent? personal assistant? dev tool?).

### 2. Craft Search Queries

Use multiple angles to find relevant alternatives. The GitHub search API endpoint:

```
https://api.github.com/search/repositories?q=KEYWORDS&sort=stars&order=desc&per_page=15
```

**Query strategies (try in order):**

a) **Topic match** — Use key topics from the reference project:
```
q=AI+coding+agent+terminal+CLI&sort=stars&order=desc
```

b) **Description/category match** — Broader category search:
```
q=personal+AI+assistant+self-hosted&sort=stars&order=desc
```

c) **Known competitors by name** — Fetch specific repos individually:
```
for repo in "owner1/repo1" "owner2/repo2"; do
  curl -s "https://api.github.com/repos/$repo"
done
```

d) **Feature-specific** — If the project has distinctive features (multi-channel, voice, gateway):
```
q=multi-channel+AI+assistant+gateway&sort=stars&order=desc
```

### 3. Parse and Rank Results

Parse search results to extract name, stars, description, topics, language:

```python
python3 -c "
import json, sys
data = json.load(sys.stdin)
for r in data.get('items', []):
    print(f\"{r['full_name']} ⭐{r['stargazers_count']:,}\")
    print(f\"  {r['description'] or 'No description'}\")
    print(f\"  Topics: {', '.join(r.get('topics',[])[:6]) or 'None'}\")
    print(f\"  {r['html_url']}\")
"
```

**Ranking criteria:**
1. **Relevance** — Does it solve the same problem? Same category/niche?
2. **Stars** — Community adoption signal
3. **Active development** — Recent commits, releases
4. **Feature overlap** — Similar architecture, platform support, integrations

### 4. Present Results

Format as a top N list (default: top 5). For each entry include:
- Name with link and star count
- One-line summary of what it does
- 1-2 sentence explanation of **why** it's similar (feature parity, same category, same philosophy)

Exclude the reference project itself from the list. If the user asked about a project that's already in your skills, load the skill first to understand it properly before searching.

### 5. Handle Edge Cases

- **No results**: Try broader queries, drop overly specific keywords, or check if the project name is a known alias
- **Rate limiting**: GitHub unauthenticated API has 60 req/hr. If you hit limits, suggest the user provides a `GITHUB_TOKEN`
- **Zero-star projects**: Filter out repos with < 10 stars unless specifically looking for niche/early-stage alternatives
- **Fake/unofficial repos**: Watch out for "leaked source" repos or obvious spam copies

## Pitfalls

- Don't just search the project's own name — that finds only the project itself. Search its **category** and **keywords**.
- Don't confuse "similar tech stack" with "similar product." Same language ≠ same category.
- The GitHub API unauthenticated rate limit is 60 requests/hour. Be efficient — batch lookups where possible.
- Parse `stargazers_count` from the top-level response, not `watchers_count` (which may differ).
- Some repos use the same name as the target project but are unrelated — verify the description.

## Verification

Did you return a list of 5+ alternatives with:
- [ ] Star counts
- [ ] Link to each repo
- [ ] Explanation of why each is similar
- [ ] No duplicates or irrelevant results
