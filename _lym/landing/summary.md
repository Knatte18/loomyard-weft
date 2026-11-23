# Report a halt on a newer board stencil as build-skew, not a hard failure

A step whose lym build is older than a board stencil it reads fails with `stencilstore.NewerBoardCopyError`.
The Go driver already recovers that stop: its hand-over row starts a fresh driver of the on-disk build, which re-runs the step.
The step's halt note, written before the driver decides, still named the stop `producer-hard-failure`.
The friction reflection then filed it as a lym defect, which is how issue #198 reached the board.

## What changes

- `loomengine.ClassifyHalt` takes a `buildSkew` flag.
  A failed, uninterrupted halt with the flag set is the new `AnomalyBuildSkew` kind, `build-skew`;
  every other failed halt stays `producer-hard-failure`, and an interrupted stop stays `interrupted-stop`.
- `loomcli`'s `failedHalt` sets the flag when the step error's chain holds a `*stencilstore.NewerBoardCopyError`,
  so the `loom-halt` note carries `anomaly: build-skew`.
- The reflection stencil `contracts/stencils/friction/friction-template-reflection.md` drops a `build-skew` halt,
  the same way it drops a spent budget or an escalation.
- `internal/loomcli/doc.go` names the new kind beside the interrupted-stop rule.

## Design decisions

- The parent narrowed the entry to the false hard-failure report, recorded as an `Added after Discussion` entry.
  The run's own records show that the driver's existing hand-over handled the incident:
  the fresh driver re-ran Code-Burler-Review 16 seconds after the halt, with no operator step.
  No hand-over before a step was added.
- The classification happens when the note is written, from the error alone, and needs no reading of the driver's later decision.
  When the on-disk binary is no newer than the board, the driver hands the run back instead of over.
  The halt is still a binary to update rather than a producer fault, so it carries `build-skew` there too.
- The run still records state `failed` for the step; only the note's anomaly kind and the reflection's handling change.

## Tests

- `TestStep_ProducerErrorWritesTheHaltNoteAndNoReflection` in `internal/loomcli/halt_test.go` is now table-driven.
  A new row drives a step whose producer returns a wrapped `NewerBoardCopyError` and expects `anomaly: build-skew`.
  On the merge base that row gets `producer-hard-failure` and fails.
- `TestClassifyHalt` gains a build-skew row, and a row in which an interrupted stop takes precedence over the flag.

## Merge danger

**Door:** two-way
**Blast radius:** friction
