# Fixer report — plan, round 2

## Changes

- F1 (BLOCKING), fixed as proposed, with the fresh baseline's point corrected in the self-check.
  Card 29: the no-review halt notes `over-cap`, and every other halt except `spent` leaves `Halt` empty.
  A resume of an entry noted `over-cap` writes `Baseline` afresh and clears the note in the same write.
  It then lands over a passing gate without judging the caps, and over a failing gate it runs the fixer on the attempts left, or reroutes or halts as a spent cap does when none are left.
  The exhausted-entry path now applies only to an entry with no `over-cap` note.
  The `Baseline` keep-rule makes an exception for the resume after an over-cap halt.
  `TestFinalize_GateAndFixer` gains a row: a two-turn fix whose last turn edits a `_test.go` file halts over-cap with `RerouteReviewed` false, and the resume over a passing gate lands.
  ImpactSummary is unchanged.
- F2 (MEDIUM), fixed as proposed.
  Card 26: the resume after an over-cap halt clears `Halt` in the same record write that refreshes the baseline.
  Every halt other than a spent cap or an over-cap change leaves `Halt` empty.
  The `Baseline` keep-rule makes the over-cap exception.
  `TestMergeIn_Fixer` gains a row under `CodeReviewOff`: an over-cap resume over a failing gate whose fixer turn commits and escalates is resumed a second time, and that resume calls `LineChange` from the first resume's baseline.
- F3 (MEDIUM), fixed as proposed.
  Card 7 defines `AlignedTip` as the merge record's aligned tip, or the entry's `AlignFrom` when the record holds none.
  `AlignedTip` is now optional only in `finalize` mode and for a darn run.
  The empty-optional test row now renders `AlignedTip` in `finalize` mode.
  Card 11: `run`'s `tips` are the from tip and the merged tip, and they fill `FixerInputs.AlignedTip` and `MergedTip`.
  Card 26: `align` passes the aligned tip, or `AlignFrom`, as the from tip.
  The first-Merge-In row of `TestMergeIn_Fixer` asserts that the spawned table carries the fork point.
  Card 29: Finalize passes an empty from tip and the merged tip card 28 wrote, which closes the same gap for the other caller of `run`.

## Self-check

- fix-claims, MEDIUM, merged with fix-proposal, MEDIUM: card 29 wrote the fresh `Baseline` at "the HEAD it resumes from", which comes before the resume's own parent merge (card 28). Fixed as proposed: the fresh `Baseline` is now the HEAD `gateAndFix` starts from, after this entry's merge.
- fix-affected-set, LOW, merged with fix-claims, LOW: card 29 left `Halt` empty on its spent-cap halt, which disagrees with card 26 and card 10. Fixed as proposed: the no-review halt now notes `spent` or `over-cap` after its cause.
- fix-consistency, MEDIUM: card 10 still said the exhausted-entry path is keyed on `Spent` alone. Fixed as proposed: it now names the over-cap resume that runs first.
- fix-consistency, LOW: card 26 lacked card 29's precedence over the exhausted-entry path and its outcome when no attempts are left. Fixed as proposed: the exhausted path is qualified by "no `over-cap`", and an over-cap resume over a failing gate with no attempts left halts naming the log.
- fix-consistency, LOW: card 26's over-cap resume test row did not assert that `Halt` is cleared. Fixed as proposed.

## Disputed

None.

## Deferred

None.
