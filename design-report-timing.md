# lyx report timing: phase minutes, parent waits and agent spawns from run records

`report-timing` · Task · enhancement, shed, orch · running · Plan-Bouncer

A mechanical time report over the hub's run records, beside `lyx report convergence`: today it took an LLM agent five minutes and an awk script to compute (2026-10-12).
The operator wants it as a tool, not an LLM job.

## What it reports

- Per run: minutes per phase (setup, Discussion, Plan, Webster, Merge, Publish to Finalize), total active minutes, minutes waiting on the parent or orch, and the span.
- Medians per phase over the landed runs in a window.
- Day or week totals: active run-minutes, parent waits (parent review, PR-Gate awaiting), arbiter spawns and minutes, alignment-agent spawns and minutes.
- Stops by who settled them: judge, shed, driver, arbiter, alignment agent, orch, operator.
  Fewer orch wakes alone can hide that load moved to the arbiter; the stop count itself shows whether runs converge.
  Check whether `lyx report convergence` already counts arbiter and alignment outcomes, and share its record reading.

## Operator decision (2026-10-15): seconds, not only minutes

Every duration the report prints carries seconds, not whole minutes only, the way the run status pane shows them since `run-pane-timing`.

## How the 2026-10-12 agent computed it

- Landed runs from the `archive/<slug>/*` tags in the records repository, runs in flight from `<slug>-weft`; each run's start from the hub `Seed-Child` row.
- History `at` stamps are completion times, so a row's interval runs from the previous row.
- Parent review windows from the round's `request.json` and `verdict.json`.
- Rework passes count in the phase of the producer that re-ran.
- Hibernation: a gap with no row from any run and no producer working; the journal's resume time confirms it.

## Data gaps found

- `attempt.json` `spawned_at` for alignment attempts is unreliable (later than the attempt's own report in `intake-bugs-1012`); archived attempts have no file mtimes.
- No token totals: `tools/tokencount` only runs through `go run`; review rounds' `round-N-usage.yaml` covers review agents only.
- Batten's Land delay after Friction-Reflect sits outside the run's own history (20.7 min for `plan-gate-fixes`).
