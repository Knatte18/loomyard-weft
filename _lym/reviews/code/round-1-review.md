---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/dobbyengine/runnotice_test.go:120
    summary: no test row pins that paused intervals are left out of the driver-start grace
    fix: In TestRunNotice_Conditions add a row with f.alive false, f.pause set to pauserecord.Coverage{Intervals} holding a paused interval (Covered false) that spans the time past driverStartGrace, and no expected notice. The row must fail when pause.Intervals is dropped from the AwakeSince call at internal/dobbyengine/runnotice.go:251.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: _lym/landing/summary.md (Design decisions) against internal/dobbyengine/runnotice.go:191
    summary: the summary says the grace applies whenever a child enters running, but memory.Since only restarts when dobby saw a non-running state
    fix: In the summary reword the claim so the grace applies when dobby sees the child enter running, noting a halt and resume between two probe ticks keeps the old since and gets no grace. The code stays as it is.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/battenshed/doc.go:119
    summary: the doc says a go child with neither lock and no strand is noticed as driver-dead, omitting the bootstrap lock and the grace
    fix: In internal/battenshed/doc.go:119 name the loom bootstrap lock among the locks that count as a live driver, and say the notice waits a minute of awake time from the running episode's since.
  - id: F4
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/dobbyengine/doc.go:136
    summary: the takeover bound omits that a machine dating the child's since from its own first sight also restarts the start grace
    fix: In internal/dobbyengine/doc.go:136 add that a driver-dead notice on such a takeover also waits the start grace from that first sight.
---

### [MEDIUM] Paused intervals in the grace are untested

**Location:** internal/dobbyengine/runnotice_test.go:120
**Issue:** `noticeCondition` passes `pause.Intervals` to `AwakeSince` for the grace, and `doc.go` and the summary both claim paused time does not count. No row sets `pause.Intervals`; `ACoveredChildSendsNothing` tests only `Covered`. Dropping the argument leaves every test green.

### [LOW] Summary overstates when the grace applies

**Location:** _lym/landing/summary.md (Design decisions)
**Issue:** The summary says the grace applies whenever a child enters running, such as a resume after a halt. `runNotice` restarts `memory.Since` only when the recorded child state differs, so a halt and resume that falls between two probe ticks keeps the old since and the grace has already passed. The bootstrap lock still covers most of that window.

### [LOW] Stale driver-dead wording in battenshed doc

**Location:** internal/battenshed/doc.go:119
**Issue:** The sentence lists the driver and run locks and the strand as the liveness signals for the driver-dead notice. The notice now also counts the loom bootstrap lock and waits the start grace.

### [NIT] Takeover bound omits the grace

**Location:** internal/dobbyengine/doc.go:136
**Issue:** The bound says a taking-over machine dates since from its own first sight so a driver-dead notice can go once more. The grace counts from that same since, so the notice also waits up to a minute there.

## Rejected

- Probe contention with `lym loom start` on the bootstrap lock (code-correctness): the probe takes the lock for an instant and `start` blocks on a blocking acquire, so it delays at most microseconds; same pattern as the driver and run lock probes.
- `battenshed/doc.go:161` stale wording (code-affected-set): the sentence stays roughly right because the grace excludes paused intervals.
- Grace boundary `<` against `<=` not pinned (code-tests): the clock moves past the grace on each tick, so the boundary is not a behaviour worth a row.
- New `loomengine` import in `battencli` (focus): other `battencli` files already import it, and `battencli` is not a leaf package.
- Bootstrap-lock row duplicating a `TestChildDriverLockHeld` row (focus): the existing rows hold other lock paths, so it is not a duplicate.
- Scope (focus): the entry offers the bootstrap lock and a grace as the smallest fixes, so taking both is licensed; no change outside the notice path rides along.
- Grace placement and empty return (focus): the grace sits after the non-running, pause and liveness-read returns and gates only `!alive`; `memory.Since` is never zero there; the empty finding and `activityStamp{}` equal the old dead-driver return, so marker and memory handling is unchanged.
