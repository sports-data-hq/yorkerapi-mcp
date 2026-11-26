---
name: cricket-match-preview
description: "Use when the user wants a cricket match preview in any format, or how a ground plays before a match: 'preview India vs Australia ODI', 'IPL preview MI vs CSK', 'PSL match breakdown', 'who's favored in the Big Bash tonight', 'how does this ground play', 'T20 World Cup match preview'. Produces a one-page report: format ratings and our pregame win probability (T20 and international ODI), venue profile, recent results, key players' form and the market's prices. Not for ball-by-ball replays (call get_deliveries) or modeling (see model-building)."
metadata:
  version: 2.2.0
  author: SportsDataHQ
---

# Cricket match preview

A one-page preview of one match from live YorkerAPI calls, in any format (Test, ODI, T20 and franchise leagues). It answers what a captain's analyst would ask: what is par here, does the toss matter, who is strong in which phase, and how much of our model's view is already in the market's price.

## Before you start
- For anything across games, seasons or players, apply the sports-data-preflight checks first (ids from search, the season convention, one league, enough data).
- Tools come from the YorkerAPI MCP server (`https://mcp.yorkerapi.com/mcp`). Call them; never recall a score.
- Seasons are a year (2024) or a split season (2024/25). Competitions by key (ipl, bbl, psl, blast, hundred, test, odi, t20i, smat...): `list_competitions` (1 cr).
- Our pregame win probability (`win_prob`, model elo-cricket-pregame v1) is served for T20 and international ODIs only. Tests and first-class have draws; domestic one-day ratings are too weak. Ratings are per format and per gender: `get_ratings` mixes men's and women's unless you pass `gender`, so always pass it.
- Teams without an id (national A sides, some domestic teams) have no rating; say so rather than borrowing another side's.
- Starting XIs are not in the data. Before the toss say "XI not announced"; after the toss the schedule's `status_detail` carries it ("Australia won toss & batted").

## Default path (about 33 credits)
1. **Find the match.** `get_schedule(competition, team_id, days)` (2 cr): start time, teams, venue, state, each side's rating, our `win_prob` (T20 and international ODI) and the favourite at the median price. Note the match id.
2. **Ratings.** `get_ratings(format, gender, team_type)` (2 cr): both sides' Elo in this format and where each ranks.
3. **The ground.** `get_venue(venue, format, since)` (5 cr): average first and second innings totals, how often the side batting first wins, toss wins and choices, draws. Use the ground's own name; a loose name can merge several grounds. Pass `since` (a recent date) to see the current pitch era as well as all time.
4. **Recent results.** `get_matches(team_id, format, limit: 5)` (2 cr) for both sides: scores, winner, toss. Add `opponent_id` for the head to head in this format.
5. **Phase strengths for 2 to 4 names.** `get_player_stats(player_id, by: "phase", format)` (5 cr): powerplay, middle and death splits for batting and bowling. Player ids from `search_players(name)` (1 cr).
6. **Duels (optional).** `get_matchup(batter_id, bowler_id, format)` (5 cr).
7. **Prices.** `get_odds(match_id)` (5 cr): opening and closing decimal price per outcome by book.

## Method
- **Par and chase edge:** par = average first innings at this ground in this format; chase par = average second innings. A gap of more than 15 runs between them is a chase-friendly or bowler-friendly ground. Say how many matches the averages rest on; rain-rule matches are left out.
- **Bat-first win rate:** `bat_first_win_pct` over the decided matches. Standard error = sqrt(p x (1 - p) / n). Inside 2 standard errors of 50% it is not an edge; at n = 80 that means anything from about 39% to 61% is noise.
- **Toss:** `toss_winner_win_pct` and the share choosing to field (`chose_to_field` / matches). The toss matters only when the win rate for toss winners clears the same 2-standard-error band, or the ground is a Test or one-day ground where a swinging first session decides the match. Do not call the toss a factor on a 50% rate.
- **Ratings to probability:** p = 1 / (1 + 10^(-gap / 400)). The served `win_prob` also carries home advantage and format effects, so quote the served number and use the formula only to explain the gap.
- **Market no-margin:** for each book with both sides, decimal d: q = 1/d for each outcome, divide by the two q's summed, then average across books. Opening vs closing shows movement. Gap = our `win_prob` minus the market's, in points; 8 or more is worth one line (information only).
- **Phase edges (T20):** powerplay overs 1 to 6, middle 7 to 15, death 16 to 20. Compare each key batter's strike rate and average with the bowling side's economy and wicket rate in the same phase. Require 100 balls in the phase before quoting a strike rate (60 balls as an absolute floor, labelled thin).
- **Matchups:** at least 30 balls and, for any "dismissed him" claim, at least 2 dismissals. Otherwise say the sample is too small.
- **Form:** last five results in the tool's order with dates and opposition rating; a win over a rating-1500 side is not a win over a 1700 side. Note which results came at which grounds and in which format.
- **Context flags:** touring side (home advantage), day/night (the schedule's description marks (N)), a new ball in two innings of a Test, dew and the toss in evening T20s, a weak or missing rating, women's vs men's competition.

## Output template
```
<Team 1> vs <Team 2> | <competition, format> | <venue> | <start> | <toss/XI status>
Ratings (<format>, <gender>): <t1> <r> vs <t2> <r> | Our pregame win probability: <t1 x%> / <t2 y%> (or "not served for this format")
Market (no-margin, <n> books): <t1 a%> / <t2 b%> | opened <a'%> | gap to our model: <pts>
Venue: par 1st <n> / 2nd <n> over <m> matches (since <date>: <n>), batting first wins <x%> of <n> (+/- <se>), toss winners win <y%>, <z%> choose to field
Form (newest first): <t1: results with dates> | <t2: ...>
Phase strengths: <named players, phase, strike rate / economy, balls>
Head to head in this format: <results with dates> (small, say so)
What to watch: 2-3 bullets from the numbers
Data: get_schedule, get_ratings, get_venue, get_matches, get_player_stats, get_odds | <competition>, <season>
```

## Worked example (live calls, Nov 25, 2026)
3rd T20I (N), England tour of Australia, Brisbane Cricket Ground, Nov 27, 2026.
- Ratings (men's T20): Australia 1734, England 1791. Our pregame win probability 44.3% / 55.7%. Market no-margin over 7 books: Australia 46.2%, England 53.8% (Australia opened at 46.8%). Gap -1.9 points, no flag.
- Venue (`get_venue`, Brisbane Cricket Ground, T20, 81 matches): par 165 first innings, 153 second; batting first won 47.4% of decided matches (standard error about 5.7 points, so no edge); toss winners won 50.0%; 59% chose to field. Verdict wording: "a ground where chasing is the habit but not the edge; the toss is not a factor."
- Form, newest first: Australia W Jun 21, W Jun 19, W Jun 17 (Bangladesh), W Feb 20 (Oman), L Feb 16 (Sri Lanka); England W Sep 19, Sep 17, Sep 15 (Sri Lanka), W Jul 11, Jul 9 (India). The results list did not include the first two matches of the current series yet: say it lags.
- Phase: Travis Head's 2026 T20 powerplay strike rate 170.2 over 198 balls (average 37.4); his middle-overs 169.8 comes from 43 balls, too few to quote.
- Head to head in T20 (newest first): England won Sep 13, 2024 (Cardiff), Australia won Sep 11, 2024 (Southampton) and Jun 8, 2024 (Barbados).

## Not enough data
- Under 10 matches at the ground in this format: "thin"; under 5: do not give par or toss numbers.
- Under 100 balls in a phase (60 as a floor), or under 30 balls in a matchup: the number is shown only as thin.
- A side with no rating: no gap and no win probability; use form and venue only.
- `win_prob` not served (Test, first-class, domestic one-day): say so and give the rating gap in plain words, plus the share of draws at the ground for Tests.
- Fewer than 5 recent results in this format: show them all and say so.
- No price posted: say so, never borrow another match's.

## When there's no game

If the game the user names isn't scheduled (off-season, not booked, already played), don't stop to ask. Do the most useful real version and say which one in the first line:
1. The next scheduled meeting of those two sides, if `get_schedule` (filtered by team) has one.
2. Otherwise their most recent meeting (`get_matches` with `opponent_id`) and where each side stands in the ratings.
3. For a matchup that can't happen soon, the ratings and ground profile labelled as hypothetical.
Answer first; offer the alternative in one closing line, never as a question before answering.

## Rules
- Say "our model" for our probability and "the market" for prices; never blend them.
- Venue numbers need matches: show the count; under 10 matches at a ground, say it's thin.
- Weather and the toss can swing a match; note when the format makes the toss matter (venue toss data).
- Odds are information about the market: no pick, stake or bet.
- Complete before writing: every step in the default path is called for both sides before the preview is written. If a call fails or returns nothing, say which section is missing rather than writing around it.
- Sequences (last 5, recent form, a run of results) are quoted in the order the tool returns them, with dates; never retell them from memory. Lists in the answer must match the tool rows.
- Rate limit: a "rate limit exceeded" reply is not a credit refusal. Wait a few seconds and retry that call once, one call at a time; if it still fails, name the section that is missing.
- Out of credits: if a call is refused for credits, pass on the API's message as it comes (it says when free credits refill and how to add more), and stop; don't retry or switch tools to get around it.
