# Pin the skill-load hold's open-turn and back-off behavior in tests

The skill-load hold landed with two checks left open: whether `TestStartup_LoadTurnAPIError` in `internal/shuttleengine/startup_test.go` pins every hold behavior, and whether an open load turn past the timeout neither skips its skills nor lets the prompt be typed.
Reading the table against `loadSkillTurn` in `internal/shuttleengine/loadturn.go` found two unpinned behaviors and a back-off that was checked only from below.
This change adds rows and bounds; it changes no production code.

## What changes

- A new row covers an open turn past its timeout that then ends.
  The skills load and the prompt follows, so the turn is neither skipped nor does the prompt get typed early.
- A new row covers an open turn after a hold.
  The start fails with a timeout at the deadline the hold moved (`time.Hour + apiErrorRetryBackoff`), and the hold-over line is logged.
- A start that succeeds must now take less than `wantHeld` plus one `apiErrorRetryBackoff`.
  A back-off longer than the scheduled one fails the row.
  This replaces the per-row `wantHeldAtMost` field, which only the ceiling row used.
- A failing start with `wantFailedAfter` set is bounded from above by the same margin.

## Design decisions

- One shared upper bound in the test body replaces a hand-set maximum per row, so each new row is bounded without extra fields.
- The remaining behaviors from the first check (busy session, turn start during the back-off, dying pane, failed send, notice once, no notice when interactive, notifier failure, rejected model, interrupted turn) already had rows and are unchanged.

## To check

- Nothing was launched for real, as the task specified.
  The rows run against the fake clock and pane in `TestStartup_LoadTurnAPIError`.
