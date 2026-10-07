---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/13-replay-corpus.md
    summary: Card 13 renames the unexported clock interface to Clock but lists neither fixture_test.go, wait_test.go nor startup_test.go, which name the type, so the shuttleengine test package stops compiling
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/10-treadle-triage-retire.md
    summary: Card 10 drops the triage path from roundOutcome and roundRecord but omits result.go, whose exported RoundSummary.TriagePath stays as a dead field, and the profile.go comment naming triage
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/plan/10-treadle-triage-retire.md
    summary: Card 10 deletes the triage stencil file but leaves its line in .gitattributes
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/plan/11-retire-outcome-asking.md
    summary: Prose that still names asking as a run outcome survives outside every card (webstercli run help text, burlerengine, frictionengine, mergeresolve, webstercli ownership, shuttleengine run_test comments) and card 11's grep covers symbols only
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/plan/05-hold-turn-end.md
    summary: Card 5 bundles the hold with AwaitOperator retirement and the whole notifier seam with its notice builder and two RunState fields, which are independently reviewable units with their own decisions and acceptance criteria
  - id: F6
    severity: MEDIUM
    class: design
    location: _lyx/plan/05-hold-turn-end.md
    summary: Card 5 defines NotifiedOffset only through a batch event Raw line and never says which offset an expired-shell hold records, though that turn end arrives with no new bytes and Attach replays its waiting line
  - id: F7
    severity: LOW
    class: design
    location: _lyx/plan/05-hold-turn-end.md
    summary: Card 5 does not say what happens when persisting NotifiedOffset fails, and an error return there would be a new negative-verdict site that the completion-signal tripwire pins by count
  - id: F8
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/12-tail-in-stencils-and-steer.md
    summary: Card 12 asks the shed driver stencil to state that every message to a working agent ends with the tail but the stencil's only messaging section sends the parent a notice and says to add nothing to it
  - id: F9
    severity: LOW
    class: design
    location: _lyx/plan/05-hold-turn-end.md
    summary: hold_test.go rows for keeps polling then Done and for expired shell with and without output duplicate rewritten wait_test.go and waiting_test.go tests, so only the notice rows are new behavior
  - id: F10
    severity: LOW
    class: design
    location: _lyx/plan/14-live-turn-scenarios.md
    summary: Card 14's last scenario, a turn end without output that holds then ends done after a tail answer, repeats card 11's rewritten TestSmokeGuardrailAskingSurfacesQuestion
  - id: F11
    severity: LOW
    class: design
    location: _lyx/plan/13-replay-corpus.md
    summary: Card 13 states no expected reading for the fork-killed, skill-load and continued-in cases, names no real source for four cases, and its harness takes the fake ClassifySkillLoad and sets no skills
  - id: F12
    severity: LOW
    class: scope
    location: _lyx/plan/01-payload-authority.md
    summary: Card 1 makes a fork's description its Label with no test asserting it and no update of the BackgroundTask.Label doc that says empty for a fork
  - id: F13
    severity: LOW
    class: consistency
    location: _lyx/plan/01-payload-authority.md
    summary: Card 1 points at prune-rest transcript lines 280 and 292-296 but the real launch is line 281 and the absorbed notices are lines 293 and 296-298 plus queued_command lines 302-303
---

### [BLOCKING] Card 13's `Clock` rename breaks test files it does not list

**Location:** `_lyx/plan/13-replay-corpus.md`, Edit group; `internal/shuttleengine/fixture_test.go:47,60,111,146`, `wait_test.go:47,66,386,1005`, `startup_test.go:36`.
**Issue:** The card turns the unexported `clock` interface in `wait.go` into the exported `Clock` and says "its uses renamed", but its Edit group names only `wait.go`, `run.go` and `claudeengine/doc.go`.
Three in-package test files name the type directly: `fixture_test.go` (`clk clock`, `withClock(clk clock)`, `withRunClock(clk clock, …)`), `wait_test.go` (`var _ clock = (*fakeClock)(nil)` and two more, plus `var clk clock = fc`), and `startup_test.go` (`var _ clock = (*frozenClock)(nil)`).
After the card as written, `go test ./internal/shuttleengine/` does not compile, so the card fails its own tier-1 gate, and its `ImpactSummary` ("adds test data and one untagged test") understates the blast radius.
**Fix:** Add `fixture_test.go`, `wait_test.go` and `startup_test.go` to card 13's Edit targets, state the rename in the card's `ImpactSummary` as touching those test files, and keep `realClock` unchanged.

### [MEDIUM] Card 10 leaves `RoundSummary.TriagePath` and a triage comment behind

**Location:** `_lyx/plan/10-treadle-triage-retire.md`; `internal/treadleengine/result.go:41`, `run.go:223,617`, `engine_test.go:314`, `profile.go:40`.
**Issue:** The card removes `roundOutcome.TriagePath` and `roundRecord.TriagePath`, but the exported `RoundSummary.TriagePath` in `result.go` is filled from both (`run.go:223` and `run.go:617`) and asserted in `engine_test.go:314`.
`result.go` is in no card, so the field survives as dead, always-empty API on the result type, against D7's "every branch that existed only to handle an asking result goes".
`profile.go:40` also still says the judge model tunes "judge/triage calls".
**Fix:** Add `result.go` (drop `RoundSummary.TriagePath`) and `profile.go` (comment) to card 10's Edit targets, and have the `engine_test.go` edit delete the `Rounds[0].TriagePath` assertion.

### [LOW] Card 10 leaves a dead `.gitattributes` line

**Location:** `_lyx/plan/10-treadle-triage-retire.md`; `.gitattributes:15`.
**Issue:** `.gitattributes` pins `contracts/stencils/treadle/treadle-template-triage.md` to LF for the embedded stencil set, and card 10 deletes that file.
The line then names a path that no longer exists.
**Fix:** Add `.gitattributes` to card 10's targets and delete that one line.

### [MEDIUM] Stale prose that names asking as a run outcome is outside every card

**Location:** `_lyx/plan/11-retire-outcome-asking.md`.
**Issue:** D7's rationale is that an outcome `Wait` cannot return leaves "dead code and stale documentation behind", and card 11's closing grep checks only the symbols `OutcomeAsking`, `AwaitOperator`, `ErrMasterAsking`, `MasterAskingError` and `runTriage`.
Prose naming the `asking/died/timeout` triple survives in files no card targets: `internal/webstercli/run.go:63` (the `lyx webster run` help text: "the shuttle spawn itself ends asking/died/timed-out"), `internal/webstercli/ownership.go:5` ("the shuttle-asking exit"), `internal/burlerengine/engine.go:121,217-218` and `doc.go:275`, `internal/burlerengine/smoke_cluster_test.go:264`, `internal/frictionengine/reflect.go:176`, `internal/mergeresolve/mergeresolve.go:108`, `internal/loomcli/smoke_attachprobe_test.go:58`, and `internal/shuttleengine/run_test.go:457,496`.
The first is user-facing CLI text, which the record's docs-in-the-same-commit rule covers.
**Fix:** Add these files to card 11's Edit targets as comment and help-text edits (each loses `asking` from its outcome list), and extend the card's closing grep to the word `asking` used as an outcome in shuttle-result contexts, with a stated allowlist (`DeadReasonAsking`, `docs/benchmarks/`).

### [MEDIUM] Card 5 is three units

**Location:** `_lyx/plan/05-hold-turn-end.md`.
**Issue:** The card holds 17 targets across six packages and covers D4 (the hold in `Wait`), the retirement of `Spec.AwaitOperator` with every caller edit, and D5's engine half: the `Runner.SetNotifier` seam, the notice builder in `hold.go`, `RunState.StrandName` and `RunState.NotifiedOffset`.
The notifier seam is public surface that card 7 consumes from three other packages, and D4 and D5 carry separate decisions and separate acceptance criteria.
The granularity rule gives a public or cross-package symbol its own card, and the plan itself stages the `OutcomeAsking` removal but bundles the `AwaitOperator` one.
**Fix:** Split card 5 into the hold with `AwaitOperator` retirement (a hold with no notifier logs at Info for every run) and a new card that adds `SetNotifier`, `hold.go`, `StrandName`, `NotifiedOffset` and the notice tests.
Renumber cards 6–14 to 7–15 with their cross-references, the Card Index and every `Applies to` line.

### [MEDIUM] The offset an expired-shell hold records is unspecified

**Location:** `_lyx/plan/05-hold-turn-end.md`, "Notify once per turn end".
**Issue:** `NotifiedOffset` is "the events-file offset just past the last notified held turn end's line", located "from its `Raw` line" in a batch.
An expired-shell hold is produced by `expiredTurnEnd` on a tick with no new bytes, from a waiting line read earlier, so no batch is in hand and the card names no offset to record.
Without one, an Attach that replays the same waiting line waits out the shell bound again and notifies a second time, against D5's "replaying events after Attach does not re-notify".
No `hold_test.go` row pairs an expired-shell hold with an Attach.
**Fix:** State that `recordWaiting` keeps the end offset of the waiting turn end's line, that `expiredTurnEnd` notifies and persists that offset, and add an Attach row to card 6's end-to-end test for a `running` record whose `NotifiedOffset` covers an expired-shell waiting line.

### [LOW] A failed `NotifiedOffset` write has no stated outcome

**Location:** `_lyx/plan/05-hold-turn-end.md`.
**Issue:** The card says Wait persists `NotifiedOffset` with `saveRunState` before calling the notifier, and that a notifier error is logged at Warn and never ends the run, but it is silent on a persistence error.
Returning it would add a negative-verdict `Errorf` return to `Wait`, which `completionsignal_enforcement_test.go` pins by count (`Wait [Errorf]` x4) and which `PATTERN-completion-signal` requires to consult the output files first.
**Fix:** State that a persistence failure logs Warn and skips that one notice, never ends the run, and so adds no return site, and have the tripwire edit leave the audited counts unchanged.

### [MEDIUM] The driver stencil's only messaging rule contradicts the tail statement

**Location:** `_lyx/plan/12-tail-in-stencils-and-steer.md`; `contracts/stencils/shed/shed-template-driver.md:235-240`.
**Issue:** D8 and the card require the driver stencil to say every message to a working agent ends with the tail, "where it describes messaging an agent".
The driver stencil sends exactly one kind of message, the notice to its parent in `## Notifying the parent`, and says "The notice is the whole message: add nothing to it, and branch on the envelope field alone".
The card names no placement and no reconciliation, so an implementer can write the tail into that section and break the verbatim `parent_notice` rule, or leave the claim with no home.
**Fix:** Say in the card where the statement lands (a sentence in `## Notifying the parent` stating that the parent notice is verbatim and carries no tail because the parent is not interrupted mid-task, and that any message to a working agent the driver sends ends with the tail), and pin both facts in `claims_test.go`.

### [LOW] `hold_test.go` repeats rewritten tests

**Location:** `_lyx/plan/05-hold-turn-end.md`, Tests.
**Issue:** The card rewrites `TestRun_Wait_AwaitOperator_AskingNonTerminal` (its `AwaitOperatorTrue_DropsAskAndFinalizesOnceOutputFilesAppear` subtest is the hold-then-Done reading) and `TestPollEventsTick_ShellExpiry` (expiry with outputs reads done, without reads asking), and also lists `hold_test.go` rows for "keeps polling … a later Stop with all outputs finalizes Done" and for the expired-shell hold and Done.
The same readings are then covered twice.
**Fix:** Keep only the notice behavior in `hold_test.go` (once per held turn end, second notifies again, batch notifies the last, silent for interactive and no-notifier, notifier error, sanitising and bounds, the notice naming the shell) and carry the no-notifier hold-then-Done and expired-shell hold rows in the two rewritten tests.

### [LOW] Card 14's last scenario repeats card 11's question smoke

**Location:** `_lyx/plan/14-live-turn-scenarios.md`, last scenario bullet.
**Issue:** The scenario "a turn end without the output file, which holds and notifies exactly once, and then ends done after an answer ending in `MessageTail`" is the same live reading that card 11's rewritten `TestSmokeGuardrailAskingSurfacesQuestion` pins (autonomous run, agent ends its turn without the file, run keeps waiting, answer through `Run.Send` ending in `MessageTail`, run finishes done); only the recording notifier differs.
D10 lists "a question" once.
**Fix:** Drop the scenario from card 14 and have card 11's rewritten question test set a recording notifier and assert one call.

### [LOW] Three corpus cases name no reading and no source

**Location:** `_lyx/plan/13-replay-corpus.md`.
**Issue:** The card gives the expected reading for the two #411 runs, the Plan-Write ask and the Master fork, but only the case names for `fork-killed-api-error`, `skill-load-turn-end` and `session-continued-in`.
It also cites a real source only for the three cases whose paths are in the record's Notes, and says only "sourced from real transcripts under `~/.claude/projects/`" for the rest.
Its harness takes `shuttlefake.Engine`'s default `ClassifySkillLoad` and starts runs with no skills, so a skill-load or continued-in case cannot express what it pins, and without a stated reading the fork-killed case cannot be told apart from card 1's non-running-entry row and `skillload_test.go`.
**Fix:** State per case the turn-end reading it pins and the existing test it does not repeat, name the source transcript of each case, and either wire `Spec.Skills` and the real `ClassifySkillLoad` into the harness for the skill-load case or fold the case into an existing table.

### [LOW] Card 1's fork label is untested and leaves a doc stale

**Location:** `_lyx/plan/01-payload-authority.md`; `internal/shuttleengine/engine.go` (`BackgroundTask.Label` doc).
**Issue:** The card makes a `subagent` entry's `description` its `Label`, which D1–D3 and D5 do not require, and none of its listed test rows asserts it.
The `BackgroundTask.Label` doc in `engine.go`, an Edit target of the same card, still says "empty for a fork".
**Fix:** Add the fork-label assertion to the card's existing waiting row and update the `Label` doc, or drop the fork label and let the notice fall back to the id as card 5 already does.

### [LOW] Card 1's transcript line pointers do not match the transcript

**Location:** `_lyx/plan/01-payload-authority.md`, fixture paragraph.
**Issue:** The card tells the implementer to copy "launch at line 280, absorbed notice at lines 292–296" of the `test-suite-prune-rest` transcript.
In that file line 280 is an unrelated Bash call, the launch of `toolu_0156x6kuzhRxYK53hcQhn2Lk` (task `bn68egcp9`) is line 281, line 292 is a tool result, the `queue-operation` notices are lines 293 and 296–298, and the `queued_command` attachments are lines 302–303.
The second transcript's pointers (lines 95, 356–362) are accurate.
**Fix:** Name the first fixture by tool-use id and task id (`toolu_0156x6kuzhRxYK53hcQhn2Lk`, `bn68egcp9`) with the corrected line numbers.
