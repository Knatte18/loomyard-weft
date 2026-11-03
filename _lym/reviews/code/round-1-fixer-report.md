# Burler fixer-report — code round 1

## Changes

All seven findings were fixed as proposed, apart from the F4 and F5 test mechanics noted below.

- **F1** — `ReviewRolePrefix` is exported from `internal/burlerengine/review.go` and used in `internal/loomcli/wiring.go` in place of the `"burler-review"` literal.
  A new `burlerDirName` constant replaces the `"burler"` literal in `review.go` (twice) and `assist.go`.
  Tests that named `reviewRolePrefix` use the exported name.
- **F2** — `composeAssist` returns, on every failure, the probe directory's instruction paths beside the reason (new `unstartedAssist`), so the seat's values are non-blank, `Table.Validate` passes, and `seatengine.Run` writes the not started marker.
  `TestEngine_Review_AssistantThatDoesNotStart` gained a row with a fan `lenses.yaml` does not define: nil error, only the chair started, an ending that starts with `not started: `.
  F1 and F2 share `assist.go` and `assist_test.go`, so they are one commit.
- **F3** — `CapAssistTimeout` returns the capped room, with no reason, for a bound of 0.
  `TestCapAssistTimeout` gained the row.
- **F4** — `TestEngine_Run_AssistantEndMarkerNamesTheEnding` gained a wait-error row (`wait error: wait failed`) and a row where the stop itself ends the wait with an error (`stopped`).
  The second row needed the fake handle to end a stopped wait with a scripted error, so `seatfake_test.go` gained `stopWaitErrs`.
- **F5** — `TestWire_ReviewKeysReachTheEnv` gained a segment with a 2-minute `keep_warm_cap` in the board's `claude.yaml`, a recorded shortname (the cap is looked up under the chair's strand name) and a 10-minute `review_timeout_min`.
  It asserts the Code-Review assistant's bound equals the timeout less shuttle's start allowance (read through `shuttleengine.LoadConfig`) less the cap.
- **F6** — `internal/burlerengine/modelkeys_test.go` is deleted.
  No assertion was folded in: the `config_test.go` malformed-entry rows (entry 1, entry 2 after an empty entry, the way-forward text) and the tapestry plan rows already make each one.
- **F7** — `docs/overview.md` names the discussion and code assist model and fan keys, `assist_reviewer_timeout_min`, the plan assist model and fan keys, and the assist fan keys in the per-perch fan sentence.

## Self-check

- fix-consistency, LOW: the `composeAssist` doc comment still read as if a failed composition returned real files — fixed as proposed (doc says placeholder paths).
- fix-consistency, LOW: the `assist_reviewer_timeout_min` template comment did not say what 0 means — fixed as proposed (`0 takes that whole capped room`).
- fix-proposal, LOW: the wiring test pinned shuttle's template start retries and startup timeout — fixed as proposed (reads the shuttle config).
- fix-proposal, LOW: `unstartedAssist`'s placeholder paths could be read later — fixed as proposed (doc says nothing may read them).
- fix-code-correctness, LOW: the same pinned template values as the fix-proposal finding — merged with it, fixed.
- fix-affected-set: no findings.

Every fork returned a report.

## Disputed

None.

## Deferred

None.
