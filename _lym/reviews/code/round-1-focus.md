---
round: 1
focus:
  - "internal/reportengine/boardhistory.go and landings.go: does the windowed board read give the same done-after times and labels as the old unbounded walk, including a run opened before the window start, a slug reopened after done, and the start point chosen by landingWalkStart and extendSpans?"
  - "internal/reportengine/batten.go: does the week-before-window start with widen-on-demand always find a run's seed (isFreshSeed, newestSeed), and what happens when no seed is found or a status file is missing at a commit in the run's span?"
  - "Memory bound: does any path still hold a parsed board or parsed status beyond the read of its commit (boardTasksAt, battenStatusReader.at, snapshotsOfRun), and does the pinning test fail if one is held?"
  - "Sweep scope from the Goal: are the timing, cost, raddle and replies reports in internal/reportengine each checked for an unwindowed walk, a per-commit read, or an all-revisions cache, and is each place found fixed and pinned by a test? Check runs.go, timing.go, report.go, and any file the diff leaves untouched that still walks history."
  - "timing.go ComputeTiming and selectInFlight: does the shared walk between landed and in-flight runs change which runs or spans either set sees?"
  - "Mutation survivors at batten.go:79 (walk), batten.go:104 (widen) and boardhistory.go:101 (add): does a test assert the behavior those statements carry, or only a read count?"
  - "Test economy: TestLandingJoiner_HoldsNoParsedBoard is reported contained by TestComputeTiming_WalksTheRecordsBranchOnce; compare their assertions before filing a fold-or-keep finding, and check the reflective joiner test for fragility."
  - "Commit trailer: the Root-Cause trailer must name the defect the diff removes; and check _lym/landing/summary.md claims, including Merge danger, against the diff."
---

Seeded from the decision record's Goal (cause, fix, operator sweep), the diff stat and the redundancy and mutation reports.
