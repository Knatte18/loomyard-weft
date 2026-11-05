```yaml
lens: code-correctness
outcome: reported
```

**Lens: code-correctness.** For each changed function or branch: the case I ran, and whether it was correct.

- **`Remove` stray-delete branch (`remove.go:332-341`):** I ran a registered pair whose worktree was refilled after removal and whose delete fails. `strayErr` is set, `res.StrayPath = target`, and the error comes back as `res, strayErr`. Correct.
- **`Remove` return paths (`remove.go` local mode, `!out.ran`, final):** I ran a handed-over remote step with `out.err == nil` and `strayErr != nil`. `strayErr` is returned. When `out.err != nil`, `strayErr` is dropped, but `StrayPath` in `res` still names the directory. Acceptable.
- **`Remove` respawn defer (`remove.go:230`):** I ran `err` wrapping `ErrStrayPathKept` with the worktree gone and the records worktree gone. No respawn, which is right. If the records worktree were still present, the push would not respawn for this error either, and that is the only gap.
- **`RemoveStrayTaskPath` (`destroy.go:1419`), five cases:
  - Absent path: returns `false, nil`. Correct.
  - Registered worktree: refused. Correct.
  - Directory holding `.git`: refused. Correct.
  - `List` fails: refused with a plain error, so it can never delete a real worktree. Correct.
  - Plain directory whose delete fails: wraps `ErrStrayPathKept`. Correct.
- **`clearStray` (`teardown.go:456`), four cases:
  - Remove succeeded and a stray is deleted: success. Correct.
  - `ErrPairNotFound` with a stray deleted: `ErrPairNotFound` is passed through, so `finishRemoval` and cancel read it as done. Correct.
  - `ErrStrayPathKept` and the delete clears it: converted to success, and the claim is then settled. Correct.
  - Delete fails, on a resumed teardown too: `strayErr` is returned and the claim is not settled, so a resume retries. Correct.
- **`teardownRefusal` (`wire.go:416`):** I ran `ErrStrayPathKept` wrapped by `%w`. The sentinel is kept and the wording names the resume. Correct.
- **`refuseNothingLeft` (`cancel.go:610`):** I ran a stray alone, once with `remove` and once without. Without `remove` it refuses and names `--remove`. With `remove` it returns nil and skips the `TaskBranchOnOrigin` probe, which is fine because there is something to delete. Correct.
- **`removePairAndBranch` (`cancel.go:666`):** I ran a stray alone with `--remove`. `ErrPairNotFound` is treated as removed, then `PairParts` and `TaskBranchOnOrigin` are read and the branch discard is skipped. Correct.
- **`beamMembers` (`beamland.go:127`):** I ran a stray-only member. `HasPair` is false. `PairWorktreeStands` returns true when `List` fails, which is a conservative error direction. Correct.
- **`PairParts.Any()` and `StrayPath`:** I ran a stray alongside other parts. `report.pairKept = parts.Any() && !remove` (`cancel.go:360`) now becomes true for a stray alone, which is intended. `leftPartNames` in `startguard.go` deliberately omits `StrayPath`, so a start is not refused by a stray, and `Add` still refuses it by name. Correct.

**Findings**

1. **LOW / design, `internal/fabricengine/destroy.go:1419-1446`.** `RemoveStrayTaskPath` deletes any directory at the slug's task worktree path that has no `.git` entry. It checks neither dirtiness nor that a pair ever stood there. The code relies entirely on caller knowledge.
   - Worktree-Teardown passes `Force: false` and still deletes whatever is there.
   - `cancel --remove` on a slug that never had a pair deletes any `<hub>/<slug>` directory, such as an operator's unrelated folder.
   - `refuseNothingLeft` now lets that through, because the directory counts as "left".
   - The old behavior was a "save anything worth keeping" warning. The cancel report now gives no sign that a directory was deleted; only a `logger.Info` line does.
   - Evidence: `destroy.go:1419-1446`, `cancel.go:642`, `teardown.go:464-466`.
   - This is the 'precondition assumed, not enforced' item in the focus file. The record's Goal licenses deleting the leftover, so I am not raising it above LOW.
   - Fix: add the deleted path to the cancel report's output, and for a cancel with no run record, board entry or branch, check that the directory looks like build output rather than user files.

2. **LOW / consistency, `internal/fabricengine/destroy.go:1419`.** `RemoveStrayTaskPath` takes no teardown lock. `PATTERN-slug-remote-order` has `Add` and `Remove` hold the slug's teardown lock exclusive. A concurrent `Add` for the same slug could create a worktree at the path between the `listsLinkedWorktree` check and the delete. The window is narrow, because the cancel and teardown paths already hold their own locks. I did not read `Add` or the cancel's locking to confirm it is closed.
   - Fix: take the slug's teardown lock exclusive, as `Remove` does, or document why the caller's locks cover it.

**Tool calls**
1. Read `remove.go:495-580` for `inspectPair` and `strayPath`; my context only had the diff.
2. Read `remove.go:195-375` for the named-return and defer flow of `removePair`.
3. Read `teardown.go:405-480` for `RemovePair` and `clearStray` as they stand after the diff.
4. `grep` for `.Remove(`, `RemoveResult` and `ErrPairNotFound` in `fabriccli`, plus `sed` of `finishRemoval` in `wire.go`, for how callers read the new error.
5. `grep` for `.Any()` and `PairParts(` callers, to check the effect of counting `StrayPath`.
6. `sed` of `startguard.go:185-260` and `cancel.go:290-364`, to confirm how `leftPartNames` and `pairKept` handle a stray.
