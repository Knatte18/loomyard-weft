---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md, Decisions 7 and 8
    summary: The daemon's and cancel's lift of the owned entry never says the sessions that pause interrupted get their carry-on line, so a bare entry removal leaves them stalled after the wake
    fix: In Decision 7, state that the lift by identity runs tender's resume sequence for that one entry, giving each non-orch session the entry recorded one `pauserecord.ResumeLine` through the guarded send and leaving the orch to its watcher, as `lyx resume` does; state in Decision 8 that cancel's lift is the same one; add an acceptance criterion that after the wake every session the pause interrupted is given its resume line.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md, Decision 6 and the 120 re-reads acceptance criterion
    summary: The criterion says 120 unconfirmed re-reads lift the owned entry and notify the orch, but Decision 6 states only that they end the request failed
    fix: In Decision 6's closing Bound sentence, after 120 re-reads without confirmation end the request failed, add that the daemon lifts its owned entry as on the wake paths (Decision 7) and queues one orch notice naming the failure.
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md, Acceptance criteria, the sleep-deny criterion
    summary: The sleep-deny criterion pins three bare verbs only, leaving Decision 10's stated bound untested, namely hybrid-sleep, suspend-then-hibernate, a flag word before the verb, a path prefix and the sudo or env prefix
    fix: Extend the sleep-deny criterion so it reads that each of the five verbs is denied, as are `systemctl --no-wall hibernate`, `/usr/bin/systemctl hibernate` and `sudo systemctl suspend`, while `systemctl status` and `systemctl status hibernate.target` pass.
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md, Decisions 5 and 7 against Acceptance criteria
    summary: Two stated paths have no criterion, a slug halted before Land leaving the request waiting with status naming why (D5), and the lift by identity lifting nothing when the live hub entry differs or was already lifted while the request still ends done (D7)
    fix: Add two acceptance criteria, one saying that an `--after <slug>` whose run halts before Land stays waiting and `status` reports the halt as its reason, and one saying that when the owned entry was resumed and another hub entry added before the wake, the daemon lifts nothing, leaves the other entry live and ends `done`.
---

### [MEDIUM] The wake and cancel lifts do not say the interrupted sessions are resumed

**Location:** `_lyx/discussion/decision-record.md`, Decisions 7 and 8.

**Issue:** In tender's own vocabulary a lift is separate from the line that gets a session going again.
`internal/tenderengine/doc.go` reads "A resume lifts entries and gives each session it interrupted one line to carry on", and `ResumeRequest` does both.
Decision 7 adds "a lift of one entry by scope, caller and paused-at time" and only ever says the daemon "lifts its owned entry".
Decision 8's cancel and the failure paths also only lift.
The pause the daemon runs Esc-interrupts every covered session in a running turn and records it in the entry.
If the new lift only removes the entry, every interrupted session sits idle after the wake, so a hibernate leaves the hub's runs stalled.
That is the outcome the support log rejects with "a lingering pause only stalls runs".
The plan writer never reads the support log, and the Notes call `ResumeRequest` a lift by slug, which reads as removal only.
The decision record has to state that the lift carries the resume lines.

### [LOW] Decision 6 leaves out the lift and notice on a confirmation failure

**Location:** `_lyx/discussion/decision-record.md`, Decision 6 and the "120 re-reads without confirmation" acceptance criterion.

**Issue:** The criterion says those 120 re-reads "end `failed`, lift the owned entry and notify the orch".
Decision 6 says only "120 re-reads without confirmation end the request `failed`".
Decision 7 states the lift and the notice for the two wake failures alone.
The lift and notice on this path appear only in a criterion, so the criterion asserts behavior that no Decision states.

### [LOW] The sleep-deny criterion does not pin the deny's stated bound

**Location:** `_lyx/discussion/decision-record.md`, Acceptance criteria, the "A spawned session's Bash call of `systemctl hibernate`" bullet.

**Issue:** Decision 10's bound makes three matching claims.
The deny matches in command position through the shared prefix in `internal/shuttleengine/claudeengine/settings.go` (`commandPositionPattern`), it allows flag words between `systemctl` and the verb, and it covers five verbs.
The criterion names `hibernate`, `suspend` and `sleep` only.
It leaves untested `hybrid-sleep` and `suspend-then-hibernate`, the `--no-wall` flag-word case that Decision 10 itself cites, and a path or `sudo` prefix.
The verbs are real: `systemctl --help` on this machine lists `sleep`, `suspend`, `hibernate`, `hybrid-sleep` and `suspend-then-hibernate`, and `loginctl --help` lists none, which agrees with the round-1 fix.
The bound's shape follows the existing denies' comments in `settings.go`, which also state what passes.

### [LOW] Two stated paths have no acceptance criterion

**Location:** `_lyx/discussion/decision-record.md`, Decision 5 ("A slug whose batten run halts before Land leaves the request waiting, and `status` says why") and Decision 7 (the lift "lifts nothing when the live entry of that scope differs", "An owned entry already lifted by then is not an error").

**Issue:** The criteria cover a run before Land and a teardown halt after Land.
No criterion covers a run that halts before Land.
They also cover the lift only on the happy path, where it "lifts only its own entry".
No criterion covers the identity check refusing a different live entry.
That case is reachable, because confirmation accepts a live hub entry "owned or not": an operator who resumes and re-pauses gets the hibernate under their own entry, and the daemon must leave that entry alone on the wake.

## Rejected

- Focus 1, Decision 6's state set: not a defect. `tenderengine.Status` (`internal/tenderengine/resume.go`, `sessionStatuses`) reports `string(state.Name)` from `shuttleengine.SessionStateName`, whose values are `busy`, `idle-done`, `idle-stalled`, `idle-fresh`, `asking`, `dead` and `unknown`, with `unknown` for a failed reading. Decision 6 accepts five, explicitly withholds `busy` and `unknown`, and withholds anything else, the zero state of a vanished strand included, by its "every session reads one of" form. The orch is a shuttle run in the prime, so status lists it. Both confirmation and the pre-hibernate check require a live hub-scope entry "owned or not", which covers the no-owned-entry case of Decision 7.
- Focus 1, the caller label: the caller label `hibernate` is consistent with the orch watcher. `internal/orchengine/pause.go`'s `settledWithoutInterrupt` skips the interrupt only for an entry whose caller parses as the orch, so the orch is interrupted by its watcher as Decision 6 says.
- Focus 3, `watch`: Decision 3 gives it a caller check, Decision 5 states its no-request and beside-a-live-daemon behavior, and two criteria pin both.
- Focus 4, liveness: Decision 4 names the test as pid plus start time, the request file records both, and `request`, `cancel` and `status` all use that test.
- Decision 10, false denies: the existing deny comments also list what a deny falsely denies, which Decision 10 omits. The attack-surface rule asks only what a guard lets through, so this is not raised.
