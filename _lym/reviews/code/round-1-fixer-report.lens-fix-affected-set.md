```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Covered sets:
- **F1, status re-reads in `shedengine`.** I searched every `state.ReadJSON[Status]` site outside tests.
  - `run.go:211` is the initial read, resolved at `run.go:240`.
  - `run.go:289` is the post-gate re-read, and the fix re-resolves it at `run.go:297`.
  - `entry.go:72` re-reads only after `persistRouted` has written the non-empty `def.OnDone`. It sits inside `enter`, which receives `st` already resolved.
  - `run.go:827` (`routedState`) reads only `PauseAfter`, not `CurrentProducer`.
  - `goto.go`, `failedstop.go` and `rename.go` are outside `stepLocked`. The review rejected the `goto` and resume case, and its premise names the `stepLocked` gated-first-row path only.
- **F2, `RowAt` callers.** `RowAt` has callers at `run.go:240`, `run.go:297` and `progress.go:88`. The new `TestRowAt` table covers the named, empty-with-producers and empty-without-producers inputs.

Calls made:
1. `grep` for `ReadJSON[Status]` and `RowAt(` in non-test `shedengine` files. My context had no full list of the re-read sites.
2. Read `run.go:815-845`, because my context did not show what the `run.go:827` read is used for.
