# Card 6 — compare-run

**Create:**
- `plan:internal/driftengine#CompareRequest` -> `type CompareRequest struct`
- `plan:internal/driftengine#CompareResult` -> `type CompareResult struct`
- `plan:internal/driftengine#RunCompare` -> `func RunCompare(ctx context.Context, req CompareRequest) (CompareResult, error)`
- `plan:internal/driftengine#SkipList` -> `func SkipList(scope ScopeRecord) []string`
- `plan:internal/driftengine#TestRunCompare` -> `func TestRunCompare(t *testing.T)`
- `plan:internal/driftengine#TestSkipList` -> `func TestSkipList(t *testing.T)`

**Edit:**
- `internal/gateslot/doc.go`

**Extends:**
- `internal/driftengine/doc.go`

**Uses:**
- `plan:internal/driftengine#RunCapture`
- `plan:internal/driftengine#ProbeLock`
- `plan:internal/driftengine#ScopeRecord`

**Intent:**
Build the compare run (D6) on the runner `RunCapture` uses: the mechanical compare `Drift-Verify` runs itself and the one `lyx drift compare` runs for its agent.

`CompareRequest` carries the told `Paths`, worktree root, the declared `compare` `CommandStep`, pool and wait dir, the skip list, the `Timeout` (the step's `timeout_min`, which a test shortens) and a clock.
`RunCompare` takes `CompareMarker`, an OS lock, before it waits for its gate slot and holds it until it ends, so a compare queued for a slot reads as activity and one left by a dead compare reads as none (D4);
a held marker refuses with `drift: a compare is already running; way forward: wait for it, then run it again`.
It writes the skip file, holds one slot (`Site: "drift compare"`), renders `{baseline_dir}` as `BaselineDir` and `{skip_file}` as `SkipFile`, runs with its log `LogsDir/compare-<UTC stamp>.log` and the timeout and tree kill of the capture, and returns the exit code, the log path and whether it passed: exit code 0 (D6), a timeout being a failure naming the timeout.
It writes no record and counts no run; `TakeRun` is the caller's.

`SkipList` is the scope's `skip` tests plus every `accepted` test, in that order, de-duplicated; a withdrawn test is off `accepted` and so is run again (D7).

`gateslot`'s `doc.go` non-tier holder list gains the drift compare; `driftengine`'s `doc.go` gains the compare section.

Tests:
`TestSkipList` (tier0) rows: `off` and `full` with no acceptance give none; `partial` gives its skips; acceptances are added; a withdrawn test is absent; a test both skipped and accepted appears once.
`TestRunCompare` (tier1, `!windows`, fake `sh` commands) rows: exit 0 passes; a non-zero exit fails and names the log; the rendered command sees the skip file holding the skip list and `{baseline_dir}` the baseline directory; while the run waits for the pool's only slot, `ProbeLock(CompareMarker)` reads held, and once it ends, free; a shortened timeout kills the tree and fails; a held marker refuses.

**ImpactSummary:** Two doc lists grow; no existing code changes.
