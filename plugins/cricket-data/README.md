# YorkerAPI: cricket skills

Cricket analysis skills on YorkerAPI: game previews, player and team research, and the community's standard metrics, with every number pulled live from the YorkerAPI MCP server.

## Skills

- `cricket-batter-vs-bowler`: Use when the user asks how a cricket batter fares against a bowler or a kind of bowling, in fan or analyst wording.
- `cricket-chase-win-probability`: Use when the user wants the win probability of a cricket run chase or how it swung.
- `cricket-match-preview`: Use when the user wants a cricket match preview in any format, or how a ground plays before a match.
- `cricket-phase-analysis`: Use when the user wants cricket numbers split by stage of an innings, in fan or analyst wording.
- `cricket-venue-effects`: Use when the user asks how a cricket ground plays or what to expect from the toss and the pitch, in fan wording.

## Data it sends

The skills call the YorkerAPI MCP server at `mcp.yorkerapi.com`. Each call sends the tool name and its inputs (teams, players, dates, seasons) with your YorkerAPI API key, and returns data. Nothing from your files or conversation is sent beyond what a tool call needs. Calls use credits on your YorkerAPI account; a free key at https://yorkerapi.com/signup includes monthly credits.

Privacy policy: https://yorkerapi.com/privacy

## Install

```
/plugin marketplace add sports-data-hq/sportsdatahq-skills
/plugin install cricket-data@sportsdatahq
```

More: https://yorkerapi.com/skills. Licence: MIT (see LICENSE).
