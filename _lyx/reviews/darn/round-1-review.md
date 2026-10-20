---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: _lyx/landing/summary.md:9
    summary: summary says the constant bounds every expected-success wait, but the pid-gone waits at lines 170, 234 and 282 keep their own fixed or budget-derived bounds
    fix: In _lyx/landing/summary.md, narrow the Changes bullet to say reapSuccessWait bounds the session-gone waits and the fresh-session wait, and add a Design decisions bullet saying the waitProcessGone waits at watchdogreap_integration_test.go lines 170 and 234 (20 s) and goneWithin at line 282 (ReapTimeout plus reapForceKillGrace plus 5 s) keep their bounds because they start after the session is gone and the reap spawns nothing on the way to killing the pids they wait on.
---

### [LOW] Summary claims more than the diff does

**Location:** `_lyx/landing/summary.md:9`, against `internal/reedcli/watchdogreap_integration_test.go:170`, `:234`, `:282`
**Issue:** The summary says the new constant bounds every wait for an outcome a step expects to succeed.
Three process-exit waits that also expect success are left on their own bounds: two fixed 20 s waits and `goneWithin`, which derives from `ReapTimeout` and `reapForceKillGrace`.
The summary's list of bounds left short covers only the early-reap check and the cancel deadline, so these three are unexplained.
They are sound as they stand.
They run after the session is gone, and `reapPaneChildren` reaches `proc.KillPID` for the waited-on pids with no process spawn on the path (`internal/reedengine/lifecycle.go:1150-1187`).
No wait in the file derives from `DiscoveryCycle`, which meets the decision record's acceptance criterion.
The defect is the claim, not the waits.

## Rejected

- Raise `goneWithin` and the 20 s `waitProcessGone` waits to `reapSuccessWait` as a MEDIUM `scope` finding (code-affected-set): the waits start after `kill-session` returns, the closure is computed before the kill, and the force-kill is an in-process signal, so a spawn stall does not stretch them; folded into F1 as a claim problem only.
- `goneWithin` as a LOW `design` finding (code-correctness): same reasoning; its budget mirrors the production reap budget the step proves.
- `readPIDFile`'s fixed 10 s wait (code-affected-set): outside the diff, a setup wait with no reap on its path, and not an expected-success reap outcome.
- The fresh-session wait no longer proving loop discovery (code-correctness): `AddPair` creates the session itself, so the old `DiscoveryCycle*10` window never measured the loop either; the next `select` still checks the loop is running.
- `cancelDeadline` and `dispatchStart` (focus point 3): the assertion is an in-process select after `cancel()` and measures the same thing; only the elapsed figure in the failure message grows, and it stays accurate.
- The `reapSuccessWait` comment (focus point 5): it starts with the symbol name, states what the constant bounds and why, names no other symbol, and its line breaks are the lint's to check.
