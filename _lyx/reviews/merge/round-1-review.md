---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/alignment/session.go:176
    summary: The alignment agent's attempt deadline is wall-clock and set before the start, so a pause, a start held by a pause, or a sleep times the attempt out
    fix: In internal/alignment/session.go make the attempt deadline measure awake unpaused time, the way shuttle's Wait does. Give SessionConfig the pause geometry and slug (or a seam returning pauserecord.Coverage) and a pauserecord.SleepWatch fed in Poll through pauserecord.MonotonicAwake. In Poll replace the check `!now.Before(current.Deadline)` with a comparison of pauserecord.AwakeSince(current.SpawnedAt, now, sleepWatch.Sleeps(), coverage.Intervals) against the attempt's timeout. Record the timeout on the attempt, or derive it from Deadline minus SpawnedAt. Set SpawnedAt after Shuttle.Start returns, or exclude the interval the start was held, so a start held at entry by a pause (decision D4) does not spend the deadline. Probe's Spec timeout must use the remaining awake time. Wire the geometry in internal/loomcli/landingdeps.go (landingAlignmentDeps) and internal/loomcli/wiring.go (newParentReviewAligner) from hubgeom.PauseGeometry. Add a case to the alignment session test for a Poll after a covered interval and after a detected sleep that reads pending.
  - id: F2
    severity: LOW
    class: consistency
    location: internal/landingshed/pralign.go:204
    summary: PR-Align's wait reads the parent-lock PauseRequested seam, which now fires on a --after naming PR-Align and so ends the alignment agent's wait mid-row, beyond the one mid-row halt decision D8 allows
    fix: In internal/landingshed/pralign.go delete the `p.deps.Lock.PauseRequested` block in `wait` (lines 204-213) and the pausedReason constant, and drop the matching case in internal/landingshed/pralign_test.go (near line 407), so a `lyx loom pause --after PR-Align` halts at the row's boundary as D8 states and `lyx pause <slug>` is the mid-row stop. If the mid-row end is wanted instead, give PRAlign its own told seam and state it in the D8 bound, in the refusal-spec row "parent lock wait paused" and in internal/landingshed/doc.go.
---

### [MEDIUM] Alignment attempt deadline ignores pause and sleep

**Location:** internal/alignment/session.go:176, internal/alignment/session.go:288
**Issue:** The parent's alignment `Session.Begin` sets `Deadline` to `now + Timeout` on the wall clock before `Shuttle.Start`, and `Poll` ends the attempt with `CauseTimeout` and stops the strand once `now` passes it.
The task's decision D4 holds a shuttle run start at entry while a pause covers the run, with its deadline starting when it starts, and D5 and D9 make every deadline exclude paused intervals and slept time.
The alignment attempt escapes both: a start held by a pause has already spent its deadline, and a pause or a night's sleep during the wait makes the next `Poll` time the attempt out, even though shuttle's own `Wait` for the same run, which excludes both, is still pending.
`Poll` checks the wall deadline before it falls back to the watched run, so the wall deadline wins.
The result is an alignment failure caused by a pause or a sleep, which is what the decision record's goal says a pause must never cause.
Both the PR-Align and the parent-review aligner use this session.

### [LOW] PR-Align reads a parent-lock pause seam whose meaning the task changed

**Location:** internal/landingshed/pralign.go:204
**Issue:** The parent's `PRAlign.wait` closes the attempt and ends with a note when `Deps.Lock.PauseRequested` reports true, which was the operator's bare `lyx loom pause`.
The task redefined that seam (internal/loomcli/landingdeps.go `parentLockDeps`, internal/landingshed/deps.go) to report a `--after` naming the row now waiting in the parent lock's queue, and the bare pause is retired.
Because PR-Align is its own current producer, `lyx loom pause --after PR-Align` now ends the alignment agent's wait mid-row.
Decision D8 states that the only mid-row halt left for a loom run is a parent lock wait ended by `--after`, and the seam's doc describes only the lock queue.

## Rejected

- Wiring seams: every `shuttleengine.NewRunner` path in the wiring files the merge touched calls `SetPause`, `landingDeps` fills `Alignment` from the same runner, and `SetStepTie` is applied after it, so the drift guards still cover both sides' seams.
- Batten notice log: `appendNoticeLog` runs once at the first composition in `deliver`, and the task's covered path returns before any composition, so a covered state-changed notice still logs once.
- Webster: `BeginBatch` runs its remaining gates as `doc.go` states, no reference to the removed pause flag or `ErrPaused` is left in webster, and `state.go` has no pause field to read.
- `PATTERN-daemon-wakeups`: the task's pause hold in `internal/pauserecord/hold.go` is listed in both the pattern and the scan and backs off from one to five seconds; the parent's statuspane loop reads files only and needs no pause or sleep handling.
- Refusal spec, orch role, `claims_test.go`, `docs/overview.md` and the `doc.go` files of battenshed, orchengine, reedengine and websterengine describe the merged behavior without contradiction.
