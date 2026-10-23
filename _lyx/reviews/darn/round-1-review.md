---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/orchengine/watcher.go:1142
    summary: a failed MailReadAt lookup going through creelFailed and holding the timeout has no test
    fix: In internal/orchengine, add a row or subtest next to TestWatcher_HandoffTimeoutCountsFromTheRequestRead that makes the fake creel fail "mailreadat" (the fake's fail hook already supports it), advances the fake clock past the handoff timeout, and asserts the phase is still handoff-requested with State.Stuck set and StuckByCreel true; then clears the failure and asserts the phase proceeds to count from the read.
    origin: lens:code-tests
  - id: F2
    severity: LOW
    class: design
    location: internal/orchengine/watcher.go:1097
    summary: the acceptance criterion that a hub pause after the read is still discounted has no handoff-phase test
    fix: Add a case to TestCycleRequest_AbandonedWithinTimeoutOfRequest (or a new row in TestWatcher_HandoffTimeoutCountsFromTheRequestRead) where a hub pause spans part of the window after the read, and assert the phase times out only once the unpaused time since the read reaches the handoff timeout, so a regression to a plain now.Sub(readAt) fails.
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: scope
    location: internal/orchengine/state_test.go:310
    summary: the clearing of RequestReadAt in ResetForFreshLaunch, enter and the cycle start is asserted nowhere; only toIdle is
    fix: In the existing ResetForFreshLaunch test at internal/orchengine/state_test.go:310-318, set a non-zero RequestReadAt in the input and assert it is zero in the result; extend an existing cycle-start test the same way for the tickIdle cycle start. Add no new test function.
    origin: lens:code-affected-set
  - id: F4
    severity: LOW
    class: consistency
    location: internal/orchengine/doc.go:190
    summary: a handoff-requested phase with no RequestID yet has no timeout, but the doc says only that an unread request never times out
    fix: In internal/orchengine/doc.go:190 widen the sentence to say the phase does not time out while its note request has no creel read, a request not yet posted included; leave the code in watcher.go:1139 as is, since the decision record's D2 accepts the unbounded wait.
    origin: lens:code-correctness
---

### [MEDIUM] Failed read lookup is untested

**Location:** internal/orchengine/watcher.go:1142
**Issue:** The decision record's D3 and the change description claim a failed `MailReadAt` is recorded through `creelFailed` and holds the timeout. The fake creel can inject the failure, but no test in the diff does, so a regression that times the phase out on a failed lookup would pass every test.

### [LOW] Pause discount after the read is untested

**Location:** internal/orchengine/watcher.go:1097
**Issue:** The record's acceptance criteria name the pause discount after the read. The new tests never open a pause after the read, so replacing `unpausedSince` with a plain subtraction would pass.

### [LOW] Clearing of RequestReadAt is pinned at one of four sites

**Location:** internal/orchengine/state_test.go:310
**Issue:** Only the return to idle is asserted. A dropped reset at `enter`, the cycle start or `ResetForFreshLaunch` would carry a stale read into the next cycle and time it out at once.

### [LOW] Doc understates the no-timeout case

**Location:** internal/orchengine/doc.go:190
**Issue:** `requestReadAt` returns zero for `RequestID == 0` too, so a phase whose post has no confirmed id also never times out, where the old rule timed it out from the phase entry.
The behavior matches the record's accepted risk; only the doc sentence is narrower than the code.

## Rejected

- Test for the `client.Log` error path in `creelSeam.MailReadAt` (lens:code-tests, NIT): the method returns the error unchanged, and the watcher-side handling is covered by F1.
- Focus question on `CycleRequestedAt` fallbacks: no path falls back to the post time or `CycleRequestedAt`; `tickHandoff` reads only `RequestReadAt`, and the idle stale-marker removal at `watcher.go:952` still ages a pending request from its request time as intended.
- Pattern-fit lens: no rule broken, no finding.
