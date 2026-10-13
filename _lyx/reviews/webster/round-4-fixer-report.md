## Changes

- F1 (`7834ac10a`): `pinGeometryOptionsLocked` passes `exactSessionWindowTarget` to the unseen-clear `set-hook`, and the expected argv in `faketmux_test.go` uses the same helper.
  `doc.go` already described `set-hook` as taking the `=<name>:` form, so it is unchanged.
- F2 (`f0c383352`): `OpenWindow` creates the window under a per-call temporary name (`lyx-opening-<pid>-<counter>`), chains `remain-on-exit off` and `@lyx_hidden` onto it, and ends the chain with `rename-window` to the requested name.
  The `OpenWindow` fake test pins the new argv, `ErrWindowReadBack` and the `doc.go` paragraph state the new mechanism.
- F3 (`1144cd2f4`): the `InvalidConfigStillRemovesAndReportsTheKey` subtest checks the session-wide `list-panes -s` listing for the removed pane id instead of an error from `display-message`.
  No other assertion in the integration tests treats a `display-message` error as proof that a pane is gone.

## Verification

- `lyx gate test ./internal/reedengine` is green.
- The `tmux`-tagged real-tmux tests (`TestOwnWindow_RealTmux`, `TestBar_...`) were not run: tmux-tier runs are refused in this session.
  `go vet -tags tmux,integration ./internal/reedengine` compiles them cleanly.

## Disputed

None.

## Deferred

None.
