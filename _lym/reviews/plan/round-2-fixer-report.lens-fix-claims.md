```yaml
lens: fix-claims
outcome: reported
```

The fix's claims about the code mostly hold, but `checkTreeAndHead`'s new signature can't call the git surface it says it uses.

**MEDIUM**: the `checkTreeAndHead` signature in card 10 (`func checkTreeAndHead(git websterengine.Git, parentBranch, reportHead string) (dirtyPaths, reasons []string, err error)`) doesn't match the calls the card says it makes.
- The card says it runs `DirtyPaths` and `Git.MergeRejection` against the told parent branch.
  - `websterengine.Git.DirtyPaths(worktree string)` (`internal/websterengine/git.go:22`) needs a worktree root, and the signature carries none.
  - `MergeRejection(worktree, commit string, parents []string, parentBranch ParentBranchFunc)` (`git.go:29`) also needs the worktree root.
  - Its parent-branch argument is a `ParentBranchFunc`, which is `func() (string, error)` (`gitwrap.go:72`), not a string.
- Card 11 says Weaver's `RunDeps.ParentBranch` may be nil, and a `string` parameter cannot carry that "no parent branch told" case.
- Resolving an abbreviated `head_sha` to a full SHA also needs the worktree.
- So the signature needs the worktree root (or the `Geometry`) and a `websterengine.ParentBranchFunc` in place of `parentBranch string`.

These claims check out:
- **Lease (F1):** a second `runplan.AcquireStateMutation` in one process blocks forever. `lock.AcquireWriteLock` opens a new `flock.New(...).Lock()`, which is a blocking `LOCK_EX` on a new file description.
- **`runplan.Distill`:** it needs a report; its signature is `Distill(r *Report) Digest`.
- **`runplan.PostBatchChecks`:** it can run over the recorded start commit with no renames. `PostBatchInputs` has `StartSHA`, `HeadSHA` and `Renames`.
- **`runplan.BatchState`:** it has `Terminal` and `Status`, so "terminal done with no digest" is something it can record.
- **`outcome.yaml`:** Webster's keys are `outcome`, `stuck_reason` and `batches_done`, decoded with `KnownFields(true)`, and `done` and `stuck` are the only legal values. Card 23's "refusing an unknown key" and its `stuck` row agree with that.
- **Wiring (F6):** `loomcli/wiring.go:852-853` sets `WebsterRun: websterengine.Run` explicitly, as card 18 now says.

Tool calls:
1. Grepped `Distill`, `PostBatchInputs`, `BatchState` and the `websterengine.Git` methods. My context had only their names, not their signatures and fields.
2. Grepped `ParentBranchFunc` and card 11's `ParentBranch` lines. My context did not have that type or Weaver's `RunDeps.ParentBranch` contract.
