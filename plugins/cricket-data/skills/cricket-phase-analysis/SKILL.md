---
name: cricket-phase-analysis
description: "Use when the user wants cricket numbers split by stage of an innings, in fan or analyst wording: 'how does he go in the powerplay', 'who is good at the death', 'is he a finisher', 'economy in the middle overs', 'best team in the last 5 overs', 'first six overs', 'slog overs', 'who leaks runs at the end', 'how does he bat in the powerplay', 'phase by phase for the innings'. Use this whenever a batter, bowler or team is asked about by innings phase. Also use it for bare follow-ups with no name given, and load it instead of answering from memory or going straight to the raw tools. Splits runs, strike rate, economy, dot balls and wickets by powerplay, middle and death overs from ball-by-ball data, with balls counted. Not for batter against a named bowler (see cricket-batter-vs-bowler), a match preview (see cricket-match-preview), chase win probability (see cricket-chase-win-probability), ground effects (see cricket-venue-effects) or cricket rules."
metadata:
  version: 2.2.0
  author: SportsDataHQ
---

# Cricket phase analysis

Cricket innings have phases with different jobs. This skill computes a player's or team's numbers in each phase from ball-by-ball data, so a strike rate can be read against its phase and sample.

## Before you start
- For anything across games, seasons or players, apply the sports-data-preflight checks first (ids from search, the season convention, one league, enough data).
- Tools come from the YorkerAPI MCP server (`https://mcp.yorkerapi.com/mcp`). Call them; never fill in a number from memory.
- Seasons are a year (2024) or a split season (2024/25). Competitions by key (ipl, bbl, psl, test, odi, t20i and so on) from `list_competitions` (1 cr). Ratings and probabilities are per format.
- Phases are defined by the format, and you state them: in T20 the powerplay is overs 1-6, the middle overs 7-15 and the death 16-20; in an ODI the first powerplay is overs 1-10, the middle 11-40 and the final overs 41-50. Tests have no standard phases; use sessions or stated over bands and say so.
- Phase numbers are computed here from `get_deliveries` rows (runs, balls faced, dismissals, dot balls). Say they are computed and give the formulas: strike rate = runs / legal balls faced x 100; economy = runs conceded / overs bowled; dot-ball rate = dot balls / legal balls. Extras and wides need care: wides are not balls faced.
- Read the field names a tool returns before using them; if a field is not returned, say so rather than inferring it.

## Default path (credits as listed by each tool; ball-by-ball pulls are the expensive step)
1. **Resolve the scope.** `list_competitions` (1 cr) for the key; `search_players(name)` for a player id.
2. **The deliveries.** `get_deliveries` for the player's innings, a team's innings, or a match, with competition, season and format filters. Paginate if the tool does.
3. **Group by phase.** Assign each ball to a phase by over number, then compute runs, balls, strike rate, economy, dot-ball rate, boundary rate and wickets per phase.
4. **Cross-check totals.** Add the phases and compare with `get_player_stats(player_id, format, competition)` (5 cr); if they disagree, say by how much and why you think so (extras, filters).
5. **Compare (optional).** `get_leaders` for the competition's leaders in the same stat, to place the player.

## Reading it
A phase strike rate over fewer than 60 balls is noise; flag it. Death-over numbers depend on when the batter came in and the match situation; if the data carries innings position, show it. Compare a player's phase numbers with other players' in the same competition and season, not with a different format.

## Output template
```
<Player or team> | <format, competition, season> | innings <n>
Powerplay: runs <r>, balls <b>, strike rate or economy <v>, dot % <v>, wickets <w>
Middle overs: ...
Death overs: ...
Cross-check: phase totals vs get_player_stats <agree / differ by <x>, why>
Read: 2-3 bullets with balls counted per phase
Data: list_competitions, get_deliveries, get_player_stats | <competition>, <season> (phase figures computed from deliveries)
```

## Rules
- Say that phase figures are computed and give the formulas used.
- State the over bands used for the format.
- Flag any phase with under 60 balls.
- Every number names its tool (the Data line, plus inline where it could be doubted) and every rate shows its sample size.
- If a tool returns too little, say "not enough data" and stop; do not estimate or borrow another sample.
- Odds and prices are information about the market only: no picks, leans, stakes, payouts or sportsbook links.
- Sequences (last 5, recent form, a run of results) are quoted in the order the tool returns them, with dates; never retell them from memory. Lists in the answer must match the tool rows.
- Out of credits: if a call is refused for credits, pass on the API's message as it comes (it says when free credits refill and how to add more), and stop; don't retry or switch tools to get around it.

## Example prompts
- "How does Bumrah bowl at the death in the IPL?"
- "Powerplay strike rate for Travis Head"
- "Which team scores best in the last five overs of T20s?"
