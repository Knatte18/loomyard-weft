---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/shuttleengine/startup_test.go:1932
    origin: lens:code-correctness
    summary: the failure branch bounds a failing start's virtual time only from below, so a deadline the hold moved too far still passes the new deadline row
    fix: In the `if tt.wantFailed` branch of TestStartup_LoadTurnAPIError, after the `elapsed < tt.wantFailedAfter` check, add a check that runs only when `tt.wantFailedAfter > 0`, failing when `elapsed >= tt.wantFailedAfter + apiErrorRetryBackoff` with the message "virtual elapsed = %s; want less than %s". The never-ends row fails at about 1h and the new row at 1h31s, so both sit under their limits; the rows whose `wantFailedAfter` is zero, such as the dying pane and the failed send, must stay unbounded because their start already waited out a back-off.
  - id: F2
    severity: LOW
    class: consistency
    location: _lym/landing/summary.md:23
    origin: handler
    summary: the summary says the shared bound pins the doubling, the ceiling and the busy session's back-off exactly, but the bound is a window of one apiErrorRetryBackoff above wantHeld
    fix: In the "Back-off from above" bullet of the change description, replace "pins the doubling, the ceiling and the busy session's next back-off exactly" with a wording that says each is pinned to within one `apiErrorRetryBackoff` above its scheduled wait. A ceiling up to about 24 seconds past loadHoldBackoffCeiling still passes, so the text must not claim more.
---

### [LOW] The failing rows pin the deadline only from below

**Location:** internal/shuttleengine/startup_test.go:1932
**Issue:** The new row "an open turn after a hold fails the start at the deadline the hold moved" sets `wantFailedAfter: time.Hour + apiErrorRetryBackoff`.
The failure branch checks `elapsed < tt.wantFailedAfter` and nothing above.
`heldDeadline` and `endLoadHold` in `loadturn.go:64-71` and `258-265` could move the deadline by twice the hold, or by a hold that includes the open turn's wait, and the row would still pass.
The diff added the upper bound for the succeeding rows and left this one out, although the commit message says the row fails the start only at the moved deadline.
Traced by hand: the run fails at 1h + 31s on the fake clock, so a limit of `wantFailedAfter + apiErrorRetryBackoff` holds.

### [LOW] The change description claims an exact pin the bound does not give

**Location:** _lym/landing/summary.md:23
**Issue:** The summary says the shared upper bound "pins the doubling, the ceiling and the busy session's next back-off exactly".
The bound is `elapsed < wantHeld + apiErrorRetryBackoff`.
A first back-off of 59 seconds passes the single-error rows, and a ceiling of 5m29s passes the ceiling row.
The bound catches a tripled back-off and a large overshoot, as the commit message says, but it does not pin the schedule exactly.

## Rejected

- Lens code-tests, LOW at `startup_test.go:1949`: the deadline move in succeeding rows is checked only from below. That assertion is not in the diff, and the diff's change is the `elapsed` bound; no decision-record check asks for the move's upper side. Not raised.
- Lens code-tests, NIT at `startup_test.go:1732`: the `wantHeld` comment says "zero for a start that fails" while a succeeding no-hold row also has zero. The interrupted-turn row had the same zero before this diff, and the reworded comment states the least-time meaning correctly. Not raised.
- Lens code-tests, LOW on the 30-second slack gaps (a first back-off up to 59s, a ceiling up to 5m29s): this is the intended design of one shared bound, and the part the summary overstates is filed as F2.
- Handler, considered and dropped: the new row "an open turn past its timeout that then ends loads its skills" duplicates "a turn past its timeout that then ends on an API error skips nothing". The existing row ends the open turn on an API error and the new one ends it normally. The decision record's second check names the plain open turn past the timeout, and the new row is the one that fails when a normal end past the timeout skips skills through `wantSkips`, so it is not a redundant test.
- Handler, checked and clean: the summary's "no production code changes" claim holds (`git diff --stat` lists only `startup_test.go`); every other hold behavior the record lists (busy session, turn start during the back-off, dying pane, failed send, notice once and none when interactive, notifier failure, model rejected, open turn at the deadline, interrupted turn) has a row in the table; the ceiling row's `wantHeld` of 30s*(1+2+4+8) + 2*300s equals the schedule `apiErrorRead` produces; `wantHeldAtMost` has no remaining reference; no PATTERN entry is declared or touched and the commit carries no `Pattern-Departure:` trailer.
