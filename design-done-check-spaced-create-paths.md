# webster: done-check rejects a Create path holding a space that webster validate accepts

`done-check-spaced-create-paths` · Note · bug, webster, quarry

Imported from [issue #805](https://github.com/Knatte18/loomyard/issues/805).

## What was seen

In the quarry hub, run `csharp-alphabet`, card 9 of the plan lists `internal/engine/testdata/csharp/tree/with space/Spaced.cs` under Create.
The directory name holds a space on purpose, as fixture coverage.
`lyx webster validate` accepts the plan.
After the batch committed the file, `record-batch 7` and `recover-batch 7` both refused with `batch_failed: glyph-rejected/9-csharp-fixture-roundtrip`, done-check target rejected before resolution ("a glyph needs a \"#\"", `no_separator`).
`lyx quarry resolve "<path>#"` answers `unit_bad_rune`, since a unit may not hold a space.
Batch 07 went terminal `failed` after one recovery, and the run blocked at Webster.

## Workaround

The parent decided to amend card 9: drop the spaced path from Create, keep it in Intent, rebaseline naming card 9, and re-drive batch 07.

## Expected

The done-check and the plan lint agree.
Either the done-check checks a Create path as a file path (existence in the batch's commits) rather than resolving it as a glyph, or `lyx webster validate` refuses a Create path the done-check cannot check, so the plan is fixed before any batch runs.


## From issue #809

Imported from [issue #809](https://github.com/Knatte18/loomyard/issues/809).

What happened: `lyx webster record-batch 7` refused with `batch_failed`: a card's done-check target, a repository-relative file path containing a space and no `#`, was "rejected before resolution: error glyph: parse ... as go: a glyph needs a "#"" (reason `no_separator`). The way forward named was `lyx webster recover-batch 7`. `recover-batch` then refused with the identical glyph-rejected error and the same way forward, so the recovery path points back at itself. The loom then halted `blocked` (anomaly `escalation-to-human`) and a second halt followed as `failed` (anomaly `producer-hard-failure`) with only the reason `loop-exited` and no further cause.

Why abnormal: the rejection is a plan-validation defect (a done-check target the glyph parser cannot take), but it was only detected at batch record time, after the batch's work was done. Neither the plan bouncer/burler stages nor recover-batch caught or repaired it, and recover-batch cannot clear a failure that its own re-validation reproduces. The `loop-exited` halt carries no diagnostic.

What to do: (1) validate done-check targets at plan review so the error surfaces before implementation; (2) have recover-batch either accept a corrected target or state that the card must be amended, instead of recommending itself; (3) give the `loop-exited` hard failure a reason; (4) consider whether a bare path with a space should be auto-addressed as `<path>#` as the error message itself suggests.

Provenance: task `csharp-alphabet`; notes loom-halt.md, loom-halt-2.md, webster-refusal-record-batch.md, webster-refusal-recover-batch.md.
