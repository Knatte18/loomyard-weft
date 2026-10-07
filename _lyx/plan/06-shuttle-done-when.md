# Card 6 — shuttle marks an ungated run's strand done-when its outputs exist

**Edit:**
- `internal/shuttleengine/run.go`
- `internal/shuttleengine/run_test.go`
- `internal/shuttleengine/doc.go`
- `PATTERN.md`
- `pattern/PATTERN-completion-signal.md`

**Uses:**
- `internal/reedengine/strand.go`

**Intent:**
`Runner.start` sets the new strand's `AddSpec.DoneWhen` to the run's `spec.OutputFiles` (already absolute after `Spec.validate`) when the run is ungated (`len(gate) == 0`), and leaves it empty for a gated run (D5).
For an ungated run the output files are the Completion Signal's own test of a finished run (`PATTERN-completion-signal`), so reed's resume now drops a finished agent strand instead of reviving it.
A gated run's strand carries no list, because its outputs existing does not mean it finished: the gate runs after them and a failing gate re-prompts the live agent, so reed must relaunch it as before.
Reed cannot know that a gate passed, so a finished gated run's strand is still relaunched when shuttle's own teardown did not run; the relaunch types no prompt.
The list is provider-neutral and set here, never in `claudeengine` (`PATTERN-shuttle-provider-seam`).

`run_test.go` pins, through the fake reed's recorded `AddStrandCalls`, that an ungated `Start` passes the output files as `DoneWhen` and a gated `StartGated` passes none.

`PATTERN.md`'s `PATTERN-completion-signal` entry and its background file gain the done-when list as the one place reed learns a strand finished: filled only by `shuttleengine`, from an ungated run's `OutputFiles`, and read only by `reedengine`'s resume (D8).
The entry line stays within the overview's line-length cap; the detail goes in the background file.
`doc.go` says the same in shuttle's own terms.

**ImpactSummary:** Every ungated shuttle strand now carries a done-when list; gated runs, drivers and operator strands are unchanged.
