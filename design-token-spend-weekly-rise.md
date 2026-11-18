# Where the Claude quota goes, and why it burns faster now than two weeks ago

`token-spend-weekly-rise` · Task · research, claude, report · draft

## Why

The operator's weekly Claude quota (Ultra) runs out faster than expected,
and they believe it burns faster now than around 2026-11-04.
They want to know what the tokens and money go to, and what changed.
`docs/research/tokens/token-usage-by-role.md` (2026-10-12) is the last such study; it covered six loom runs only.

## Scope

Read every Claude Code transcript on this machine for 2026-10-28 to 2026-11-18, not only run sessions:
the hub orchs, beam orchs, run roles, raddle and motif runs, dobby agents (tidy, retro), subagents and forks,
and sessions in other hubs (lyt, cs, odd) and outside LoomYard.
Weigh usage the way the quota counts it as far as is known (model, output, cache write, cache read, input),
and state the weighting used.
Codex usage is out of scope.

## Report

A report under `docs/research/tokens/` with:

1. Usage per day and per ISO week, so the rise (or its absence) is plain.
2. Per week, the split by session kind (orch, beam orch, run role by producer, subagent, dobby agent, outside LoomYard), by model and by hub.
3. Per recipe and profile: runs per week and mean usage per run, from `lym report cost` and the transcripts.
4. What changed between W45 and W47: more runs, dearer runs, a role or model that grew, parallel runs, longer orch sessions, cache misses or anything else the data shows, each with its share of the change.
5. The five largest cuts the data supports, each with its estimated weekly saving, ranked; no code change is made.

The run reads data and makes no model calls of its own beyond its own session.
