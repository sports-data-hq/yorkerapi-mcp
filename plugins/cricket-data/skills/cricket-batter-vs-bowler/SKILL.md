---
name: cricket-batter-vs-bowler
description: "Use when the user asks how a cricket batter fares against a bowler or a kind of bowling, in fan or analyst wording: 'Root vs Jadeja', 'does Smith handle Archer', 'is he scared of left-arm pace', 'struggles against leg spin?', 'who keeps getting Rohit out', 'his bunny in T20s', 'Babar versus Naseem', 'any good against turners', 'does he fear pace', 'who has dismissed him the most times'. Use this whenever a batter and bowler, or a batter and a bowling type, are mentioned together. Also use it for bare follow-ups with no name given, and load it instead of answering from memory or going straight to the raw tools. Reports balls, runs, dismissals and strike rate for the pair or type, and says plainly when the sample is too small. Not for a match preview (see cricket-match-preview), a phase split (see cricket-phase-analysis), ground effects (see cricket-venue-effects) or chase win probability (see cricket-chase-win-probability)."
metadata:
  version: 2.0.0
  author: SportsDataHQ
---

# Cricket batter vs bowler

Duel records are small and easy to over-read. This skill reports a batter's results against a bowler with the balls counted, and makes no claim under 30 balls.

## Before you start
- For anything across games, seasons or players, apply the sports-data-preflight checks first (ids from search, the season convention, one league, enough data).
- Tools come from the YorkerAPI MCP server (`https://mcp.yorkerapi.com/mcp`). Call them; never fill in a number from memory.
- Seasons are a year (2024) or a split season (2024/25). Competitions by key (ipl, bbl, psl, test, odi, t20i and so on) from `list_competitions` (1 cr). Ratings and probabilities are per format; a duel record is per format too, so do not mix them without saying.
- `get_matchup(batter_id, bowler_id, format)` (5 cr) is the duel record. Read the field names it returns before using them (balls, runs, dismissals, dot balls, boundaries); if a field is absent, say so.
- Players by `search_players(name)`. Check the id belongs to the player the user means.
- Handedness, bowling type, pace versus spin and bowling speed are not claimed unless a tool returns them. A "how does he play spin" request needs `get_deliveries` rows that carry bowler type; if they do not, say it is not in the data.

## Default path (about 5 to 25 credits)
1. **Resolve both players.** `search_players(name)` for the batter and the bowler.
2. **The duel.** `get_matchup(batter_id, bowler_id, format)` (5 cr). Compute strike rate = runs / balls x 100 and balls per dismissal; say they are computed if not returned.
3. **Check the sample.** Under 30 balls: report the balls, runs and dismissals as a record, say "not enough data" for any claim about who has the edge, and stop. At 30 balls or more, continue.
4. **Context.** `get_player_stats(player_id, format, competition)` (5 cr) for each player's overall numbers, so the duel can be set against the batter's overall strike rate and the bowler's overall economy.
5. **Where the dismissals came (optional).** `get_deliveries` for those matches, if a duel filter exists, to see overs and phases; say it was computed.

## Reading it
A duel under 30 balls is a record, not a pattern; between 30 and 60 balls say the edge is tentative. Dismissals are rare events, so a bowler who has dismissed a batter twice in 40 balls has not shown he owns him. Compare the duel strike rate with the batter's overall rate in the same format before saying he struggles or dominates.

## Output template
```
<Batter> vs <Bowler> | <format, competition if filtered> | balls <n>
Record: runs <r>, dismissals <d>, strike rate <v> (computed), balls per dismissal <v>, dot-ball % <v>
Batter overall (same format): strike rate <v> | Bowler overall: economy <v>
Read: <record only if balls >= 30; otherwise "not enough data (<n> balls)">, with what the sample cannot say
Data: search_players, get_matchup, get_player_stats | <format>, <competition>
```

## Rules
- Minimum 30 balls before any claim about who has the edge; under that, report the record and say not enough data.
- Say strike rate and balls per dismissal were computed if the tool does not return them.
- Do not discuss pace, spin, handedness or speed unless a tool returned them.
- Every number names its tool (the Data line, plus inline where it could be doubted) and every rate shows its sample size.
- If a tool returns too little, say "not enough data" and stop; do not estimate or borrow another sample.
- Odds and prices are information about the market only: no picks, leans, stakes, payouts or sportsbook links.
- Out of credits: if a call is refused for credits, pass on the API's message as it comes (it says when free credits refill and how to add more), and stop; don't retry or switch tools to get around it.

## Example prompts
- "How has Kohli fared against Starc in ODIs?"
- "Has Bumrah dismissed Head in T20s?"
- "Record of Smith against Ashwin in Tests"
