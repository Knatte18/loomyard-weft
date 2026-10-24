---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: internal/shuttleengine/claudeengine/activity_test.go:TestTurnEndUsageLimited
    summary: no row pins that the newest main-chain turn end decides, so an earlier limit entry followed by a plain turn end is unpinned
    fix: add a row to the TestTurnEndUsageLimited table with the transcript lines limitTurnEnd then agentTurnEnd, no fork, want false; it pins the decision record's bound that a limit an earlier turn already reported never silences a later notice.
    origin: lens:code-tests
  - id: F2
    severity: LOW
    class: scope
    location: internal/shuttleengine/claudeengine/activity_test.go:TestTurnEndUsageLimited
    summary: the sidechain exclusion in readTurnLimit has no row
    fix: add a row to the TestTurnEndUsageLimited table whose newest main-transcript line is a sidechain assistant entry with isApiErrorMessage true and error rate_limit, after agentTurnEnd, no fork, want false.
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: consistency
    location: internal/shuttleengine/wait.go:327
    summary: usageLimited runs a transcript read and logs "no hold notice" for held turn ends that would not have notified anyway
    fix: in the Wait switch at wait.go:325-334, keep the usageLimited case but have it apply only where a notice would otherwise be sent or queued (not for an interactive run, and not when HoldDeferral is set and turnStartRead is true); or reword the Info line so it does not claim a notice was suppressed. Confirm how notifyHeld filters interactive runs before choosing.
    origin: lens:code-correctness
---

### [LOW] No row pins "newest main-chain turn end decides"

**Location:** internal/shuttleengine/claudeengine/activity_test.go:TestTurnEndUsageLimited
**Issue:** Every main-chain row ends with the limit, server-error or plain turn end after only an ordinary earlier turn end. A scan that took any limit entry in the tail would pass every row, so the decision record's bound is not pinned.

### [LOW] Sidechain exclusion in readTurnLimit is untested

**Location:** internal/shuttleengine/claudeengine/activity_test.go:TestTurnEndUsageLimited
**Issue:** `readTurnLimit` skips sidechain entries, and no row would fail if that filter were dropped.

### [LOW] usageLimited is asked, and logged, for turn ends that would never notify

**Location:** internal/shuttleengine/wait.go:327
**Issue:** The `usageLimited` case sits before the `HoldDeferral` cases, so it reads the transcript and logs that no notice is sent for every held turn end that reaches the switch.
That includes runs and states in which no notice would be sent in any case.
The behaviour is harmless and matches the decision record's ordering, but the log line can mislead.

## Rejected

- Code-correctness #2, an unparsable previous timestamp yields a zero fork bound: the doc comments state this fallback, it needs a malformed timestamp Claude Code does not write, and the cost is at worst one skipped notice.
- Code-correctness #3, the newest main-chain turn end is read from the live transcript: this is what the decision record specifies, and the lens itself marked it a known limitation.
- Code-tests #3, no row covers the `raw` set at the `EventWaiting` site: the lens could not confirm that site reaches `usageLimited`, and the construction is a one-line field copy.
- Code-tests, the unparsable-timestamp row: it pins a fallback rejected above.
- Lens code-affected-set and code-pattern-fit reported nothing; I checked the deferred-hold interaction myself: any event read clears `pendingHold` (`wait.go:300-303`) before the switch runs, so an earlier queued hold cannot notify after a limited turn end.
