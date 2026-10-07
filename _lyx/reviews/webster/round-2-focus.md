---
round: 2
focus:
  - "Go comments the branch adds or changes in internal/batcher, internal/websterengine, internal/webstercli, internal/gitrepo/blobread.go and tools/tokencount (including new test files and the new doc.go sections): settle whether each follows PATTERN-comment-line-breaks, one sentence per line with no mid-sentence break at a fixed column; untouched legacy lines are out of scope."
  - "internal/websterengine/rebaseline_test.go, the Regression330 test: settle whether a test still pins that Rebaseline naming a card is accepted after BeginBatch rewrote a begun card, with no card-set or hash refusal, and whether the ErrBatchOrder expectation lives in its own case."
  - "Behaviors the diff adds in internal/websterengine/runlevel.go newPartition (first-init and --fresh Run refusing on CheckBatchOrder before saving state), internal/webstercli/validate.go (no-state validate refusing the fresh partition's order error) and internal/websterengine/partition.go formBatches (transient refusal naming the plan when the batchifier's Batch errors): settle whether a test reaches each, extending an existing test rather than adding a duplicate."
  - "The batchifier-failure refusal text in internal/websterengine/partition.go and internal/websterengine/rebaseline.go: settle whether the message is declared in exactly one place."
  - "Test functions and helpers in internal/batcher/cost_test.go: settle whether each carries a doc comment like its siblings in the package."
---

The focus names where to look; the base diff review and the rubric's other dimensions still apply in full.
