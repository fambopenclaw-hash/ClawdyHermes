# Repo Migration Handling on GitHub Pages

## When GitHub says "This repository moved"

If `git push` succeeds but prints:

```
remote: This repository moved. Please use the new location:
remote:   git@github.com:NEW_OWNER/NEW_REPO.git
```

**Do not ignore it.** The next push will fail with the same warning. Fix immediately:

```bash
git remote set-url origin git@github.com:NEW_OWNER/NEW_REPO.git
```

Then update memory so future sessions don't hit the stale remote.

## URL changes after a rename

| Before | After |
|--------|-------|
| Repo `user/github.io` | Repo `user/reports` |
| URL `user.github.io/github.io/` | URL `user.github.io/reports/` |
| Remote `git@github.com:user/github.io.git` | Remote `git@github.com:user/reports.git` |

The GitHub API will still serve the old repo for a while due to forwarding, but all pushes must go to the new remote.

## CDN cache after deployment

Even when `curl -s` returns 200 with the new HTML, the browser may show stale DOM for 1-3 minutes. This is a GitHub Pages CDN quirk, not a deploy failure.

**Verification checklist:**
1. `curl -s -o /dev/null -w "%{http_code}"` → 200
2. `curl -s "https://..." | grep "unique-string-from-your-change"` → match found
3. Browser with `?nocache=1` → fresh DOM
4. Wait 60s and check again if still stale

## Real example (this user's setup)

- Old remote: `git@github.com:fahmiamni/github.io.git`
- New remote: `git@github.com:fahmiamni/reports.git`
- Published at: `https://fahmiamni.github.io/reports/`
- Local clone: `/tmp/fahmi_github_io/`
- The old `~/.openclaw/workspace/github.io/` clone still has the old remote — never push from there
