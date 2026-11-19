```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals the fixer carried out as written (F1, F2, F3, F4, F5, F6, F7, F8, F9, F11).

**Finding 1: LOW — F4: the same missing re-run mark is left in the other live-record paths** (`internal/gatefixer/rowgate.go`)
- Changed text: `spawning`'s live-record branch now wraps `inner` in `w.rerun.Run(inner)`.
- The sibling paths that judge over a finished fixer's tree still pass the bare closure:
  - `waitOnly` (`return w.waitThenRun(store, live, inner, markOnPass)`).
  - The fixer entry's `Final`, which is `w.waitOnly(finalInner, true)` where `finalInner` is the turn-decorated `original.Gate`.
- Case: a driver restarts with a live fixer, and the entry's `Final` closure then runs. `WithReachTurn` sees no re-run mark, so a failing result counts one arrival.
- Nothing in the change set pins the `Final` path. The new row, `TheJudgingRunAfterALiveFixerRunsUnderTheEntrysMark`, covers only `entry.Gate`.
- The reviewer's one-line proposal fixes the named branch but leaves the same defect class in `Final`.

**Finding 2: LOW — F6: the cap turns a moderately large two-site edit into a whole-file mark** (`internal/testreach/changedlines.go`)
- Changed text: `maxLineDiffCells = 4_000_000`, with the fallback marking every `headMiddle` line.
- Prefix and suffix trimming only removes lines outside the outermost edits.
- Case: a file of about 2100 lines with one edit near the top and one near the bottom has a middle of about 2100 × 2100 = 4.4M cells. That is over the cap, so every line, unchanged ones included, is reported as changed.
- Consequence: `lym gate mutate` plans mutants on lines the diff never touched, so survivors get blamed on the diff.
- The fallback is correct for the memory bound. It silently trades precision for a size that ordinary large files reach, and the doc comment does not say how large that size is.
- A cheaper fallback would run the diff per hunk between common anchor lines, or at least say in the doc where the cliff is.

No findings on F1, F2, F3, F5, F7, F8, F9 and F11:
- F1: the union write keeps old ids and takes the new hash for a changed one. The listed file grows without pruning, which is harmless.
- F2: the test fails if the closure returned HEAD as the base, since it would then print "reaches none" plus the reach command.
- F11: `state.WriteJSON` replaces the index atomically, as the `state.go` doc states, so the lock-free read is sound.

Tool calls: none; the context already held every file read, so I made no further calls.
