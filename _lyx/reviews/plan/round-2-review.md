---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/11-orch-resume-context-engine.md:33
    summary: A hook-delivered pointer skip confirms the step through typeReloadStep, whose first typing moves PhaseEventsOffset to the cursor, so the phase waits for a turn end after the skip rather than the first turn end after the boundary that D11 and the card itself promise
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/11-orch-resume-context-engine.md:29
    summary: ResumeContext and the verb plan no refusal for an absent orch state, while LoadState reads an absent state as phase idle, so the verb would render the reload prompt and write a mark where the acceptance criteria require a JSON error and no mark
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/06-watchdog-discovery-backoff.md:37
    summary: The discovery loop's event branch (stat, consume, ignore an event that finds no file) has no tier-1 test over a fed channel and no test at all for the no-file event, contrary to the record's test-speed constraint, its acceptance criterion and the overview's opener-seam decision
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/plan/08-readiness-reading.md:42
    summary: internal/shuttleengine/doc.go keeps stating that session state runs in shadow mode with no consumer acting on it and lists the signal kinds without session start; card 7 does not edit that doc and card 8 limits its doc edit to the verified-send section
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/11-orch-resume-context-engine.md:35
    summary: The pointer-skip Bound states only the double-delivery cost and omits what the skip lets through, a mark written for a render whose output the provider drops, which D11's Bound names
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/plan/06-watchdog-discovery-backoff.md:28
    summary: The discover-signal wake is a new routing edge into the daemon, but card 6's Intent states no Bound naming who can trigger it and its worst-case cost of one list-sessions per base cadence
  - id: F7
    severity: LOW
    class: scope
    location: _lyx/plan/02-poll-floors.md:38
    summary: Shuttle's LoadConfig gains the one Warn for a sub-second poll_interval_ms, but no planned test asserts it, while the acceptance criteria ask each declared loop's package to prove the floor with one Warn
  - id: F8
    severity: NIT
    class: consistency
    location: _lyx/plan/14-idle-cpu-benchmark.md:9
    summary: The before figure is taken from the production lyx described as built from main, but production is built at prod, which can lag main; D6 asks for the before figure on main
---

### [MEDIUM] A hook-delivered skip does not end the phase at the first turn end after the boundary

**Location:** `_lyx/plan/11-orch-resume-context-engine.md:33`, with the draft-case row at line 38.

**Issue:**
The card puts the skip inside `typeReloadStep` and has it "confirm the step as typed, so the phase ends at the first turn end after the boundary".
`typeReloadStep` (`internal/orchengine/watcher.go:787`) sets `ReloadTypedAt`, `PhaseEventsOffset = w.cursor` and resets `w.seen` on the step's first typing.
`tickResuming` then ends the pointer step only on a turn end read past that offset.
A turn end that came after the boundary but before the pointer step is not counted.
In the draft case that is the operator's own turn: the color and plugins steps wait for the draft to clear, so the operator's turn has already ended when the pointer step skips.
The phase then stays in `resuming` until the operator's next message.
If no message comes, `toIdle(st, "resume timed out")` records an abort and zeroes `LastContextTokens`, even though the hook delivered the pointer.
The card's own test row ("ending the phase at the next turn end") pins this behaviour.
D11 says "The phase ends at the first turn end after the boundary as today, whether the operator's message or the typed pointer started that turn."

**Fix:**
Make the skip end the phase on a turn end at or after `State.CompactionBaseline` rather than one after the skip's cursor.
For example, the skip could end the phase at once through `toIdle` when such a turn end has already been read since the boundary, and otherwise keep the offset at the boundary's events position.
Repin the draft-case row to that rule, and add a row where the hook delivered and no turn follows, asserting the phase does not end in `resume timed out`.

### [MEDIUM] No refusal for an absent orch state

**Location:** `_lyx/plan/11-orch-resume-context-engine.md:29`; `_lyx/plan/12-orch-resume-context-verb.md:28`.

**Issue:**
The acceptance criteria say `lyx orch resume-context` "outside the prime, with no orch state, or on a render failure, ... prints a JSON error and writes no mark".
`orchengine.LoadState` (`internal/orchengine/state.go:115`) returns `State{Phase: PhaseIdle}` when the state file is absent.
`ResumeContext` as planned picks its pointer by phase, so an absent state would render the reload prompt, write a mark and print context.
Card 12's tests cover outside-the-prime and a missing stencil, but not an absent state.

**Fix:**
Have `ResumeContext` (or the verb) read the state's presence, for example through `state.ReadJSON`'s `found`, and return an error with no mark when it is absent.
Add the row to `resumecontext_test.go` or `cli_test.go`.

### [MEDIUM] The discovery loop's event branch has no tier-1 test, and its no-file case no test at all

**Location:** `_lyx/plan/06-watchdog-discovery-backoff.md:37`; the overview's `opener-seam-exported-from-reedengine` decision.

**Issue:**
The record's `PATTERN-test-speed` constraint says the watchdog's event path is tested with a fake event channel in tier 1 and once live in the `tmux` tier.
Its acceptance criteria include "an event that finds no signal file runs no cycle".
The overview's shared decision says tier-1 tests pass an opener over a fed channel, "so a `reedcli` test that builds that channel imports `fswatch`".
Card 6 plans only a table test over the pure `discoveryWake`.
`onSignal` is reached only after a file is found, so that table cannot show the stat, the consume, or that an event finding no file runs nothing.
Both loop-level tests (`watchdog_integration_test.go`, `watchdogreap_integration_test.go`) are `//go:build tmux`, which the plan's verify compiles and never runs.
Their planned step covers only the found-file case.
Card 4 does give the resize loop this coverage, so the gap is specific to `reedcli`.

**Fix:**
Add a row to `TestWatchdogDaemon`, or to an existing `reedcli` tier-1 test where the loop can be driven, asserting that an event with no signal file on disk runs no cycle.
If `runWatchdogLoop`'s direct `reedengine.ListSessions` call keeps the loop out of tier 1, say so in the card.
Then reword the overview decision so it does not promise a `reedcli` tier-1 channel test.

### [MEDIUM] `internal/shuttleengine/doc.go` is left stale

**Location:** `_lyx/plan/08-readiness-reading.md:42`; card 7's target list.

**Issue:**
The package doc (`internal/shuttleengine/doc.go:241-243`) says "Session state, in shadow mode: no consumer acts on it".
It then lists the signals as "a turn start, a turn end ..., an API-error turn end, an ask, an idle notice and a session end".
After card 8, `Runner.SessionIdle` and the verified send's `busyReading` act on the state.
After card 7, there is a session-start signal and `RunSessionState.SessionStartAt`.
Card 7 does not list `internal/shuttleengine/doc.go`, and card 8's Intent limits its edit to "the verified-send section".
The record's scope puts `shuttleengine`'s `doc.go` in the same commit where its behaviour changes, and CLAUDE.md requires that.

**Fix:**
Add `internal/shuttleengine/doc.go` to card 7, adding the session-start signal and its no-move rule to the signal paragraph.
Widen card 8's doc sentence to replace the shadow-mode statement with the readiness reading's consumers.

### [LOW] The pointer-skip Bound omits what the skip lets through

**Location:** `_lyx/plan/11-orch-resume-context-engine.md:35`.

**Issue:**
The skip lets a file under `.lyx/orch/` suppress the typed pointer.
The card's Bound states only the cost of not skipping (the pointer arrives twice).
It omits the case D11's Bound names: a mark written for a render whose output the provider drops skips a pointer the session never got.

**Fix:**
Extend the Bound with that case, and with what bounds it: the mark is written only after a successful render, must be dated at or after the boundary, and must equal `PendingResume`.

### [LOW] No stated Bound for the discover-signal wake

**Location:** `_lyx/plan/06-watchdog-discovery-backoff.md:28`.

**Issue:**
The discover signal is a new routing edge into the daemon, and any process of the operator can touch the hub scratch dir.
Card 6 describes the planner's base-cadence rule, but its Intent has no Bound stating what the edge lets through and what bounds it.
D5's Bound states it: a toucher in a loop costs one `list-sessions` per 5 s per hub, today's cadence.

**Fix:**
Add a `Bound:` sentence to card 6 carrying D5's statement.

### [LOW] Shuttle's floor Warn is untested

**Location:** `_lyx/plan/02-poll-floors.md:37-38`.

**Issue:**
Card 2 has shuttle's `LoadConfig` log one Warn for a sub-second `poll_interval_ms`.
It asserts that Warn for orch (`TestLoadConfig_PresentFileOverrides`) and for batten (`innerrun_test.go`), but the shuttle tests it names assert only the floor value.
The acceptance criteria ask each declared loop's package test to prove "floored with one Warn".

**Fix:**
Add a sub-second row with a `logcapture` Warn assertion to an existing `shuttleengine/config_test.go` load test.
Do not add a new top-level test, since the package is at its budget.

### [NIT] The before binary is described as built from `main`

**Location:** `_lyx/plan/14-idle-cpu-benchmark.md:9`.

**Issue:**
`update-plugins.sh` builds the production `lyx` at `prod`, which moves only when the operator runs it, so it can lag `main`.
D6 asks for the before figure "taken on `main` before the change".

**Fix:**
Either build the before binary from `main`'s tip into a scratch directory, or keep the production binary and stop describing it as built from `main`.
The report already names each binary's commit.

## Focus directive verdicts

- Removed `reedengine` fields and the re-signed `startWatchLoop`: card 4 lists `watchloop_test.go` and the `tmux`-tagged `watchdog_integration_test.go`, the only test files referencing `SignalTick`, `watchdogSignalTick`, `tickerPeriodFor` or `startWatchLoop`.
  Card 5 lists the `tmux`-tagged `naming_integration_test.go` for `repairNames`'s new result, and card 6 lists both `tmux`-tagged `reedcli` loop tests for `runWatchdogLoop`'s new parameter.
  Not a defect.
- Test budget: every card that adds a top-level test lists `cmd/lyx/testdata/test-budget.yaml` (cards 1, 2, 4, 5, 6, 7, 9, 10, 11, 12, 13).
  Cards 3 and 8 add rows only, to packages at their cap (`shedrun` 22/22, `shuttleengine` 160/160).
  Not a defect.
- Guard downgrades and escape hatches: card 8 (the needle and box downgrade), card 7 (`Spec.ContextAfterCompaction`) and card 13 (`//lyx:one-shot`) each state what they let through and what bounds it.
  The two gaps found elsewhere are F5 and F6.
- Wall-clock in `watchNames` and the cadence tests: card 5 drives `watchNames` through an injected `After` that fires at once, and card 6's `discoveryWake` takes `now` as a parameter.
  Not a defect.
- Card 14's procedure runs on a scratch hub cloned with `lyx fabric clone --into`, starts every daemon with `--hub-path` set to that hub, and ends only processes found by the scratch path.
  It touches no process of the operator's hub.
  Not a defect; F8 is a wording point.
