```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set check, rerunning the searches behind the set-premised findings to see that the fix covers every member.

Findings left unchecked: none with a set premise. F1, F2, F3 and F4 were searched. F5 concerns one bullet, not a set.

- **LOW**, `_lym/landing/summary.md` `## Tests`: "`TestTargetOf_...` drives the real reads over both removals" is stale. The F2 fix added a third `removals` row, `ThePathIsARegularFile`, and that test ranges over `removals`. The fix updated the neighbouring `TestWorktreeGone` bullet but not this one.
- **LOW**, `_lym/landing/summary.md` `## What changes`, PruneLedger bullet: it still names only "a worktree left without its `.git` entry" and "a missing path". The first bullet now also lists a file at the path, which PruneLedger newly drops too, where before it kept the entry.
- **LOW**, `internal/runpusher/seams_test.go` `removals`: the unchanged comment "removals change a pair's worktree as a removal leaves it" no longer fits the added `ThePathIsARegularFile` row, since no removal leaves a regular file. The `TestTargetOf_...` doc, "covers a run whose pair a removal took away", has the same gap.

No other misses:
- **F1:** every non-test caller of `TargetOf`, `TargetRefused`, `Start`, `StartIfAhead` and `PruneLedger` is in beamcli, battencli or loomcli, which the new blast radius names.
- **F3:** no directory-only presence read is left in `internal/runpusher`; every pair presence read goes through `worktreeGone`.
- **F4:** the other "gone" and "stands" docs in the package, including `parent.go:51` and `parent.go:57`, agree with the new meaning.

Tool calls (nothing written or committed):
1. `git grep` for runpusher callers: my context had only an `internal`-scoped, partly noisy search, and F1 needed the repository-wide set.
2. `git grep` for presence reads in runpusher, pushledger and `loomcli/pusher.go`: F3's set after the change was not in context.
3. `git grep` for gone-worktree descriptions, which failed on a bad pathspec: F4 and F2 doc siblings were not in context.
4. The same search with a corrected pathspec: same reason.
