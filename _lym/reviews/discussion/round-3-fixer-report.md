# Fixer report — Discussion round 3

## Changes

- F1 (MEDIUM, design), fixed differently in part.
  D4 now places the push verdict, states its pass-through cases and hold behaviour, and the verb's own skips;
  the Constraints `PATTERN-owner-machine` bullet says the same.
  Not made as proposed: a `fabricengine` function "taking the pair's location and slug" that runs the whole check.
  It cannot run the reconcile or read the pause:
  `internal/pairteardown` (which builds the reconcile's seams, `internal/loomcli/landingdeps.go:756-760`) and `internal/hubgeom` (`PauseGeometry`) both import `fabricengine`.
  The function instead takes the pair's location, slug and the reconcile's report or error, and each caller runs the reconcile only once its own skips leave the check open.
  Also not made as proposed, after the self-check: the verb does not take loom's pause and Finalize skips (see Self-check, fix-proposal 1 and 2).
  A hold's line names a re-run of the verb, and a re-run that changes no map still pushes when the pair holds unpushed commits, so a held commit has a way out in a hand-worked task.
- F2 (MEDIUM, design), fixed as proposed, with one change.
  D5 states that the commit takes the weft off the behind-only branch and onto the replay, that the replay's `dirtyTrackedOn(touched)` covers the maps commit's paths, and the live-pass bound with its `lym fabric commit` way forward.
  It does not claim the stop "touches no state": `replayWeft` tags set-aside commits before the dirtiness refusal, and D5's maps commit stays on the branch.
  D5 now says the branch is left unmoved with that commit on it.
  The heading now reads "Uncommitted maps pass preflight and are committed before a weft pull", since a weft pull can still stop on a map.
  Criteria: one new criterion for the live-pass stop.
  The replay-path criterion is folded into the existing prime-sync D5 criterion, which now says the maps commit is replayed onto origin's tip, instead of adding a near-duplicate line.
- F3 (LOW, consistency), fixed as proposed: two D4 criteria for what the task pair's and the prime's pushes carry, conditioned on pushes being on and, in the task pair, on the verdict letting the push go.
- `support-log.md`: a Round 3 entry under Review rounds.

## Self-check

- fix-consistency, MEDIUM, F3 criteria unconditional against the held-push criterion: fixed as proposed (qualified).
- fix-consistency, LOW, PATTERN changes entry lacked the hold clause: fixed with own change.
  Adding the clause put the entry's rule text over the gate's 280-character cap (`pattern-change-over-cap`).
  The hold rule stays in D4 and in the Constraints `PATTERN-owner-machine` bullet, and the PATTERN entry keeps its round-2 text.
- fix-consistency (noted out of lens), the reconcile would run before loom's skips: fixed with own change, each caller runs the reconcile only when its skips leave the check open.
- fix-claims, MEDIUM, `weftDirtyStop` misdescribed: fixed as proposed, D5 names the behind-only branch, not `weftDirtyStop`, as the path no longer taken.
- fix-claims, LOW, "one commit ahead": fixed as proposed, "at least one commit ahead".
- fix-proposal, BLOCKING, pause skip lets a hand call push past loom's stop: fixed as proposed, the verb drops the pause skip, with a criterion.
- fix-proposal, MEDIUM, Finalize skip leaves a hand call's push unchecked: fixed as proposed, the verb drops the Finalize skip, in the same criterion.
- fix-proposal, LOW, F3 criteria unconditional: duplicate of fix-consistency's MEDIUM, fixed there.
- fix-proposal, LOW, live-pass criterion needs an injected write: fixed as proposed.
- fix-way-forward, LOW, a hold's way forward never pushes the waiting commit: fixed as proposed, with a re-run that pushes unpushed commits, named in the hold's line, with a criterion.

Every fork returned a report.

## Disputed

None.

## Deferred

None.

## Rulings

None.
