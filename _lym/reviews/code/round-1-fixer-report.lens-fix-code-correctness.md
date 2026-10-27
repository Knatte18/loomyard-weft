```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fixer's change set (F1–F4) for code correctness.

**Findings**

1. **LOW**: `heartbeat` / `logRunChanges` in `internal/dobbyengine/passes.go`: `primeOwed` is not cleared when a heartbeat primes the memory itself.
   - Input: the start heartbeat's `Runs()` read fails, so `primeOwed` is true and `loggedRuns` is nil.
   - Tick 1's `drivePasses` read fails too, so `logRunChanges` never runs.
   - A heartbeat ten minutes later reads the runs successfully, sees `loggedRuns == nil`, and primes it. `primeOwed` stays true.
   - The next tick's `logRunChanges` then primes silently again and returns. A run whose label changed between the heartbeat and that tick gets no change line.
   - This needs two consecutive run-listing failures, so it is rare, and the effect is one lost log line.
   - Fix: set `d.primeOwed = false` in the `if d.loggedRuns == nil` branch of `heartbeat`.

**Checked, no finding**
- **Failed start read, then a good tick:** `primeOwed` is true, `logRunChanges` primes silently and clears it, and later ticks log changes normally. This is the F1 case and it holds.
- **Fresh start with a good read:** `heartbeat` primes, `primeOwed` stays false, and `logRunChanges` behaves as before.
- **Direct-tick tests:** `TestLog_ChangesOnly` and similar call `tick` on a zero daemon. They never set `primeOwed`, so first-seen lines still log. The gate run was green.
- **New test row:** without `primeOwed`, tick 1's `logRunChanges` would log `run a: running`, so the row would fail with the defect in place. The `Fatalf` on the unreadable-heartbeat count guards the assumption that read 1 is the start heartbeat's.
- **F3 and F4:** comment-only edits.
- **F2:** comment and doc text only. It matches the reviewer's account of the drop-then-reseed order, and the `SeedAdoptGate` signature and behaviour are unchanged.

**Tool calls:** none. My context already held every changed file.
