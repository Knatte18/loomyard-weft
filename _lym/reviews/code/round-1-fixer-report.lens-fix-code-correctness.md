```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the code the fix changed (`removeEntry`, `metBelow`, the `validateWrite` removal and the added tests) for wrong results, broken callers and tests that pass with the defect in place.

**Findings**

1. **MEDIUM** — `Store.RemoveTask` / `removeEntry` (`internal/boardengine/store.go`), through `Board.RemoveTask` and `lym board remove`.
   - The strip now edits the `depends_on` of every dependent, including a task a run holds.
   - Run-lock judging forgives a changed `depends_on` only for slugs of done entries: `removedDoneSlugs` and `scopeChanged(before, now, removedDone)` (`runlock.go:164-220`).
   - Removing an open entry that a run-held task depends on now reads as a scope change on the held task, so the run lock should refuse it.
   - Before the fix the remove left a dangling slug and passed the lock.
   - Evidence: `runlock.go:175` refuses `scopeChanged` for kept held entries, and `removedDone` is built only from done entries.
   - No test covers a remove of an open entry that a run-held task depends on.
   - I did not read `checkRunLocks`' callers, so whether `boardCriticalSection` runs it on this path is unconfirmed.

2. **LOW** — Bolt replay (`replay.go:757-780`, `replayRun.removeEntry`) is the twin of `RemoveTask` and still refuses a removal while any dependent holds the slug ("dangling dependency: … depends on the entry").
   - The live board now strips instead.
   - The fix changed one side only, so a remove that succeeds on the board is reported refused on replay.

**Checked, no finding**
- `MergeTasks` keeps dependents of the upserted slug: `removeEntry(slug, slug != slugStr)` passes `false` for `gone` and `true` for `stays`. The merge row of `TestRemoveTaskStripsTheSlugFromDependsOn` expects `[gone]` for both dependents and would fail if the strip were unconditional.
- The first subtest of that test fails with the defect in place, because the dependents would keep `gone`.
- `metBelow`: a note returns false before the done check, and a missing slug still returns true, so `Prune` still releases a pruned done note's dependents. The `Pending` row in `TestKindDependencyRules` would fail with the old order, where a done note counted as met.
- The cycle row reaches `findCycle` through `waitsOn`, so it holds.
- I judged the removal of the `validateWrite` note blocks and of `CheckDependsNote` complete; `go vet` and tier0 passed.

**Further tool calls (1)**
- One `Bash` call reading `board.go:275-300`, the `removedDone` grep over `internal/boardengine/*.go`, and `replay.go:750-780`. My context did not hold the run-lock code, the `Board.RemoveTask` wrapper or the replay remove path.
