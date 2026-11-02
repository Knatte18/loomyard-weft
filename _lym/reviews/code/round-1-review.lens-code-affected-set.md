```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the diff forces outside the lines it touches.

**Per change, the set I found**

1. **New `startStallTimer` package variable in `gitexec.go`:** covered.
   - The only reader is `runAttempt` (`gitexec.go:301`). The only writer is `SetStallCheckForTest` (`export_test.go:22`), which restores the previous value in `t.Cleanup` at line 50.
   - A grep of every `.go` file found no other user of `startStallTimer`, `SetStallCheckForTest`, `SetStallBoundForTest` or `stallBound`.
   - The default `time.AfterFunc(bound, check).Stop` keeps the old behaviour. `defer stopStallTimer()` stands where `defer stallTimer.Stop()` stood. The `stallBound < deadline` guard is unchanged, so the unarmed row still arms nothing.
   - The seam's doc comment is self-contained, and its claim that `tracePath` is unused holds.
   - `PATTERN-test-speed` ("Time is injectable") says a timeout a test would otherwise wait out is never a package-level `var`. This var follows the file's existing `remoteDeadline`, `stallBound`, `runCommand` and `sleepBeforeRetry` precedent. `TestRun_RemoteBounds` is serial, and its doc comment names the process-global state. I raise no finding.
2. **`SetStallCheckForTest` in `export_test.go`:** covered. The header comment of `export_test.go` was updated, and the helper's doc comment states that the test must not be parallel. The only caller is `TestRun_RemoteBounds`, and all three stall rows call it. The retry attempt is covered too: the silent row's `ready` uses `attempt`, which the helper counts across the test and the row's count file accumulates across attempts.
3. **Behaviour and doc statements about the stall check:** covered.
   - The `runAttempt` doc comment (`gitexec.go:271-272`) and the `RunWithin` doc comment ("a deadline at or under the stall bound leaves the stall check unarmed") still hold, because the default path is unchanged.
   - A grep of `.md` and `.yaml` files outside `_lym` and `.lym` found no mention of `stallBound`, "stall bound", "stall check" or `startStallTimer`. No doc, stencil or fixture needs an update.
4. **Test and fixture side effects:**
   - The silent row sets `SetRemoteDeadlineForTest(t, 10*deadline)` inside the subtest, and its cleanup restores the outer `deadline` before the next row. The later rows depend on `deadline`, so the restore is correct.
   - The trace-leftover check on the silent row is unchanged and still valid.
5. **Repository rules and PATTERN entries:**
   - The rubric's "Declared PATTERN changes landed" does not apply: the directive declares no entries, and the diff touches no `PATTERN.md` or `pattern/` file.
   - The diff makes no change that touches `PATTERN-gitrepo-client-boundary`, `PATTERN-spawn-observability` or `PATTERN-test-isolation`. It adds no spawn, no raw git call and no kit import.
   - No `Pattern-Departure:` trailer is present or needed. The commit message's claims match the diff.
   - The diff changes no stencil.

**Findings:** none.

**Tool calls**
1. `grep -rniE "stallBound|stall bound|stall check|startStallTimer"` over `.md` and `.yaml` files outside `_lym` and `.lym`, to check for docs that describe the changed behaviour. My context only had a `.go` grep. It returned nothing.
