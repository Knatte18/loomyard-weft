---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/04-reed-resize-events.md
    summary: Card 4 drops watchTiming.SignalTick but omits internal/reedengine/watchdog_integration_test.go, whose fastWatchTiming sets t.SignalTick, so go vet -tags tmux stops compiling
  - id: F2
    severity: BLOCKING
    class: scope
    location: _lyx/plan/10-shell-changedir.md
    summary: Card 10 adds TestShell_ChangeDir to internal/shell, already at its budget row of 11, without listing cmd/lyx/testdata/test-budget.yaml; card 11 adds resumecontext_test.go to internal/orchengine, at 126 of 126, without listing it either
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/07-session-start-signal.md
    summary: Card 7 adds the SessionStart hook and its payload parse but leaves internal/shuttleengine/claudeengine/doc.go, which enumerates the recording hooks and states that no recording hook prints to standard output, unedited and false
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/plan/08-readiness-reading.md
    summary: Card 8 downgrades the pane probe so a ready reading passes beside a running-turn needle or a missing box, and its Intent states what passes but not what bounds it (D7's landed check and watcher-only text)
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/08-readiness-reading.md
    summary: Card 8 plans a new readiness_test.go table over the private judgeReadiness whose draft, unmatched-turn-start and short-pane rows are planned again through Runner.SessionIdle in session_test.go and TestSend_WaitsForIdleSession
  - id: F6
    severity: MEDIUM
    class: design
    location: _lyx/plan/05-reed-wake-cadence.md
    summary: Card 5 tests watchNames by driving real timers at a short timing and asserting the cadence doubles to the ceiling, a wall-clock-dependent assertion of behaviour its own NextWakeCadence table already pins
  - id: F7
    severity: MEDIUM
    class: scope
    location: _lyx/plan/14-idle-cpu-benchmark.md
    summary: Card 14 measures on the operator's live hub and ends its production watchdog process, where D6 says to start a hub with at least three idle sessions
  - id: F8
    severity: LOW
    class: design
    location: _lyx/plan/07-session-start-signal.md
    summary: Cards 7 and 13 each add an escape hatch without stating its bound in Intent; Spec.ContextAfterCompaction is a verbatim shell command validate does not inspect, and //lyx:one-shot lets any literal through
  - id: F9
    severity: LOW
    class: consistency
    location: _lyx/plan/04-reed-resize-events.md
    summary: Card 4 plans a new test for watchDefaultTiming's floor, while TestWatchState's DefaultTimingMatchesTheSixConstants subtest already pins watchDefaultTiming and can take the assertion
  - id: F10
    severity: LOW
    class: scope
    location: _lyx/plan/06-watchdog-discovery-backoff.md
    summary: The discovery loop's 5 s to 60 s backoff, its discover-signal wake and the longer idle-exit and reap delay are documented in no doc.go; reedcli has none, card 6 edits no doc and card 5's reedengine doc text covers only the signal and name repair
  - id: F11
    severity: LOW
    class: consistency
    location: _lyx/plan/13-daemon-wakeups-guard.md
    summary: Card 13 adds two top-level cmd/lyx tests where the decision record's test-budget constraint says cmd/lyx gains one
  - id: F12
    severity: LOW
    class: consistency
    location: _lyx/plan/00-overview.md
    summary: Shared decision opener-seam-exported-from-reedengine says reedcli imports neither fswatch nor fsnotify, but FileWatchOpener's channel carries fswatch.Event, so card 6's tests that feed the channel must import fswatch
  - id: F13
    severity: NIT
    class: consistency
    location: _lyx/plan/05-reed-wake-cadence.md
    summary: Card 5 pins the DiscoverSignalFileName literal in watchdog_test.go, a tautology beside TestReedGeometry and the cold-boot integration assertion, which already check the told path
---

### [BLOCKING] Card 4 breaks the tmux-tier build

**Location:** `_lyx/plan/04-reed-resize-events.md`

**Issue:** Card 4 removes `watchTiming.SignalTick`.
`internal/reedengine/watchdog_integration_test.go` (`//go:build tmux && linux`) defines `fastWatchTiming`, which assigns `t.SignalTick = 20 * time.Millisecond` at line 33, and every self-heal test in that file calls it.
The card's `Edit:` list does not name that file, so after card 4 the plan's own `go vet -tags tmux ./...` fails to compile `internal/reedengine`.
The card states that "the existing `tmux`-tier self-heal tests keep running the production opener live", which also needs `fastWatchTiming` changed, since `startWatchLoop` in `watchloop_test.go` gains the opener argument.

**Fix:** Add `internal/reedengine/watchdog_integration_test.go` to card 4's `Edit:` list, and say in `Intent` that `fastWatchTiming` drops its `SignalTick` line and its comment.

### [BLOCKING] Cards 10 and 11 push packages past their test-budget rows

**Location:** `_lyx/plan/10-shell-changedir.md`, `_lyx/plan/11-orch-resume-context-engine.md`

**Issue:** `cmd/lyx/testdata/test-budget.yaml` holds `internal/shell: 11`, and `internal/shell` has 11 top-level tests today (`TestShell_Quote` through `TestForGOOS`).
Card 10 adds `TestShell_ChangeDir`, a twelfth, and does not list the budget file, so `TestTestBudget_PackagesStayWithinBudget` fails from card 10 on and in the plan's final `go test ./...`, because no later card raises the `internal/shell` row.
`internal/orchengine` sits at 126 of 126.
Card 11 creates `resumecontext_test.go` and covers the mark codec in `state_test.go`, so it adds at least one top-level test, and it does not list the budget file.
It only passes if card 9's raise, which card 9 sizes "as the new tests need" for its own tests, happens to cover card 11 too.
The repo's rule is that the card whose new top-level tests pass a row lists the budget file under `Edit:`.

**Fix:** Add `cmd/lyx/testdata/test-budget.yaml` to the `Edit:` lists of cards 10 and 11, with `Intent` raising `internal/shell` and `internal/orchengine` by the tests each card adds.

### [MEDIUM] Card 7 leaves the claudeengine package doc false

**Location:** `_lyx/plan/07-session-start-signal.md`

**Issue:** `internal/shuttleengine/claudeengine/doc.go` maps each recording hook's payload to a signal (line 74), lists the recording hooks (line 81) and states "No recording hook prints to standard output" (line 84).
Card 7 adds a `SessionStart` hook whose second command prints the context JSON, a `SessionStart` payload parse and the `SessionStartContext` renderer, and it edits only the file comment of `settings.go`.
The repo's docs-in-the-same-commit rule, and the decision record's Scope-In docs line, require the package doc to follow the behaviour.

**Fix:** Add `internal/shuttleengine/claudeengine/doc.go` to card 7's `Edit:` list.
`Intent` names the `SessionStart` row in the payload mapping, the hook in the recording-hook list, and the fact that the session-start entry's second command, unlike the recording commands, prints the provider's additional-context JSON.

### [MEDIUM] Card 8 states no bound for what the readiness reading now lets through

**Location:** `_lyx/plan/08-readiness-reading.md`

**Issue:** Card 8 downgrades a guard.
The pane's running-turn needle and a missing box no longer hold a ready reading, and the card says the probe "still answers idle and logs the disagreement".
That states what passes but not what bounds it.
D7's Bound gives the bound: the verified send's landed check retries or reports a submission that did not land, and only the watcher's own commands and prompts are typed, never operator text.
The rubric's attack-surface rule requires that bound in the introducing card's `Intent`.
The idle override is bounded in the card: 10 s of idle polls, logged at Warn.

**Fix:** Add a `Bound:` sentence to card 8's `Intent`.
It says that a ready reading beside a needle or a missing box lets an injection reach a screen state the hooks do not report, such as a menu, and that the landed check and the restriction to the watcher's own text bound it.

### [MEDIUM] Card 8 tests the same readiness scenarios three times, once through a private helper

**Location:** `_lyx/plan/08-readiness-reading.md`

**Issue:** `readiness_test.go` is a new table over the unexported `judgeReadiness` with draft, short-pane and unmatched-turn-start rows.
`session_test.go` then drives the same draft and unmatched-turn-start cases through `Runner.SessionIdle`, the public surface, and `TestSend_WaitsForIdleSession` in `run_test.go` gains draft and unmatched-turn-start rows a third time.
`PATTERN-test-economy` pins behaviour at the public surface and adds a test only for behaviour no existing test covers.
The decision record's constraint says the reading "extends the existing session-state tests' tables where a scenario fits".
`judgeReadiness` is a helper that serves `Runner.SessionIdle` and `busyReading`, and both public-surface tests already exercise it.

**Fix:** Drop `internal/shuttleengine/readiness_test.go` from card 8's `Create:` list.
Carry the per-state-and-cause rows (`idle-done`, `idle-stalled`, `asking`/`awaiting-input`, `busy`/`background`, `busy`/`turn`, `asking`/`ask`, `dead`, `unknown`) as a table in the `Runner.SessionIdle` test in `session_test.go`.
Keep `TestSend_WaitsForIdleSession`'s rows to what is specific to the send: one held reading and the needle case.

### [MEDIUM] Card 5's watchNames test depends on the wall clock

**Location:** `_lyx/plan/05-reed-wake-cadence.md`

**Issue:** Card 5 plans "a test driving `watchNames` with a short timing against the fake tmux, asserting the pass cadence doubles to the ceiling while nothing drifts and resets after a repair".
`watchNames` runs on a real timer, so the test has to observe intervals between passes in real time.
That breaks the testing rule against real-time dependence, and `PATTERN-test-speed` requires injectable clocks.
The doubling-and-reset rule is already pinned by the `NextWakeCadence` table this card adds to `watchdog_test.go`.
Card 6 does the same job without the clock by putting the timing in the pure `discoveryWake` planner and testing it as a table.

**Fix:** Restrict the `watchNames` test to what the table cannot show: `repairNames`'s `repaired` answer chooses the next cadence, observed through a timing or clock seam and not through elapsed wall time.
Alternatively, give the name-repair loop a pure planner as card 6 does, and drop the cadence-timing test.

### [MEDIUM] Card 14 measures on the operator's live hub and kills its daemon

**Location:** `_lyx/plan/14-idle-cpu-benchmark.md`

**Issue:** D6's procedure starts a hub with at least three idle reed sessions.
Card 14 takes the before figure from "the daemon the production `lyx` runs on the operator's hub" and the after figure "after the production daemon's process is ended".
The executing agent would end a process on the operator's live hub, which the decision record does not license.
The operator's hub also cannot be held at "at least three idle sessions" for ten minutes, because runs there start and stop.

**Fix:** Run both figures on a scratch hub the card starts, with three idle reed sessions.
The before figure comes from a `main` build's watchdog and the after figure from this branch's `./deploy-dev` build, each started in the foreground with `--hub-path`, `--tmux` and `--shell`.
No process on the operator's hub is touched.

### [LOW] Cards 7 and 13 add escape hatches without stated bounds

**Location:** `_lyx/plan/07-session-start-signal.md`, `_lyx/plan/13-daemon-wakeups-guard.md`

**Issue:** Card 7's `Spec.ContextAfterCompaction` is a shell command rendered verbatim into a provider hook, and "`validate` does not inspect it".
The card says the field is "empty for every run but the one that sets it" but not who may set it.
D11 bounds it to `orchcli` alone, and only card 12 states that bound.
Card 13's `//lyx:one-shot <reason>` lets any sub-second literal in a declared file past the guard.
The card states the scan's literal-shape bound, but not that the annotation enforces announcement and not absence (D2's Bound), and not whether an empty reason passes.

**Fix:** In card 7's `Intent`, add that `orchcli` is the only setter and that the command runs only at a compaction of that run.
In card 13's `Intent`, add that the annotation lets the line through and is bounded by review of the stated reason, and say whether the guard refuses a bare `//lyx:one-shot` with no reason.

### [LOW] Card 4 plans a new test where an existing subtest pins the same timing

**Location:** `_lyx/plan/04-reed-resize-events.md`

**Issue:** "A new test asserts `watchDefaultTiming`'s periodic fields (`PollCycle`, `Dormant`) are at or above one second".
`TestWatchState` in `internal/reedengine/watchloop_test.go` already runs `DefaultTimingMatchesTheSixConstants`, which pins what `watchDefaultTiming` returns.
The floor is one more assertion on the same value.

**Fix:** Put the floor assertion into the `DefaultTimingMatchesTheSixConstants` subtest, which loses `SignalTick` in this card anyway, and plan no new test.

### [LOW] The discovery loop's new cadence is documented nowhere

**Location:** `_lyx/plan/06-watchdog-discovery-backoff.md`

**Issue:** `internal/reedcli` has no `doc.go` and no package comment.
`docs/overview.md` points readers to `internal/reedengine`'s package documentation for reed.
Card 5's edit of `reedengine/doc.go` covers the discover signal, who touches it, and the name-repair backoff.
Card 6 changes the daemon's observable behaviour: discovery backs off from 5 s to 60 s, wakes on the signal, and idle exit and orphan reap can take up to three minutes.
Card 6 edits no doc, although the decision record's Scope-In docs line names `reedcli` where its behaviour changes.

**Fix:** Add `internal/reedengine/doc.go` to card 6's `Edit:` list, stating the discovery cadence, the signal wake with its base-cadence bound, and the longer idle-exit and reap delay.
Alternatively, have card 6 create the doc for `reedcli`.

### [LOW] Card 13 adds two cmd/lyx tests where the record allots one

**Location:** `_lyx/plan/13-daemon-wakeups-guard.md`

**Issue:** The decision record's test-budget constraint says "`cmd/lyx` gains one test".
Card 13 adds the guard and "a second small test" for the detector.
The detector cases can be subtests of the guard's own test function.

**Fix:** Make the crafted-snippet detector cases subtests of the one `//lyx:guard` test, and raise `cmd/lyx` by one.

### [LOW] The opener shared decision contradicts card 6's tests

**Location:** `_lyx/plan/00-overview.md`

**Issue:** Shared decision `opener-seam-exported-from-reedengine` says "`reedcli` imports neither `fswatch` nor fsnotify, and tier-1 tests pass an opener over a channel they feed".
`FileWatchOpener` returns `<-chan fswatch.Event`.
A `reedcli` test that builds and feeds such a channel has to name `fswatch.Event`, so it imports `fswatch`.

**Fix:** Narrow the decision to `reedcli`'s production files, or have `reedengine` export its own event type.

### [NIT] Card 5 pins a constant's literal

**Location:** `_lyx/plan/05-reed-wake-cadence.md`

**Issue:** `watchdog_test.go` "pins `DiscoverSignalFileName`".
The name's behaviour is that teller and daemon agree on one path, and two other tests already check that: `TestReedGeometry`, through `DiscoverSignalPath`, and the cold-boot integration test, through the file at the told path.

**Fix:** Drop the pin and keep the table test over `NextWakeCadence`.

## Focus verdicts

- Attack surface on cards 4, 5, 6, 11 and 12: no defect in any of them.
  Card 4 says that an event finding no file runs nothing and that an apply runs only after the debounce.
  Card 6 says that `next` never runs a signal's cycle sooner than the base cadence.
  Card 11 states the skip condition and the double-pointer bound.
  Card 12 states the hook's bound.
  The bound gaps are in cards 8, 7 and 13 (F4 and F8).
- Card order: no violation found.
  `NextWakeCadence` and `DiscoverSignalFileName` (card 5) precede card 6.
  `OpenFileWatch` (card 4) precedes cards 5 and 6.
  `ChangeDir` (card 10), `ResumeContext` (card 11) and `SessionStartContext` (card 7) precede card 12.
  `IdleProbe.Reason` (card 8) and `RunSessionState.SessionStartAt` (card 7) precede card 9.
- Fidelity: D1 to D11 and every Scope-In item are carried, apart from the doc gaps in F3 and F10.
  The two carry-over entries license no scope of their own, and cards 11 and 12 implement them through D11's own text, which licenses them.
- Member glyphs and re-sign arrows: every `Edit` card that names symbols lists member glyphs.
  The arrows on `watchLoop`, `repairNames`, `runWatchdogLoop` and `buildSettings` match the stated signature changes.
  The signatures in the tree of `busyReading`, `Runner.newSendContext`, `probeIdle` and `logIdleStateDisagreement` do not change under the cards' intents.
- `## verify:`: it runs every targeted package through `./...` in both runnable tiers and compiles `tmux` and `llm`, in the template's order.
  No `docs/` tooling is touched.
- `ImpactSummary`: every one is a blast-radius conclusion, not a restatement of `Intent`.
- Granularity of cards 1, 7, 10 and 13: no defect.
  `Shell.ChangeDir` extends a cross-package seam and earns its own card, and the private types in card 8 stay with their surface.
