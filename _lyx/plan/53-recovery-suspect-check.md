# Card 53 — recovery-suspect-check

**Edit:**
- `internal/websterengine/state.go`
- `internal/websterengine/suspect.go`
- `internal/websterengine/batchfail.go`
- `internal/websterengine/recordbatch.go`
- `internal/websterengine/recoverbatch.go`
- `internal/websterengine/doc.go`
- `internal/websterengine/batchfail_test.go`
- `internal/websterengine/recoverbatch_test.go`

**Intent:**
Fix PR #321 round-3 finding A's last item: a batch failed on a correctness finding names its suspect paths only in the digest's prose, and `PersistRecoveryTerminal` checks none of them, so a recovery strand that ignores the revert instruction still records the batch done.

The check here is not accept-audit's "matches the base" rule.
A Master that implemented a card itself writes that card's own target files, which the recovery strand must re-derive rather than leave at the start commit, so a strict revert check would fail every honest recovery of such a card.
What a strand that ignored the instruction leaves behind is the flagged content itself, unchanged or uncommitted, and that is what the check looks for.

- `state.go`: `BatchState` gains `SuspectPaths []SuspectPath` (`json:"suspectPaths,omitempty"`).
  `SuspectPath` carries `Path` (`json:"path"`) and `Blob` (`json:"blob,omitempty"`): the git blob id of the path's worktree content when the batch failed, empty when the file was absent or lies outside the task worktree's tracked tree.
- `suspect.go`: `worktreeBlob(worktree, path string) (string, error)` (`git hash-object`, "" for an absent file) and `commitBlob(worktree, commit, path string) (string, error)` (`git rev-parse <commit>:<path>`, "" when the commit does not hold the path).
- `batchfail.go`: `failBatchInput` gains `WorktreeRoot`.
  `failBatch` computes each suspect path's `Blob` before archiving, so a git failure still mutates nothing, and records `Batch.SuspectPaths`.
  `recordbatch.go`'s `failFromFindings` passes `deps.Geom.WorktreeRoot`.
- `recoverbatch.go`:
  - `recoverSpawn` carries `prior.SuspectPaths` onto the fresh recovery record, as it carries `AuditWarnings`.
  - `PersistRecoveryTerminal`, after resolving `head` and before `postBatchChecks`, checks every `bs.SuspectPaths` entry and collects a reason for each that fails:
    - a plan file whose hash differs from `PlanFileHashes` (`checkSuspectPaths` with any base): `suspect path <p> still differs from the plan as the run recorded it`;
    - a path in the tracked tree that `checkSuspectPaths` with `head` as base reports differing: `suspect path <p> has changes the recovery report's head <head> does not hold`;
    - a path in the tracked tree with a non-empty `Blob` whose `commitBlob` at `head` equals `Blob` and differs from its `commitBlob` at `bs.StartSHA`: `suspect path <p> still holds the content the audit flagged; revert it to <StartSHA> or re-derive it`.
    A path `checkSuspectPaths` reports unverifiable is not checked: the failing verb already archived the batch's report, and webster rewrites its own state.
  - Any collected reason fails the batch through `failBatch` with those reasons and the same suspect paths, so the next `recover-batch` checks them again and its strand reads the reasons in its failure digest.
  - The doc comment states the check and its one accepted residual: a strand whose re-derivation is byte-identical to the flagged content is failed again, and the operator's way forward is to revert the path and edit its card.
- `doc.go`'s recover-batch paragraph states the check.

Tests:

- `batchfail_test.go`: a failed batch whose suspect path is a modified tracked file records `SuspectPaths` with that file's `git hash-object` blob;
  an absent suspect path records an empty `Blob`.
- `recoverbatch_test.go`, over the existing recovery fixture with a batch failed on a Master parent-write to a tracked file `internal/x.go` whose flagged content is `forged`:
  - `TestPersistRecoveryTerminal_FailsWhenSuspectContentSurvives`: the strand commits `internal/x.go` still holding `forged` → `ErrBatchFailed` naming the path and `revert it`, the record terminal failed with `SuspectPaths` kept;
  - `TestPersistRecoveryTerminal_FailsOnUncommittedSuspectPath`: the strand commits other work and leaves `forged` uncommitted → `ErrBatchFailed` naming the path and the head;
  - `TestPersistRecoveryTerminal_PassesReverted`: the strand restores `internal/x.go` to `StartSHA` → done;
  - `TestPersistRecoveryTerminal_PassesRederived`: the strand commits different content → the suspect check passes and the batch records done;
  - `TestRecoverSpawn_CarriesSuspectPaths`: the recovery record spawned from the failed one carries its `SuspectPaths`.

**ImpactSummary:** A failed batch persists its suspect paths with the flagged content's blob, and recovery records the batch done only when each suspect path was reverted or re-derived and committed; every other recovery path is unchanged.
