# Where the Claude quota goes, and why it burns faster now than two weeks ago

`token-spend-weekly-rise` · Task · research, claude, report · draft

## Why

The operator's weekly Claude quota (Ultra) runs out faster than expected,
and they believe it burns faster now than around 2026-11-04.
They want to know what the tokens and money go to, and what changed.

## Tool

Use `tools/tokencount` (`tools/tokencount/tokencount.sh`), which already reads run transcripts by role, segment and recipe, with the orch charge.
Run it once per ISO week from W45 to W47 (`-since` with slugs chosen per week, or the run's own split of one `-since 2026-11-03` count),
plus `-detail` where a role needs its token columns.
Sessions it does not count are only the gap to fill by hand:
the hub orchs and beam orchs beyond its orch charge, dobby agents (tidy, retro), raddle and motif runs if it skips them,
other hubs (lyt, cs, odd) and sessions outside LoomYard.
Read those transcripts under `~/.claude/projects` directly, priced the way tokencount prices.
Where tokencount cannot do a needed split, say so in the report as a gap in the tool; change no code.
Codex usage is out of scope.

## Report

A report under `docs/research/tokens/` with:

1. Usage per ISO week W45 to W47, and per day where the transcripts allow, so the rise (or its absence) is plain.
2. Per week, the split by segment and role, by model, and by session kind (run, orch, beam orch, dobby agent, other hub, outside LoomYard).
3. Per recipe and profile: runs per week and mean cost per run.
4. What changed between W45 and W47: more runs, dearer runs, a role or model that grew, parallel runs, longer orch sessions,
   or anything else the data shows, each with its share of the change.
5. The five largest cuts the data supports, each with its estimated weekly saving, ranked.
