```yaml
lens: fix-affected-set
outcome: reported
```

**Fix-affected-set check: no findings.** The searches below found no member of the F1, F7 or F9 sets that the fix missed.

- **F9, every writer of a loop-log mark:** the signal mark (`loop.go:454`), the exit mark (`loopwait.go:181`) and the start mark (`loopwait.go:153`) all go through the newline guard. The only unguarded append, `appendLoopLog(loop.LogPath, stdout.Bytes())` at `loop.go:562`, writes child output and not a mark, so it should stay unguarded.
- **F1, every reader of the recorded child's stop files:** `recordedStopFiles` has two callers, `loopwait.go:513` (gated by `inFlight`) and `loopwait.go:582` (gated by `rec.inFlightAt(after)`). The remaining `StopFiles` calls belong to the loop's own live child, not to a dead loop's record:
  - `loop.go:300`: the stop's own `res.traceID`.
  - `loop.go:528`: the stderr path `runChild` captures into.
  - `loop.go:652`: the interrupted trace file of the live child.
- **F7, every caller of `recordTerminatingSignals`:** the only non-test caller is `loop.go:186`, which uses both returned functions. The two checks at the post-stepping point and just before the envelope write cover every `runLoop` exit that follows the stepping.

I did not run the F5 and F11 searches (single-file test rows) or the F2, F6 and F8 searches, since none of those findings has a set premise.

Tool calls: both were `grep` over non-test Go files, one for the mark writers and one for the stop-file readers and signal callers. My context held only the files I had read earlier, not a repository-wide search.
