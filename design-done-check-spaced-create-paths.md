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
