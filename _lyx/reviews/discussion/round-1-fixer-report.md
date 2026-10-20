# Fixer report — discussion round 1

Every finding's premise was checked against the code at HEAD (`internal/shedadapters/bouncer.go` `Call` and `atHardCap`, `internal/shedengine/run.go` `episodeStuckCount`, `internal/shedengine/reroute.go`, `internal/shedengine/goto.go` `Relaunch`, `internal/sheddriver/driverules.go`, `internal/sheddriver/drive.go`, `internal/shedverbs/goto.go`, `internal/burlerengine/cluster.go`, `internal/loomcli/bootstrap.go` and `run.go`, `internal/verifytree/failure.go`); all hold.

## Changes

All in `_lyx/discussion/decision-record.md` unless named otherwise.

- F1, fixed as proposed: D5 defines a live settle record (it names the run directory's current generation, and no round report past its settled round is on disk) and compares only a live one; a non-live record takes the round-by-round path, where a settled verdict is settled, never cleared. D5 adds the generation record. D2 and the Scope Out reroute bullet name the round N+1 report as what makes the record no longer live. New criteria: an amendment's second Call judges round 1 without re-archiving; the verify retry's and the reroute's round N+1 are judged.
- F2, fixed as proposed: D5's code identity excludes the `_lyx` durable tree; the Open risks bullet is removed; new criterion on status and carry-over commits.
- F3, fixed as proposed (exclusion option): D5's document identity excludes the carry-over blocks and the `Added after Discussion` entries; new criterion.
- F4, fixed as proposed: D6's amendment seed Stuck carries a `generation_start` mark that ends the episode; added to D1's enders and the tolerant-records constraint; the failed-Commit-then-writer-goto criterion asserts the count starts at 0.
- F5, fixed as proposed: D2 refuses on any live record, settled or interrupted, and its Bound states what the interrupted state admits; new criterion.
- F6, fixed differently: the proposal to clear the carried record's settled mark was not made. D2 reads liveness, so an amendment generation is not refused; new criterion.
- F7, fixed as proposed (one round past the allowance): D6 sets the hard cap and rewrites the Bound; new criterion.
- F8, fixed as proposed: D3's rule sits ahead of the hand-over rule; Bound and criterion updated.
- F9, fixed as proposed: D3's rule reads the status error for a status-built stop; criterion extended.
- F10, fixed differently: the proposal to key on the driver's unspent retry was not made. The driver writes its verify-retry record before its goto (`internal/sheddriver/drive.go:287-288`), so that key would refuse the retry itself. The bypass instead holds while the failure record's `written_at` is later than the live settle record's settle time, so it lasts until the generation settles again. Bound and criterion updated.
- F11, fixed as proposed (exclude option): Merge-Review takes no amendment, and a changed merge-in gets a full generation; new criterion.
- F12, fixed as proposed: D4 states the pause label's values and bound, that goto's `--by` stays `shed`-only, and that loom's bare pause stays refused; the criterion and the rationale are updated.
- F13, fixed as proposed in D4.
- F14, fixed as proposed: the tackler and the arbiter run the seam at validation; Scope In and a criterion updated.
- F15, fixed as proposed: D8 keeps the key and the `awaitRunLockHalted` arm, and rewrites only its comments.
- F16, fixed as proposed: D9 retries every sentinel case, and a resumed reviewer's mismatch falls through to a fresh attempt 1; the title, Goal, Scope and criterion are aligned.
- F17, F18, F19, F20 and F21, fixed as proposed.
- `_lyx/discussion/support-log.md`: Turn 2 and Q13 updated, Q14–Q16 added, and a Round 1 entry.

## Self-check

- fix-consistency LOW, D2's "one goto" disagrees with its Bound: fixed, reworded to "admits gotos in one case".
- fix-consistency LOW, Scope In missing the validation seam and `generation_start`: fixed, both added.
- fix-consistency LOW, D5 case 2 and D6's opening omit the Merge-Review exception: fixed.
- fix-consistency LOW, support-log Q13, Turn 2 and Review rounds stale: fixed.
- fix-claims MEDIUM, the soft-cap escalation is described wrongly: fixed as proposed; the D6 Bound and its criterion now describe the soft-cap rule as it is.
- fix-way-forward MEDIUM, the writer-row exit from an amendment's hard-cap escalation makes no progress: fixed, the claim is removed and the Bound leaves accept as the exit, as at any hard cap.
- fix-way-forward LOW, `generation_start` can be lost or repeated: fixed, carried by the successful seed's Stuck only, once per generation; new criterion.
- fix-way-forward LOW, D2's way forward says "starts an amendment" for Merge-Review: fixed, it now says "starts a new generation".
- fix-proposal MEDIUM, case 1's re-mark moves the write time past the failure record: fixed, the record keeps the generation's first settle time.
- fix-proposal LOW, the pause label accepts `shed`: fixed, the pause verbs refuse it.
- fix-proposal side note, an older binary's budget halt has no `budget_capped` entry: added to Open risks.

## Disputed

None.

## Deferred

None.
