---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/battenshed/notice.go:467
    summary: no test row covers a usage-limit run winning over an api-error or quiet run, or the usage-limit key standing while another run of the child is not limited
    fix: In internal/battenshed/notice_test.go add rows to TestNotice_UsageLimitNotifiesOncePerLimitWithoutARevive, or a sibling table test, with two runs. One run is usage-limited and the other has a plain API error, and the test asserts exactly one usage-limit notice and no api-error notice. Then one run stays limited while the other reads clean across two calls, and the test asserts no second notice.
  - id: F2
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/shuttleengine/activity.go:162
    summary: the readRunActivity copy of UsageLimit, and its ownTurnEnd gating, is asserted by no test
    fix: In internal/shuttleengine/activity_test.go add a row beside the existing API-error rows (around lines 158 to 191). One turn end is the run's own, and its TurnEndActivity carries UsageLimit{Limited true, ResetsAt set}, which the AgentActivity must carry too. A second row is a turn end before the prompt offset carrying a limit, and the AgentActivity.UsageLimit must stay zero.
  - id: F3
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/orchengine/state.go:463
    summary: ResetForFreshLaunch clears the three usage-limit fields, and TestResetForFreshLaunch does not assert it
    fix: In internal/orchengine/state_test.go TestResetForFreshLaunch, set UsageLimitAt, UsageLimitResetsAt and UsageLimitWakeAt in the input of the idle or non-idle subtest and assert all three are zero in the result.
  - id: F4
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/orchengine/usagelimit.go:55
    summary: the failure paths of tickUsageLimit have no test, namely a wake line that fails to render, a creel inject that fails and leaves the wake due, and a hold that is refused
    fix: In internal/orchengine/watcher_usagelimit_test.go add a case with a missing or multi-line stencil, asserting UsageLimitWakeAt moves 30 minutes on and nothing is sent. Add a case with a failing fake creel inject, asserting the wake stays due and the next tick sends it.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/orchengine/usagelimit.go:33
    summary: a limit first read after its reset time has passed waits 30 minutes, although D5 sends the wake line one minute after the reset
    fix: In recordUsageLimit, when limit.ResetsAt is not zero and ResetsAt plus usageLimitWakeDelay is not after now, set UsageLimitWakeAt to now so the next tick wakes the orch at once; keep the 30-minute retry only for a zero reset time. Add a table row to the watcher usage-limit test for a reset already past.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/claudeengine/usagelimit.go:54
    summary: a month and day that is not a real date, such as Feb 31, is normalised by time.Date to another date instead of reading as no reset time as D2 says
    fix: After building reset in the month-and-day branch, return the zero time when reset.Day() differs from day or reset.Month() differs from month.Month(), and add a Feb 31 row to TestUsageLimitResetsAt.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/orchengine/usagelimit.go:64
    summary: the wake inject passes ttl 0, which creel turns into its default, while every other watcher inject passes stepInjectTTL
    fix: Pass stepInjectTTL as the ttl in the creel.Inject call in tickUsageLimit.
---

### [MEDIUM] Batten usage-limit precedence and any-run retention untested

**Location:** internal/battenshed/notice.go:467
**Issue:** The decision record's D3 puts the usage-limit notice ahead of the api-error and quiet checks, and keeps its key while any run of the child still reads limited.
The one test drives a single limited run, so a regression in either the precedence or the any-run loop passes.

### [MEDIUM] Shuttle activity copy of UsageLimit untested

**Location:** internal/shuttleengine/activity.go:162
**Issue:** The existing rows in `activity_test.go` set only the API-error fields, so dropping `UsageLimit` from the copy, or copying it for a turn end that is not the run's own, fails no test.
The run's own turn end is the only one whose limit may reach batten.

### [MEDIUM] Fresh launch clearing of the limit record untested

**Location:** internal/orchengine/state.go:463
**Issue:** `ResetForFreshLaunch` clears the new fields, and the summary and `doc.go` both claim it.
`TestResetForFreshLaunch` sets and asserts none of them.

### [LOW] tickUsageLimit failure paths untested

**Location:** internal/orchengine/usagelimit.go:55
**Issue:** The doc comment promises that a render failure defers the wake and a creel failure leaves it due.
The one watcher test covers only the happy path.

### [LOW] Past reset time waits a full retry interval

**Location:** internal/orchengine/usagelimit.go:33
**Issue:** `wake.After(now)` is false for a reset that is already past, so the wake falls to `now + 30 minutes`.
A watcher that read the limit late, for instance after a restart, leaves the orch unwoken for half an hour although the limit has lifted.

### [LOW] Impossible month and day reads as a date

**Location:** internal/shuttleengine/claudeengine/usagelimit.go:54
**Issue:** `time.Date` normalises `Feb 31` to Mar 3 without an error.
D2 reads an unreadable clause as no reset time.

### [LOW] Wake inject uses ttl 0 instead of the watcher's constant

**Location:** internal/orchengine/usagelimit.go:64
**Issue:** Creel maps 0 to its ten-minute default, so the behaviour equals `stepInjectTTL`, but the call reads as no expiry and departs from the other injects.

## Rejected

- Runner.UsageLimit delegate and the wire.go field mapping untested (lens:code-tests, LOW): thin delegates over covered readings, and the notice test and the watcher fake pin their consumers.
- No minute above 59 row in `TestUsageLimitResetsAt` (lens:code-tests, LOW): the guard shares a branch with the hour-range row that is covered.
- Old state record reading with no limit untested (lens:code-tests, LOW): the new fields are plain optional JSON, and the tolerant-records rule is pinned generally by its own test.
- No second limited turn end in the watcher test (lens:code-tests, LOW): the re-read path is a two-line assignment already exercised by the wake moves.
- The board entry's second Expected bullet, api-error dedup for plain errors: not in the decision record's scope, and a finding about the answer key is not the diff's to fix.
