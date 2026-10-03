---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/plan/11-anomalies-under-step.md:21
    summary: The step entry observation is taken before the seed-and-commit bootstrap and reed Up, so a step refused there after a completed step consumes the clean-handoff marker and the next step files a false crash-resume
  - id: F2
    severity: MEDIUM
    location: _lyx/plan/11-anomalies-under-step.md:45
    summary: Card 11's tests only inject c.entryObservation; no case drives one completed step and then a second step through the real marker write and entry observation, which is the sequence F1 asks to protect
  - id: F3
    severity: LOW
    location: _lyx/plan/11-anomalies-under-step.md:42
    summary: The bound paragraph does not name the residual false crash-resume after a BuildShed refusal or a shed.Step error that leaves the status running, neither of which writes a fresh marker
  - id: F4
    severity: LOW
    location: _lyx/plan/10-anomaly-pending-retry.md:38
    summary: No listed test pins the cancelled-context arm's pending behaviour or the unreadable and undecodable status arms the Intent says now reach the filing pass
  - id: F5
    severity: LOW
    location: _lyx/plan/10-anomaly-pending-retry.md:3
    summary: selfreport_github_test.go's filing-failure test documents that a failed title is left out of the marker, which card 10 makes false, and the file is not in card 10's Edit list
  - id: F6
    severity: LOW
    location: _lyx/plan/12-halt-filing-ownership.md:24
    summary: The halt-note line says Tier 1 files the halt event even when this pass's filing failed or was skipped; the event is then filed only if a later pass runs, and neither card 11 nor card 12 names that residual
  - id: F7
    severity: NIT
    location: _lyx/plan/11-anomalies-under-step.md:39
    summary: The halt-note line says files this halt event itself while the stencil keys on files the halt event itself, so card 12's claim that the sentence matches the line is not literal
---

### [MEDIUM] Entry observation placed before the bootstrap consumes the marker on a refused step

**Location:** `_lyx/plan/11-anomalies-under-step.md:21`

**Issue:**
Card 11 places `observeEntry` in `loomPreStepUnmarked` "directly after the busy probe finds the run lock free and before the seed-and-commit bootstrap".
`observeEntry` consumes (deletes) the step clean-handoff marker (`consumeStepHandoffMatch`, `internal/loomcli/selfreport.go`).
Every return after that point but before `shed.Step` refuses the step without calling `AfterStep` (`internal/shedverbs/step.go`: a PreStep error returns before `AfterStep`) and without `PostStep` rewriting the marker:
`seedAndCommitBootstrap`'s origin, seed, ownership and commit stages, the bootstrap-lock acquisition, and `c.reed.Up()`.
So after a completed step, a step refused at, say, a transient reed or commit failure spends the voucher, and the step ly-drive runs next observes `running`, history non-empty, lock free and no marker, and files a public `crash-resume` issue for what was a clean handoff.
That is the outcome F1's "so a completed step does not read as a crash" rules out.
Nothing requires the earlier placement: for a seeded run the bootstrap leaves the status file untouched (`loomshed.Seed` returns `ErrSeedExists`), so an observation after `reed.Up()` reads the same state, and it then also follows the ownership check as `loomPreRun`'s does.

**Fix:**
Take the observation at the end of `loomPreStepUnmarked`, after the busy probe, the bootstrap and `reed.Up()` have all succeeded, and state in the card that only a step that reaches `BuildShed` consumes the marker.
Keep the busy-refusal test's assertions.

### [MEDIUM] No test pins the completed-step-then-step sequence

**Location:** `_lyx/plan/11-anomalies-under-step.md:45`

**Issue:**
The crash-resume case injects a crash-resume observation on `c.entryObservation` directly, and `TestStepCmd_BusyRefusal_BeforeBootstrap` pins only the busy side of the placement.
No case runs `loomPostStep`'s marker write followed by the next step's entry observation and `loomAfterStep` with the fake filer, so a mismatch between what `recordStepHandoff` records and what `observeEntry` reads, or an observation moved out of `PreStep`, would file a crash-resume on every driven step with all of card 11's tests green.
`loomPreStepUnmarked` itself cannot run in a Tier 1 test (its bootstrap needs a real fabric), and `stepEnvelopeOver` clears `PreStep`, so the plan needs a seam for it.

**Fix:**
Factor the observation into one receiver method (for example `observeStepEntry`) that `loomPreStepUnmarked` calls, and add a `halt_test.go` case that drives the generic step body twice over a shed whose first row returns Done and leaves the run `running`, with `PreStep` set to that method: the second step files no `crash-resume`, and the same sequence with the marker removed between the steps files one.
`stepEnvelopeOver` seeds the status file on every call, so the two-step case seeds once.

### [LOW] The bound omits the residual false crash-resume after a refused or errored step

**Location:** `_lyx/plan/11-anomalies-under-step.md:42`

**Issue:**
Even with F1's placement, two paths spend the marker and write no fresh one:
a `BuildShed` refusal (it runs after `PreStep` and never reaches `AfterStep`), and a `shed.Step` error that leaves the status `running` (a status read failure, an unknown `current_producer`, a persist failure), since `PostStep` runs on success only.
The next step then files a `crash-resume`, the same issue `run` files after the same error.
The "What this lets through" paragraph names only the filing bounds, not this residual.

**Fix:**
Name the residual in the bound: one `crash-resume` issue, titled by producer and history length so it files once, after a step refused at `BuildShed` or one whose error left the run `running`, as `run` already does after the same error.

### [LOW] Card 10's tests miss the cancelled-context and unreadable-status arms

**Location:** `_lyx/plan/10-anomaly-pending-retry.md:38`

**Issue:**
The Intent states two new behaviours no listed case pins:
a cancelled context with a lone crash-resume retries the pending entries, and with no crash retries nothing;
an unreadable status file or an undecodable product now calls `runFilingPass` and retries the pending entries.
Only the missing-status arm, the knob off and the busy error are listed.

**Fix:**
Add cases: a cancelled context with no crash leaves the pending entries and calls no filer;
a cancelled context with a crash-resume files the crash-resume and retries the pending entries;
an unreadable status file and an undecodable product each still retry the pending entries.

### [LOW] An existing engine-boundary test's doc goes false

**Location:** `_lyx/plan/10-anomaly-pending-retry.md:3`

**Issue:**
`TestDetectAndFileAnomalies_FilingFailure_LeavesMarkerUnfiledAndDoesNotPropagate` in `internal/loomcli/selfreport_github_test.go` documents that a failing call "leaves that title out of the marker".
After card 10 the title lands in `Pending`; the test still passes because it checks only `Titles`, so its doc becomes false with nothing failing.
The file is not in card 10's Edit list.

**Fix:**
Add `internal/loomcli/selfreport_github_test.go` to card 10's Edit list: the test's doc says the title stays out of `Titles` and is kept pending, and the test asserts the pending entry with the body sent at the engine boundary.

### [LOW] The halt-note line's ownership claim has an unnamed residual

**Location:** `_lyx/plan/12-halt-filing-ownership.md:24`

**Issue:**
Card 11 sets the line from `c.cfg.Selfreport` alone, and Tier 1 runs first in both hooks.
When this pass's filing of the halt fails (card 10 keeps it pending) or the pass was skipped (a context cancelled after `shed.Step` returned), the line still tells the reflection not to file the halt event.
The event then reaches GitHub only if a later Tier 1 pass runs in that worktree; a run abandoned at the halt never files it.
Addition (b) promises only that a resumed halt is filed, so this is within the answer key, but neither card 11's nor card 12's bound states it.

**Fix:**
Name the residual in card 11's and card 12's "What this lets through": with the knob on, a halt event whose Tier 1 filing failed or was skipped is filed by the next Tier 1 pass, and is never filed if no later `run` or `step` runs.

### [NIT] The halt-note line and the stencil sentence word the claim differently

**Location:** `_lyx/plan/11-anomalies-under-step.md:39`

**Issue:**
The note line reads `Tier 1 anomaly filing files this halt event itself; file only the lyx problems behind it.`, while card 12's stencil sentence keys on a note that "says Tier 1 anomaly filing files the halt event itself", and card 12 says the two match.

**Fix:**
Word the note line `Tier 1 anomaly filing files the halt event itself; file only the lyx problems behind it.` so the stencil quotes it verbatim.

## Focus verdicts

- Card 11 and F1's observation placement: the tests do not pin the real sequence (F2), and a step can leave a state the next observation reads as a crash-resume, both before `shed.Step` (F1, fixable by placement) and from a `shed.Step` error or a `BuildShed` refusal (F3, a residual to name).
- Card 11's bound and transient failures: not a defect.
  A transient-marked failure persists `failed`, and `detectHaltAnomaly` titles it `<producer>#<success count>`, so the stated filed-title dedupe bounds re-stepped transient failures to one issue per producer per success count, the same filing `run` already does for the same failure.
  Addition (a) keeps the existing skips and adds none, so a transient skip would be scope the answer key does not license.
- Card 10's entry conditions: consistent.
  The cancelled-context arm is a partial skip, which the Intent states explicitly, and "every path reaches the filing pass" is scoped to "past them".
  F2's resumed-run test is the done-when case; the missing arms are F4.
- Cards 11 and 12 coupling: the line is written on all four paths through `reflectHalt`, its wording differs slightly from the stencil's (F7), and it can be untrue for this pass (F6).
  Addition (a)'s "does not double-file" is met: Tier 1 files the halt event under its title and the reflection skips it when the note says so.
- Fidelity: every directive in F1 and F2 is carried by card 10, 11 or 12.
  The `testing.Testing()` refusing filer and `loomPostRun`'s move onto the seam are the instrument F1's "tests through `loomCLI.fileIssue`" requires and keep tests from filing real issues; neither adds observable scope, so neither is a finding.
