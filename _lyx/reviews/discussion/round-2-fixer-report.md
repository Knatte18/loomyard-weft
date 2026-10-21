# Fixer report — discussion round 2

## Changes

All changes are in `_lyx/discussion/decision-record.md`.

- F1 (BLOCKING), fixed as proposed: deleted D4's clause that dropped a moved unit equal to the template while the marker is absent.
  Added that a key move writes every moved unit and that the walk's one-time base strip, which runs after the key moves (D3), drops such a unit on an unmigrated board, so no other caller drops one.
  The self-check added the `lenses` exception and the kept-pin exception to that sentence.
- F2 (MEDIUM), fixed as proposed: D4 now writes a source base the move empties as the header-only base, never deleted.
  The plan-writer note names the delete of an emptied hub base file, source or target.
  The key-move acceptance criterion adds that a move out of a base holding only the moved unit leaves the base present and key-less.
- F3 (MEDIUM), fixed as proposed: rewrote D1's guard paragraph.
  It says Webster-Review's auto-accept takes open BLOCKING findings, since its Bouncer sets no `blocking_escalates`, and Discussion-Review gets two rounds.
  It names the reviewer and fixer model downgrades.
  The Bound names the verify gates, PR-Gate's alignment approve or the parent, and says why Merge-Review bounds none of the task's own change.
  The self-check extended the model list to Darn-Review and to Plan-Review's and Discussion-Review's later rounds.
- F4 (LOW), fixed as proposed: added an Auto-mode assumptions bullet on the worktree former-value drop changing a resolved value where the hub base holds another tuned value.
- F5 (LOW), fixed as proposed: extended the worktree criterion to an unmarked pair reconciled through `hubreconcile`'s `Options.Pair` on a board that already holds the marker.
- F6 (LOW), fixed as proposed: added `lyx fabric clone` of an existing weft to the older-binary refill risk.

`_lyx/discussion/support-log.md` needed no change.

## Self-check

- fix-consistency, MEDIUM: D1's model downgrade list omitted Darn-Review. Fixed as proposed, merged with the matching fix-proposal finding.
- fix-consistency, LOW: D4's "writes every moved unit" contradicted the kept target pin. Fixed as proposed: the sentence now excepts a unit a kept target pin overrides.
- fix-claims: no findings.
- fix-proposal, MEDIUM: "no other caller drops one" was false for `lenses` as a key-move target. Fixed as proposed: added "bar `lenses`' template-equal entries (D2)".
- fix-proposal, MEDIUM: D1's list omitted Darn-Review and Plan-Review. Fixed as proposed: both are named, and so is Discussion-Review's third round.
- fix-proposal, LOW: the plan-writer note no longer limited the delete change to hub base files, against local and worktree layers. Fixed as proposed.
- fix-way-forward: no findings.

## Disputed

None.

## Deferred

None.
