---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed-differently
    reason: "The proposed key of PID and start does not dedupe a multi-slot hold: tryAcquireSlot stamps h.Started = time.Now() per slot (internal/gateslot/pool.go:322), so each slot's record has its own start. D6 instead defines a hold as one holder PID with every slot it holds, starting at the earliest Started among them, and keys the notice by PID and that start."
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
---

# Round 1 fixer-report — Discussion decision record

Every change is in `_lym/discussion/decision-record.md`.

## Changes

- **F1**: D4 now keeps the watch memory in a watch map of its own, keyed by slug, as `LandNotices` is, outside `recordOwed`'s reach.
  The entry is dropped only by the done-run clear, or by the watch once the run's batten status no longer exists and the run owes no line.
  D1 names `d.listedRuns` as the run list the watch reads.
  New acceptance bullet: a landed run posts its landed line once across several ticks before the clear.
- **F2**: D3's skip now rests on the watch's own copy of the land_notice reading, taken while the run is ahead of Land.
  Only a landed reading whose copy is on is skipped, and a cancelled reading is never skipped.
  The rationale and the landed acceptance bullet are rewritten to match, and there is a new bullet for a cancelled run with land_notice on.
- **F3**: D1 makes a never-read child part absent.
  D3 counts a child state as a stop only when it was read, and an absent child part is no stop.
  New acceptance bullet: a run before Run-Shed posts no line.
- **F4**: D3 states that a cancel which removes the run's records is reported by the cancel's own notice, not by the watch.
  The Goal's cancelled bullet now covers only a cancel that keeps its records.
- **F5**: D2's `stops` rule is now defined over D3's whole-reading stop.
  New acceptance bullets: blocked to failed posts one line, and a pause lifting over a blocked child posts one line.
- **F6**: D1 states that the watch runs only on ticks whose scheduling steps run, and what delays it.
- **F7**: D2 states that a run whose `run_watch` has never read is not judged until a read succeeds.
- **F8**: D1's rationale uses a blocked Land row as its example in place of the pause.
- **F9** (fixed differently): the reviewer proposed keying the notice by PID and start.
  That key would still give one notice per slot, since each slot's record is stamped with its own start (`internal/gateslot/pool.go:322`).
  D6 instead defines a hold as a PID with every slot it holds, starting at the earliest `Started`, and keys the notice by PID and that start.
  One notice names every slot.
  The acceptance bullets now cover the slot list and a multi-slot hold posting once.
- **F10**: D6 skips the check with one Warn per failure episode on a failed `Pool.Holders` read, and the acceptance bullet covers it.
- **F11**: D6 refuses a negative `gate_hold_factor` and a `gate_hold_min` below one, and states the one-notice-per-hold bound.
  New acceptance bullet for the refusal.
- **F12**: D6's rationale pins `nanotime1` in `src/runtime/sys_linux_amd64.s`, which calls clock_gettime with clock id 1, `CLOCK_MONOTONIC`.
- **F13**: Constraints now name `PATTERN-tolerant-records` for the state record only.
  D6 states that the holder record is YAML decoded without `KnownFields`, so an older record reads as no marker.

Changes from the self-check:

- D1 orders the watch after the landing notice step, so the watch's last copy before Land is the reading the landing notice decides with.
- D2 logs one Warn per failure episode on a failed `run_watch` read.
- D3 states that the cancel notice is tried once, and that a failed post leaves the orch with no lym line about that cancel.
- D3's rationale now says the landing notice drops its memory at the latest on the tick the run first derives as landed.
- D4 lists the gate-hold keys among the entry's fields, and gives every run an entry whatever its level.
  It drops an entry only once the run's batten status is gone, never because one tick's listing left the run out.
  Its rationale now cites the landing notice's map as the precedent.
- D6's `Pool.Holders` failure example now names an unreadable or unparsable held record or a failed lock probe, in place of a half-written record.

## Self-check

- fix-consistency, LOW, D4's field list omits the gate-hold key, and a run at level `off` has no stated entry: fixed as proposed.
- fix-consistency, LOW, D4's rationale cites the run notices as its precedent: fixed as proposed, now citing the landing notice.
- fix-way-forward, LOW, the cancel notice's single try has no stated bound: fixed as proposed.
- fix-way-forward, LOW, a never-read `run_watch` gives no signal: fixed as proposed, with one Warn per episode.
- fix-claims, MEDIUM, a half-written holder record cannot occur under the atomic write: fixed as proposed.
- fix-claims, LOW, "a tick before" overstates the timing: fixed as proposed.
- fix-proposal, MEDIUM, the copy can disagree with the landing notice when land_notice flips on the last tick before Land: fixed as proposed, with the watch ordered after the landing notice step.
- fix-proposal, LOW, one missed listing drops the watch entry: fixed as proposed, with the entry dropped only once the batten status is gone.

## Disputed

None.

## Deferred

None.

## Rulings

None.
