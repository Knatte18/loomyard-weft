# Round 2 fixer report — discussion

Every finding's premise held against the code; all thirteen are fixed in `_lyx/discussion/decision-record.md`, and the support log gains a round 2 entry.

## Changes

- F1 (BLOCKING), fixed differently in one part: D7 step 1 takes the parent lock of the worktree's branch through `internal/parentlock` as its own owner and holds it until step 8, and step 4 runs under the weft write lock (`.weft/weft.write.lock`); the acceptance criterion says the verb holds the lock.
  The proposal had the lock taken right after the record is written; the verb takes it only after the pass wait instead, because `parentlock.Acquire` waits with no time limit, so a pass in Finalize queued behind the verb would keep the pass in flight while the verb waits on it, a deadlock.
  Step 4's failure path and step 8 release the lock, and a killed run's lock is freed with its process.
- F2 (BLOCKING): the Not-landed recovery renames `B/.lyx` back first, or refuses when both exist, and its reset and deletion pathspecs exclude `.lyx`, which the recovery never resets or deletes; the recovery criterion says `.lyx` is kept.
- F3 (BLOCKING): step 1's wait covers every dobby step 2 will end, every dobby of the machine for the prime, and reads a pass in flight while its driver lock is held or its dobby has a live child other than a spared push or sync; step 2 notes the wait has left no pass among the descendants.
- F4 (MEDIUM): D7 states that `lyx adopt` refuses before its first step while a move record stands in the prime or a worktree it would resume runs in or add a pair under, with the way forward; the criterion lists it.
- F5 (MEDIUM): D5 states that `Add` from `lyx fabric add`, `lyx beam create` or adopt refuses a fork point with a standing move record, with the way forward; the criterion lists it.
- F6 (MEDIUM): the record holds `U`, written at step 4 under the weft write lock, and an integration has landed when `HEAD`'s committed marker is `B` and `U` is an ancestor of `HEAD`; a weft pull and a replaying weft push refuse while the record stands (D7 step 1, D10).
  Step 8 now removes the record before spawning the push, since that push would otherwise stop on its own record.
- F7 (MEDIUM): the creel daemon is dropped from the prime refusals and the criterion, with the reason; the hibernate refusal names `lyx hibernate cancel`.
- F8 (MEDIUM): D5 adds D10's pull and push comparison and the integration's source anchor to the uses of the markerless value; D10 says a markerless tip compares by it; the criterion names the case.
- F9 (MEDIUM), fixed differently after the self-check: D10's checkout way forward names the `--set` then checkout together with its cost, re-anchoring the shared branch being left, and why that cost is accepted; the criterion matches.
  The proposal's first option, a `lyx fabric add` pair on the target branch, was tried and dropped: `Add` wires at its fork point's anchor (D5), so an adopted records branch at another anchor would be mis-wired, and `--adopt-local` reaches only unmarked local branches (`internal/fabricengine/doc.go`, AdoptLocal paragraph).
- F10 (MEDIUM): the conflict way forward tags the local tip under `archive/<slug>/<tip>`, saves the changes of the conflicting and every later local commit outside the weft, resets, re-runs and writes all of them back at `T`.
- F11 (LOW): D7 refuses a dirty weft on a first run, naming `lyx fabric commit` or removal for paths under `A/<names>` and restoring the committed marker for the marker; the dirtiness sentence points at that refusal and the recovery check.
- F12 (LOW): D7 refuses a weft with no committed marker on a first run, naming `lyx fabric reconcile`.
- F13 (LOW): the criterion spares only pushes and board syncs outside the ended dobbies' process trees.

## Self-check

- fix-consistency, MEDIUM, `--adopt-local` way forward contradicts D5: fixed with own change (merged with the three below; the pair route was dropped, see F9).
- fix-claims, MEDIUM, `--adopt-local` does not wire at the target's anchor: fixed with own change (merged, see F9).
- fix-way-forward, MEDIUM, `--adopt-local` way forward leads to a mis-wired pair: fixed with own change (merged, see F9).
- fix-proposal, MEDIUM, F9's proposed pair cannot wire at the target's anchor: fixed with own change, dropping the pair route rather than adding a D5 rule for adopted branches, which no finding asked for (merged, see F9).
- fix-consistency, LOW, the prime refusal's lead-in still claims daemon state under the prime's anchor: fixed as proposed, it names the hibernate daemon only.
- fix-way-forward, MEDIUM, `lyx fabric commit` cannot clear a dirty marker: fixed as proposed, the marker's way forward restores it.
- fix-way-forward, MEDIUM, the conflict way forward can loop on the dirty-weft refusal: fixed as proposed, the changes are saved outside the weft and the reset leaves a clean weft clean.

## Disputed

None.

## Deferred

None.
