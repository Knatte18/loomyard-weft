```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens. I traced the diff's changes to the Goal points of the decision record, which has no `Added after Discussion` entries.

## Decisions (Goal points)
- **G1**: read the board history once, bounded by the report window, and keep only each slug's done-after time and its labels at a landing.
- **G2**: hold no parsed board past the read of its commit.
- **G3**: a test pins the bound. The joins read each board commit at most once and hold no board beyond the one being read.
- **G4**: sweep every report in `internal/reportengine` (timing, cost, raddle and replies included) for three patterns: an unwindowed walk, a read per commit, or a cache of every revision. Fix each place found and pin each fix with a test.

## Changes and their traces
- **C1**: new `boardhistory.go` (`readBoardHistory`, `boardTasksAt`, `boardHistory.add`, `doneAfter`, `labelsAt`), plus the `landings.go` rewrite (`landingWalkStart`, the `timeJoinOwners`/`joinTime` split, `assemble`, and the removal of `boardDoneAfter`, `tasksAt`, `labelsOf` and `boardTasks`). Traces to G1 and G2.
- **C2**: `landings_test.go`, with `TestCompute_ReadsEachBoardCommitOnceFromTheWindowOn` and `TestLandingJoiner_HoldsNoParsedBoard`. Traces to G3.
- **C3**: `batten.go`, the windowed records walk with `widen`, the per-run `battenStatusReader`, `isFreshSeed` and `newestSeed`. Traces to G4: the unwindowed records walk and `commitsOf`'s cache of every revision.
- **C4**: `report.go`, `runs.go` and `timing.go`, which build one `battenIndex` and pass it to `selectRuns` and `selectInFlight`. Traces to G4: the timing report walked the records branch twice.
- **C5**: `runs_test.go` (`TestCompute_ReadsBattenStatusesOnlyOverEachRunsSpan`) and `timing_test.go` (`TestComputeTiming_WalksTheRecordsBranchOnce`). Traces to G4 ("pin each fix with a test").
- **C6**: `doc.go` paragraphs. Forced by C1 and C3.
- **C7**: `fake_test.go`, with the `fileReads` and `logQueries` counters. Forced by C2 and C5.

## Builders per decision
- **G1**: C1, partly. The bound fails when the batten walk widens; see F1.
- **G2**: C1.
- **G3**: C2.
- **G4**: C3, C4 and C5.
  - Cost, raddle and replies hold no `Log` call. The grep of `Log` and `FileAtRevision` calls shows cost reading at tag commits only, raddle at the board tip only, and replies reading no git at all.
  - So no fix was due there. The diff does not fix them, and no change is needed.

## Findings

- **F1**
  - Severity: MEDIUM
  - Class: scope
  - Location: `internal/reportengine/landings.go:148-155` and `internal/reportengine/batten.go:107-127`
  - Summary: the board read's floor is not bounded by the report window, so G1's "bounded by the report window" does not hold in general.
    - `floor` drops to the oldest time-join owner's `open` (`landings.go:150-153`). Those openings come from `batten.checkpoints` (`runs.go:210`).
    - The checkpoints come from the batten walk. Once any run lacks a fresh seed, `widen` falls back to a walk of the whole history (`batten.go:120-125`). The doc comment at `batten.go:108-109` says so.
    - After that fallback, every slug's checkpoints are in view. That includes an untagged, abandoned slug: `timeJoinOwners` gives it a nil floor (`landings.go:317-322`), so its newest checkpoint of any age opens a window.
    - That can drag `readBoardHistory` back over the whole board branch: one read per commit again, the cost the Goal names.
    - The same fallback also leaves the records walk unwindowed, which is the G4 pattern the sweep was meant to remove.
  - Evidence: `landings.go:150-155`, `landings.go:316-323`, `batten.go:120-125`, `runs.go:210`.

- **F2**
  - Severity: MEDIUM
  - Class: decision
  - Location: `internal/reportengine/landings.go:148-155`, `internal/reportengine/boardhistory.go:449-460` and `internal/reportengine/doc.go` (the added checkpoint paragraph)
  - Summary: the diff changes join results in ways the Goal does not license, and no `Added after Discussion` entry records them.
    - The Goal asks to read the same information with a bound ("keep only what the joins need"). It does not ask to change what the joins produce.
    - Change one: a checkpoint older than the records walk now opens no time-join window, so a landing that used to map can come out unmapped.
    - Change two: `labelsAt` gives no labels, plus a "no board commit holds an entry" finding, when the entry is absent from the board in effect at the floor but present in an older commit.
    - `_lym/landing/summary.md` states both under "Design decisions", and the package doc records the first. Both are the writer's own unreviewed calls, not the amended entry.
  - Fix: leave the code in place and defer to the run's parent, who records or refuses the departure with `lym loom decision add`. Alternatively, restore the old join semantics within the bound.
  - Evidence: `_lym/landing/summary.md`, "Design decisions" bullets 1–2; `doc.go` diff lines 469-473; `boardhistory.go:449-460`.

I made no tool calls; everything came from inherited context.
