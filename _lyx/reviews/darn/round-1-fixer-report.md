# Round 1 fixer report

## Changes

- **F1** (fixed differently, commit `04cb57517`): `testInnerRunRefusedResume` now asserts that every collected driver-busy notice leads with `[batten notice] myslug: awaiting at Publish; driver pane busy`.
  The proposed `awaiting at ` substring check is replaced by this exact-prefix check, which also pins the row name and the order of state text and busy condition.
  The proposed `/wt/task` deny is not added, since the busy notice in this fixture carries no path, so the deny could never fail; `TestRenderNotice` already pins the relative paths and the absent attach clause for the one renderer both notices share.
- **F2** (fixed as proposed, commit `2fa288541`): `pairRelative` strips the pair prefix only at the start of the text or after a character that cannot be part of a path, so `/other/wt/task/x` stays whole.
  Its doc comment states the rule.
  `TestRenderNotice`'s awaiting row now carries such a path in the child's reason and asserts that it survives while the in-pair path is made relative.

`lyx gate test ./internal/battenshed` passed after each fix.

## Disputed

None.

## Deferred

None.
