# Measure whether landed runs got faster since turn-savings

Adds a research study, `docs/research/speed/task-runtime-before-after-2026-11-06/`, that compares the active time of landed darn and tapestry runs before and after project `turn-savings` (P1), and an index line in `docs/research/README.md`.
It reads recorded data only: `lym report timing`, `cost`, `convergence`, `lym gate report`, the archive tags and the landing commits. No lym code changes and no agent ran.

## What the report finds

- Darn code runs are slower after the cut, not faster: 31.7 min active on average before (median 31.3, n = 52) and 45.0 after (median 39.7, n = 130), 1.3 to 1.6 times in every size band; taking gate time out leaves the difference.
- Tapestry whole runs with a Discussion take 188.8 min before (n = 7) and 244.8 after (n = 6), for larger runs; per 100 changed lines they are level (3.2 and 3.1). The sample is small and no significance is claimed.
- The phases `agent-turns` touched (darn write, tapestry plan and Webster) show no saving, so the data supports no share for P1. The report names the changes of the same week that it cannot separate from P1: added recipe rows, the Drift rework, gate changes, and runs in parallel rising from 1.8 to 3.4.

## Design decisions

- Active time is `active_seconds` of the timing report. A code run whose whole span lies in machine sleep has no active time and is counted and left out, rather than read as zero (25 runs).
- Size is the added plus deleted lines of the landing commit. The convergence report leaves every tapestry run unmapped, so those take the one first-parent commit up to 12 minutes before the run's last stamp; a fast-forwarded multi-commit branch is measured by its last commit only.
- Profile is recorded for 19 runs, all after the cut, so runs are not compared by profile; bench, trial, audit and `docs` runs are kept apart by slug name.
- `slack-relay` (Sketch-fed) and `cold-gate-fixer` (no Discussion or Plan time) are named and left out of the whole-run means. Archive tags before 2026-10-27 hold only row stamps, so the window stays.
- Scripts are bash and jq (`extract.sh`, `join-gate.sh`, `producers.sh`, `analyze.sh`, `analyze.jq`); `runs.tsv`, `gate-by-slug.tsv` and `producers.tsv` are the tables behind the report.

## Checks

The study ships no Go code, so there is no test. `analyze.sh` was run on the committed tables and its output matches the figures in `report.md`.
