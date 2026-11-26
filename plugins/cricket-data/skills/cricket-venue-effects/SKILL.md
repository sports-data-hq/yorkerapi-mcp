---
name: cricket-venue-effects
description: "Use when the user asks how a cricket ground or pitch plays, in fan or analyst wording: 'what is a good score at Lord's', 'bat or bowl first at Mohali', 'does it turn here', 'seamers' paradise?', 'high scoring ground?', 'do chasers win at Sharjah', 'how do captains call at the toss', 'is Eden Gardens a flat deck', 'should we bat first', 'does the pitch help spinners', 'does it favor the toss winner'. Use this whenever a ground, stadium or pitch is the subject. Also use it for bare follow-ups with no name given, and load it instead of answering from memory or going straight to the raw tools. Reports innings totals, batting-first and chasing results, toss choices and phase scoring for a ground and format, with match counts. Not for a full match preview (see cricket-match-preview), a live chase's probability (see cricket-chase-win-probability), a player's phase stats (see cricket-phase-analysis), weather or capacity."
metadata:
  version: 2.1.0
  author: SportsDataHQ
---

# Cricket venue effects

Grounds differ: size, pitch and conditions change par scores and what the toss is worth. This skill reports what a ground has produced, from YorkerAPI venue and match data, with the sample size beside every figure.

## Before you start
- For anything across games, seasons or players, apply the sports-data-preflight checks first (ids from search, the season convention, one league, enough data).
- Tools come from the YorkerAPI MCP server (`https://mcp.yorkerapi.com/mcp`). Call them; never fill in a number from memory.
- Seasons are a year (2024) or a split season (2024/25). Competitions by key (ipl, bbl, psl, test, odi, t20i and so on) from `list_competitions` (1 cr). Ratings and probabilities are per format.
- `get_venue(venue, format)` returns the ground's profile: average first and second innings totals, how often the side batting first wins, toss wins and choices. Read the fields it returns before quoting.
- Pace-versus-spin splits, weather, pitch reports and boundary dimensions are not in the data unless a tool returns them. Do not say a ground helps spin without a returned figure that shows it; at most, compare wickets by type if `get_deliveries` rows carry bowler type, and say you computed it.
- A ground's numbers mix competitions and years. Filter by competition and recent seasons when the tool allows, and say what you filtered.

## Default path (about 10 to 20 credits, plus any ball-by-ball pulls)
1. **Find the ground and format.** `list_competitions` (1 cr); the venue name from `get_schedule(competition, days)` (2 cr) or `get_matches` (2 cr).
2. **The profile.** `get_venue(venue, format)` (5 cr): average first and second innings totals, batting-first win rate, toss behaviour, matches counted.
3. **Recent matches.** `get_matches(competition, venue)` (2 cr): the last 5 to 10 results and totals, to see whether the profile still holds.
4. **Phases at the ground (optional).** `get_deliveries` for those matches: powerplay and death-over scoring computed by phase; say it is computed.
5. **Match-up (optional).** For a named side, results at the ground from `get_matches(team_id)` (2 cr).

## Reading a ground
Par is the typical first-innings total, not a target: show the average and the spread. A batting-first win rate near 50% means the toss is not worth much; far from it, say which way and over how many matches. Under 10 matches, call the ground thin and give the matches rather than a rate. Compare recent seasons against the full record: a re-laid pitch or new rules can change a ground.

## Output template
```
<Venue> | <format, competition, seasons> | matches <n>
Average first innings <v> | second innings <v> | spread <min-max>
Batting first wins <%> of <n> | chasing wins <%>
Toss: winners choose <bat/field> <%> | toss winner wins <%>
Recent <k> matches: <result, totals>
Phase scoring (optional, computed): powerplay <rpo> | middle <rpo> | death <rpo>
Read: 2-3 bullets with match counts (thin if under 10)
Data: get_venue, get_matches[, get_deliveries] | <competition>, <season>
```

## Rules
- Show the number of matches behind every venue figure.
- Do not describe pitch or weather behaviour that no tool returned.
- State filters (competition, seasons) so the figures can be reproduced.
- Every number names its tool (the Data line, plus inline where it could be doubted) and every rate shows its sample size.
- If a tool returns too little, say "not enough data" and stop; do not estimate or borrow another sample.
- Odds and prices are information about the market only: no picks, leans, stakes, payouts or sportsbook links.
- Sequences (last 5, recent form, a run of results) are quoted in the order the tool returns them, with dates; never retell them from memory. Lists in the answer must match the tool rows.
- Out of credits: if a call is refused for credits, pass on the API's message as it comes (it says when free credits refill and how to add more), and stop; don't retry or switch tools to get around it.

## Example prompts
- "How does Wankhede play in the IPL?"
- "What is the par score at Chepauk?"
- "Should teams bat first in Dubai?"
