---
dispositions:
  - id: F17
    disposition: fixed-differently
    reason: The premise holds, but the proposed justification and a single return shape do not fit the code. Wait's liveness read gives OutcomeDied for a removed strand only on a handle Run.Stop marked (wait.go:1489-1492); a StopStrand stop gets errStrandNotTracked. Writing nothing after a handle Run.Stop would also report no session end, since Run.Stop reports none (stop.go:43-57) and only finalize does (wait.go:2038). Card 14 now splits the return by the stop's form and drops the liveness-read justification.
---

## Changes

All changes are in `_lym/plan/14-in-run-relaunch.md`.

- **F17 (fixed differently).**
  - **Load-turn paragraph.** The sentence that returned "the stop's outcome" is replaced. A stop-ended relaunch removes the strand and returns as the record-write paragraph states. On the start path only `StopStrand` can end it, because no caller holds the handle yet. That relaunch returns the run's `identity` with Outcome `OutcomeDied`, beside an error wrapping `ErrNotStarted`. It leaves `startupStrandRemoved` false, so `startRetryable` never launches a stopped run again.
  - **Step 1 of `relaunchUnderAccount`.** A terminal record now ends the relaunch "as a stop-ended relaunch" instead of with "the stop's outcome".
  - **Record-write paragraph.**
    - The relaunch removes the strand through `removeStrandIfPresent`, also when the stop came before the mark, and logs a failed removal at Warn.
    - After a handle `Run.Stop` (`markedStopped`), the relaunch finalizes `OutcomeDied`. `finalize` keeps the record stopped through `stopOutcome`, stamps the run end and reports the run's one session end.
    - After `StopStrand`, the relaunch finalizes nothing and returns `identity` with `OutcomeDied`, beside nil on the `Wait` path.
  - **Proposals not taken.** I did not say the result has "the shape `Wait`'s liveness read gives a strand a stop removed", because that holds only for a handle stop. I did not keep one write-nothing shape for every stop, because after a handle `Run.Stop` it would report no session end and stamp no run end.
  - **Rows.**
    - The Wait-path `StopStrand` stop-race row now asserts `Wait` returns identity with `OutcomeDied` beside nil, and that one session end (the stop's) is reported.
    - New row: a handle `Run.Stop` from the same hook, which gets a finalized `OutcomeDied` Result and one session end (the run's).
    - The before-the-mark row now asserts identity with `OutcomeDied`, the record stopped, no `EndStrandProcess` or `RelaunchStrand`, and no strand up.
    - New row: a skill-load turn driven through `start` with start retries set, whose `OnEndStrandProcess` hook runs `StopStrand`. It asserts no run, an error wrapping `ErrNotStarted`, identity with outcome died, the record stopped, no strand up, one session end and no second launch.
  - **Uses list.** Adds `Run.identity`, `Run.markedStopped` and `removeStrandIfPresent`.

## Self-check

- fix-consistency, LOW: the Wait-path `StopStrand` stop-race row did not assert the returned outcome. Fixed as proposed: the row now asserts identity with `OutcomeDied` beside nil.
- fix-claims, MEDIUM: "the shape `Wait`'s liveness read gives a strand a stop removed" holds only for a handle `Run.Stop`. Fixed as proposed: the justification is dropped, and the return is split by the stop's form.
- fix-claims, LOW: "leaves the stop to remove its own strand" fails when the stop's removal fails. Fixed as proposed: the relaunch now removes the strand through `removeStrandIfPresent` for a stop before the mark too.
- fix-proposal, MEDIUM: after a handle `Run.Stop`, writing nothing skips `finalize`, so no session end is reported and no run end is stamped. Fixed with own change: a handle stop now ends through `finalize(OutcomeDied)`. A `StopStrand` stop, which already reported its session, returns `identity` with `OutcomeDied`. A new row covers the handle stop. The fork suggested also stamping the run end on the `StopStrand` path; I left that out, because `StopStrand` stamps none for any run and the review asks for no change there.
- fix-proposal, LOW: the new start-path row pinned only the Outcome. Fixed as proposed: the row now asserts the identity fields and exactly one session end, so a relaunch that finalized again would fail it.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
