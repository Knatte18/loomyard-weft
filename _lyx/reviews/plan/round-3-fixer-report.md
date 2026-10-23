# Plan round 3 — fixer report

## Changes

- **F1** (fixed differently, in one row): card 29's `TestFinalize_GateAndFixer` paragraph gains the three `RerouteReviewed`-false rows the fix proposed.
  These are the over-cap resume over a failing gate with one attempt left, the second resume after an `escalate` keeping the first resume's `Baseline`, and the two-turn spent over-cap halt resumed over a failing gate.
  The proposed third row said the halt names the log.
  The card instead names the fixer's commits, because card 29's Intent says a spent cap with no review on the way back "halts with `LandingStopPrefix` instead, naming the fixer's commits".
  After the self-check, the second row also names its second resume's gate and verdict: a failing gate and a `fixed` turn.
- **F2** (fixed as proposed): card 35's `TestDecideDrive` keeps one marked blocked-stop row, plus the changed-build-with-escalation row and the Webster reroute-cap row.
  Card 36's `TestCheck_RefusesLandingHalts` keeps one landing-halt row, a Finalize `LandingStopPrefix` status, plus its two eligibility rows.
  Each card states why one row covers every kind of halt.
- **F3** (fixed as proposed): card 40's Intent gains the sentences that bound the kept lock:
  - owner takeover (card 27)
  - the fixer session bound (card 11)
  - marker liveness, with the gate's pre-tier ceiling and tier timeouts
  - the operator's release verb

  The sentences also state that a read error keeps the lock while the read fails.
- **F4** (fixed as proposed): card 7 makes `PlanDir` and `FailureRecordPath` Optional in both modes, and `aligned_tip` Optional only in `finalize` mode.
  As a result, a `merge-in` table with an empty `AlignedTip` is an error naming the field.
  The test paragraph gains that row.

## Self-check

- fix-consistency, LOW: cards 35 and 36 claimed `TestHandsBack` "pins each kind's reason", which overstates card 34's rows.
  Fixed: both now say it pins the predicate, meaning the `LandingStopPrefix` and the reroute-cap reason.
- fix-claims, LOW: card 40's list of bounds omitted the bound on the verify run itself.
  Fixed: the list names the gate's pre-tier ceiling and tier timeouts.
- fix-affected-set, LOW: card 37's `TestInnerRun_BinaryChangeSkipsALandingHalt` was a twin of F2 with two per-kind rows.
  Fixed: it now has one landing-halt row plus the Webster row, with the same reason.
- fix-proposal, MEDIUM: card 29's second new row did not name the second resume's gate or verdict, so the path to its `LineChange` assertion was undetermined.
  Fixed: the row now says the second resume runs over a failing gate and its fixer answers `fixed`.
- fix-proposal, MEDIUM: card 40 said the hold is unbounded only for an unreadable status file, but a malformed verify marker is also a read error (`internal/verifytree/marker.go:44`).
  Fixed: the sentence names both files.
- fix-proposal, LOW: duplicate of the fix-claims gate-bound finding; fixed with it.
- fix-proposal, LOW: duplicate of the fix-consistency "each kind" finding; fixed with it.

## Disputed

None.

## Deferred

None.
