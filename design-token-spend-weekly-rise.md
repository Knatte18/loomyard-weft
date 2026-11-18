# Claude usage outside runs: orchs, beam orchs, dobby agents, other hubs

`token-spend-weekly-rise` · Task · research, claude, report · running · Drift-Baseline

## Why

The operator's weekly Claude quota (Ultra) runs out faster than expected.
The run spend is known: `tools/tokencount -since 2026-11-03` and `lym report cost --since 2026-11-01` (2026-11-18) show
tapestry at about $62 per run, Plan about 45 % of that and Webster about 33 %, and the rise coming from more tapestry runs.
What is not known is the usage outside runs.

## Scope

Count every Claude Code session on this machine from 2026-11-03 to 2026-11-18 that `tools/tokencount` does not count:
the hub orch sessions in full (beyond tokencount's per-run orch charge), beam orchs, dobby agents (tidy, retro, board agents),
motif and raddle runs, sessions in the other hubs (lyt, cs, odd) and sessions outside LoomYard.
Read their transcripts under `~/.claude/projects`, priced as `tools/tokencount` prices.
Where tokencount could count a kind with a small change, say so in the report; change no code.
Codex usage is out of scope.

## Report

A report under `docs/research/tokens/` with, per ISO week W45 to W47:
the usage of each session kind above, by model, set beside the run spend from `tools/tokencount`,
the share of the weekly total each takes, and any kind that grew markedly, with why.
