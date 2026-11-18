---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/relaunch.go:46
    summary: Automatic recovery discards a warning that a live session holder could not be ruled out.
    fix: >-
      Capture the warning returned by checkResume in relaunchForReprompt and refuse automatic recovery when it is nonempty.
      Return the warning as the recovery failure so handleGatedBoundary preserves it beside the original strand error.
      Do not call Prepare or AddStrand when holder absence is inconclusive.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/relaunch.go:45
    summary: Clearing Skills disables the resumed writer's plugins and removes its subagent skill instructions.
    fix: >-
      Preserve the original Skills when preparing a resumed session.
      Separate suppression of the initial skill-load turn from the skill metadata used for settings and standing instructions.
      Make the resumed launch deliver the gate prompt while retaining the original plugin scope and subagent skill passage.
      Correct the existing gate test assertion that currently requires Skills to be nil.
  - id: F3
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/relaunch.go:79
    summary: Historical session and turn events let the replacement strand skip its startup gates.
    fix: >-
      Record the events-file boundary before launching the replacement and restrict its startup evidence to events after that boundary.
      Apply that boundary to both session-start and turn-start evidence in the startup probe.
      Ensure the replacement reaches its own ready state and receives any required trust dismissal before the gate retry proceeds.
  - id: F4
    severity: BLOCKING
    class: consistency
    origin: lens:code-affected-set
    location: internal/shuttleengine/relaunch.go:69
    summary: The replacement launch leaves the retained launch manifest describing the original launch.
    fix: >-
      After saving the replacement strand's run record, call copyManifest with the existing run root, RunID and run directory.
      Preserve the original launch path's nonfatal logging policy for a failed copy.
      This must retain the manifest written by the replacement Prepare call as PATTERN-engine-parity requires.
  - id: F5
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/shuttleengine/relaunch.go:96
    summary: Retirement of the tracked old strand bypasses the stop-owned record-before-removal transition.
    fix: >-
      Move retirement of the old recorded strand into the stop-owned transition in stop.go and invoke that from recovery.
      Settle its persisted record before reed removal, preserving the completion-signal rule and the gate state needed for replacement.
      Persist the replacement as running only after successful registration; a failed replacement must leave a coherent old record.
  - id: F6
    severity: MEDIUM
    class: scope
    origin: lens:code-scope
    location: internal/loomcli/arm.go:445
    summary: A retry after an already-reported interruption still starts without reconciling the leftover shuttle records.
    fix: >-
      Add reconciliation before an interrupted step is restarted, including when its loop envelope was already delivered and its pid file retired.
      Identify the interrupted step from its persisted step identity and reconcile its running shuttle records only after proving their waiter gone.
      Preserve other steps and live waiters, and retain or retry cleanup failures before proceeding with the replacement step.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shedverbs/loop.go:376
    summary: No test exercises the new orphan-cleanup hook during panic recovery.
    fix: >-
      Extend the existing panic behavior coverage with a panic after a child and trace have been recorded.
      Set OnStepOrphaned and assert its trace argument and note in the interrupted cause.
      Keep a no-child case asserting that the hook is skipped; use the existing fake child and clock seams.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/gate_test.go:536
    summary: The recovery test omits tracked gone strands and the new boundary-failure behavior.
    fix: >-
      Extend TestGate_RepromptToGoneStrand with tracked dead and cleared-binding strands and failures of holder lookup,
      resume validation, preparation, old-strand retirement, registration, record persistence and replacement startup.
      Assert the verdict, attempts, preserved original send error, persisted record and surviving strands through the gate surface.
      Include inconclusive resume warnings, original skill configuration and historical startup events in this table.
---

### [BLOCKING] An inconclusive holder check can launch the session twice

**Location:** internal/shuttleengine/relaunch.go:46
**Issue:** A readable registry can contain an unreadable or malformed entry for a live session.
Claude's `sessionHolder` skips that entry (`claudeengine/resume.go:126`).
Its `checkResumable` returns a warning that a live holder could not be ruled out (`:88–94`).
Recovery discards that warning and launches anyway.
This violates the recovery path's stated condition that the original session has no live holder.

### [BLOCKING] Suppressing a load turn changes the session's settings

**Location:** internal/shuttleengine/relaunch.go:45
**Issue:** `Prepare` consumes `Skills` for more than the initial load message.
It rebuilds scope and subagent instructions in `claudeengine/claudeengine.go:243` and `:280`.
With the shipped session scope enabled, an empty skill list disables every installed plugin in `scope.go:86`.
It also removes the subagent skill-loading passage in `subagents.go:63`.
Previously loaded transcript text does not preserve plugin availability in the rewritten settings.
This consolidates the same defect reported by the correctness and affected-set lenses.

### [BLOCKING] The old process's events satisfy the new process's startup probe

**Location:** internal/shuttleengine/relaunch.go:79
**Issue:** Recovery keeps the events file and calls `awaitStartup` with startup-event proof enabled.
`checkLivenessTick` accepts a historical session-start followed by a historical turn-start (`wait.go:1516`).
`turnStartedAfterSessionStart` scans from the file's beginning (`:1689`).
Those events normally exist before the failed gate.
The new strand is therefore marked started without a pane capture or trust-prompt dismissal.
The outer wait already carries a started state, so later liveness ticks do not repair the missed startup handling.
A replacement held at a startup gate can stall until the run deadline.

### [BLOCKING] The retained manifest misses the replacement launch

**Location:** internal/shuttleengine/relaunch.go:69
**Issue:** `Prepare` rewrites the launch manifest, but recovery never refreshes its retained copy.
The original launch calls `copyManifest` after saving the run record (`run.go:743`).
`PATTERN-engine-parity.md` requires that copy for a launch once its run record is saved.
The retained manifest otherwise describes the original settings and instructions after the replacement has changed them.
The constraint makes this BLOCKING even though it does not itself prevent the producer running.

### [BLOCKING] Old-strand removal bypasses the stop contract

**Location:** internal/shuttleengine/relaunch.go:96
**Issue:** `removeGoneStrand` directly removes a strand still named by the persisted running record.
The replacement record is saved only after preparation, removal and registration complete.
`PATTERN-shuttle-provider-seam.md` requires run-strand removals to use the stop verb's record-first transition.
It explicitly leaves enforcement inside `shuttleengine` to review.
An interruption or registration failure after removal leaves the old running record naming the removed strand.

### [MEDIUM] Reconciliation does not run on every retry it must repair

**Location:** internal/loomcli/arm.go:445
**Issue:** The goal asks for reconciliation on a re-step after an interrupted stop, before starting the step.
The new hooks run while reporting an interruption.
`deadLoopStop` then delivers the envelope and retires the pid record (`loopwait.go:601`).
If the interruption was already reported by the old code, the retry has neither record nor envelope to trigger that hook.
The same gap remains after a new cleanup attempt fails: `stepRunsNote` renders the failure but preserves no obligation.
`runUntilStop` then reaches its spawn branch (`loopwait.go:108–118`) without reconciling the remaining runs.

### [MEDIUM] Panic cleanup has no behavioral assertion

**Location:** internal/shedverbs/loop.go:376
**Issue:** The existing unit panic row leaves `OnStepOrphaned` nil (`loop_test.go:466`).
The integration panic test also leaves it nil and checks process death rather than shuttle cleanup.
Removing the new hook block would leave both tests passing.
The missing assertion concerns a new recovery behavior, not an internal helper's implementation.

### [MEDIUM] Recovery failures and tracked-strand retirement lack coverage

**Location:** internal/shuttleengine/gate_test.go:536
**Issue:** Every new gate-resume row starts with an untracked strand.
The resume fake always passes validation and never fails holder lookup.
The table pins success, a live holder and a missing holder capability, but not the new retirement or failure outcomes.
Tests of the original launch do not exercise this replacement path's reused record and gate state.
The required additions belong in the existing behavior table.

## Focus conclusions

The interrupted-producer check does not require history's last producer to equal the current producer.
`shedengine` records the completed row and its routing target separately from `CurrentProducer`.
The interrupted paths read the current status, and existing tests cover a child moving that status before interruption.
The reported Code-Bouncer to Code-Burler-Review distinction is therefore not itself a defect.

A clean terminating signal is already marked by `recordTerminatingSignals`.
`keepLoop` also records the loop process's exit status.
The existing signal and keeper tests cover those paths, so no missing exit-marker change is raised.

The watchdog keeps its own cleanup hook; the new orphan hook is skipped for that kill.
The step-stop table covers tied dead and untracked strands and preserves unrelated steps and terminal records.
Completed-output protection intentionally retains its existing behavior.
The remaining retry gap is F6.

## Validation

Reviewed commits from `22a2c25b767617b80815edf3f0748f816ac96796` through `fea250c53aba2761c05b0eeca30049ac0234c89f`.
The board entry and decision record contain no added scope amendments or declared PATTERN changes.
No target file was changed.

Attempted `lym gate test --tier 0 ./internal/shuttleengine ./internal/shedverbs ./internal/loomcli`
in the foreground with a one-hour command timeout.
It failed before running tests because the hub's gate-holder and run-process directories are read-only in this session.
This environment limitation is not a finding against the diff, and no passing test run is claimed.
