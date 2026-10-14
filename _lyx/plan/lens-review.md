# Lens review

Every fork returned a report.

- criteria-coverage, MEDIUM, no batcher row for a symbol-level Rename of a created member: fixed, card 2 adds the row `internal/a#Foo` -> `plan:internal/a#Baz` after a create of `plan:internal/a#Foo`, sizes `[1, 1]`.
- criteria-coverage, LOW, no rebaseline row for D1: rejected, `internal/batcher` is the one grouping site both a first init and a rebaseline derive through (`PATTERN-batcher-registry`), so a rebaseline row would retest the batcher rows.
- criteria-coverage, LOW, D4 pinned only in the stencil, not a rendered prompt: rejected, `RenderForkPrompt` fills the body verbatim (`internal/websterengine/render.go`, `stencil.FillOptional` over `composeForkTemplate`), so the `wordingClaims` row fails when the rule leaves the body.
- criteria-coverage, LOW, the drop rule's re-add branch had no row: fixed, card 6 adds a row where the emptying card also creates `internal/g/new.go#`.
- criteria-coverage, LOW, no recover-batch row for the over-long line: rejected, recover-batch renders through the same `renderCardGates` and `guardNotice`, and its guard-step row pins the shared path.
- test-economy, LOW, card 6's no-`go.mod` and no-marked-test rows repeat card 5's scan rows: fixed, dropped; a zero `guardTests` is reached by every existing `TestCardGateCommand` row.
- test-economy, LOW, `TestRenameBoundary_BatchesCheckAndGateTheirOwnTree` re-derives the partition: rejected, deriving the partition through the cost batchifier is what ties the split to the done-checks and gate it fixes; literal batches would pass with D1 absent.
- test-economy, LOW, `TestExecutionBatches`' no-state row repeats the batcher row: fixed, dropped; only the recorded-partition row remains.
- test-economy, LOW, the unmappable and edit-same-package batcher rows repeat `TestRefCovers` rows: rejected, each pins a batch-level acceptance criterion through `renameBoundaries`, which `TestRefCovers` never reaches.
- test-economy, LOW, the over-long step is tested twice: rejected, the `TestCardGateCommand` row pins the returned length and the omitted step, and the prompt row pins the too-long fragment's fill; they are different outputs.
- row-reachability, MEDIUM, the fix-line row's pointer assertion cannot fail: fixed, the row now asserts the pointer occurs after the scan-error text's `internal/g/g_test.go`, which only the fix line produces.
- row-reachability, MEDIUM, card 6's unreadable-directory rows never restore the mode: fixed, both rows restore it in `t.Cleanup`.
- row-reachability, LOW, the drop row's fixture file set was unstated: fixed, card 6 names `internal/g` holding exactly `g.go` and `g_test.go`.
- row-reachability, LOW, the identity row cannot fail on a wrong D1: rejected, it pins the "identity unchanged" criterion, and `identityBatcher.Batch` (`internal/batcher/identity.go`) ignores target groups by design.
- forced-edits, LOW, batcher feasibility sentences not amended: fixed, card 2 names `costBatcher.Batch`'s doc comment, `cost.go`'s header and `doc.go`'s feasibility sentence.
- forced-edits, LOW, `RenderForkPrompt` and `RenderRecoveryPrompt` doc comments describe `cardGates` as the step list only: fixed, card 6 lists both members and states the change.
- forced-edits, LOW, `writeGateFile` writes only `x` and the card named no source-writing helper: fixed, card 5 declares `writeGuardFixture` in `guardstep_test.go`, and card 6 uses it.
- forced-edits, LOW, the walk refusal fired after mutations under a transient row: fixed, the scan now runs before the reports-dir `os.MkdirAll` in `BeginBatch` (`internal/websterengine/beginbatch.go:505`) and in `recoverSpawn` before `archiveStaleReport` (`internal/websterengine/recoverbatch.go`), and the recover row asserts a seeded stale report is left in place.
