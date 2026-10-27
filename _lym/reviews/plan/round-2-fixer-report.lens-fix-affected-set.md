```yaml
lens: fix-affected-set
outcome: reported
```

Task: for each set-shaped finding (F1, F2), re-run its search and check that the fix covers every member.

Unchecked findings: none. F3 is decision-class and wasn't part of this fix.

**F1, MEDIUM.** Changed text: card 8's `TestRun_WorktreeGoneExits`, where `stopAt` 3 was added only to the two `Deps.Sleep` rows. The intro sentence and the rows that remove the worktree inside a tick still name no `stopAt`.

The premise is that the tick seam's cancel can mask an exit. It covers every row whose removal tick equals `stopAt`, not only the Sleep row. In `loop_test.go:96-98` the seam calls `h.cancel()` when `len(h.ticks) >= h.stopAt`, and `loop.go:227` then returns nil before the post-tick `worktreeGone` test and before `CheckBinary`.

The worst member is the row "a worktree removed during tick 1 with a `CheckBinary` fake … `CheckBinary` is never called". With `stopAt` 1, which is common in the existing harness callers (`loop_test.go:118`, `:237`), that row passes even with the post-tick exit deleted, because the cancel alone keeps `CheckBinary` from being called. The rows that remove during tick 2 (the `Tidy`-gained and `Tidy`-cleared rows) need `stopAt` ≥ 3 for the same reason.

Fix: put "with `stopAt` 3" in the harness sentence that opens the test, for every beam row. The prime row's "goes on ticking to `stopAt`" stays as it is.

**F2: no findings.** The rows that remove the worktree inside a tick are covered by the generic sentence "The rows that remove the worktree inside a tick reach this branch with a failed read". That sentence holds: the `Tick` seam is the tick's last step (`loop.go`, `tick`), so nothing recreates `.lym/beam` after the hook's removal.

Tool calls:
1. Grepped the `newLoopHarness` callers and read `loop_test.go:86-112` for the seam's cancel. My context had the harness's fields but not the cancel logic or the `stopAt` values callers use.
