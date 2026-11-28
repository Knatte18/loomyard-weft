```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the review's proposals that the fixer carried out as proposed (F1, F3, and F2's ENOTDIR read).

no findings

I tried these cases and none breaks:

- **F3, leftover directory in the parent seams:** `RunPastFinalize` now answers true, `UnpushedRecords` answers "", and `HoldPair` and `WakePair` do nothing. This matches what a missing path already did, and the seam's contract ("or its pair is gone from this machine"). `heldByRefOrLanding` also treats a leftover code directory as no checkout.
  - Pair removal takes the code worktree before the records worktree, so a records worktree can outlive the code one. The missing-path case already behaved this way, so I don't count it against the fix.
- **F2, ENOTDIR read as gone:** a file at the path, or a file where a parent directory should be, holds no worktree. No caller of `worktreeGone` needs an error there.
- **F1, one-way door and wider blast radius:** the deletion text agrees with `PruneLedger`, which keeps only entries that hold landings.

Further tool calls (2 of 6):
- Grep of `internal/runpusher/parent.go` and `internal/fabricengine/landing.go` / `pusherreads.go` for the parent-seam consumers; my context lacked how `RunPastFinalize`, `UnpushedRecords` and `HoldPair` are used.
- `landing.go:262-320` and a grep of `internal/fabricengine/remove.go`; my context lacked how fabric reads a leftover pair and the order in which removal takes the worktrees.
