---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shedengine/step_test.go:57
    summary: StartedAt is pinned only on the done path; the stuck, reroute and reroute-capped entries and the fold keeping StartedAt have no test row
    fix: In internal/shedengine, add StartedAt assertions (assertRFC3339UTC and StartedAt <= At) to the existing step tests that produce a stuck entry, a reroute entry and a reroute-capped entry, and add one appendOrFold row asserting the folded entry keeps the first call's StartedAt.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/shedverbs/status.go:124
    summary: idle time between repeated calls folded into one budget-exempt stuck entry is not reported, though a lasting stuck loop is the Goal's own case
    fix: Either make the fold keep the per-call gaps the idle report needs, or state in the IdleGap comment and internal/shedverbs/doc.go that gaps inside a folded entry are not reported.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/startup_test.go:1882
    summary: Result.APIError on a capped hold, and a capped hold returning the run when every output file exists, have no test
    fix: Add a capped counterpart of TestRun_LoadTurnModelRejectedEndsTheStart that asserts Result.APIError is the newest error text, and a variant with every output file present that asserts the run is returned instead of failing the start.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/startup_test.go:1925
    summary: the parent notice text that names the cap is never asserted; the test counts notices only
    fix: In TestStartup_LoadTurnAPIError, compare the captured notices entry on the capped row against the cap count (for example a wantNoticeContains field holding the "At 8 API errors" wording built from loadHoldMaxAPIErrors).
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shedverbs/status_test.go:554
    summary: no test checks a non-empty idle_gaps in the status envelope or its JSON field names
    fix: Seed one history over the five-minute floor in a status command test and assert idle_gaps[0] carries after, after_outcome, before, from, to and idle with the expected values.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/shuttleengine/engine.go:34
    summary: the OutcomeAPIError doc and docs/overview.md:650 still list only the re-send limit and a rejected model, omitting a start whose skill-load hold reached the cap
    fix: Add the capped skill-load hold to the OutcomeAPIError comment in engine.go and to the api-error description at docs/overview.md:650-652.
---

### [MEDIUM] StartedAt is pinned only on the done path

**Location:** internal/shedengine/step_test.go:57
**Issue:** `rerouteStuck` takes a new `startedAt` parameter on three entry paths and `appendOrFold` keeps the folded entry's `StartedAt`, yet only the done entry is asserted. The idle gap after a stuck step, which the Goal asks to surface, depends on these values, and dropping them leaves every test green.

### [LOW] Idle time inside a folded stuck entry is not reported

**Location:** internal/shedverbs/status.go:124
**Issue:** A fold keeps the first call's `StartedAt` and `At` and moves only `LastAt`. Hours between repeated budget-exempt stuck calls vanish into one entry, and `idleGapsOf` measures only the gap into and out of it. The Goal's stuck-step scenario can fall in this case.

### [LOW] Capped hold's Result.APIError and all-outputs-exist branch are untested

**Location:** internal/shuttleengine/startup_test.go:1882
**Issue:** The new row checks the start error, outcome, typed loads and deadline extension. It never reads `Result.APIError`, set in the `capped` branch of `abandonLoad`, and nothing runs the capped error into the all-outputs-exist return.

### [LOW] Cap-naming parent notice text is unasserted

**Location:** internal/shuttleengine/startup_test.go:1925
**Issue:** The Goal asks that the cap be named. The notice sentence that names it is runtime output, and the test compares only the notice count.

### [LOW] Non-empty idle_gaps envelope is unasserted

**Location:** internal/shedverbs/status_test.go:554
**Issue:** The envelope test sees only an empty list, so a rename of a JSON tag on `IdleGap` passes every test.

### [LOW] OutcomeAPIError doc and overview omit the capped hold

**Location:** internal/shuttleengine/engine.go:34
**Issue:** The diff updated `internal/shuttleengine/doc.go` for the new api-error path but left the `OutcomeAPIError` comment and the api-error description in `docs/overview.md` listing the older paths only.

## Rejected

- Possible double count of one API error through both the StopFailure signal and a Stop-event activity read (code-correctness, unverified): the Claude engine maps StopFailure to the signal and builds events from Stop lines only, so one turn end reaches one path.
- The tail gap after the last entry is not reported (code-correctness, NIT): the Goal asks for time between a stuck step and the next step, which a tail has none of.
- Missing skipped, reroute and out-of-order rows in TestIdleGapsOf (code-tests): the skipped arm shares the goto condition and the other cases are covered, so no separate finding.
- Halt note carrying the cap text (focus item): the start error names the cap and the shed layer surfaces a start error as the failed run's error, which meets the Goal's api-outage reading; nothing in the diff contradicts it.
