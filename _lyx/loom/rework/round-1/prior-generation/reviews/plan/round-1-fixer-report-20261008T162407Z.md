# Fixer report — plan, round 1

## Changed

Nothing fixed: no file under `_lyx/plan` was edited this round.

## Validation

Both findings hold against the plan and the decision record.

- F1 is valid.
  The two decisions added after Discussion ("The segment color goes through the send path after readiness" and "Burler: the merged raw /color typing is a blocking finding") are carried by no card.
  `internal/shuttleengine/run.go:494` still plays `ColorSequence` through `playInputs`, and `Runner.TypeColor` in `internal/shuttleengine/session.go:202` does the same.
- F2 is valid.
  D4 states the late-landing catch only for a prompt whose submission was sent ("The box reading D3 takes before its clear shows an empty box"), and the acceptance criterion "A sent text that never appears in the pane fails with `ErrSubmissionNotLanded`" contradicts card 4's empty-box success on the unsettled and never-appeared causes, where no Enter was sent.

## Disputed

None.

## Deferred

- F1: deferred on the parent's instruction.
  `ly:orch`, acting as the operator's delegate, told this fixer to keep the plan untouched and not rewrite cards against HEAD, letting the gate's attempt cap run out so the row hands back to it.
- F2: deferred on the same instruction from `ly:orch`.
- The gate's findings (`path-missing`, `glyph-not-found`, `create-already-exists`, `caller-uncovered` on cards 2, 3, 4, 6, 7, 9 and 10): deferred on the same instruction.
  Each one comes from checking the plan against a HEAD where the plan's cards have already been implemented: Create targets now exist, Delete targets are gone, and new callers sit in `send.go` and `stopper_test.go`.
  Repairing them would mean rewriting cards against HEAD, which `ly:orch` told this fixer not to do.
