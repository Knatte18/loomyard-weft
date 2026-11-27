---
verdict: BLOCKING
findings:
  - id: F37
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:142
    summary: "D3 and D7 assume every halt leaves a history entry, but a producer error (launch never ready, agent died, and the other error stops that leave state failed) appends none (appendHistory in internal/shedengine/run.go skips an empty outcome); D3 counts no hand-back or arbiter restep record, so such a stop counts zero unless lym retried it and the third halt at a row never turns severe, and D7 has no entry to record class, settle, halts_at_row and halt on, nor a key that changes between two such halts."
    fix: "In D3, state that a stop leaving no history entry counts once too: the driver gives each stepped stop an identity that stays the same across a driver restart, every settle record of it carries that identity (the hand-back and arbiter restep records included), and the count is the distinct stops at the row over the halt entries, the handed-on Awaiting entries and the identities of entry-less stops that D3 does not exclude. Narrow the nothing-count bullet so that a hand-back or arbiter restep record excludes only a stop already counted or excluded. In D7, say where the four facts live for a halt with no history entry (a stop record the driver writes through internal/state and the dobby's child read reads, never an earlier row's newest entry), and put the stop identity in the halt: key, so two successive failed halts at one row get different keys and D8's dedup, D7's posted record and D10's chase treat them as two halts. Add to D3's tier0 criteria an agent launch that fails three times at one row with no history entry (one fresh start, then a hand-back, a resume, a third failure) reaching settle: operator, and to D7's criterion a failed-state hand-back that records the four facts and a second one that gets a new key and a second post."
prior:
  - id: F26
    status: resolved
  - id: F27
    status: resolved
  - id: F28
    status: resolved
  - id: F29
    status: resolved
  - id: F30
    status: resolved
  - id: F31
    status: resolved
  - id: F32
    status: resolved
  - id: F33
    status: resolved
  - id: F34
    status: resolved
  - id: F35
    status: resolved
  - id: F36
    status: resolved
---

### [BLOCKING] A failed-state halt is neither counted nor recorded

**Location:** `_lym/discussion/decision-record.md:142` (D3), with the same premise in D7's first paragraph and D10's first stall.

**Issue:** A producer that returns an error returns no outcome, and `appendHistory` in `internal/shedengine/run.go` then appends nothing;
the run is persisted `failed` with its history unchanged.
An agent that never started or died before its verdict is such an error
(`judgeDiedError` in `internal/shedadapters/bouncer.go`, the chair's `ErrNotStarted` and `ErrAgentDied` returns in `internal/shedadapters/multillm.go`).
A resume writes no entry either.

The count therefore breaks for the launch-failure class D2 bounds at one fresh start:

1. The first failure is re-stepped by lym, and its settle record counts one.
2. The second failure is handed back. It has no halt entry, and D3 says a hand-back record counts nothing.
3. Every later failure after a resume is handed back the same way, so the count stays at one.

D2's "third halt at the row" is unreachable for it, against D3's first sentence ("each counted once"), D3's Bound,
and the declared `PATTERN-stop-settlement` text, under which `stopclass`'s table decides by a count that includes each stop.
A stop the tackler escalates and the arbiter resumes counts nothing at all.

D7 records `class:`, `settle:`, `halts_at_row:` and `halt:` "on the halt's history entry", which such a halt does not have.
The newest entry belongs to an earlier verdict, and its row, index and `repeats` do not change between two failed halts.
So lym cannot post a severe verdict or let D10 chase a `settle: orch` one,
and a key derived that way would mute the second distinct halt through D7's posted record and D8's dedup.

## Rejected

- D4 calls the bounce budget's row "the one row that matches a prefix", while `preconditionUnmet` in `internal/sheddriver/driverules.go` matches one too.
  The new rows' rule is sound, and the slip is below the bar for a new finding in an incremental round.
- Two distinct keyless relays of one slug within `slack.remind_after_min` mention the operator once.
  It is below the same bar.
- A `budget` escalation over an unread previous ledger can carry a cause detail when the harden `OnHalt` reap fails, so D6's "no cause detail" test misses it.
  It needs two independent faults in one campaign round and is below the same bar.
