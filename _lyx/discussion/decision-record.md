# Decision record — webster-reset-reopens-dead-batch

## Goal

After `lyx webster reset --to batch-start --batch NN` over a batch whose recovery ended dead, Merriam runs `lyx webster recover-batch NN` itself instead of writing `outcome: stuck` again.

## Scope

- `RecoveryRetry` in `internal/websterengine/recoverbatch.go`: the retry verdict `status`, `recover-batch` and `begin-batch` read.
- `existingReportRemedy` in `internal/websterengine/beginbatch.go`: its retry wording.
- The Master stencil `contracts/stencils/webster/webster-template-master.md`: the dead rungs' wording of what `recovery_retry: true` means.
- The docs naming the retry rule: `internal/websterengine/doc.go`, the `recover-batch` long help in `internal/webstercli/recoverbatch.go`, and the `RecoveryStartSHA` field comment in `internal/websterengine/state.go`.
- Out of scope: the reset verb's own state writes, which already clear the count and the start, and the recovery cap.

## Decisions

### A dead recovery with no counted recovery earns a retry

Decision: `RecoveryRetry` reports true for a terminal dead recovery record whose counted recoveries are zero, with no commit check.
The commit check stays the rule for a record with one counted recovery.
Rationale: `reset --to batch-start` clears the batch's count and recovery start, so the batch has its full allowance and its first counted recovery is owed, as it is for a fork that reported stuck.
Every reader of the verdict, `status`, `recover-batch`'s terminal envelope and `begin-batch`'s report-present remedy, then names `recover-batch` for the reset batch, and the stencil already acts on `recovery_retry: true`.
Bound: the new edge lets through only a record at zero counted recoveries; the retry it grants is itself a counted spawn, so the cap of two and the commit rule bound every later retry.
A dead recovery whose spawn rendered an amendment and so counted nothing also reads as retryable, which the uncounted-spawn rule already allows when `recover-batch` is run.

### The reset keeps the batch's terminal dead record

Decision: the reset verb's state writes stay as they are: the batch stays terminal and dead, with its count and start cleared.
Rationale: a non-terminal recovery record makes `recover-batch` attach to the dead strand instead of spawning, and rewriting the digest status would falsify the batch's report.

## Constraints

- One rule decides the retry: `status`, `recover-batch` and `begin-batch` keep reading `RecoveryRetry` alone.
- `PATTERN-refusal-way-forward`: the begin-batch remedy keeps naming the step to take.
- `PATTERN-comment-line-breaks`: one sentence per line in every comment touched.

## Auto-mode assumptions

- The operator expects Merriam to run the recovery itself after the reset, the first of the two outcomes the board entry names as acceptable.

## Open risks

- A Master session already running when the reset lands reads the verdict only on its next status read, so the orch still resumes the run after the reset.

## Acceptance criteria

- `RecoveryRetry` gives true for a terminal dead recovery record at zero counted recoveries with an empty recovery start, and keeps the count at zero.
- `RecoveryRetry` still gives false for a dead recovery at one counted recovery with no commits of its own, and at the cap.
- `begin-batch`'s report-present remedy for a reset dead batch names `lyx webster recover-batch NN`.
- `lyx webster status` reports `recovery_retry: true` for a reset dead batch.
- The Master stencil states that `recovery_retry: true` covers a batch whose count a reset cleared.
