# webster done-check: Create paths with a space and test targets quarry does not find

`done-check-spaced-create-paths` · Task · bug, webster, quarry · running · Darn-Burler-Fix

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

## Operator decision (2026-10-22)

The done-check accepts a Create path holding a space, matching what `webster validate` already accepts; `webster validate` is not made to refuse it.
The `loop-exited` halt reason is out of scope here: task `loop-exit-halt-names-its-cause` takes it.

## Folded in: test targets quarry does not find (operator, 2026-10-22)

Imported from [issue #823](https://github.com/Knatte18/loomyard/issues/823).

In task `machine-handover`, `lyx webster record-batch` refused batches `01` and `09` with `create-not-done[blocking]`: `Create target "plan:<pkg>#<TestName>" still does not resolve ... quarry answered not_found (the unit exists and the member is missing)`.
Batch `09`'s target sat in an external test package (`internal/loomcli_test#...`).
The refusal cannot tell a card that never wrote the test from a lookup that misses one that exists, and names no file or package quarry searched.
In `run-process-ownership` (2026-10-22) the same refusal named two tests the batch had written in other files than the card said.

What to do:
- Check whether quarry resolves `plan:` targets in `_test` external test packages and in test files created in the same batch.
- Make the `create-not-done` message name the files quarry searched, so a missing test is told apart from a lookup miss or a test in another file.
