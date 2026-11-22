# Measure where darn runs lost time since 2026-11-18

`darn-slowdown-measure` · Task · research, darn, gate · running · Record-Write

## Goal

Find why darn runs take about twice as long as before 2026-11-18, and name each cause with its share of the time.

## What is known

- The median darn rose from about 24 to about 47 minutes.
- The records push at each transition (`71d6ee1a0`) explains only a few minutes per run.
  Loom row gaps went from about 2 s to about 6 s, and batten start from about 7 s to about 60 s; `926b5c28e` already took the push off the start rows.
- The number of full verify runs per darn is about the same before and after (3 to 5); the extra gate runs seen are failures within a second, each followed by a short rework.
- Each full verify run takes longer: first runs went from about 9 to 12 minutes to about 11 to 16, later runs from about 3 to 4 minutes to about 4 to 7.
- The darn-write and code-burler-fix gates in the hub's `gate.yaml` did not change in the period.
  Test functions grew about 8 % since 2026-11-16, and test files at tier 1 or 2 about 15 %.

## Questions

- Per row: wall time of each row before and after, from the `history` of `_lym/shed/<slug>/status.json` in the archive tags of `loomyard-weft` (`refs/tags/archive/<slug>/<sha>`), and the gaps between rows.
  Use about 20 darn runs from 2026-11-16 to 2026-11-17 and about 20 from 2026-11-21 to 2026-11-22, leaving out runs with a pause over 30 minutes.
- Agents: writer and review-agent time per row, apart from the gates inside it.
- Verify: how much of each run's time is build, vet and test, how often the Go build and test cache hits, and how many tests each scope runs.
- Load: how many runs and gate slots were busy at the same time in each period, hardener live tests included.
- The gate failures within a second: what fails, and whether the rework after each is needed.

## Bounds

- Read records, gate logs and git history first.
- At most two timed runs: one `lym gate test` of one fixed package set at tiers 0 and 1, once with no other run on the machine and once under ordinary load.
  If the machine is never idle, one run and a note saying so.
- The report ends with each cause, its share of the time, and a concrete fix for each, as candidate notes.
