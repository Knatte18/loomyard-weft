# Round 2 fixer report — plan

All three findings are fixed.
`lyx loom validate-plan` on the edited plan returns `ok: true`, with informational `create-new-unit` findings only.
The `approved:` key in `00-overview.md` is untouched.

## Changes

- **F1 (card 4's caller edits overwritten by card 7 and card 8).**
  Old card 4 (`04-batcher-takes-sizes.md`) is folded into the recorded-partition card, now `04-recorded-partition.md`, placed straight after the calibration report.
  The merged card carries the `Batcher.Batch(plan, cards, sizes)` signature, `Batch.Profile` and `Batch.Estimate`, `Identity()`, the order assertion, the recorded partition and every verb's move to `ExecutionBatches`, so each call site changes once.
  Its `Uses` are `DiskSizes` and `SizeSource`; the former `Uses` of `Identity` went, since the card now creates it.
  The cost batchifier and the profiles cards keep numbers 5 and 6, and `Identity` is created before card 6 uses it.
  Old cards 8 and 9 are now `07-rebaseline-tail.md` and `08-calibrate-template.md`; the old `04-batcher-takes-sizes.md` and `07-recorded-partition.md` are deleted.
  Headings, the Card Index, the shared decision naming the no-commit card and every in-card "card N" reference follow the new numbers.
  `webstercli/rebaseline.go` is still edited by the merged card and by the rebaseline-tail card, which replaces the `Batches` dep; the merged card says so.
  `registry_test.go`'s probe batcher is still edited for compilation before card 6 deletes the file, as round 1 judged.
- **F2 (no test for the `BaseSHA` extraction).**
  Card 3 gains `tools/tokencount/count_test.go` in its `Edit` targets and a `TestCountRun` extension: two `begin-batch` tool-use and tool-result pairs, an error envelope without `start_sha` before them and an unrelated tool result mentioning `start_sha`, asserting `BaseSHA` is the earliest successful envelope's value and stays empty when none exists.
  The card now states the recognition rule: the `tool_result` paired by `tool_use_id` with a `Bash` `tool_use` invoking `lyx webster begin-batch`, whose content parses as a JSON object with a non-empty `start_sha`.
- **F3 (`TestActive` negative-coefficient row).**
  Card 6 drops that row from the `TestActive` plan and says the `verbs_test.go` rebaseline step pins that a coefficient error reaches `Active`'s caller naming `batcher.yaml`; the other `TestActive` rows are unchanged.
  This replaces the row round 1 kept as a pass-through proof, since the `verbs_test.go` step already asserts it.

## Deferred

None.
