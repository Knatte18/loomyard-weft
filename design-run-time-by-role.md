# Tokencount reports wall-clock time per role

`run-time-by-role` · Task · feature, shed, loom · running · Webster

## Seen (operator, 2026-10-11)

`tools/tokencount` reports each finished run's tokens and estimated cost per run and per role, but nothing reports where a run's wall-clock time goes.
The operator wants to see the time each producer takes and where most of a run's time is spent.

## Wanted

- `tools/tokencount` reports wall-clock time beside tokens: per run and per role, from the transcript lines' timestamps it already reads, with sub-agents under their parent's role as for tokens.
- Time no session spends working (parent review, PR-Gate, waits for a gate slot, a parked driver) shows as its own line per run, read from whatever the run's shed/loom status records; if the status records no per-phase timestamps, say so in the report and add them.
- Measured by hand on 2026-10-11: each archived `status.json` history entry carries the UTC time its row finished, so the gaps between entries cover the whole run, waits included; no entry records when a row started, and Publish and Finalize record no inner breakdown.
- Each role's share of the run's total time, so the slowest producer stands out.
- Update the package doc and the launchers' report description.
