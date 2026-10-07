# Fabric's pushes and exported names

All six batches reported `done`.

## Push behavior (batch 01, cards 1–3)

- Card 1: `Add`'s warp and weft branch pushes retry a bare `(failed)` rejection through a bounded, per-`Topology` injectable seam in `pushretry.go`.
- Card 2: `rollbackAdd` deletes the warp branch its `Add` created, locally, and on origin only when the probe found it absent and the lease holds.
- Card 3: in the prime, `PushPairAnchored`, `Fabric.Commit` and the `sync` verb push the records side only, and `push` reports `code_push_skipped` when the code branch has unpushed commits.
- Deviation: `IsPrimeWorktree` was added to `internal/fabricengine/worktreelist.go`, outside any declared target.

## Neutral names (batches 04–07)

- Batch 04: hubforge's five side-named accessors and bare-remote fields now read `PairCodeWorktree`, `PairRecordsSibling`, `PrimeRecords`, `CodeBare` and `RecordsBare`.
- Batch 05: fabricengine's side-named exports and `ResetPairWarp` took code/records names.
  The fork touched comments and tests in 19 files outside the Edit list (reported deviations), and `gofmt`-ed two files that were already unformatted.
- Batch 06: `FABRIC_SKIP_GIT` and `FABRIC_SKIP_PUSH` replace `WEFT_SKIP_*` with no fallback, and the refusal says "records sibling of a pair".
- Batch 07: result fields became `CodeWorktree`/`RecordsWorktree` with JSON `code_worktree`/`records_worktree`, the remove envelope keys became `CodeBranch…`, and mutation details read `side=code` or `side=records`.

## Files, handles and the exemption (batch 08, cards 8–10)

- Card 8: `warpprobe.go`, `warpbinding.go` and `weftgit.go` were renamed with `git mv` to `recordsprobe.go`, `codebinding.go` and `recordsgit.go`, and the `Fabric` handle fields became `code` and `records`.
- Card 9: `fabricengine.RecordsLockDirPath` serves hubgeom's lock directory, and the retired-spelling tests moved into `refscanner_test.go`.
- Card 10: `vocabExempt` and its stripping were deleted, and `pattern/PATTERN-fabric-vocabulary.md` says so.

The plan-level verify is left to the gate.
