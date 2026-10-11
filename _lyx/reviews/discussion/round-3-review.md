---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:98-101
    summary: D6 leaves pause_requested clearable by the re-launch's goto and states a status-lock span that cannot hold across the step, so a plain pause landing between the state check and the goto is erased
    fix: "In D6's resume bullet, replace 'The re-launch, in Go and under the run's status lock, consumes the attempt: it refuses ..., then checks ..., writes the applied mark (D1), performs the verdict's goto when it names a target, and steps the run.' with wording that puts the state check, the applied mark and the goto in one status-file write under one hold of the status lock, and the step after that hold is released (the step's own persists take the status lock). In the same bullet, change 'it keeps `pause_before` and `pause_after` as they stand, so it removes no pause condition either' to 'it keeps `pause_requested`, `pause_before` and `pause_after` as they stand, so it removes no pause'. Add to the Acceptance criteria: a plain `pause` that lands after the arbiter's verdict and before the re-launch's step survives the re-launch and pauses the run."
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:119
    summary: D7's busy-refusal example needs the applied mark written before the loop takes the run lock, an order D6 never fixes
    fix: "In D6's resume bullet, state that the re-launch takes the run lock before it consumes the attempt, so a busy refusal leaves the attempt unapplied. In D7, replace 'as after a busy refusal of the re-launched loop or a driver that died after writing the mark' with 'as after a driver that died after writing the mark, or a step that failed before its first status write'. In the Acceptance criteria, replace 'such as after a busy refusal' in the applied-resume criterion with 'such as after the driver died between the applied mark and the step'."
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/support-log.md:58
    summary: The rejected recommend-only alternative is rejected partly because the driver would run a goto on another agent's say-so, which the chosen design now does
    fix: "In the support log's Rejected alternatives bullet 'The arbiter only recommends, the driver executes every verb', replace 'a Haiku driver would then run `circling accept` or a goto on another agent's say-so with no added check' with 'a Haiku driver would then run `circling accept` on another agent's say-so with no added check, whereas the one move the driver makes on a verdict, its re-launch's goto, is checked in Go against the attempt's recorded state (D6)'."
---

### [MEDIUM] The re-launch's goto can still erase a plain pause

**Location:** `_lyx/discussion/decision-record.md:98-101`

**Issue:** `internal/shedengine/goto.go` `Goto` writes `PauseRequested = false`, `PauseBefore = ""` and `PauseAfter = ""` in its persist.
D6 defines the re-launch's goto as "`lyx loom goto`'s backward move, except that it keeps `pause_before` and `pause_after`", so it still clears `pause_requested`.
That clearing is harmless only while the check and the goto are one atomic write, since the check requires `pause_requested` unset.
D6 says the whole consumption, the step included, runs "under the run's status lock", which cannot hold: the status lock is a gofrs/flock file lock (`internal/lock/lock.go`), taken per write by `state.UpdateJSON`, and the step's own `persistTransient` takes it again, so a single hold across the step would self-deadlock.
A plan writer must therefore choose where the hold ends, and the natural reading, a check write followed by a call to a `Goto` variant that does its own read and `UpdateJSON`, leaves a gap.
`lyx loom pause` with no flag (`internal/shedverbs/pause.go`) takes only the status lock and sets `pause_requested`, so a plain pause landing in that gap is cleared by the goto and the step proceeds over it.
That contradicts D6's "a `resume` verdict never un-pauses a stop a human set" and the Open risks claim that such an action is never overridden.
The acceptance criteria cover a pause "during a live arbiter" only, not one landing after the verdict.
No other path clears the fields before the check: the arbiter's `circling` verbs read the status file and never write it (`internal/loomcli/circling.go`), and `decision add` does not touch it.

### [LOW] D7's busy example presumes an ordering D6 does not fix

**Location:** `_lyx/discussion/decision-record.md:119`

**Issue:** D7 names "a busy refusal of the re-launched loop" as a case that leaves an applied attempt with an unchanged stop key.
That holds only if the re-launch writes the applied mark before it tries the run lock.
D6 orders the check, the mark, the goto and the step, but says nothing about the run lock.
If the re-launch takes the run lock first, which also keeps an orch `goto` (which holds the run lock, `goto.go`) from interleaving with the consumption, a busy refusal writes no mark and never reaches D7's refusal, and acceptance criterion "An applied `resume` whose re-launch never stepped, such as after a busy refusal" names a case that cannot occur.
The refusal's stated reason, "an applied resume left the stop key unchanged", matches every case that does reach it: a recurring error stop, a driver dead after the mark, and a step that fails before its first write.
D7's "This costs the parent one wake and never a second re-launch" states the bound on the parent's wakes the focus asks about.

### [LOW] A rejected alternative's rationale contradicts the chosen design

**Location:** `_lyx/discussion/support-log.md:58`

**Issue:** The recommend-only alternative is rejected because "a Haiku driver would then run `circling accept` or a goto on another agent's say-so with no added check".
Since the round-2 change, the driver's re-launch performs the arbiter's named goto (D3, D6), so half of that rationale now describes the chosen design.
The difference that still holds is D6's Go state check, which the rationale should name instead.

## Settled focus questions

- D3 and D6, pause erasure: the arbiter no longer runs `goto`, D6's check precedes the goto, and the recorded `pause_before` and `pause_after` comparison refuses any `--before`, `--after` or `--clear` set during the arbiter. The one remaining ordering is F1.
- D6's resume bullet, the Question ledger and Notes: no text still claims the re-launch performs an un-pause. `internal/loomcli/resume.go` `resumeFromRunState` writes no status file and only wakes the parked driver, and `stepLocked` in `internal/shedengine/run.go` short-circuits only `StateDone`, matching D6's "steps an `awaiting` or `paused` run as it stands" and the ledger entry. The Notes sentence on a pending circling decision matches `resume.go`'s file header. D6's "the run must move" also holds when a recorded `pause_before` names the goto target: the step then pauses at once, which honours the human's condition and changes the stop key.
- D1 and D7, the refusal's reason: settled under F2; the reason text is right, one example is not.
- D2's input list: not a defect. The list names every input the eligibility rules read: the state and row, the escalation and decision files for the Bouncer case, the Publish row's verify-failure record for the exclusion (batten's `ChildPublishFailure` in `internal/battenshed/deps.go` is the record that exists), and `kind`, `transient` and `interrupt_policy` for the error cases. The fork's escalation is correctly stated as outside both files.
