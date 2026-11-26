---
name: cricket-chase-win-probability
description: "Use when the user wants the win probability of a cricket run chase or how it swung: 'what are the chances of this chase', 'win probability at the 15th over', 'how did the match turn', 'was the chase ever in doubt', 'the biggest swing of the innings', 'what does the model give the chasing side'. Reports our win probability through the second innings (get_win_probability), the swings between overs, the required rate, and compares it with the market's view as information. Not for a pre-match preview (see cricket-match-preview), phase stats (see cricket-phase-analysis) or ground effects (see cricket-venue-effects)."
metadata:
  version: 2.0.0
  author: SportsDataHQ
---

# Cricket chase win probability

A chase is a stream of small decisions, and a win probability curve shows where it turned. This skill reads our win probability for a match and explains the swings from the balls that caused them.

## Before you start
- For anything across games, seasons or players, apply the sports-data-preflight checks first (ids from search, the season convention, one league, enough data).
- Tools come from the YorkerAPI MCP server (`https://mcp.yorkerapi.com/mcp`). Call them; never fill in a number from memory.
- Seasons are a year (2024) or a split season (2024/25). Competitions by key (ipl, bbl, psl, test, odi, t20i and so on) from `list_competitions` (1 cr). Ratings and probabilities are per format.
- `get_win_probability` returns our model's win probability for a match through its course. Read the fields the tool returns (how it is indexed: by over, ball or innings stage) before quoting any; if the model is not served for a format or competition, say so. Our pregame probability is served for T20 and international ODIs; Tests and first-class cricket have draws and are not covered the same way.
- The required run rate and balls remaining are computed here from the target and the score: required rate = runs needed / balls remaining x 6. Say it is computed.
- For a match still in progress, use only what the tool returns now; do not project a score yourself.

## Default path (credits as listed by each tool)
1. **Find the match.** `get_matches(competition, team_id)` (2 cr) for a completed match, or `get_schedule(competition, days)` (2 cr) for one upcoming or in progress.
2. **The curve.** `get_win_probability(match_id)`: our win probability for the batting side through the chase.
3. **What swung it.** `get_deliveries` for the overs where the curve moved most: wickets, boundaries, expensive overs.
4. **The market (optional).** `get_market_vs_model(match_id)` or `get_odds(match_id)` (5 cr): the market's price against ours, reported side by side as information.
5. **Context (optional).** `get_venue(venue, format)` (5 cr): how often chasing sides win here, with the number of matches.

## Reading the curve
Report the starting probability (before the first ball of the chase), the lowest and highest point, and the three largest swings, each with the over and the balls that caused it. A probability is not a prediction of the result: a side at 10% wins about one chase in ten. Early in the innings the curve is mostly the target and the pitch; late in the innings it is mostly wickets in hand and balls left.

## Output template
```
<Chasing side> chasing <target> vs <Bowling side> | <competition, format> | <venue>
Start of chase: <x%> | low <x%> at over <o> | high <x%> at over <o> | final <result>
Largest swings: <over: change, caused by ...> x3
Required rate at key points: <over: runs needed, balls left, required rate> (computed)
Market (optional): <side %> vs ours <x%> (information only)
Venue: chasing side wins <%> of <n> matches here
Data: get_matches, get_win_probability, get_deliveries[, get_odds, get_venue] | <competition>, <season>
```

## Rules
- Say "our model" for the curve and "the market" for prices; never blend them.
- A probability describes a distribution of outcomes, not a certainty; do not call a chase "over" at 95%.
- Venue chase records need a match count; under 10 matches at a ground is thin.
- Every number names its tool (the Data line, plus inline where it could be doubted) and every rate shows its sample size.
- If a tool returns too little, say "not enough data" and stop; do not estimate or borrow another sample.
- Odds and prices are information about the market only: no picks, leans, stakes, payouts or sportsbook links.
- Out of credits: if a call is refused for credits, pass on the API's message as it comes (it says when free credits refill and how to add more), and stop; don't retry or switch tools to get around it.

## Example prompts
- "How did the chase swing in the IPL final?"
- "What are the chances of this chase after 15 overs?"
- "Was the chase ever in doubt?"
