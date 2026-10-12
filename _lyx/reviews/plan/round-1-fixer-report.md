# Fixer report — plan, round 1

## Changes

- **F1** (MEDIUM, scope), fixed as proposed.
  `_lyx/plan/07-dialog-notice.md` gains two `TestWait_DialogNotice` rows: an ASK line on `sessionFakeEngine`, which has no `InputBoxReader`, and an ASK line on `readinessEngine` with `tooShort` set over a caret-less pane, each sending no notice with the clock past `permissionNoticeBound`.
  The `watchDialog` paragraph names `sessionFakeEngine` beside the no-reader guard and the `tooShort` `readinessEngine` beside the too-short failed-read branch.
  Premise checked: `readBox` (`internal/shuttleengine/readiness.go`) returns `boxReading{}` for an engine without `InputBoxReader`, `sessionFakeEngine` asserts no `InputBoxReader`, and `readinessEngine` carries `tooShort` (`internal/shuttleengine/fakes_test.go`).
  The rows extend the existing table, so the budget row is unchanged.
- **F2** (LOW, decision), fixed as proposed.
  `_lyx/plan/08-fork-ask-live-scenario.md` fixes the command as `rm -f ./smoke-ask-$$`, a computed-path `rm` through a special parameter card 4 lets through, which the decision record assumes Claude Code still prompts on in bypass mode.
  The never-retry rule joins the card's existing no-ask sentence, which already said such a run fails and is the open-risk outcome, so that point is stated once.

## Disputed

None.

## Deferred

None.
