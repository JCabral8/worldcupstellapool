# Scoring & Live Standings

How the World Cup Stella Pool is scored, and how to keep the leaderboard live
during the group stage.

## The rule

**3 points for each correct finish position.**

Every entry ranks all four teams in each of the 12 groups (1st → 4th). For each
group, each position where your pick matches the actual table earns 3 points.

- Max per group: 4 × 3 = **12 points**
- Max overall: 12 groups × 12 = **144 points**

**Tiebreakers** (in order):

1. Most points
2. Alphabetical (placeholder — swap in a coin flip / head-to-head if you prefer)

(Group winners are not tracked separately — only total correct positions matter.)

The same engine does both **live** (provisional tables during the group stage)
and **final** scoring — "live" just means `results.json` holds today's standings
instead of the finished ones.

## How scoring runs

The site (`index.html`) scores entirely in the browser, read-only:

1. It reads everyone's picks from `entries.json`.
2. It reads the current standings from `results.json`.
3. It computes points and renders the **🏆 Scores** view (leaderboard +
   a "Pick vs. Result" grid where correct positions are highlighted green).

No credentials live in the page. Scoring needs no write access — it only reads
the two public JSON files over the CDN.

## Keeping it live (daily)

Edit **`results.json`** whenever standings change and the leaderboard updates on
the next page load (the site cache-busts each fetch).

```json
{
  "updated": "Jun 18 (MD3)",
  "final": false,
  "standings": {
    "A": ["MEX", "KOR", "CZE", "RSA"],
    "B": ["SUI", "CAN", "BIH", "QAT"],
    "C": []
  }
}
```

- `standings.<GROUP>` is the current table order, 1st → 4th, using the
  **team codes** below.
- A group with `[]` (or omitted) is treated as **not yet scored** — it simply
  doesn't contribute points until you fill it in. So mid-group-stage you can
  post only the groups that have meaningful tables.
- Set `"final": true` once the group stage is complete — the view switches its
  label from *Live* to *Final*.
- `updated` is a free-text "as of" label shown on the scoreboard.

### Two ways to update

- **Manual (recommended for a small pool):** edit `results.json` on
  GitHub (web UI → edit file → commit to `main`). Zero infrastructure, fully
  reliable, no API keys.
- **Automated:** a scheduled GitHub Action can pull standings from a football
  data API once a day and commit `results.json`. More setup (API key as a repo
  secret, network access); only worth it if you don't want to touch it daily.

> Source of truth is whatever the official FIFA table says (points → goal
> difference → goals scored → head-to-head → fair-play → drawing of lots).
> Just mirror that order into `standings`.

## Team codes

| Group | Teams (code) |
|-------|--------------|
| A | MEX, RSA, KOR, CZE |
| B | CAN, BIH, QAT, SUI |
| C | BRA, MAR, HAI, SCO |
| D | USA, PAR, AUS, TUR |
| E | GER, CUW, CIV, ECU |
| F | NED, JPN, SWE, TUN |
| G | BEL, EGY, IRN, NZL |
| H | ESP, CPV, KSA, URU |
| I | FRA, SEN, IRQ, NOR |
| J | ARG, ALG, AUT, JOR |
| K | POR, COD, UZB, COL |
| L | ENG, CRO, GHA, PAN |

## Security note

This page used to ship a GitHub write token in client-side JavaScript so the
browser could append entries to `entries.json`. That token was public to anyone
who opened the page and has been **removed**. If it hasn't been revoked yet:

1. GitHub → **Settings → Developer settings → Personal access tokens**
2. Find the token starting `github_pat_11ABUEACI0…` and **Revoke** it.

Since picks lock at kickoff, the site no longer needs to write anything — both
`entries.json` and `results.json` are edited server-side (GitHub web UI or an
Action) and read-only in the browser.
