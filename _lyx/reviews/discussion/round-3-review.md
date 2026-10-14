---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:136
    summary: D9's hand-over exemption cannot tell the hand-over's park marker from the successor's own, so a successor that hands back without an arbiter hand-on leaves the hold running to the backstop
    fix: In D9's end-condition bullet, replace the identifying clause with a test that singles out the hand-over's own marker, for example that a park marker does not end the hold while the newest settle record at this stop key is the `hand-over`, so a successor's later hand-back record (any `hand-back` settle record after the hand-over) lets its marker end the hold; then add an acceptance criterion that a watched blocked eligible stop whose successor driver's tackler hands back, with no arbiter attempt, gets its notice on the successor's park marker and not at the backstop.
  - id: F2
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:138
    summary: The stateChangeReady bullet rests on two false premises, that a hold always outlasts noticeReportWait and that the driver writes its stop report only after the verb's verdict, and its end condition names a stop report batten never reads
    fix: In D9, change the end condition at line 135 from a driver stop report or park marker to the driver's park marker, the one record batten's `StopReport` seam reads (`readStopReport` in `internal/battencli/wire.go`), and say that the Go driver's report files written by `startReport` before the tackler and the arbiter do not end the hold; in the stateChangeReady bullet, replace the clause after the measure with the plain reason that the report wait must start no earlier than the hold's end, dropping both premises; change acceptance criterion line 190's driver stop report to the driver's park marker.
  - id: F3
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:145
    summary: D9 measures the post-hold report wait from the hold's end but names no record that time comes from, so a batten restart between the hold's end and the notice either loses it or falls back to the episode's since and sends at once
    fix: In D9's stateChangeReady bullet, name the source of the hold's end for each end condition - the park marker's time, the hand-on attempt's verdict file time (the attempt record carries no hand-on time), the backstop's computed time, and, where no record carries one (an applied resume or the arbiter switched off), the time batten first observed it, with a restart taking its own start time; extend the Bound's restart sentence and the restart criterion at line 190 to cover the end as well as the start.
---

### [MEDIUM] The hand-over exemption also swallows the successor's own park marker

**Location:** `_lyx/discussion/decision-record.md:136`

**Issue:** D9 exempts "a stop report or park marker left by a Go driver's hand-over, one with a settle record of action `hand-over` at this stop key later than the stop".
That record stays in the settle file for the rest of the episode, so the successor's own park marker also has a `hand-over` record at this stop key later than the stop.
In `internal/loomcli/drive.go`, `handOver` appends the `hand-over` record and then calls `handBack`, which writes the report and park marker.
A successor driver at the same blocked stop sees `HandedOver` true, so `decideDrive` (`internal/loomcli/driverules.go`) routes it to the tackler, not to the arbiter.
A tackler that hands the stop back ends in `handBackAs`, which writes the same `driver-parked` marker at a later time.
Batten reads that marker through `readStopReport` as the path and the write time only, and both markers' times are later than the hand-over record, so the decided test cannot separate them.
Read as written, the successor's tackler hand-back leaves the hold on until the backstop: the parent waits up to `arbiter_timeout_min` plus the report wait for a stop no arbiter will settle.
The criterion at the hand-over case pins only the path where the successor's arbiter hands on, so it never exposes this.
`handBackAs` writes a `hand-back` settle record while the hand-over's own `handBack` call writes none, so a test keyed on the newest settle record is available.
The identifying clause is the one round 2's F3 fix suggested; this finding sharpens it rather than reopening the decision that the hand-over's marker does not end the hold.

### [LOW] The report-wait bullet's premises are false, and batten sees no stop report but the park marker

**Location:** `_lyx/discussion/decision-record.md:138`

**Issue:** "A hold always outlasts `noticeReportWait`" is false: the hold ends at once on a hand-on, which can come a minute after the stop.
Measuring from the hold's end is still right, but the stated reason is not.
"The driver writes its stop report only after the verb's verdict" holds for the LLM driver at awaiting and blocked stops, but not for the Go driver.
`goDriver.tackle` and `goDriver.arbitrate` both call `startReport`, which writes a stop report file before the tackler or the arbiter runs.
Batten never sees those files: `StopReport` in `internal/battencli/wire.go` reads only the park marker and its time.
D9's end condition, "the driver writes a stop report or park marker" (line 135), and the criterion at line 190 name the stop report as a separate signal.
A plan writer who adds a reader for report files to honour that wording ends the hold the moment the Go driver starts its tackler or arbiter, which reopens the incident window D9 closes.

### [LOW] The hold's end time, which the report wait now starts from, has no named source

**Location:** `_lyx/discussion/decision-record.md:145`

**Issue:** D9 moves `stateChangeReady`'s report wait to start from the hold's end, and its Bound makes the start restart-safe by reading it from the status file and the attempt record.
It says nothing of where the end's time comes from.
`loomengine.ArbiterAttempt` carries `SpawnedAt` and `Deadline` but no hand-on time, and an applied resume or the arbiter switched off leaves no batten-readable time at all.
A batten restart after the hold ended and before the notice must re-derive the end.
If it falls back to the episode's since, `waited` is true at once and the notice goes with `report none yet`, the incident's own shape.
The decided rule settles the hand-on path's timing (focus 1) only while batten keeps running.

## Focus verdicts

- Focus 1, D9's hand-on timing: not a defect, apart from F3's restart gap.
  In `stateChangeReady`'s awaiting-with-parent-notice branch, measuring `waited` from the hold's end makes a hand-on with no report wait `noticeReportWait` from the hand-on.
  A park marker written after the hand-on qualifies, since it is not before the episode's since, and moves the branch to the `noticeRelayWait` and `undecided` case.
  The criterion for the hand-on path pins the same two outcomes.
- Focus 2, D9's hand-over end condition: the intent, that the hand-over's marker does not end the hold, is decided and a criterion covers the arbiter path; the identifying test is defective (F1).
- Focus 3, D4's `run_settings` refusal: not a defect.
  D4 reorders `PrepareRunSettings` to `ValidateLocal`, then `CheckValues`, then `Migrate`.
  After D2, `review_max_bounces` is category none, so `ValidateLocal` passes it, and darn's `CheckValues` refuses the 0 before the hook rewrites it to `skip_review`.
  The refusal's wording names `review_max_bounces` and the hub's `darn.yaml` `skip_review`, and the criterion matches.
  The reorder for the other modules with a `Migrate` hook (batcher, orch, shuttle) changes no outcome: orch and shuttle have no `CheckValues` and `ValidateLocal` checks keys, not values, and batcher's retired keys sit under its open `profiles` map.
- Focus 4, the single eligibility function: not a defect.
  D5 states one Go function answering the whole awaiting and blocked eligibility, carrying both the cause rule and D6's verify-halt rule, for the verb, the Go driver's `EscalationPending` fact and batten's hold, and D9 says `battenshed` holds no copy.
  Today's `arbiterEligibility` is a pure function over `arbiterStop` in `internal/loomcli`, and `battencli` already imports another CLI package (`orchcli`), so the wiring D9 names is reachable.
