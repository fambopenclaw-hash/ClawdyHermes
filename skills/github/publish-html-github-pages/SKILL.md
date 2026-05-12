---
name: publish-html-github-pages
description: >-
  Publish a static HTML file to a GitHub Pages repository (username.github.io or 
  org.github.io) — copy to local clone, commit, push, verify deployment, and 
  optionally update the site's index page with a new card/link. Also handles 
  delivering the published URL to the user.
version: 1.0.0
author: Hermes Agent
---

# Publish HTML to GitHub Pages

## When to Use

Trigger this skill when:
- User asks to "update my github.io page", "publish to github pages", "put this on my site"
- After generating an HTML report from markdown (`markdown-to-html` skill), user wants it publicly available
- User wants to add a new HTML page to an existing `username.github.io` repository
- User asks to "deploy to GitHub Pages" a standalone HTML file

Do NOT use for:
- Creating or cloning repos (→ use `github-repo-management`)
- Setting up GitHub Pages for the first time (→ use `github-repo-management`)
- Converting markdown to HTML (→ use `markdown-to-html` first)

## Prerequisites

- The HTML file must already exist locally
- The GitHub Pages repository must already be cloned locally
- SSH or HTTPS auth to GitHub must be configured

## Workflow

### 1. Locate the GitHub Pages repo

First, find the user's GitHub Pages repo. Common locations and repo names:
- `~/.openclaw/workspace/github.io` (OpenClaw workspace convention)
- Search: `find ~ -maxdepth 4 -type d -name "*.github.io" 2>/dev/null` (username.github.io pattern)
- Search: `find ~ -maxdepth 4 -type d -name "github.io" 2>/dev/null` (plain github.io pattern)
- If not found locally, clone fresh: `gh repo clone owner/username.github.io` or `gh repo clone owner/github.io`

Check the remote to confirm:
```bash
cd /path/to/repo
git remote -v
# Should show: origin git@github.com:username/username.github.io.git
```

### 2. Check SSH auth

```bash
ssh -T git@github.com 2>&1 | head -3
# Expected: "Hi username! You've successfully authenticated..."
```

If SSH fails, suggest `github-auth` skill.

### 3. Copy the HTML file into the repo

```bash
cp "/path/to/source.html" /path/to/repo/
```

### 4. Ensure the new HTML file has a backlink to index.html

Before committing, the new HTML file MUST include a link back to `index.html` so users can navigate home. Common patterns:

```html
<!-- Option A: Back link in header banner area -->
<a class="back-home" href="index.html">Back to Home</a>

<!-- Option B: Back link in content area (most common) -->
<a href="index.html" class="back-link">← Back to Index</a>
```

Check which pattern the existing pages in the repo use (search for `back-home` or `back-link` or `index.html` in existing HTML files) and match the convention. Add the backlink near the top of the page body (header area) and optionally again in the footer.

### 5. Check if an index page exists

```bash
ls /path/to/repo/index.html 2>/dev/null
```

If the index page uses a card-grid style (common pattern), read it to understand the layout before adding a new card.

### 6. (Optional) Add a card to the index page

If the index page has a sectioned card layout:
1. Read the full `index.html` to understand its structure
2. Identify the most appropriate category section (e.g., Oil & Gas, Reports, AI)
3. Increment the category count (e.g., "2 reports" → "3 reports")
4. Add a new card `<a>` tag with:
   - `class="card accent-{color}"`
   - `href="filename.html"`
   - Title, description, and tag text consistent with existing cards
5. Use `patch` to add the new card at the top of the card grid (before the first existing card)

### 6. Commit and push

```bash
cd /path/to/repo
git add "filename.html"
# If index was updated:
git add index.html
git commit -m "Add: {description of report} report"
git push
```

If push fails with "rejected" error:
```bash
git pull --rebase origin main
git push
```

### 7. Verify deployment

GitHub Pages takes ~20-30 seconds to deploy. Verify:
```bash
sleep 30
curl -s -o /dev/null -w "%{http_code}" "https://{username}.github.io/{repo-name}/{filename}"
# Expected: 200
```

Also verify the index page if updated:
```bash
curl -s -o /dev/null -w "%{http_code}" "https://{username}.github.io/{repo-name}/"
# Expected: 200
```

### 8. Construct and deliver the URL

The URL format depends on the repo name:
- `username/username.github.io` → `https://username.github.io/filename.html`
- `username/github.io` → `https://username.github.io/github.io/filename.html`
- `username/my-site` → `https://username.github.io/my-site/filename.html`

Deliver the URL to the user via the current platform.

## URL Format

| Repo | Published URL |
|------|--------------|
| `user/user.github.io` | `https://user.github.io/file.html` |
| `user/github.io` | `https://user.github.io/github.io/file.html` |
| `user/docs` | `https://user.github.io/docs/file.html` |

## Pitfalls

- **Spaces in filenames:** If the source HTML file's path has spaces, use quotes or copy to `/tmp/` first before moving to the repo
- **Push rejection:** Always `git pull --rebase` before retrying — the remote may have changed
- **GitHub Pages delay:** Deployment takes ~20-30 seconds — don't report "404" immediately, wait and retry
- **No `gh` CLI:** The skill works with plain `git` + SSH — no `gh` CLI required
- **Card placement:** When adding to index, add the new card at the TOP of the card grid (first position) so it's the most recently added item
