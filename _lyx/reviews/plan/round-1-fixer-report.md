# Round 1 fixer report — plan

All five findings are fixed.
`lyx loom validate-plan` on the edited plan returns `ok: true`, with informational `create-new-unit` findings only.
The `approved:` key in `00-overview.md` is untouched.

## Changes

- **F1 (cards 4, 7, 8 edit the same call sites).**
  Old card 7 (`07-batch-order-assertion.md`) and old card 8 (`08-recorded-partition.md`) are one card, `07-recorded-partition.md`.
  It creates `CheckBatchOrder`, `ExecutionBatches` and the partition, removes `SequenceBatches` and the cycle machinery, and edits each verb file once.
  The merged card states both bounds: the order assertion lets nothing new through and is bounded by Plan-Gate, and the recorded partition changes only at first init or rebaseline.
  The old order-assertion and recorded-partition files are deleted.
  Old card 9 is now `08-rebaseline-tail.md` and old card 10 is `09-calibrate-template.md`; headings, the Card Index, the shared decisions and every in-card "card N" reference follow the new numbers.
  `webstercli/rebaseline.go` stays edited by cards 4, 7 and 8, because each edit is forced by compilation until card 8 replaces the `Batches` dep.
- **F2 (`profiles:` not an open map).**
  Card 6 gains `ConfigOpenMaps()` returning `profiles`, passed by the shared load helper, set on the `batcher` entry in `internal/configreg/configreg.go`, and pinned by a `TestLookup` row in `configreg_test.go`.
  Its `TestActive` plan gains a row showing a file whose `profiles:` lacks `cautious` fails to load when `active:` names it.
  Card 6's `ImpactSummary` names the `configreg` change.
- **F3 (card 1 attribution against the real transcript).**
  Card 1 now says the fork reads its prompt with `Read`, that the `<number><TAB>` line prefix is ignored, and that a pointer line has nothing after the closing backtick, which excludes the `: <command>` gate lines.
  Its `count_test.go` fixture is built in the `Read` form with a gate line naming a third card that must not count.
- **F4 (card 6 repeats card 2's coefficient rows).**
  Card 6's `TestActive` plan keeps the rows `Active` adds and one row showing a negative coefficient's error reaches `Active`'s caller naming `batcher.yaml`; the missing, negative and unknown coefficient cases stay in card 2's `ProfileWeights` rows.
- **F5 (card 10 wording).**
  The sentence now says the worktree's `batcher.yaml` holds no `profiles:`, so the template's profiles fill in.

## Deferred

None.
