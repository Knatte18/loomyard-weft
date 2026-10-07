---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: internal/websterengine/runlevel.go:446
    summary: Run's --fresh re-init forms and records a new partition and refuses on CheckBatchOrder before archiving state, and no test reaches either behavior
  - id: F2
    severity: LOW
    class: consistency
    location: internal/websterengine/rebaseline_test.go:598
    summary: The Regression330 test keeps an ErrBatchOrder expectation that TestRebaseline_CardSet already covers, and its no card-set refusal check cannot fail because the order check runs before every card-set and hash check
  - id: F3
    severity: NIT
    class: consistency
    location: internal/batcher/estimate_test.go:42
    summary: The new TestSegmentCost, TestDiskSizes and editCard carry no doc comment, while the new tests and helpers of cost_test.go and config_test.go in the same package do
---

Range: `status.json` gives `product.parent` = `main`, and `_lyx/loom/rework/` does not exist, so the range is `git diff $(git merge-base main HEAD)..HEAD`.
Build, `go vet` and `go test` over the touched packages pass.
Card 6's `Delete` group is satisfied: `batcher_test.go` and `registry_test.go` are gone, and no caller of `batcher.Select`, `register`, `lookup` or `SequenceBatches` remains.
Card 8's no-commit branch matches the plan: no `template.yaml` change beyond card 6's.

### [MEDIUM] --fresh re-init partition and order refusal are untested

**Location:** `internal/websterengine/runlevel.go:446` (the `opts.Fresh` branch of the state phase in `Run`), tests in `internal/websterengine/runlevel_test.go`.
**Issue:** Card 4 requires that on a new state, "none on disk, or the `--fresh` re-init", `Run` batches with the active batcher, refuses on `CheckBatchOrder`'s error before it saves anything, and records the partition.
The first-init half is covered: the `TestRun_RefusesBeforeSpawn` row "a first init whose batches run a dependency backward" and the `TestRun_MasterSpawn` row "a first init under a grouping batcher records the partition…".
The `--fresh` branch has its own `newPartition()` call placed before `archiveStateFile`, and its own `RecordPartition(st, batches)` on the re-initialised state, and no test reaches either.
`ErrBatchOrder` appears in `runlevel_test.go` only at the first-init row, and the only `Partition` assertion is the first-init one.
Dropping `RecordPartition` from the `--fresh` branch, or moving `newPartition()` after `archiveStateFile`, leaves every test green, although the first would silently put a `--fresh` run on the identity fallback and the second would archive a run's state and then refuse.
**Fix:** Extend existing tests, not a new one.
Give `TestRun_RefusesBeforeSpawn` a `fresh` field and add a row that seeds a matching state, runs `--fresh` over a two-card plan whose card 1 uses card 2's target, and checks `ErrBatchOrder`, the way forward, and that `state.json` is still in place and unarchived.
In the `--fresh` stale-state case of the re-init table (the one whose verify closure asserts the live `state.json` no longer carries the stale fingerprint), assert the re-initialised state records the active batcher's partition.

### [LOW] Regression330 test: redundant ErrBatchOrder half with a vacuous check

**Location:** `internal/websterengine/rebaseline_test.go:598` (`TestRebaseline_AfterBeginBatchRewroteBegunCard_Regression330`).
**Issue:** The focus asks whether a test still pins that `Rebaseline` naming a card is accepted after `BeginBatch` rewrote a begun card, with no card-set or hash refusal, and whether the `ErrBatchOrder` expectation lives in its own case.
No begin-batch variant can pin the accept any more: the rewrite happens only when a begun card uses a handle a later card declares, which is a backward dependency, so the plan that provokes it is refused by the order assertion.
The accept after a moved begun-card hash is still pinned by `TestRebaseline_AfterRecordBatchBoundBegunCard_Regression330` in `rebaseline_git_test.go`, over a plan with no backward edge, and the hash move itself by `moveBegunCardHashes`' test and `TestRestamp`.
The second half of this test asserts `errors.Is(err, ErrBatchOrder) && !errors.Is(err, ErrRebaselineCardSetChanged)`.
`rebaselineBatches` runs `CheckBatchOrder` before any card-set or hash check, and `ErrBatchOrder` does not wrap `ErrRebaselineCardSetChanged`, so the second operand cannot fail and the test no longer exercises what its comment says it guards.
The `ErrBatchOrder` expectation itself is covered by the "a tail that uses a later batch's target refuses with ErrBatchOrder" row of `TestRebaseline_CardSet` (card 7's own row for it) and by `TestCheckBatchOrder` in `sequence_test.go`, so here it is a duplicate living inside an unrelated test rather than its own case.
**Fix:** Drop the `Rebaseline` half and its `ErrBatchOrder` comment, keeping the begin-batch observation (the rewrite happens and the recorded hash follows it), and rename the test and fix its doc comment to say that; leave the `ErrBatchOrder` coverage to the `TestRebaseline_CardSet` row.

### [NIT] estimate_test.go declarations lack the doc comments their siblings carry

**Location:** `internal/batcher/estimate_test.go:42`, `:50`, `:154`.
**Issue:** The focus asks whether the test functions and helpers of `cost_test.go` carry a doc comment like their siblings: they do, every one.
The siblings in `estimate_test.go`, added by the same branch, do not: `editCard`, `TestSegmentCost` and `TestDiskSizes` have none, although every new test and helper in `cost_test.go` and `config_test.go` does and round 1's F5 fixed exactly this gap in `cost_test.go`.
**Fix:** Add a doc comment to each stating the behavior it covers or builds.

### Checked, no finding

- `PATTERN-comment-line-breaks`: every comment line the branch adds or changes in `internal/batcher`, `internal/websterengine`, `internal/webstercli`, `internal/gitrepo/blobread.go` and `tools/tokencount`, new test files and `doc.go` sections included, holds one sentence, and every break inside a sentence falls at a semicolon or a comma before a conjunction.
  Untouched wrapped lines in `config.go` and `verbs_test.go` are legacy and out of scope.
- The batchifier-failure refusal text is declared once, in `batchCards` (`internal/websterengine/partition.go`); `formBatches` and `rebaselineBatches` reach it through that function, and `rebaseline.go` holds no second copy.
- `validate`'s no-state order refusal is reached by `TestValidateCmd_RefusesBackwardPlanWithNoRun`, and the `formBatches` transient refusal by the `TestRun_RefusesBeforeSpawn` row "the batchifier fails" with `requireNoRecordedState`.
- `refusal-spec.md` rows for batch order, recorded partition mismatch and `batcher.yaml did not load` land with their tests (`sequence_test.go`, `partition_test.go`, `verbs_test.go`).
