# Fixer report — discussion, round 1

## Changes

All in `_lyx/discussion/decision-record.md` unless noted.

- F1: D2's Decision requires the verify-failure record to belong to the halt being judged (row is the current producer, head is the pair's HEAD, written during the halting attempt, the match battenshed's retry makes), and says the Publish check takes the same match.
  The Bound now says a stale record from an earlier verify leaves a stop eligible.
  New acceptance criterion: a blocked Merge-In stop of another cause with an earlier verify's record on disk stays eligible.
- F2: D2's Bound gains the sentence on the no-batten and spent-or-capped-retry case parking for the parent.
- F3: D4's Decision exempts a changed file with no symbols when it is the file part of any card target, symbol or file-only, through `glyph`'s conversions.
  D4 gains a Bound on what the list lets through and reports.
  The criterion on symbol-less files is reworded to the file-part rule, and a new criterion covers an import-only change in a targeted file.
- F4: D4 and its acceptance criterion name the Master's summary narrative instead of the PR narrative.
- F5: D5's Decision names the hub's loom recipe file as the row-order source, pointed at by the stencil per `PATTERN-producer-pointer-rule`, compared against the producer half of the peer's run status, with a non-recipe producer as unknown order.
- F6: D5's Bound names the behavioural interaction between text-only overlaps, caught only at the second run's Merge-In verify, and the narrowed PATTERN-entry rule, at both `parent-review` and `pr-gate`.
- F7: chose the scoping option: the relaxed pause check holds only on the verdict-in-effect admit, and an unmoved run keeps the equality check.
  New acceptance criterion names the choice.
- `_lyx/discussion/support-log.md`: ledger Q2 notes the F7 revision; the Review rounds section records round 1.

## Disputed

None.

## Deferred

None.
