```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's F1 and F2 proposals as carried out in card 8, by building a case where either one goes wrong.

no findings

- F1 (a `stateRead` episode of its own): I tried to build a case where the episode flaps or double-counts and found none.
  `loop.go` runs `d.failures.Recover` right after `tick` and before the inserted read (`internal/dobbyengine/loop.go:221-225`), so only a separate episode keeps the read's episode intact.
  The marker-gone branch exits before any log, so `stateRead` has no exit-path interaction.
  The decode branch logs its own Warn, as `ReadState` does (`internal/dobbyengine/state.go:87-88`), and ending `stateRead` there with `Recover` is correct, because the read did not fail with a non-decode error.
- F2 (the undecodable-record row): the row distinguishes the decode branch from a wrong implementation.
  - An implementation that treats `ErrDecode` like a read error keeps the start `Tidy` and records a `StopStrand`, so the row's "no `StopStrand`" fails it.
  - An implementation that skips the read leaves `keptTidy` as the start `Tidy`, which the row also catches.
  - `.git` stays present through tick 1, so the post-tick check does not exit.
  - The Sleep wrapper's removal is caught by the top-of-iteration check, which the card keeps (line 33), before tick 2's `UpdateState` could rewrite the record.
  - The hook writes after `tick`'s own `UpdateState`, since the `Tick` seam is the tick's last step, so the bytes survive to the post-tick read.

Tool calls:
- Read `ReadState`'s decode handling in `internal/dobbyengine/state.go` and `loop` in `internal/dobbyengine/loop.go:200-240`, to check where the post-tick read sits relative to `Recover` and `CheckBinary` and whether "one Warn as `ReadState` does" holds. My context held only `loop.go:221-225` and `240-256`.
