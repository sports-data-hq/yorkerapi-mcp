---
name: cricket-match-preview
description: "Use when the user wants a cricket match preview in any format, or how a ground plays before a match: 'preview India vs Australia ODI', 'IPL preview MI vs CSK', 'PSL match breakdown', 'who's favored in the Big Bash tonight', 'how does this ground play', 'T20 World Cup match preview'. Produces a one-page report: format ratings and our pregame win probability (T20 and international ODI), venue profile, recent results, key players' form and the market's prices. Not for ball-by-ball replays (call get_deliveries) or modeling (see model-building)."
metadata:
  version: 2.0.0
  author: SportsDataHQ
---

# Cricket match preview

A one-page preview of one match from live YorkerAPI calls, in any format (Test, ODI, T20 and franchise leagues).

## Before you start
- For anything across games, seasons or players, apply the sports-data-preflight checks first (ids from search, the season convention, one league, enough data).
- Tools come from the YorkerAPI MCP server (`https://mcp.yorkerapi.com/mcp`). Call them; never recall a score.
- Seasons are a year (2024) or a split season (2024/25). Competitions by key (ipl, bbl, psl, blast, hundred, test, odi, t20i...): `list_competitions` (1 cr).
- Our pregame win probability is served for T20 and international ODIs only (Tests and first-class have draws; domestic one-day ratings are too weak). Ratings are per format.

## Default path (about 22 credits)
1. **Find the match.** `get_schedule(competition, days)` (2 cr): start time, teams, venue, each side's rating, our pregame win probability (when served) and the market favourite.
2. **Ratings.** `get_ratings(format)` (2 cr): both sides' Elo in this format.
3. **The ground.** `get_venue(venue, format)` (5 cr): average first and second innings totals, how often the side batting first wins, toss wins and choices.
4. **Recent results.** `get_matches(team_id, competition)` (2 cr) for both: last 5-10 results.
5. **Key players (optional).** `get_player_stats(player_id, format, competition)` (5 cr) for 2-4 names; for a batter vs bowler duel, `get_matchup(batter_id, bowler_id, format)` (5 cr) and only trust it at 30+ balls.
6. **Prices (optional).** `get_odds(match_id)` (5 cr).

## Output template
```
<Team 1> vs <Team 2> | <competition, format> | <venue> | <start>
Ratings (<format>): <t1> vs <t2> | Our pregame win probability: <t1 x%> / <t2 y%> (or "not served for this format")
Market: favourite <team> at <price>
Venue: par 1st inns <n>, batting first wins <x%> of <n> matches, toss winners choose <bat/bowl>
Form: last 5 for both
Players to watch: <name: format stats with innings/balls>
Data: get_schedule, get_ratings, get_venue, get_matches | <competition>, <season>
```

## When there's no game

If the game the user names isn't scheduled (off-season, not booked, already played), don't stop to ask. Do the most useful real version and say which one in the first line:
1. The next scheduled meeting of those two sides, if `get_schedule` (filtered by team) has one.
2. Otherwise their most recent meeting and where each side finished last season, from the last completed season's data.
3. For a matchup that can't happen soon, the ratings matchup labelled as hypothetical.
Answer first; offer the alternative in one closing line, never as a question before answering.

## Rules
- Say "our model" for our probability and "the market" for prices.
- Venue numbers need matches: show the count; under 10 matches at a ground, say it's thin.
- Weather and the toss can swing a match; note when the format makes the toss matter (venue toss data).
- Odds are information about the market: no pick, stake or bet.
- Complete before writing: every step in the default path is called for both sides before the preview is written. If a call fails or returns nothing, say which section is missing rather than writing around it.
- Sequences (last 5, recent form, a run of results) are quoted in the order the tool returns them, with dates; never retell them from memory. Lists in the answer must match the tool rows.
- Out of credits: if a call is refused for credits, pass on the API's message as it comes (it says when free credits refill and how to add more), and stop; don't retry or switch tools to get around it.