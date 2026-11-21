# Research: what the reach and mutation reports change in landed darn runs

Adds the report `docs/research/speed/testmap-use-measure-2026-11-21/report.md` and the scripts that print its tables.
It reads the 10 newest darn runs landed after `6dbc7a05` (listed in `runs.tsv`) from their archived records, verify logs, gate call traces and session transcripts.
It asks whether the reach turn, the mutation step, the failure-report triage lines and the review markers change what lands.
No test, agent or lym code ran or changed.

## Findings

- **Reach turn.**
  7 turns came up in 4 runs, and the session added tests after 3 of them, in 2 runs.
  Of the 4 turns that changed nothing, 3 named functions that tier-1 tests call but no coverage map holds.
  A turn costs 5 to 60 seconds of agent time.
  Code-Review cited the reach report in 3 findings in 2 runs, and all 3 were fixed.
- **Mutation step.**
  Code-Review saw 27 survivors in 6 runs and raised 6 findings from 7 of them, in 3 runs, all LOW and all fixed.
  Of the other 20, 7 were killed by tier-1 tests the mutation run does not run, 8 sat on lines a removal restored verbatim, and 5 were unkillable.
- **Time.**
  The reach and mutation steps added 1223 seconds of row gate wall time over the 10 runs: 122 per run on average, 80 at the median.
  The mutation step's share was 410 seconds: 41 per run on average, 17 at the median.
  One large removal diff took 248 seconds of mutation on its own.
- **Triage.**
  6 failure reports carried triage lines, and no fixer used one to call a failure unrelated.
  All 6 failed tests that got "reaches none of this run's changes" failed because of the run's own change.
  The change reached them through a tree text scan, an embedded markdown file or new nil seams, none of which is a function in a map.

## Verdict

Keep the reach turn and the mutation step and tune them, and turn off the "reaches none" triage line.
The tuning:

- leave out of a reach turn any function whose package has tier-1 tests the maps do not hold, or map tier-1 tests;
- mark a survivor that a tier-1 test covers, or one on a restored line;
- print "reaches none" only when the diff changes nothing but Go functions, or drop it.

## Design decisions

- The step times come from the traces of the row gate calls only.
  A session's own gate call detaches to a front run whose trace the archive does not keep.
- The redundancy step's span is subtracted from the time after the tiers because that step predates `6dbc7a05`.
  The reach analysis has no log line of its own, so it is counted together with the map refresh.
- The scripts find the hub, records repository and transcript directory from the worktree, as the earlier speed reports' scripts do.
  Each location can be overridden by an environment variable.

## Merge danger

**Door:** two-way
**Blast radius:** docs
