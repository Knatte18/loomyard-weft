# Card 2 — selfexec-replaced

**Create:**
- `plan:internal/selfexec#Watcher.Replaced` -> `func (w *Watcher) Replaced() bool`

**Edit:**
- `internal/selfexec#Watcher.Check`
- `internal/selfexec#TestWatcher_Check`

**Intent:** Give the binary-identity check a detecting form that never execs, for creel's daemon (D2), which exits on a replaced binary instead of re-execing.
`Replaced` re-stats the recorded path exactly as `Check` does and returns true once a changed identity is seen on two consecutive calls, the same debounce `Check` applies; it never calls the exec seam and logs the detection once at Info.
A Watcher with no start identity, a failed stat and an unchanged identity answer false.
`Check` and `Replaced` share one unexported step that updates the previous-identity state and reports a changed identity seen twice, so the debounce is written once; `Check`'s behavior and log lines stay as they are.
`TestWatcher_Check` gains a `replaced` mode field: its rows run through `Replaced` instead of `Check` and assert the answers per call and zero exec calls, covering an unchanged identity, a change seen once, a change seen twice (true on the second), a flip back, a failed stat between two changed reads, and a watcher whose start stat failed; through the test's existing `logcapture`, the row reading a change seen twice asserts one Info line naming the path, and the others assert none.

**ImpactSummary:** Check keeps its behavior through a shared helper; Replaced is additive, used by card 14's daemon verb; the leaf import set is unchanged.
