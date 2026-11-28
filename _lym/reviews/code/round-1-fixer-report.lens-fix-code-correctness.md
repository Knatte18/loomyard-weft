```yaml
lens: fix-code-correctness
outcome: reported
```

Task: fix-code-correctness lens over the F2–F4 code changes.

no findings

What I traced:
- **`worktreeGone`:** a standing worktree, a missing path, a directory without `.git` and a regular file all give the documented answer. Any other `Lstat` error is returned.
- **`pairOf`:** a failure of `NamedWorktree` or `worktreeGone` gives `standing` false with the error, and every caller checks the error first.
- **`pairOf` callers on a leftover directory:** `RunPastFinalize` returns true, `WorkUnderOtherCommit` returns false and `UnpushedRecords` returns "". `HoldPair` and `WakePair` do nothing. These match the missing-path behaviour and the seam docs in `parent.go`.
- **New tests:** the `ThePathIsARegularFile` row fails without the ENOTDIR branch. The `unreadable` entry fails `TestPruneLedger` if the failure branch drops the entry or swallows the error. The gate log shows both ran and passed.

Tool calls:
- One `sed -n` read of `internal/runpusher/parent.go`. My context held only grep hits from it, and I needed to see how the `pairOf`-backed seams are used.
