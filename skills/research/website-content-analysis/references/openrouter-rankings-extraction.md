# OpenRouter Rankings Extraction (2026-05-24)

## Context
OpenRouter's /rankings page (https://openrouter.ai/rankings) is a Next.js SPA. All data loads client-side. `browser_snapshot` showed only "Loading rankings chart" and "Loading rankings chart and rows" — no actual data visible.

## Technique Used
Async JavaScript injection via `browser_console` to wait for client-side rendering, then extract the full rendered DOM text.

## Exact Steps

### Step 1: Initial landing
```
browser_navigate(url="https://openrouter.ai/rankings")
```

### Step 2: Check snapshot
```
browser_snapshot()
```
→ Shows "Loading rankings chart and rows" — dynamic content.

### Step 3: Extract rendered text
```
browser_console(
  expression="(async () => { await new Promise(r => setTimeout(r, 5000)); console.log(document.body.innerText); })()"
)
```

This waits 5 seconds for client-side data to load, then logs the full page text.

### Step 4: Read the output
```
browser_console()
```

## Result
The 5-second wait was sufficient. The console output contained the full LLM Leaderboard with ranked models including:
- Rank number
- Model name
- Provider name
- Token count (e.g., "3.21T tokens")
- Growth percentage (e.g., "73%")

## Data Retrieved
**Top 10 by weekly token usage (This Week)**:
1. DeepSeek V4 Flash — 3.21T tokens (+73%)
2. Hy3 preview (Tencent) — 3.08T tokens (+16%)
3. Claude Opus 4.7 (Anthropic) — 1.84T tokens (+18%)
4. Claude Sonnet 4.6 (Anthropic) — 1.82T tokens (+18%)
5. Owl Alpha (OpenRouter) — 1.15T tokens (+46%)
6. Gemini 3 Flash Preview (Google) — 1.13T tokens (0%)
7. DeepSeek V4 Pro — 1.03T tokens (+17%)
8. DeepSeek V3.2 — 1.03T tokens (+3%)
9. Step 3.5 Flash (Stepfun) — 728B tokens (+10%)
10. Kimi K2.6 (Moonshot AI) — 675B tokens (+41%)

## Key Observations
- 5 seconds was the right delay for this page. Too short (< 2s) and the data wouldn't load.
- The page also had a Top Models chart and Market Share section — those needed separate extraction.
- The API endpoint `/api/v1/models` exists but returns the full model catalog unsorted, not the rankings.
- Next.js dpl (deployment) hash in URLs changes per deployment — don't try to fetch data build manifests directly.
