```yaml
round: 1
files:
  - internal/reedengine/ownwindow_integration_test.go
commits:
  - 7499bb1f
  - the follow-up commit "reedengine: fix F2, F3 — make the zoom and re-pin tests fail without the code they pin"
```

## Changes

- F1: fixed as proposed. `ResizeLaysTheSharedWindowOutEvenlyAgain` is registered only under `if runtime.GOOS != "windows"`, and its early return is gone.
  The comment above reads as the reason the resize steps are left out on Windows.
- F2: fixed as proposed. The new `ResizeLeavesAZoomedSharedWindowZoomed` zooms the first pane, grows the window, and asserts `window_zoomed_flag` is still 1.
  A second shared window ("zoomed-control") is resized after the zoomed one and waited on until its panes are even,
  so the zoomed window's hook has run before the flag is read and removing the guard fails the test.
- F3: fixed with an own change. The proposal was a resize after a leave under an unchanged split,
  which passes with the re-pin line deleted, since the join already pinned the same layout.
  `LeavingASharedWindowRelaysItOut` now joins three panes under `width`, sets `window_split` to `height`, removes one,
  and asserts the two remaining panes are stacked, then, off Windows, that they stay stacked and window-wide after a window resize.
  That fails without the re-pin, because the stale hook would lay them out side by side.
  The one-pane full-width assertion became the pane-width equals window-width check.
- The `numberOf` and `panesEven` helpers replace the subtest's local `width` closure.

## Self-check

- fix-code-correctness, MEDIUM: F3 test passes without the re-pin. Fixed with the `window_split` change above.
- fix-code-correctness, LOW: F2 test may pass with the guard removed because the hook may not have run. Fixed with the control window.
- fix-code-correctness, LOW: `numberOf` and `panesEven` call the outer `t`. Disputed, premise false: see Disputed.
- fix-consistency, LOW: stale comment above the resize subtest. Fixed.
- fix-consistency, LOW: one-pane leave case no longer covered. Fixed: the leave test asserts pane width equals window width after the resize.
- fix-proposal, MEDIUM: duplicate of the correctness F3 finding. Fixed.
- fix-proposal, MEDIUM: duplicate of the correctness F2 finding. Fixed.
- fix-proposal, LOW: the dropped full-width assertion. Fixed, as for the consistency finding.
- fix-affected-set: no findings.

## Disputed

- `premise false`, fix-code-correctness LOW on `numberOf` and `panesEven`: the finding names no wrong result, only the outer `t`.
  `display`, which both helpers call, already uses the outer `t` and its failures still fail the run, so the helpers behave as the existing ones do.

## Deferred

None.

## Rulings

None.

## Verification

`lym gate test --tier 2 ./internal/reedengine` passed (exit 0) after each round of edits.
The summary does not show whether the tmux-bound `TestOwnWindow_RealTmux` ran or was skipped here, so the new subtests are unconfirmed against a real tmux.
