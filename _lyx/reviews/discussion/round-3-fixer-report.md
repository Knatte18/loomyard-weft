# Fixer report — discussion, round 3

Every finding's premise held against the code and every finding was fixed.

## Changes

- F1 (BLOCKING), fixed as proposed.
  Premise checked: `fabricengine.ReconcileOrigin`'s branch moves (`reconcileRecordsBranch` through `ffWorktree` and `replayWeft`, `internal/fabricengine/originreconcilemoves.go`) test no marker and no move record.
  D10 gains a paragraph: the origin reconcile, hub-wide or run-scoped, applies the pull's marker test and the move-record check before moving a prime's or pair's weft, leaves it unmoved, reports it stopped with the way forward, and never fails its caller for it; one `fabricengine` function holds the comparison.
  D10 says the code branch's move goes ahead, since it changes no marker and can bring the warp the new anchor.
  D7 step 1's list of what a standing record stops, D7's closing paragraph on other machines, Scope, D5's list, the Open risks bullet and two Acceptance criteria name the origin reconcile.
  D10's rationale lists every path that can change a checked-out marker.
- F2 (LOW), fixed as proposed.
  D10's checkout refusal names a warp update by hand first when the current warp checkout does not track the target's anchor; the sentence justifying the `--set` cost moved to `support-log.md`'s Rejected alternatives; the checkout criterion names the warp update.
- F3 (LOW), fixed as proposed.
  D5's list names a replayed local commit's parent tree; D10's integration reads D5's value for a markerless parent tree.
- F4 (LOW), fixed differently.
  D7 step 1 has every holder of the move lock write its pid and start time into the move record, and the move-lock refusal reads it.
  Not made: the proposal's "the Windows live-dobby refusal read[s] the pid from it".
  That refusal names a live dobby's pid, not the verb's, and the review itself says its source is the run-process records ("The Windows live-dobby refusal does have a source, the run-process records").
- F5 (LOW), fixed as proposed.
  The `--set` criterion names the live dobby's pass and task gate and `EnsureLive`'s report.
- `support-log.md` gains a Round 3 entry.

## Self-check

- fix-consistency, MEDIUM, the origin reconcile left out D2's empty-marker refusal: fixed as proposed; it stops on an empty own marker, naming the file and way forward.
- fix-consistency, LOW, the empty-marker criterion did not name the origin reconcile: fixed as proposed.
- fix-consistency, LOW, D6's `Add` writes no pid: fixed as proposed; step 1's pid write names every holder of the move lock, D6's `Add` included.
- fix-consistency, LOW, the landed recovery's "keeping the record" and the refusal's missing starting case: fixed as proposed; the landed re-run keeps `A`, `B`, kind and `U` and rewrites the pid, and the refusal bullet names the starting case.
- fix-claims, MEDIUM, `lyx adopt --drop-local` resets a weft to origin's tip with no marker test, against the rationale (`internal/fabricengine/destroy.go:1855`, `internal/adoptcli/adopt.go:345`): fixed with my own change; the drop-local reset applies the same test and refuses before tagging or moving, naming the `--set` and the adopt again, and the rationale, Scope, D5's list and the empty-marker criterion name it.
- fix-proposal, MEDIUM, the checkout's warp update changes the branch being left without naming that cost: fixed as proposed; the refusal names the code commit's cost, and committing the anchor directory is given as the lighter route.
- fix-proposal, LOW, "covered" undefined in the criterion: fixed as proposed; the criterion uses step 1's wording.
- fix-proposal, LOW, the reconcile's code-side move against the pull's stop of both sides was unflagged: fixed as proposed; the sentence names the difference.
- fix-way-forward, MEDIUM, the prime's stop under `lyx orch start` names a `--set` that D7's prime refusals then refuse: fixed as proposed; a prime stop names ending the hub's reed sessions, the orch's included, and waiting out its batten runs, then the `--set`, then `lyx orch start`.
- fix-way-forward, LOW, the merge example's cost: merged with the fix-proposal MEDIUM above and fixed there.

## Disputed

None.

## Deferred

None.
