---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/relaunch.go:262
    summary: The resumed strand's awaitStartup runs with eventsProof on the reused events file, so the first session's session-start and turn-start lines mark it started on the first tick and its startup probe, gate dismissal and readiness wait never run.
    fix: >-
      In relaunchForReprompt, record the events file's size before Prepare and make the resumed startup read only lines past that offset.
      Do it by giving Run a startup events offset that checkLivenessTick's eventsProof branch and sessionStartRecorded honour
      (turnStartedAfterSessionStart and sessionStartRecorded read from that offset instead of the file's start), set by relaunchForReprompt and zero for a first launch.
      Update the checkLivenessTick doc in wait.go, which states that only a fresh events file feeds eventsProof, to the new rule.
      Add a TestGate_RepromptToGoneStrand row whose events file already holds a session start followed by a turn start and whose resumed pane is not yet ready,
      asserting that the resume waits for the new pane before the gate goes on.
  - id: F2
    severity: BLOCKING
    class: design
    origin: assist:assist
    location: internal/shuttleengine/relaunch.go:45
    summary: Clearing spec.Skills to skip the skill-load turn also changes what Claude's Prepare writes into the resumed session's settings, so with session scope enabled every installed plugin is disabled and the subagent skill passage is dropped.
    fix: >-
      Keep spec.Skills unchanged in relaunchForReprompt.
      Add a Spec field, for example SkipSkillLoadTurn, that each engine's Prepare reads to put the prompt pointer on the launch line instead of the skill-load message and return no PromptLine,
      while it still builds the session scope, settings and standing passages from Skills; set it in relaunchForReprompt.
      Change TestGate_RepromptToGoneStrand to assert the prepared spec keeps the run's skills and sets the new field.
  - id: F3
    severity: BLOCKING
    class: consistency
    origin: assist:assist
    location: internal/shuttleengine/relaunch.go:69
    summary: The resume's Prepare rewrites the run directory's manifest.json but never refreshes the kept copy under manifests/, which PATTERN-engine-parity requires of a launch once its run record is saved.
    fix: >-
      In relaunchForReprompt, after the saveState that names the new strand succeeds,
      call copyManifest(runDirRoot(runner.cfg, runner.anchorPath), run.state.RunID, run.runDir) and log a failure as a non-fatal Warn,
      as Runner.launch does after its own saveRunState.
  - id: F4
    severity: BLOCKING
    class: design
    origin: assist:assist
    location: internal/shuttleengine/relaunch.go:96
    summary: removeGoneStrand removes the run's tracked strand while run.json still reads running and names it, outside the stop verb's record-first order that PATTERN-shuttle-provider-seam requires; a failure or kill after it leaves a running record naming a removed strand.
    fix: >-
      Reorder relaunchForReprompt so the old strand is removed only after the record names the new one:
      Prepare, AddStrand, saveState naming the new strand, and only then remove the old strand when reed still tracks it and it is removable,
      logging a failed removal as a Warn without failing the resume.
      Drop removeGoneStrand's call before AddStrand.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-affected-set
    location: internal/shuttleengine/stepstop.go:91
    summary: StopStepRuns now writes stopped into a running record whose strand is untracked or has a cleared pane binding, so Attach's leftoverThenAgeVerdict treats it as respawn-eligible at any age and skips the minAge guard that holds a young run whose agent may still work in a pane reed cannot address.
    fix: >-
      In StopStepRuns, settle a tied run whose strand reed tracks with a cleared pane binding (tracked, not live, empty PaneID, anchor not hidden) or does not track
      only when no live process holds its session: ask the record's engine through SessionHolderFinder and leave the run running, listed under a new StepStopReport field such as Held, when a holder is live or none can be named.
      Render that field in loomcli's stepRunsNote, document it in StopStepRuns' doc and shuttleengine's doc.go step-stop paragraph,
      and add rows to TestRunner_StopStepRuns_StopsOnlyTheStepsRuns for a held and an unheld untracked run.
  - id: F6
    severity: MEDIUM
    class: design
    origin: assist:assist
    location: internal/shuttleengine/relaunch.go:46
    summary: relaunchForReprompt discards checkResume's warning that a live holder could not be ruled out, so an unreadable or undecodable registry entry for the session lets the resume launch a second process on it.
    fix: >-
      In relaunchForReprompt, keep the warning checkResume returns and, when it is non-empty, return it as an error before Prepare,
      so handleGatedBoundary ends the loop naming the gone strand and the unconfirmed holder.
      Add a TestGate_RepromptToGoneStrand row whose resumable fake returns a CheckResume warning and assert no AddStrand call and the warning in EndedEarly.
  - id: F7
    severity: MEDIUM
    class: scope
    origin: lens:code-scope
    location: internal/loomcli/arm.go:445
    summary: The board entry asks to reconcile the dead step's running shuttle runs on the re-step before the step starts, but the diff stops them only inside the stop that reports the interruption, so a failed stop or a halt reported before this change still leaves the leftover run for the re-step.
    fix: >-
      Before the until-stop loop spawns its first child for a run whose last stop was interrupted, stop the shuttle runs of the interrupted step's recorded trace id through the same stopStepRuns hook,
      and write the note into the loop log; read the trace id from the loop's delivered envelope or the interrupted stop's record that the re-step already reads.
      Add a loop test row where the stop at interrupt time failed and the re-step's hook call settles the run before the child starts.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/gate_test.go:536
    summary: TestGate_RepromptToGoneStrand covers only an untracked strand; no row reaches a dead pane or cleared binding that removes the old strand, a resumed provider that never becomes ready, or the resume's failures at holder lookup, checkResume, Prepare, AddStrand and the record save.
    fix: >-
      Add rows to TestGate_RepromptToGoneStrand, in its table:
      a tracked strand with a dead pane and one with a cleared pane binding, each asserting the resume and the old strand's removal;
      a resumed provider that never becomes ready, asserting the teardown's outcome comes back as the Wait result;
      and a failing SessionHolder, CheckResume, Prepare, AddStrand and record save, each asserting a failed verdict naming both failures and no strand left behind.
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shedverbs/loop.go:376
    summary: The panic recovery's new OnStepOrphaned call for an in-flight child has no test; the panic row in loop_test.go sets no hook and panics before a child is recorded.
    fix: >-
      Add a row to the panic case of TestLoop_InterruptedChildFillsTheErrorForm, or the test that holds the loop-panicked row,
      where the panic fires after a child and its trace id are recorded, with OnStepOrphaned set;
      assert the hook got that trace id and the cause reads loop-panicked: <value>; <note>.
  - id: F10
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/relaunch.go:246
    summary: When saveState fails, run.state keeps the new strand's GUID, name, session, Started and PromptLine after that strand is removed, so the finalize that follows records a strand that no longer exists.
    fix: >-
      In relaunchForReprompt, copy run.state before the assignments under recordMu and restore that copy when saveState fails, before removing the new strand.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/shuttleengine/relaunch.go:232
    summary: The resume's Prepare overwrites the run directory's prompt.md, which RunState.PromptPath names, with the re-prompt text, so the run's original prompt is lost on success and on every failure after Prepare.
    fix: >-
      Before calling Prepare in relaunchForReprompt, rename the run's prompt.md to a kept name such as prompt.original.md when no such file exists yet,
      or document in relaunchForReprompt's doc comment and doc.go's gate paragraph that a resume replaces prompt.md with the re-prompt.
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/shuttleengine/stepstop.go:93
    summary: A tied run whose output files all exist keeps reading running after stopRecorded, yet StopStepRuns lists it under Stopped, so the interrupted cause claims a stop that did not happen.
    fix: >-
      In StopStepRuns, reload or have stopRecorded report the outcome it left, and list a run whose record still reads running under a new StepStopReport field such as Finished instead of Stopped;
      render that field in loomcli's stepRunsNote and add a finished-files row to TestRunner_StopStepRuns_StopsOnlyTheStepsRuns.
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: _lym/landing/summary.md:18
    summary: The change description says the package has always refused to respawn over an untracked strand, but Attach's leftoverThenAgeVerdict makes an untracked run respawn-eligible once its directory is older than minAge.
    fix: >-
      Reword that sentence in _lym/landing/summary.md to say Attach refuses to respawn over a young untracked strand and allows it past minAge, and that the holder check is what rules out a live writer here.
  - id: F14
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomcli/arm.go:445
    summary: No test pins that specFor wires OnStepOrphaned; deleting the line passes every test.
    fix: >-
      In the loomcli test that checks specFor's loop spec, assert spec.Loop.OnStepOrphaned is non-nil and that calling it reaches the receiver's stop function with the steps directory and the given trace id.
  - id: F15
    severity: NIT
    class: design
    origin: lens:code-tests
    location: internal/shedverbs/loop.go:609
    summary: The OnStepOrphaned contract that an empty note adds nothing to the cause has no row; every fake hook returns a non-empty note.
    fix: >-
      Add a row to TestUntilStop_DeadLoopStopsLoopExited or TestLoop_InterruptedChildFillsTheErrorForm whose hook returns an empty note and assert the cause carries no trailing separator.
---

### [BLOCKING] The resumed strand's startup is satisfied by the first session's events

**Location:** internal/shuttleengine/relaunch.go:262
**Issue:** `relaunchForReprompt` keeps the run's events file and calls `awaitStartup`, which calls `checkLivenessTick` with `eventsProof` set.
`turnStartedAfterSessionStart` reads from the file's start (`wait.go:1689`), and a gated run at its boundary always holds the first session's session-start and turn-start lines.
So the new strand is marked started on the first tick (`wait.go:1516`), with no pane capture, no startup-gate dismissal and no readiness wait.
The `checkLivenessTick` doc (`wait.go:1469-1473`) states that only a fresh events file feeds `eventsProof`, which the resume now breaks.
The test seeds only `STOP:turn1`, so it never reaches this.
Found by the correctness and affected-set lenses, the assistant and the handler.

### [BLOCKING] Clearing Skills changes the resumed session's plugins and passages

**Location:** internal/shuttleengine/relaunch.go:45
**Issue:** Claude's `Prepare` builds the session scope from `spec.Skills` (`claudeengine.go:243`, `scope.go` `resolveSessionScope`), which disables every installed plugin whose namespace no preloaded skill names.
It also builds the subagent passage from the skills (`subagents.go:63`).
With `Skills` nil, the resumed writer's settings disable its plugins and drop that passage, though the original session ran with both.
The skills' text loaded in the transcript does not restore what the rewritten settings take away.

### [BLOCKING] The kept launch manifest misses the resumed launch

**Location:** internal/shuttleengine/relaunch.go:69
**Issue:** `PATTERN-engine-parity` says a launch keeps a copy of its manifest at `manifests/<run id>.json` once the run record is saved.
`Runner.launch` calls `copyManifest` after `saveRunState` (`run.go:743`).
The resume's `Prepare` rewrites `manifest.json` in the run directory, but `relaunchForReprompt` never refreshes the copy, so it describes the original launch's settings and passages.

### [BLOCKING] The old strand is removed before the record stops naming it

**Location:** internal/shuttleengine/relaunch.go:96
**Issue:** `PATTERN-shuttle-provider-seam` routes every removal of a run's strand through the stop verb's record-first order and leaves calls inside `shuttleengine` to review.
`removeGoneStrand` removes a tracked strand while `run.json` still reads running and names it.
The record names the new strand only after `AddStrand` and `saveState` succeed.
A kill or an `AddStrand` failure in between leaves a running record naming a removed strand: the state the board entry's second part is about.

### [MEDIUM] Settling an untracked run skips Attach's duplicate-agent age guard

**Location:** internal/shuttleengine/stepstop.go:91
**Issue:** `StopStepRuns` now records stopped for a tied run whose strand reed does not track or holds no pane binding for.
Its stated reason is that no process waits on it, but the agent runs in a tmux pane outside the killed step tree, and a cleared binding is the case where reed says the agent may still be working (`run.go` `liveStrand`).
`leftoverThenAgeVerdict` (`attach.go:597-609`) treats a terminal outcome as respawn-eligible at any age, while a running record younger than `minAge` returns `verdictError`.
The stop therefore lets the re-step respawn beside a writer that may still be alive.

### [MEDIUM] The resume discards an unconfirmed-holder warning

**Location:** internal/shuttleengine/relaunch.go:46
**Issue:** Claude's `sessionHolder` skips unreadable and undecodable registry entries (`claudeengine/resume.go` `sessionHolder`).
`checkResumable` then returns a warning that a live holder of the session could not be ruled out.
`checkResume` only logs that warning, and `relaunchForReprompt` goes on to launch.
The resume's own doc comment conditions it on no live process holding the session, which is then unproven.
The assistant filed this BLOCKING; it is held at MEDIUM because it needs a corrupt registry entry to occur.

### [MEDIUM] The leftover runs are not reconciled on the re-step

**Location:** internal/loomcli/arm.go:445
**Issue:** The board entry asks to reconcile a running shuttle run with a gone waiting process on the re-step, before the step starts.
The diff stops the step's runs once, inside the stop that reports the interruption (`runChild`, `deadLoopStop`, the panic recovery).
A stop that fails, a reed status error, or a halt reported before this change leaves the run for the re-step to trip on, and nothing retries it.
The summary's claim that the leftover is settled before any re-step holds only when that one stop succeeds.
The scope lens filed this LOW and the assistant MEDIUM; it is held at MEDIUM because it is the entry's explicit request.

### [MEDIUM] The resume's tracked-strand, startup-failure and failure paths are untested

**Location:** internal/shuttleengine/gate_test.go:536
**Issue:** Every row of `TestGate_RepromptToGoneStrand` starts from an untracked strand, so `removeGoneStrand`'s removal and the dead-pane and cleared-binding branches of `liveStrand` never reach the resume.
No row covers a resumed provider that never becomes ready, or a failure of `SessionHolder`, `checkResume`, `Prepare`, `AddStrand` or the record save.
Merges the tests lens's two MEDIUM items and its LOW failure-path item with the assistant's F8.

### [MEDIUM] The panic recovery's orphan hook is untested

**Location:** internal/shedverbs/loop.go:376
**Issue:** The panic row in `loop_test.go` sets no `OnStepOrphaned` and panics before a child is recorded, so removing the new block passes every test.
The tests lens filed this LOW and the assistant MEDIUM; a behaviour the diff adds with no coverage is held at MEDIUM.

### [LOW] A failed record save leaves the run state naming the removed strand

**Location:** internal/shuttleengine/relaunch.go:246
**Issue:** The assignments to `run.state` happen before `saveState`, and the failure branch removes the new strand without restoring them.
The finalize that ends the loop then writes a record naming that removed strand.

### [LOW] The resume overwrites the run's original prompt.md

**Location:** internal/shuttleengine/relaunch.go:232
**Issue:** Claude's `Prepare` writes `spec.Prompt` to `<runDir>/prompt.md` (`claudeengine.go:223`), which `RunState.PromptPath` names.
After a resume, or any failure after `Prepare`, the run directory holds the re-prompt in place of the original prompt.

### [LOW] StopStepRuns reports a run with finished files as stopped

**Location:** internal/shuttleengine/stepstop.go:93
**Issue:** `stopRecorded` leaves a run whose output files all exist reading running (`stopOutcome`), yet `StopStepRuns` lists it under `Stopped`, so the note that ends the interrupted cause claims a stop that left the record running.

### [LOW] The change description misstates Attach's untracked-strand rule

**Location:** _lym/landing/summary.md:18
**Issue:** The summary says the package has always refused to respawn over an untracked strand.
`dispositionCandidate` sends an untracked strand to `leftoverThenAgeVerdict`, which allows the respawn once the run directory is older than `minAge` (`attach.go:568-609`).

### [LOW] The loomcli wiring of OnStepOrphaned is untested

**Location:** internal/loomcli/arm.go:445
**Issue:** No loomcli test reads `spec.Loop.OnStepOrphaned`, so deleting the wiring line in `specFor` passes every test.

### [NIT] The empty-note skip has no row

**Location:** internal/shedverbs/loop.go:609
**Issue:** `LoopSpec.OnStepOrphaned`'s doc says an empty note is skipped, but every fake hook in the tests returns a non-empty note.

## Focus conclusions

- Gate re-prompt to an untracked strand: the holder check and a non-resuming engine end the loop with both failures named, and no strand is added (verified in `relaunchForReprompt` and the test rows).
  The resume itself is defective per F1 to F4 and F6.
- Step stop: the diff stops only runs of the given step and leaves runs of other steps and terminal runs alone (`stepstop.go`, `TestRunner_StopStepRuns_StopsOnlyTheStepsRuns`).
  F5 and F12 are its defects, and F7 is the re-step gap.
- Interrupted child and watchdog: a child exiting without an envelope fills the error form with the hook's note, and a watchdog-killed child never calls `OnStepOrphaned` (`runChild`, `TestLoop_WatchdogKillsAnIdleChild`).
- Second part, bullet 2: not a defect.
  The interrupted record reads the status file's `CurrentProducer`, which names the routed-to producer once `Code-Bouncer` bounces back, while the history's last entry is the row that completed.
  The summary's account matches the code; no change was needed.
- Second part, bullet 3: not a defect.
  A terminating signal leaves the loop's own mark, and the keeper records the exit, so a markless exit is a `SIGKILL` or machine stop that no process survives to record.
- Doc updates: the `shedverbs/doc.go` and `shuttleengine/doc.go` paragraphs match the code as the diff leaves it, except the `checkLivenessTick` doc in `wait.go`, which F1 covers.
