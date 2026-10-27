```yaml
lens: fix-proposal
outcome: reported
```

My task was to challenge the reviewer's proposals for F1 and F2 that the fixer carried out in card 8, rather than to check that they were applied.

Proposals challenged: F1 (after-tick-1 Sleep row, `stopAt` 3 and its new assertions) and F2 (read first, then test `worktreeGone` once, plus the `state.json`-as-directory row). F3 is decision-class and out of my lens.

**F1: no breakage found.**
With the top-of-iteration check present, tick 1 ends with `.git` present. The first `Deps.Sleep` call then runs in `daemon.sleep`'s timer goroutine (`loop.go:345-348`): it removes the worktree and sends on `Watch`, and the loop's select receives that event. Iteration 2's top check exits, so `len(h.ticks) == 1` and nothing recreates `.lym/beam`.
With the check deleted, tick 2's `UpdateState` (`loop.go:275`) recreates the stub, and the two new assertions fail as the row claims.

**F2-a, LOW: the new `state.json`-as-directory row doesn't say whether its tick-1 hook sends a watch event.**
Changed text: card 8, row "a record holding a `Tidy` at start whose `state.json` tick 1's hook replaces with a directory … with `stopAt` 3 and the same `Deps.Sleep` wrapper removing the worktree on its first call".
Evidence:
- The harness line (card 8:50) says the default tick hook "removes the whole worktree on tick 1 and sends one watch event". The F1 sibling row overrides that explicitly ("tick 1's hook sends no watch event"); this row doesn't.
- If the hook does send an event, `daemon.sleep` takes it in its first `select` with no `Deps.Sleep` call (`loop.go:339-341`). The wrapper's "first call" is then the wake-floor `Sleep` on the loop goroutine itself (`loop.go:359`).
- From there, its send on `Watch` blocks unless the channel has a free buffer slot. `newLoopHarness` builds no `Watch` channel; each test makes its own (`loop_test.go:173-177`), and the card says only "a `Watch` channel". The row could then hang until the harness's one-minute timeout, instead of reaching the top-of-iteration exit the way the F1 row does.
- Fix: say in the row that tick 1's hook sends no watch event, as the F1 row does.

**F2-b, LOW: the rewritten kept-tidy paragraph's three branches leave out the decode-failure read.**
Changed text: card 8:36ff. It says the read treats "an undecodable record as absent with one Warn". The branches it then lists are "gone", "failed with an error other than a decode failure" and "the read succeeded".
Evidence:
- `ReadJSONNoMkdir` returns `ErrDecode` as an error (`internal/state/state.go:193-194`), so a decode failure is neither a "succeeded" read nor the excluded error class.
- The paragraph implies `keptTidy` takes nil (absent record, nil `Tidy`) but never says so. An implementer could equally keep the earlier value.
- Fix: in the last branch, say a decode failure counts as a read that succeeded with no record, so `keptTidy` takes nil.

Tool calls:
1. `grep` of `Deps.Sleep` call sites and `daemon.sleep` in `loop.go`: I needed to know which goroutine makes the first `Sleep` call and whether an already-pending event skips it, and my context had only `loop.go`'s `loop` function.
2. `grep` of `Watch` in `loop_test.go` and `loop.go`: I needed to know whether the harness builds a `Watch` channel and whether it is buffered, which my context lacked.
3. `sed` of `loop_test.go:168-182`: I needed to see how an existing test builds its events channel and how big its buffer is.
