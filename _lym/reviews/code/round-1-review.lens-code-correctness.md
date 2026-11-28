```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each changed function and branch on a concrete input.

**Cases run**
- `gitrepo.UpdateRemoteBranchLeased` auth branch: stderr is GitHub's 403 lines with no "stale info". The error wraps both `ErrPushAuthRefused` and the git error, and a lost lease still returns `ErrLeaseRejected` bare. A non-`GitError` error leaves `gitErr` nil, so the branch is skipped. Correct.
- `seedPushOutcomeUnknown` and `seedPushCause` (destroy.go:1845, 1949):
  - An auth refusal on the records push gives false, so no withdraw runs and `fail` removes the hub with the credentials way forward.
  - An auth refusal on the board push calls `fail(..., recordsTip)`, which withdraws the records branch that already landed.
  - Correct.
- `deleteLeftoverUnderLock` (cleanupremotewarp.go:169):
  - A local-only `archive/<slug>/<tip>` tag in the code store is pushed before the delete. Correct.
  - A failed push sets Error and keeps the branch. A partial push keeps the tags already pushed, recorded on rec. Correct.
  - A dry run, or a nil `openPRHeads`, returns before any push. Correct.
  - A tip held only by another local branch or by another slug's tag: see F5.
- `existingHubError` (clone.go:42):
  - A hub with no Bolt `.git` names `--reset`. Correct.
  - A clone killed after `ensureBoardWorktree`: see F1.
- `checkResetTarget` / `hubWorkOriginLacks` (clone.go:815-879):
  - A clean hub, a pair's `.git` file, an unpushed commit on a branch, and a code store with no refs all behave correctly. With no refs, rev-list's pseudo-option counts 0.
  - A commit held only by a local-only tag, and uncommitted edits or a stash in the prime: see F2 and F3.
- `FetchRecords` / `FetchPairTips` / `fetchRunForCatchUp`: a lost layout returns `ErrRecordsLayout` and nothing is fetched. A fetch failure is wrapped and reads as unreachable origin. Correct.
- `SettleGonePairTeardown` (bool return): no record gives (false, nil); a held lock or a part of the pair left gives (false, err) or (false, nil); a dropped record gives (true, nil). Correct.
- `remover.run` / `settleGonePair` (remove.go:113-149):
  - Ordinary and local verdicts with `--remote` and `ErrPairNotFound` both reach settle.
  - Without `--remote`, `ErrPairNotFound` is returned as is. With no record, it is returned unchanged. A settle error names the re-run.
  - Correct, except the stray-path case and the unreported mutations: see F4 and F6.
- `RunTeardownChild` with empty input: the record is kept. A parent with no job writes no record (remoteteardown.go:372-377), so nothing is orphaned. Correct.
- `recordsArchiveWayForward` without the condition: when origin holds nothing of the pair, the re-run now settles through remove. Correct.

**Findings**

- **F1 — MEDIUM, design, `internal/fabricengine/clone.go:43`.**
  `existingHubError` treats any hub whose Bolt checkout holds `.git` as whole.
  A clone killed after `ensureBoardWorktree` (clone.go:465) but before its excludes, `.lym`, anchor commit and layout resolve (clone.go:471-576) leaves a partly cloned hub.
  That hub now gets "work in it" as its way forward, though nothing finishes it, and `RefuseHubWithoutBolt` (clone.go:81-84) passes it.
  The `--reset` clone that would replace it, and that `hubWorkOriginLacks` would allow, is no longer named.
- **F2 — MEDIUM, design, `internal/fabricengine/clone.go:870`.**
  `rev-list --count --branches --not --remotes --tags` counts a local-only tag as something origin holds.
  A commit held only by a local `archive/<slug>/<tip>` tag that a teardown child never pushed counts as held, so `--reset` deletes the only copy. Such a tag is what the pending-teardown record names.
  This is the same defect the cleanup sweep fix removes (cleanupremotewarp.go:169).
  A stale remote-tracking ref for a branch origin no longer holds is counted the same way, through `--remotes`.
- **F3 — MEDIUM, design, `internal/fabricengine/clone.go:851-879`.**
  `hubWorkOriginLacks` reads only linked worktrees and branch commits.
  Uncommitted edits in the prime code worktree, its stash (`refs/stash`) and commits on a detached HEAD pass the refusal, and `resetHub` removes them.
  The summary's design decision exempts only the records store.
- **F4 — LOW, design, `internal/fabriccli/remove.go:113`.**
  When `refuseWhenNothingRemains` returns `ErrPairNotFound` naming a stray path left alone (remove.go:534-536), settling replaces that error with a success.
  The result carries no `StrayPath`, so `stray_path` is never reported.
- **F5 — LOW, design, `internal/fabricengine/cleanupremotewarp.go:169`.**
  The landed check counts any other local branch (`--glob=refs/heads/*`) and any tag (`--tags`) as holding the tip (destroy.go `checkUnlandedRemoteTip`).
  Only the slug's own archive tags are pushed, so a tip held only by a local branch of another name is still deleted from origin.
  The design decision acknowledges other tags but not local branches.
- **F6 — LOW, design, `internal/fabriccli/remove.go:149` and `internal/fabricengine/pendingteardown.go:421`.**
  `SettleGonePairTeardown` records its tag pushes on a `Mutations` it discards.
  The settled removal returns an empty `MutationRecord`, so the envelope reports no mutation although archive tags were pushed to origin.

Tool calls:
1. Read `refuseWhenNothingRemains` (internal/fabricengine/remove.go:510-540) and `clearStray` (internal/pairteardown/teardown.go:560-600). My context lacked what the not-found error carries and whether pairteardown passes it on.
2. Read pairteardown/doc.go:228-242 and the returns of `Teardown.Run`. My context lacked whether `td.Run` returns `ErrPairNotFound` to the remover in production.
