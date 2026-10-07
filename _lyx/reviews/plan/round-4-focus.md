---
round: 4
focus:
  - "Sequencing of the Batcher signature change: for the batch-forming call in `internal/webstercli/rebaseline.go` and in `websterengine.Rebaseline`, list which cards edit it and whether any card's edit of it is later replaced wholesale by another card; settle whether every edit survives or exists only to keep the tree compiling between two cards."
  - "`RunTally.BaseSHA` in `tools/tokencount`: settle whether the plan states by what ordering the earliest `lyx webster begin-batch` envelope is chosen (file-name order versus the line's `timestamp`), and whether the `TestCountRun` fixture pins that ordering and describes the existing `webster` session fixture accurately."
  - "Refusal coverage in the rebaseline tail (`CheckBatchOrder` over kept batches and re-batched tail, `State.Partition` replaced only after every check passes): settle whether a test row reaches the order refusal and pins that the refusal leaves `State` unchanged, as `PATTERN-refusal-way-forward` requires."
  - "`TestActive` in `internal/batcher` under `PATTERN-test-economy`: settle whether any two rows reach the same error path through `Active`."
---

Entries name where to look; none states a verdict.
