```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each review proposal the fixer carried out as proposed. That is F1 only, since F2 followed the parent's decision.

**MEDIUM: the new `TestJournal_Retention` row asks for an assertion that test cannot make.**
- **Changed text:** `_lym/plan/07-chase-clock.md`, the Tests paragraph. The new row reopens the fold and asserts "that a post to `CREEL` is still refused as an unknown receiver".
- **Why it fails:** `TestJournal_Retention` works on a bare `pool` (`newPool`, `openJournal`), and its `check` asserts on `p.programs` directly. It has no `Store`, no strand list and no delivery seam, so no post goes through it. The refusal lives in `Store.checkReceiverLocked`, which reads `s.state.programs`.
- **Evidence:** `internal/creelengine/journal_test.go:412-420` builds the test from `newPool` and `openJournal`. Its check is `if !p.programs["ly:BATTEN"] ...` on the writer's pool and on the refolded journal (around `journal_test.go:497`). `internal/creelengine/pool.go:199-203` is the refusal path.
- **Fix:** state the row as a check that the refolded pool's `programs` has no `<shortname>:CREEL` entry, which is the condition `checkReceiverLocked` refuses on. The alternative is to move the post-refusal assertion into a Store-level test such as `TestScheduler_Chases` with `env.reopen`.

One gap is outside my lens. Per card 7, `carryRecords` snapshots a chaseable message's `ReadAt`, `ConsumedSeconds`, `Chase` and `MarkSeconds`, but not `Chased` or `ChaseOf` on a carried unread chase. A chase carried past retention may therefore lose `ChaseOf`, which card 8's reply to a chase depends on. The new row carries such a chase without asserting it.

Tool calls:
1. Read `internal/creelengine/journal_test.go:412-470` and `carryRecords`/`prune` in `fold.go`, to see what the retention test can assert and what a carry keeps; my context had only the test's line number.
2. Grepped `journal_test.go:470-540` for Store or post use in the retention test, to confirm it asserts only on `pool` fields; my context lacked the rest of the test.
