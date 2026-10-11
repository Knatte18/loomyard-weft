---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/03-never-ready-pane-tail.md
    summary: The named no-tail rows for the skill-load death and a cause-bearing exit never reach the Started or cause condition, because skillReed serves an empty startup capture, so only the empty-capture condition is ever exercised
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/04-startup-probe-cadence.md
    summary: TestStartup_GateDismissalGap is a new top-level test for the gate-never-clears teardown that TestStartup_UndismissableGateUntilWindowExpires already drives, whose no-key case is the empty-sequence row of the same table
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/01-claude-lone-eol-mark.md
    summary: TestStart_LoneEOLMarkBoxTakesStartupSends serves the lone-% box only until the first send, so the skill-load send's idle reading never sees the % box the acceptance criterion names
  - id: F4
    severity: NIT
    class: consistency
    location: _lyx/plan/04-startup-probe-cadence.md
    summary: Card 4 rewrites TestStartup_ReadyProbeTiming's rows and doc comment but leaves its testtiming:keep reason, which still claims exactly two probe intervals
---

### [MEDIUM] Card 3's no-tail rows do not reach the conditions they name

**Location:** `_lyx/plan/03-never-ready-pane-tail.md`, the Tests list; `internal/shuttleengine/fakes_test.go:561` (`skillReed.CapturePane`).

**Issue:** Card 3 adds the tail exactly when three conditions hold: `cause` nil, `run.state.Started` false, `run.lastStartupCapture` non-empty.
It names one test row per exclusion, but two of those rows never reach the condition they stand for.

- `TestStartup_SkillLoadTurns`'s "pane that dies during a load turn is a died startup" row runs on `skillReed`, whose `CapturePane` returns only the texts sent so far.
  At the startup probe nothing has been sent, so `checkLivenessTick` records `""` as `lastStartupCapture`.
  The row's "no `The pane's last lines:`" assertion therefore holds through the empty-capture condition, and an implementation that dropped the `Started` check would still pass it.
  D4 and its acceptance line ("a pane that dies during the skill-load turn gets no tail") depend on exactly that check.
- `TestStartup_SendFailuresTearDown`'s "a skill-load send that stays busy" row also runs on `skillReed` (empty startup capture), and `Started` is true by the time any send runs.
  Its "exactly once" assertion cannot detect a second tail whatever the cause check does.
  The only exits where the cause check decides anything are the reed-status retry exits (cause present, never ready).
  The decision record's addendum names those exits explicitly ("the reed-status retry exits carry a cause and get no tail").
  No named row covers them with a prior capture: `TestStartup_MechanismFailure`'s rows fail or miss the strand before any capture is taken.

The rubric's reachability rule applies: a prescribed branch whose named test does not reach it.

**Fix:** Give the skill-load-death row a non-empty startup capture, for example a `skillReed` field holding a fixed screen line that `CapturePane` serves ahead of the sent texts, set only on that row.
The row's no-tail assertion then turns on `Started`.
Add a row to `TestStartup_MechanismFailure` whose reed first answers one live status with a non-empty pending capture and then fails `maxStatusRetries` times.
Assert that its error carries the cause and does not contain `The pane's last lines:`.
Both stay in `startup_test.go`, which card 3 already targets, and add no top-level test.

### [LOW] TestStartup_GateDismissalGap duplicates the gate-never-clears scaffold of an existing test

**Location:** `_lyx/plan/04-startup-probe-cadence.md`, the `TestStartup_GateDismissalGap` bullet; `internal/shuttleengine/startup_test.go:302` (`TestStartup_UndismissableGateUntilWindowExpires`).

**Issue:** The throttle itself is new behavior, so this is not a pure duplicate.
But all three planned rows share one scenario: a gate that never clears, a start that fails wrapping `ErrNotStarted` with a died record, and assertions on the `SendKey` calls.
`TestStartup_UndismissableGateUntilWindowExpires` already drives that scenario for an empty dismiss sequence.
That empty sequence is also one of the throttle's prescribed branches ("an empty sequence ... sets neither").
The decision record's `PATTERN-test-economy` constraint says each new scenario extends an existing table or test in its package where one covers the surface.

**Fix:** Turn `TestStartup_UndismissableGateUntilWindowExpires` into the table-driven gate-never-clears test.
Its current case becomes the empty-sequence row, and card 4's three rows join it with their play-time assertions.
Drop `TestStartup_GateDismissalGap` from card 4's `**Create:**` list.
Card 4 already targets `startup_test.go`.

### [LOW] Card 1's end-to-end test shows the lone `%` to the first send only

**Location:** `_lyx/plan/01-claude-lone-eol-mark.md`, the `start_test.go` paragraph.

**Issue:** The fake reed "serves `pane-idle-eol-mark.txt` until a send arrives, then whatever the test's paste hook needs for the send to land".
Read that way, only the first send (the color command) has its idle reading and box text checked against the `%` box.
The skill-load send's guard reads whatever the hook serves after the color send landed.
The acceptance criterion says the start "passes the color send and the skill-load send" on that pane.

**Fix:** State that the capture goes back to `pane-idle-eol-mark.txt` once each send has landed.
Each send's pre-write guard, the skill-load send's included, then reads the lone-`%` box.

### [NIT] Stale `//testtiming:keep` reason on TestStartup_ReadyProbeTiming

**Location:** `_lyx/plan/04-startup-probe-cadence.md`, the `TestStartup_ReadyProbeTiming` bullet; `internal/shuttleengine/startup_test.go:162`.

**Issue:** Card 4 replaces the "probe cadence matches Wait" row and has the doc comment and the file header drop the matches-Wait claim.
The `//testtiming:keep` line above the test still reads "exactly two probe intervals otherwise", which the new rows no longer pin.

**Fix:** Have card 4 also reword the keep reason to what the new rows pin: no sleep when ready on the first probe, the base interval while the pane redraws, and the backoff to the dismissal gap while it does not.

## Focus verdicts

- Card 1, the new top-level test: no existing `claudeengine` test drives a start through Claude's own box readings.
  `corpus_test.go` wires a `shuttlefake.Engine` that does not use Claude's `IdleSession` or `InputBoxText`.
  The acceptance criterion names this end-to-end check, and the budget arithmetic is right: 103 tests excluding `TestMain`, raised to 104.
  Not a defect; see F3 for the capture design.
- Card 4, the `TestStartup_ReadyProbeTiming` rows: the replacement rows pin the new cadence, and their numbers check out.
  The redraw row gives 3 status calls and 2 s.
  The backoff row sleeps 1+2+4+8+10+10 = 35 s over 7 status calls.
  The new top-level test is F2.
  `internal/shuttleengine` holds 160 tests against a budget row of 161, so one new test fits.
- Card 4 member glyphs: `playInputs` keeps its signature and delegates, and `startupTickCap` keeps `(startupTimeout, interval time.Duration) int` with a different argument at its call site.
  `Run.awaitStartup` and `Run.checkLivenessTick` keep theirs.
  No re-sign arrow is missing.
- Cards 1–3 glyphs: each `Edit` lists member glyphs matching its `Intent`, plus file paths only for test files, `doc.go` and non-Go files.
  No card targets only non-test Go file paths while naming symbols.
- Card 4 against the decision record: no card amends `PATTERN-daemon-wakeups`.
  The base, doubling while the capture does not change, the dismissal-gap ceiling and the reset on change are all carried.
  The `Wait` remark restates D6's own rationale.
  The Debug log on a held dismissal is part of D6's held-probe branch and adds no behavior.
  Both are licensed.
- Card 4 attack surface and reachability: the `Intent` states that the throttle only removes key presses and that the startup deadline and tick cap still end a gate that never clears.
  The held-dismissal branch is reached by the never-clears row (probes at 1–9 s and 31 s are held).
  The first-key-failed branch is reached by the `SendKeyErr` row.
- Card 3 reachability: see F1.
  The all-captures-failing row does reach the empty-capture exclusion: `lastStartupCapture` stays empty when every capture fails.
- Card 5 and `## verify:`: `./...` covers `claudeengine`, `shuttleengine`, `shedadapters` and `cmd/lyx`.
  The integration tier runs, and the order matches the plan stencil.
- Card order: card 3 needs nothing from card 2's code, and card 4's edits to `startup_test.go` and `doc.go` build on cards 2 and 3 without reaching forward.
  No forward dependency.
