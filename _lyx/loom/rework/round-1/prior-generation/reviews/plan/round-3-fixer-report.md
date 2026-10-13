# Round 3 fixer-report

## Changes

- **F1** (fixed as proposed): `_lyx/plan/05-window-order-and-shell.md`'s Intent now places `shellWindowOf` in `internal/reedengine/selvagepane.go` beside `recordedPaneIDs`, naming `selvagePaneAllowlist` as the reason.
  It adds that no other new member in `windoworder.go` names a Selvage identifier outside a call position or composite-literal key, and reaches the Selvage pane only by calling `shellWindowOf`.
  The Create glyph `plan:internal/reedengine#shellWindowOf` is package-scoped and unchanged.
  Premise checked: `selvagepane_enforcement_test.go` allows only `selvagepane.go`, `state.go` and `config.go`, and `recordedPaneIDs` lives in `selvagepane.go`.
- **F2** (fixed as proposed): `_lyx/plan/07-focus-rules.md`'s Intent names three more `doc.go` sentences for rewrite: the Own windows `@lyx_own, @strand_color and @lyx_unseen` sentence, the multi-client warning bullet's "never changes the argv", and the Detached windows sentence calling the current window "the strands' window".
  Each sentence was confirmed present in `internal/reedengine/doc.go`.
- **F3** (fixed as proposed): card 7's Intent renames the `ListClientsError_WarnsAndDoesNotChangeBehaviour` subtest in `internal/reedengine/attach_test.go` to `ListClientsError_WarnsAndAppendsNoSelection`.

## Disputed

None.

## Deferred

None.
