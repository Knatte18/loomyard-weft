```yaml
lens: fix-affected-set
outcome: reported
```

Lens: fix-affected-set. For each finding whose premise is a set, I re-ran its search and checked that the fix covers every member.

Unchecked: none. I checked all three findings: F1, F2 and F3.

no findings

- **F1 (who reads and who writes the transition tip):**
  - Card 1 and card 13 only consume `CodeTip` and `CodeTipMachine` through `ReadRun`. Neither reads the trailer directly.
  - The production writers of `statuscommit` are `loomcli` `wiring.go:207,329` and `battencli` `commitstatus.go:131,181`. The changed `Commit` closures in card 8 cover both, and through them `commitRunRecords`, `commitCancelRecord` and readopt's `CommitFailed`.
  - Card 12's give-up writes the trailer, and `lastTransitionCodeTip` is the one shared reader for cards 6, 7 and 12.
  - Three commits correctly stay without `Run-Transition`: tender's `commitRunIntent` (card 9), the board writes (card 10) and Finalize's commits (card 11). None of them is a transition.
- **F2 (unexported hooks a test sets from the external package):** `owedTipPushedHook` is the only hook any card names, and the new `SetOwedTipPushedHookForTest` covers it.
- **F3 (doc comments still describing `Spawn`):**
  - `loomcli` `wiring.go:77` is the field doc in `commitStatusDeps`, which card 8 already targets.
  - `wiring.go:194-198` is now a card 8 target.
  - `battencli` `commitstatus.go:55` (`commitStatusDeps`), `commitstatus.go:175` (`newCommitStatusSeam`) and the doc of `battenCommitStatusDeps` all belong to card 8 targets.
  - `readopt.go:31` `SpawnPush` is a different seam, which stays.

Tool calls:
1. Grepped cards 1–5, 9, 10, 11 and 13 for code-tip and trailer readers, and the `loomcli`/`battencli` sources for `Spawn` doc lines. My context held only cards 0, 6, 7, 8, 12 and 14, and none of those files.
2. Read the `newCommitStatusSeam` doc in `commitstatus.go` and grepped for `statuscommit.New` and `commitStatusDeps{` builders. I needed the symbol that owns line 175 and the full list of status-seam writers.
3. Grepped cards 9, 10 and 11 for `commit`, to find records commits outside the seams that might be transitions. Those cards were not in my context.
4. Grepped every card for `Hook`, to find the set of unexported hooks for F2. Unchanged cards were not in my context.
