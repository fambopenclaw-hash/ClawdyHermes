# Global News Fallback — Noise Filtering Guide

## Why This Is Needed

The fallback query `q=world+news+international` (used when `q=global+news` is
dominated by the Canadian "Global News" TV network) has its own problem: it
returns a flood of non-news content — press releases, government PR, university
announcements, sports, gaming, and travel-industry puff pieces.

Without aggressive filtering, the "Global" category is filled with irrelevant
items even on the fallback query.

## Observed Noise Patterns (from live RSS dumps)

### 1. City/State Government Press Releases
- "City of Austin Promotes 'Live Music Capital of the World' on International Stage"
- "State of California attracts global investment through coordinated international engagement at the 2026 SelectUSA Investment Summit"

### 2. University Announcements
- "RIT students experience global learning with international fellowships"

### 3. Sports Articles
- "These Bruins Are Skating in the World Championship Quarterfinal" (NHL)
- "Germany win Alpine International Baseball Trophy undefeated"
- "Major champions Watson & Garcia headline world-class International Series field in Morocco"
- "No league goals is no World Cup worry for South Korea skipper Son"
- "World Cup notes: Messi among injured international stars"
- "Photos of tiny Curacao making World Cup soccer history"

### 4. Gaming / Entertainment
- "EA Sports FC 26 adds new International Tournament mode in The World's Game update"
- "Football Manager 26's International Management is out now for the World Cup"
- "Coming-of-age film 'The World of Love' invited to Shanghai International Film Festival"
- "Beauty of Joseon launches global campaign with New York pop-up"

### 5. Travel / Aviation Industry PR
- "Vinci Airports Commits to Transform Monterrey International Airport in Mexico Into a World-Class Hub Ahead of FIFA World Cup"
- "Aviation groups launch regional air traffic initiative ahead of FIFA World Cup 2026"
- "International airlines urged to stick to safety measures in wake of Ebola outbreak" (this one IS real news — don't filter all aviation topics)

### 6. Other Filler
- "History is made: Class of 2026 takes its place in International Swimming Hall of Fame"
- "World News in Brief: Sudan and Haiti updates, Afghan women's rights" (actually real news — don't filter all "World News" titles)

## What to Keep

Legitimate global news articles come from:
- Major wire services (Reuters, AP, AFP)
- Reputable news outlets (NYT, BBC, The Guardian, The Independent, LA Times, etc.)
- UN / WHO / Amnesty International / ICRC / other international orgs
- Coverage of actual geopolitical events, conflicts, treaties, elections, disasters

## Python Filtering Heuristic

```python
NOISE_SOURCES = [
    "city of austin", "california governor", "selectusa",
    "rochester institute", "rit students",
    "nhl.com", "world championship", "world cup", "ea sports",
    "football manager", "international series field",
    "international baseball", "world aquatics",
    "international swimming", "world of love",
    "international film festival", "fifa world cup",
    "international airport review", "international tournament mode",
    "altchar", "golf digest", "travel and tour world",
    "the sixth axis", "the lufkin", "wbsc", "korea joongang",
    "international management", "international fellowships",
    "international engagement", "international investment",
    "international fundraising", "boston entrepreneur",
    "foster care awareness", "champions world international",
    "vinci airports", "monterrey international airport",
    "world's busiest airport", "delta juggles",
    "tracking trump's crackdown", "us news & world report",
    "bbc refusing", "world cup halftime show",
    "class of 2026 takes its place in",
]

# Check both title and source domain
skip = any(phrase in raw_title.lower() for phrase in NOISE_SOURCES)
```

## Pro Tip

If after aggressive filtering you get < 3 items, try yet another fallback:
`q=latest+world+events+geopolitics` or `q=world+news+conflicts+dipomacy`
