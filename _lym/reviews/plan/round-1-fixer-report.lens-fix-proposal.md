```yaml
lens: fix-proposal
outcome: reported
```

Fix-proposal lens: I challenged each proposal the fixer carried out by building concrete cases where it fails. Three of them break.

1. **BLOCKING — card 5, the new `TestSeedChild_RunSettings` row (F8):** The row's fake now calls `configreg.PrepareRunSettings` inside the battenshed test. That creates an import cycle, so the test won't compile.
   - Evidence: the test file holding `TestSeedChild_RunSettings` declares `package battenshed`, an in-package test, and `internal/configreg/configreg.go:16` imports `internal/battenshed`.
   - A fix needs the test in an external `battenshed_test` package, or the error built some other way. Copying the text is not an option, since that is the authored text F8 rejects.

2. **MEDIUM — card 8, `RewriteRunSettings` snapshots and restores outside the write lock (F2):** `RewriteRunSettings` takes the snapshot before it calls `boardCriticalSection` and writes it back after that call returns. `boardCriticalSection` takes the board write lock and releases it on return.
   - Evidence: `internal/boardengine/board.go:103-107`, where the lock is taken and its release is deferred inside `boardCriticalSection`.
   - Case 1: another `lym board upsert` lands between the snapshot and the lock. A failing `beforeCommit` then restores the stale snapshot and erases that write.
   - Case 2: the restore runs after the lock is released, so a writer that took the lock in the meantime also has its write overwritten.
   - Case 3: "any later step returns an error" also covers the `*DecisionRecordError` that `boardCriticalSection` returns last (board.go:192-202). That error comes after the render and after `spawnSync`. A restore at that point rewrites `board.json` under a detached sync that may already be committing the moved board.

3. **MEDIUM — card 9, the `websterInFlightWarnings` bullet against the new line "Each site calls the walk at the verb's entry that first reaches one of those routes" (F1):** `websterInFlightWarnings` runs only after the merge.
   - Evidence: `internal/fabriccli/merge_verbs.go:146-149`, where `res, err := fabric().MergeIn(args[0])` comes first and `warnings = websterInFlightWarnings(loc())` follows.
   - So the walk runs after merge-in has changed the worktree, not at the verb's entry. On a conflicted merge the worktree is mid-merge, so the walk skips it and the stamp stays stale. On a clean merge the walk commits reconciled files on top of the fresh merge.
   - The card's "at the verb's entry" claim does not hold for this site. Either move the call into the merge verb's `RunE` before `MergeIn`, or state the after-merge placement and what it means.

No findings on F3, F4, F5, F6, F7, F9 or F10.

Tool calls:
- `grep` of `internal/fabriccli/merge_verbs.go` for `websterInFlightWarnings` call sites: my context held only lines 1-80 of that file, not the caller at line 149.
- `grep` of the battenshed test's `package` line and of configreg's import of battenshed: I needed both to judge the import-cycle risk of F8's proposal, and my context held neither.
