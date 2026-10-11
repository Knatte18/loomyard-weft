---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D6 (re-launch state check), D3 (arbiter goto), Open risks second bullet
    summary: The arbiter's own goto clears an orch or operator pause set during a live arbiter, so D6's state check passes and the re-launch undoes that pause, contrary to D6's claim that a resume verdict never un-pauses a stop a human set.
    fix: In D3, remove `lyx loom goto` from the arbiter's mutating verbs and let a `resume` verdict name a goto target instead. In D6, have the Go re-launch perform that goto itself under the status lock, as part of consuming the attempt, and refuse it (taking the hand-on path) when `pause_requested` is set or `pause_before` or `pause_after` differs from the value the verb recorded in the attempt at spawn. Then the state check runs before anything clears a human pause. Rewrite D6's blocked-stop precondition from "a goto recorded since the stop" to "the verdict names a goto target the re-launch admits". Restate D2's and D3's bounds for the moved goto. Add an acceptance criterion that an orch `pause` (plain, `--before` or `--after`) during a live arbiter whose verdict moves the run back is handed on, and neither the goto nor the re-launch happens. Reword the Open risks bullet to match.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D6 (resume bullet, un-pause clause); support-log.md Question ledger (goto-paused entry); decision-record.md Notes for the plan writer (second bullet)
    summary: D6's clause that the re-launch performs the same un-pause lyx loom resume does when a bare re-launch would not step the run rests on a false premise. lyx loom resume writes no status (it wakes the parked driver), and a shed step re-calls current_producer from paused and awaiting alike.
    fix: In D6, delete "and over an `awaiting` or `paused` state performs the same un-pause `lyx loom resume` does when a bare re-launch would not step it". Replace it with "a re-launch steps an `awaiting` or `paused` run as it stands, since a step short-circuits only a `done` run and re-calls `current_producer`". In support-log.md's Question ledger, mark the goto-paused question resolved by the code. In Notes for the plan writer, drop the sentence asking the Plan to check whether a bare re-launch steps a goto-paused run.
  - id: F3
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D1 (applied mark), D7 (same-stop rule)
    summary: The applied mark is written before the run steps, so it records that an attempt was consumed, not that the run moved. A re-launch that ends before stepping (a busy error, or driver death right after the mark) leaves the stop key unchanged, and D7 then hands on a settled stop as if it had recurred.
    fix: In D7, after the sentence about the applied mark, add that an applied attempt whose stop key is unchanged is handed on whether the stop recurred or the re-launch never stepped. Say that the refusal's reason states this as "an applied resume left the stop key unchanged" and never claims recurrence. Add that this costs the parent one wake and never a second re-launch.
  - id: F4
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md D2 (sentence beginning "Go checks what the status file and envelope carry")
    summary: D2's list of the inputs Go checks for eligibility omits the Publish row's verify-failure record, which D2's own exclusion of the target-naming Publish halt reads.
    fix: In D2, add "the Publish row's verify-failure record" to the list after "the escalation and decision files".
---

### [BLOCKING] The arbiter's goto erases a human pause before D6's check can see it

**Location:** `_lyx/discussion/decision-record.md` D6, D3 and Open risks.

**Issue:** D6 admits the re-launch when `pause_requested` is unset and the run's history is the attempt's plus one `goto` entry, with the run `paused`.
It then asserts "a `resume` verdict never un-pauses a stop a human set", and Open risks repeats that an orch `pause` "is not overridden".
`internal/shedengine/goto.go` (the `UpdateJSON` mutate in `Goto`) sets `cur.PauseRequested = false` and clears `PauseBefore` and `PauseAfter` on every goto.
`lyx loom pause` with no flag only sets `pause_requested` on a halted run (`internal/shedverbs/pause.go`), and leaves state and history alone.
Take this interleaving at a `blocked` stop: the arbiter is running, the orch runs `lyx loom pause`, and then the arbiter runs `lyx loom goto --to X`.
The goto clears the orch's `pause_requested`, and the run is left `paused` with history equal to the attempt's plus one goto to `X`.
That is exactly the state D6 admits, so the re-launch steps the run and the human pause is lost with no trace.
The same applies to an orch `pause --before` or `--after` set during the arbiter, and to an arbiter goto at an error stop (D3).
The acceptance criterion for an orch `pause` during a live arbiter passes only for the ordering in which the arbiter runs no goto.
After the fact, the re-launch cannot tell whether a pause request existed, so no added state comparison in D6 can close this.
The check has to run before the goto clears the fields, which is what the fix does by moving the goto into the Go re-launch.

### [MEDIUM] D6's un-pause clause names a mechanism that does not exist

**Location:** `_lyx/discussion/decision-record.md` D6 resume bullet, with the matching Question ledger entry and Notes bullet.

**Issue:** `internal/loomcli/resume.go`'s `resumeFromRunState` never writes the status file.
It refuses or wakes the live parked driver with the resume line.
`stepLocked` in `internal/shedengine/run.go` short-circuits only `StateDone`.
`blocked`, `failed` and `awaiting` deliberately fall through and re-call `current_producer`, and `paused` does too.
Step 3 pauses again only on `pause_requested` or a `pause_before` naming the row.
The loop verb (`internal/shedverbs/loop.go`) has no pre-step refusal of a `paused` or `awaiting` state.
So a bare re-launch already steps every state D6 admits, given `pause_requested` unset.
The clause asks the Plan to copy an un-pause that `lyx loom resume` does not perform, and could lead to a new status write.
The ledger leaves open a question the code answers.

### [LOW] An applied mark cannot tell a recurred stop from a re-launch that never stepped

**Location:** `_lyx/discussion/decision-record.md` D1 and D7.

**Issue:** D6 writes the applied mark before the run steps, which correctly prevents a second application.
D7 then reads "applied and the stop key unchanged" as a recurrence.
A busy refusal of the re-launched loop leaves the key unchanged without the producer ever being re-called.
So does a driver that dies between the mark and the step.
The outcome is fail-safe, since the stop is handed on to the parent.
But the arbiter's settlement is lost, and the reason the parent reads misdescribes the cause.

### [NIT] D2's list of Go inputs omits the verify-failure record

**Location:** `_lyx/discussion/decision-record.md` D2.

**Issue:** D2 lists what Go checks: "the state, the row, the escalation and decision files, `kind`, `transient` and `interrupt_policy`".
The Publish-halt exclusion in Scope and D2 depends on whether the verify-failure record names a target.
That is a file outside the status file and envelope, and the list leaves it out.

## Focus verdicts

- Error stops (D1, D6, D7): `appendHistory` in `internal/shedengine/run.go` returns an unchanged copy when `outcome` is empty, so a recurring producer or bootstrap error leaves `current_producer`, history length and `failed` unchanged.
  D1 names the stop key once, and D6 and D7 refer back to it.
  The applied mark separates an unapplied `resume` (returned on a D6 wake) from an applied one (refused and handed on).
  The remaining gap is F3.
  Budget-exempt Stuck folding (`appendOrFold`) cannot keep the key across a stop, because folding needs two consecutive entries from the same producer, and no admitted path produces that between two arbiter stops.
- Un-pause (D6): not bounded to the state the arbiter left, as F1 shows.
  The un-pause step itself is a false premise (F2).
  The Open risks bullet does describe what D6 bounds, but its claim of "not overridden" falls with F1.
- Attack surface of D2 and D3: not a defect.
  D2 states what an error stop without the fork skips (the fork's diagnosis and the repair cap).
  It bounds that by the budget, the applied-attempt refusal and the absence of repair verbs.
  An arbiter goto at an error stop changes the stop key and so escapes the applied-attempt refusal, but the budget still bounds it to one re-launch per slot, as D2 says.
  The transient half is Go-visible: `internal/shedverbs/loop.go` re-steps a transient failure once, so a transient `kind` reaching the driver has failed twice.
- Watched marker (D2): not a defect.
  D2 refuses the target-naming Publish halt in every run and never reads the marker, so a missing marker after a failed `MarkWatched` admits nothing.
  The exclusion is broader than batten's retry condition, which also requires the child's HEAD and a record written during the halting attempt (`internal/battenshed/doc.go`).
  It is broader in the safe direction.
