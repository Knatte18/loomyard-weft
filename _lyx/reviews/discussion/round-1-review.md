---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: decision
    location: _lyx/discussion/decision-record.md D1, D6, D7
    summary: At an error stop D1's attempt key and D7's same-stop rule match the same case, so the verb cannot tell an unapplied verdict (D6 wake) from a resume that already ran and recurred, and either a recurring error stop re-launches forever on a stale resume or the D6 recovery is refused.
    fix: In D1, key an attempt to the stop and record when its verdict is applied, for example by having the driver's re-launch after a resume (or the verb, before it returns resume) write an applied mark into the attempt directory. State in D1 that a re-run finding an unapplied attempt returns its verdict (D6), and in D7 that a re-run finding an applied resume attempt for the same key is refused and handed on. Use one key in both decisions (D1 says current_producer, history_length and state; D7 says row and history length) and name it once, in D1. Add an acceptance criterion that a producer error recurring at the same row after an arbiter resume is handed on without a second spawn and without a second re-launch.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D6, Open risks
    summary: D6's un-pause cannot tell a pause or goto the arbiter left from one the orch or operator set while the arbiter ran, so a resume verdict overrides a human's pause, and the Decision names no bound for it; Open risks only describes it.
    fix: In D6, bound the un-pause to state the arbiter's own records left, for example by applying it only when the run's last history entry is a goto the attempt recorded among the verbs it ran, or the run's state, row and history length equal what the arbiter left; otherwise the driver hands on with the verdict path. Replace the second Open risks bullet with that bound, and extend the acceptance criterion on an orch goto during a live arbiter to an orch pause during a live arbiter, which is handed on, not un-paused.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D2
    summary: D2 makes the repair fork's escalation a stencil-only bound but does not state what an error stop that reaches the verb without it can now skip, namely the fork's diagnosis and the repair cap, which D6 then lets an arbiter resume override.
    fix: Add a Bound line to D2 stating that a driver that runs the verb at a producer, bootstrap or reinvoke-interrupted error without a fork escalation skips the fork's diagnosis and the repair cap, and that this is bounded by the spawn budget, by the corrected same-stop rule of D7 (F1), and by the arbiter's lack of repair verbs (D3).
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D2
    summary: The watched-marker bound covers only a stale marker; a watched run whose marker is missing, which batten tolerates when MarkWatched fails and logs that the child keeps its own notifications, lets the arbiter in at the batten-owned Publish halt and race batten's own goto and resume there.
    fix: In D2, state the bound for a missing marker as well, for example that the verb refuses the Publish halt with a target-naming verify-failure record whenever it cannot positively read the run as unwatched, or state that a missing marker under a live batten admits the arbiter and that batten's retry then sees the arbiter's goto, and why that race is acceptable.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Scope
    summary: Scope's bullet for the orch role stencil's escalation and stop sections points at D14, but the record ends at D13, which is the orch-stencil decision.
    fix: In Scope, change (D14) to (D13).
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D7, D8
    summary: D7 refuses a negative arbiter_max_spawns naming the key, while D8 floors arbiter_timeout_min at 1 otherwise, which leaves a negative timeout silently floored or ambiguous.
    fix: In D8, state the negative case for arbiter_timeout_min explicitly, either refused naming the key as D7 does or floored to 1, and add it to the acceptance criterion on absent keys.
  - id: F7
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md Acceptance criteria
    summary: D10's hold ends when the marker's arbiter shuttle run is dead or the marker is absent, but the batten acceptance criterion covers only the deadline and a hand-on, so the dead-run and missing-marker paths that keep a lost marker from suppressing a notice are not pinned.
    fix: Extend the batten acceptance criterion to say that a marker whose arbiter shuttle run is dead, and a missing marker, hold nothing and the notice goes as today.
---

### [BLOCKING] D1's attempt key and D7's same-stop rule collide at an error stop

**Location:** `_lyx/discussion/decision-record.md` D1, D6, D7

**Issue:** D1 says a re-run of the verb for the same stop (keyed on `current_producer`, `history_length` and `state`) returns the finished attempt's verdict and spawns nothing; D6 relies on that for the unparked wake.
D7 says a stop that reaches the verb again unchanged after an arbiter `resume` (same row and history length) is refused and handed on.
At a Bouncer escalation or a `blocked` stop the two never meet, because the step after a resume appends history.
At an error stop they do: `internal/shedengine/run.go`'s `appendHistory` records nothing when a producer returns an error with no outcome, so a `producer` or `bootstrap` error that recurs after the arbiter judged a plain re-launch safe has the same producer, the same history length and the same `failed` state.
The verb then sees an attempt with that key and a `resume` verdict, and nothing on disk says whether the verdict was applied.
If D1 wins, the driver is served the stale `resume` and re-launches again, spending no budget slot, with no bound and no parent wake: the error-stop re-launch D6 allows over a spent repair cap becomes unbounded.
If D7 wins, the D6 wake after a driver death is refused, contradicting the acceptance criterion that such a driver applies the verdict without a second spawn.
D3's error-stop power, judging a plain re-launch safe, rests on this same rule for its bound, so the defect also leaves that power unbounded.
The two decisions also name different keys, one with `state` and one without.

### [MEDIUM] D6's un-pause overrides a pause the arbiter did not set

**Location:** `_lyx/discussion/decision-record.md` D6, Open risks

**Issue:** D6 has the driver perform `lyx loom resume`'s un-pause over "an `awaiting` or `paused` state the arbiter's records left", but the driver has no way to tell which party left the state.
`lyx loom goto` leaves the run paused, and `lyx loom pause` on a halted run is consumed into a `paused` state at the next boundary, so an orch or operator who pauses or moves the run while the arbiter runs can have that action undone by a `resume` verdict.
The second Open risks bullet describes this, but D6 bounds the un-pause only by the spawn budget and D7's same-stop rule, neither of which limits overriding a human's stop.

### [MEDIUM] D2's stencil-only bound states no consequence

**Location:** `_lyx/discussion/decision-record.md` D2

**Issue:** D2 says Go cannot see whether the repair fork escalated, so the driver stencil alone keeps an error stop from reaching the verb without one.
A Haiku driver that skips the fork can still run the verb at a `producer`, `bootstrap` or reinvoke `interrupted` error, and D6 then lets an arbiter `resume` re-launch over a spent repair cap with no recorded verb.
The rubric's attack-surface question asks the Decision to state what this lets through and what bounds it, and D2 states neither.

### [MEDIUM] A missing watched marker admits the arbiter at the batten-owned Publish halt

**Location:** `_lyx/discussion/decision-record.md` D2

**Issue:** D2 bounds a stale marker (it refuses the arbiter at the Publish halt and falls back to today's escalation), but not a missing one.
`internal/battenshed` logs and carries on when `MarkWatched` cannot write the marker, so a live batten can watch a run whose marker is absent.
The verb then reads the run as unwatched and admits the Publish halt, which is the race the support log's rejected alternative "Arbitrate the batten-owned Publish halt too" names as the reason for the exclusion.

### [LOW] Scope points at a missing D14

**Location:** `_lyx/discussion/decision-record.md` Scope

**Issue:** The Scope bullet for the orch role stencil names D14; the orch stencil decision is D13 and there is no D14.
Every other Scope and Decision cross-reference resolves.

### [LOW] Negative timeout handling differs from negative budget handling

**Location:** `_lyx/discussion/decision-record.md` D7, D8

**Issue:** D7 refuses a negative `arbiter_max_spawns` naming the key; D8 gives `arbiter_timeout_min` "floor `1` otherwise", which reads as flooring a negative value, so the two keys of one feature treat a negative value differently with no stated reason, and no criterion pins either reading for the timeout.

### [LOW] The batten hold's release paths are not all in the acceptance criteria

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria

**Issue:** D10 defines a live marker as one whose arbiter shuttle run is alive and whose deadline plus the report wait has not passed, so a dead shuttle run or an absent marker releases the hold.
The criterion covers the deadline and a hand-on only, which leaves the two paths that stop a lost marker from suppressing a needed notice untested by any listed behaviour.

## Rejected

- D7 and D8's zero values against `config-layers-run-settings`: the record cites it correctly (a present file lacking a key reads the Go zero value), the acceptance criteria match D7 and D8, and `hubreconcile.Ensure` walks every pair after a binary change, so the template default reaches existing pairs. Not a defect.
- The spawn count against a timeout or a failed spawn: D7 writes the record when the verb spawns and D8 says a timeout spends its slot, so both errors only shrink the budget and fall back to hand-on. Not a defect.
- D3's and D12's bounds on `circling`, `decision add --by arbiter` and `goto`: each is stated in the granting Decision, and `goto`'s preconditions match `internal/loomcli/cli.go`'s help text. The strand-name check is stated as unverified beyond the name, as `--by parent` is. Not a defect.
- Completeness-before-leanness over Scope, Constraints and Open risks: the support log's lens dispositions are each reflected in a Decision, Scope or a criterion, and no deliberation narrative of note remains in the record beyond what F2 moves out of Open risks.
