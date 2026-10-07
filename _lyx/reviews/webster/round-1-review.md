---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/websterengine/partition.go:1
    summary: New and changed Go comments are hard-wrapped at a fixed column mid-sentence, breaking PATTERN-comment-line-breaks across batcher, websterengine, webstercli, gitrepo and tools/tokencount
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/websterengine/rebaseline_test.go:612
    summary: Regression330 test was rewritten to expect ErrBatchOrder, so the accept-after-begin-batch-rewrite behavior it guarded is no longer pinned by any test
  - id: F3
    severity: MEDIUM
    class: scope
    location: internal/websterengine/runlevel.go:451
    summary: Three behaviors the diff adds have no test, namely the first-init Run refusal on ErrBatchOrder before any save, validate refusing the fresh partition's order error with no state, and the batchifier-failure transient refusal
  - id: F4
    severity: LOW
    class: consistency
    location: internal/websterengine/rebaseline.go:96
    summary: rebaselineBatches repeats the batchifier-failure refusal text that formBatches already declares, so the message lives in two places
  - id: F5
    severity: NIT
    class: consistency
    location: internal/batcher/cost_test.go:26
    summary: The batchSizes helper and TestCostBatchifier_Limits carry no doc comment while their sibling test functions do
---

### [BLOCKING] Fixed-column comment wrapping in new and changed Go comments

**Location:** `internal/websterengine/partition.go:1` and the other files below.
**Issue:** `PATTERN-comment-line-breaks` requires one sentence per line with a further break only at an independent-clause boundary, and no column limit.
Many comments the diff adds or changes break mid-sentence at about 100 columns, for example `partition.go` (file header, `ErrPartitionMismatch`, `formBatches`, `ExecutionBatches`, `mapPartition`, `RecordPartition`), `sequence.go` (header, `ErrBatchOrder`, `backwardEdgeFinding`, `firstSharedRef`), `cost.go` (header), `batcher.go` (`Batch.Profile`, `Batcher.Batch`), `config.go`, `registry.go`, `estimate.go`, `internal/batcher/doc.go`, `internal/websterengine/doc.go` (the two new sections), `internal/gitrepo/blobread.go` (header), and the new or changed comments in `tools/tokencount/calibrate.go`, `count.go` and `main.go`.
The same wrapping appears in new test file headers and doc comments (`partition_test.go`, `sequence_test.go`, `cost_test.go`, `config_test.go`, `calibrate_test.go`).
The rule governs new and changed comments only, so untouched legacy lines in those files stay as they are.
**Fix:** Reflow each new or changed comment to one sentence per line, with a further break only where a clause with its own subject and verb follows a comma plus conjunction or a semicolon; leave untouched lines alone.

### [MEDIUM] Regression #330 test no longer pins its behavior

**Location:** `internal/websterengine/rebaseline_test.go:612` (`TestRebaseline_AfterBeginBatchRewroteBegunCard_Regression330`).
**Issue:** The test guarded that `Rebaseline` naming card 3 is accepted after `BeginBatch` rewrote a begun card, with no card-set or hash refusal.
The diff changed it to expect `ErrBatchOrder`, because its fixture (`handlePlan(true, true)`) has begun card 1 using the handle card 2 declares, a backward `Uses` edge that `SequenceBatches` used to reorder around.
The refusal now comes before the begun-card hash checks, so the rewritten-begun-card path the regression is about is never reached, and no other test pins that accept.
**Fix:** Keep the regression's positive assertion.
Build the fixture so the rewrite of a begun card still happens while the plan holds no backward dependency, then assert `Rebaseline` accepts; if the scenario cannot be expressed without a backward edge, pin the rewritten-card hash path another way and keep the `ErrBatchOrder` row only as a separate case in `TestRebaseline_CardSet`, which already has one.

### [MEDIUM] Added behaviors with no coverage

**Location:** `internal/websterengine/runlevel.go:451` (`newPartition`), `internal/webstercli/validate.go:98`, `internal/websterengine/partition.go:37` (`formBatches`).
**Issue:** `PATTERN-refusal-way-forward` and the rubric's test-economy rule need each added behavior and refusal reached by a test.
No test reaches these:
- `Run` at first init (no state, and the `--fresh` re-init) refusing on `CheckBatchOrder`'s error before it saves anything;
- `validate` with no state refusing the fresh partition's order error;
- the transient refusal naming the plan when the batchifier's `Batch` returns an error.
`ExecutionBatches` is covered by `partition_test.go`, but `Run`'s own first-init path and `scopedValidate`'s no-state path are separate call sites.
**Fix:** Add a row to the existing first-init `Run` test (a plan with a backward `Uses` edge, asserting the error wraps `ErrBatchOrder` and no `state.json` exists), a row to the existing validate scope test in `cli_test.go`, and a row in `TestExecutionBatches_Refusals` for a batchifier that returns an error.

### [LOW] Duplicated batchifier-failure refusal text

**Location:** `internal/websterengine/rebaseline.go:96`.
**Issue:** `rebaselineBatches` builds the same `webster: batch the cards of plan %s: %w; way forward: transient, re-run the verb` message that `formBatches` declares, so a wording change must be made twice.
**Fix:** Extract the message into one helper in `partition.go` that `formBatches` and the tail batching in `rebaselineBatches` both call.

### [NIT] Missing doc comments on cost test helpers

**Location:** `internal/batcher/cost_test.go:26` (`batchSizes`) and `TestCostBatchifier_Limits`.
**Issue:** Sibling test functions and helpers in the package carry a doc comment stating what they assert or return; these two do not.
**Fix:** Add one-sentence doc comments.

## Verdicts on the focus directive

- Card 4 callers: `Run`, begin-batch, await-batch, record-batch, recover-batch, `validate` and `rebaseline` all form batches through `ExecutionBatches`, `formBatches` or the tail re-batch; no path recomputes batches from disk sizes mid-run, and only a state with no `Partition` falls back to `batcher.Identity()`. Not a defect.
- `CheckBatchOrder` refuses every edge from a batch to an earlier one, including two writers of one target out of card order, with the refusal text and way forward the plan and `refusal-spec.md` name. A partition that reorders independent cards with no dependency edge is accepted, as the plan specifies. Not a defect.
- Card 5: `cost.go` matches the feasibility rules and tie rule; `cost_test.go` carries a seeded brute-force comparison. Not a defect.
- Card 6: `Active` resolution, the load errors naming `batcher.yaml`, the `ConfigOpenMaps` wiring through `configreg.Modules()` and the `template.yaml` values all match the card. The deleted `batcher_test.go` and `registry_test.go` behaviors survive in `TestActive` and `identity_test.go`, and no reference to `Select`, `register` or `lookup` remains. Not a defect.
- Card 7: the kept prefix comes from `State.Batches` records, a card-set change at or before the last begun batch refuses with `ErrRebaselineCardSetChanged`, and `Partition` is replaced only after every check passes. Not a defect.
- Card 8: no commit; `template.yaml` changed only in card 6 and no calibration output was committed. Not a defect.
- Docs: the `doc.go`, `docs/overview.md`, `PATTERN.md`, `refusal-spec.md`, `loom-plan-spec.md` and `webster-spec.md` changes land in the cards the plan assigns them to; the only doc finding is F1.
- Build, `go vet` with every tag, `go test ./...` and the `integration` run over the verify packages all pass.
