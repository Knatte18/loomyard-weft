```yaml
lens: code-correctness
outcome: reported
```

Task: run the code-correctness lens over the diff, taking one concrete input per changed function or branch.

**Per-function list**

- `anchorSeams.runFail` and `anchorSeams.headSHA`:
  - Zero value: `fail` is nil, so `runFail` returns nil. `weftHead` is nil, so `headSHA` reads `gitrepo.New(weftPath).CurrentSHA()`. Correct, and the same as production before.
  - Injected `weftHead` returning an error: `moveWeftContent` returns `true, err` after the commit and keeps the record. Correct, same ordering as before.
- `moveWeftContent` with `anchorSeams{}` from `addanchor.go:92`: runs the production path. Correct.
- `BeginAnchorChange` copying `t.anchor` into `AnchorChange.seams`:
  - A hook set before `Begin` reaches `moveWeft` and `integrate`. Correct.
  - A hook set to nil before the re-run `Begin` is not carried into the re-run. Correct, matches the tests.
  - No other caller of the removed globals is left. Greps outside the four edited files found none.
- `integrate` and `moveWeft`: `c.seams.runFail(false)` and `runFail(true)` keep the old before/after order and the old rollback/no-rollback split. Correct.
- `Topology.gcAutoThreshold` and `housekeepStores(..., threshold)`:
  - `NewTopology` sets 6700, so production runs `gc.auto=6700` as before. Correct.
  - A `Topology` built any other way would have threshold 0 and pass `gc.auto=0`, which disables auto gc.
  - The grep for `Topology{` found only `topology.go:37` (`NewTopology`), so that case can't occur. Correct.
- `RecordPushFailure` calling `recordPushFailure(..., installedNotifier())`:
  - Nil installed notifier: `*n` is never dereferenced when `Load()` returns nil, so `installedNotifier()` returns nil and nothing is mailed. Correct.
  - The notifier is now read before the record lock instead of inside it. This is harmless, since a swap between the two moments has no observable effect.
- `SyncOptions.notifier()` with `Notifier` nil: falls back to the installed notifier. Production passes zero options, so behavior is unchanged. Correct.
- `recordPushFailure` branches:
  - `Notified` or `Deferred`: returns the record without mailing. Correct.
  - Nil `notify`: returns the record, unmailed. Correct.
  - Notifier error: logged, record left unnotified. Correct.
  - Not posted: record left unnotified. Correct.
- `pushclassified.go:283` using `j.opts.notifier()`: a push with a `Notifier` mails only through it. Production, with nil, mails through the installed one. Correct.
- `ReconcileOptions.callHook` and `verdictHook` with nil hooks: no-ops. Correct.
- `lsRemoteOnce` and `lsRemoteHeads`:
  - The deadline is now read from `opts.deadline()` at the same point as before.
  - Both ls-remote goroutines call the same `CallHook` concurrently. That is the same concurrency the old global hook had, and the tests' `callLog` already tolerated it. Correct.
- `fetchMovedHeads` with an empty `heads`: still returns before calling the hook. Correct.
- `judgeAgainstOrigin` calling `opts.verdictHook()` per branch: same order as before. Correct.
- Test helpers:
  - `SetAnchorFailHookForTest` and `SetWeftHeadSHAForTest` mutate only the one hub's `Topology`, so parallel subtests don't see each other's values.
  - `SetGCAutoThresholdForTest` is the same. It is set before `AddPair` and `Remove`, so the value is in place when `Remove` reads it.
- Split `TestStoreHousekeeping`: `n != 0` ("not packed") and `n == 0` ("unexpectedly packed") match the old `packed != wantPack` for `wantPack` true and false. The skip, key-kept and log assertions all moved unchanged into `TestStoreHousekeeping_Warnings`.

**Findings:** none. I found no wrong result, swallowed error or inconsistent failure state in the changed code.

**Tool calls:** none. My context already held everything the lens needed.
