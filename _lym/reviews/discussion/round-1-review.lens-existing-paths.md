```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced each state the record introduces or changes through the code that already runs.

**Paths traced, per decision**

- **D1 (index beside the shared maps, written only by the prime's `--map` run): survives.** `testcover.WriteMap` has only two callers, both in `internal/gatecli/testmappackage.go:709,728` and both inside the `--map` run. Nothing else writes the shared map directory, so an index rebuilt after that run's maps matches them.
- **D1 (worktree map against hub map): survives.** Only a non-prime `--map` run writes the anchor maps. The redundancy step reads only the hub's maps (`internal/gatecli/redundancy.go:286`), so it never sees the worktree maps D3 writes.
- **D3 (refresh skips a package with modified or untracked files): breaks.** See F2.
- **D3 and `verifytree.Verify`'s skipped return: survives.** A skipped verify writes no new `reach.json`, and the one already there was written for the same tree, so it still matches HEAD.
- **D5 and D10 (rubric markers): survives.** The redundancy marker is filled at `internal/shedadapters/rubric.go:18,33` and handed over through `shedrecipe` and the Bouncer. A sibling marker fits the same seam.
- **D7 (triage lines and the card fixer): survives.** `fixAndRerun` copies the first run's stderr (`internal/gatecli/detachfixer.go:198-203`), so the lines reach the card fixer. The re-run only appends to that stderr.
- **D7 (triage lines and the report's line cap): breaks.** See F1.
- **D12 (template fill of `reports`): survives.** The hub's base `gate.yaml` tunes `batch`, `darn-write`, `code-burler-fix`, `webster` and `parent-align` without a `reports` key (`_bolt/_lym/config/gate.yaml:43-51`). Config load therefore fills the template's `reports` into each of them, as `internal/gateslot/doc.go:113` states.
- **D9 (line ranges): survives.** The map stores content keys (`BlockKey`), but the existing mutation path rebuilds line ranges from the coverage binary's profile (`Toolchain.ProfileBlocks`, `internal/testcover/runner.go:296`). Line ranges are still available to the planner.

**Findings**

- **F1 — MEDIUM, class `design`, at D7 (`decision-record.md` lines 128-130).** Triage lines can push a failure report past its 80-line cap, and a cut report makes `gateslot.FailedTests` return `ok=false`. The record's claim that the parsers read the report "unchanged" then fails, and so does its acceptance criterion.
  - Evidence: `internal/gatecli/failurereport.go:29-30` sets `failureReportLines = 80`, and lines 200-214 cut the report and append a `... ` line. In `internal/gateslot/runlog.go`, `FailedTests` returns false on that `... ` line, because it is neither a run header nor indented, and its doc comment says "a cut report… give false".
  - Effect in `internal/loomshed/verifygate.go:226-247`: `namedFailures` gets `ok=false`, so the run of the failed tests alone is treated as naming none and the gate falls back to the full command.
  - Effect in `internal/landingshed/verifygate.go:309`: the rerun widens from the failed tests alone to the whole verify.
  - Fix: keep triage lines out of the cap budget, or give up a triage line before any test's output is cut. Add an acceptance criterion for a report at the cap.
- **F2 — MEDIUM, class `design`, at D3 (`decision-record.md` line 70, and the matching Open risk).** D3 skips a package only when that package has modified or untracked files. The `--map` write also refuses when a package the test binary links is dirty, so the refresh spends its build and test runs under `reach_budget_min` and then writes no map.
  - In that case the note gives the wrong reason, or none, for the stale package.
  - Evidence: `internal/gatecli/testmap.go:583-585`, where `checkClean` "needs that no file a package of trees owns is modified or untracked", and `trees` covers linked packages. The test-speed background at `pattern/PATTERN-test-speed.md:305` says the same.
  - Fix: skip a package whose own files or any linked package's files are uncommitted, and name the dirty linked package in the note.

Tool calls:
1. Read `internal/loomshed/verifygate.go` 225-250 and `internal/landingshed/verifygate.go` 295-325, to see what the callers of `FailedTests` do on `ok=false`. My context held only their line numbers.
2. Grepped for `testcover.WriteMap` callers, read `internal/gatecli/testmap.go` 576-600 for the scope of `checkClean`, and grepped for where `redundancy_report` is filled. My context held none of these.
