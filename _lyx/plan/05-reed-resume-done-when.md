# Card 5 — reed resume drops a finished strand

**Edit:**
- `internal/reedengine/state.go`
- `internal/reedengine/strand.go`
- `internal/reedengine/lifecycle.go`
- `internal/reedengine/lifecycle_test.go`
- `internal/reedengine/doc.go`
- `internal/reedcli/resume.go`

**Intent:**
A reed strand may carry a list of done-when paths, set by its spawner at add time, and `Resume` drops from state, instead of relaunching, a non-live strand whose list is non-empty and whose every path exists (D5, #409).

- `Strand` gains `DoneWhen []string` (`json:"doneWhen,omitempty"`), so a strand added before this change, or one whose spawner sets no list, decodes with none and is relaunched as today.
- `AddSpec` gains `DoneWhen []string`, copied verbatim onto the new `Strand` by `addStrandLocked`; reed never interprets the paths beyond the existence check below.
- `planResumeLaunches` takes an existence seam (`func(path string) bool`, `os.Stat` in production) and returns the strands to relaunch and the strands to drop.
  A strand is dropped only when it is not live, not hidden, its `DoneWhen` is non-empty, every listed path exists, and no other strand in state names it as `Parent`; every other non-live, non-hidden strand is relaunched exactly as before.
- `Resume` removes each dropped strand from state before the launch loop, persists, logs one Info line per drop naming the strand and its guid (`PATTERN-spawn-observability`'s teardown half), and reports the count on `ResumeResult.Dropped`.
  A dropped strand has no live pane, so there is nothing to kill.

Bound: only non-live strands are dropped, and only when every listed path exists; a live strand is never touched, and a strand with no list, a partly missing list or a child is relaunched as before.

`lifecycle_test.go`'s `planResumeLaunches` table gains the rows the acceptance criteria name: no list relaunches, a partly missing list relaunches, a fully present list drops, a live strand with a fully present list is untouched, a hidden strand is neither launched nor dropped, and a strand with a child is relaunched.
`resume.go`'s Long text says a finished strand (every done-when path present) is dropped, and its envelope gains `dropped`.
`doc.go` records the done-when list as the one place reed learns a strand finished, and that reed only stats the paths.

**ImpactSummary:** Resume's relaunch set shrinks only for strands that carry a done-when list, which no spawner sets until card 6.
