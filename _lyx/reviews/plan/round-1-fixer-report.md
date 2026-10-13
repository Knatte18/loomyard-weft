# Fixer report — plan, round 1

Every finding's premise held against the code (`attach_test.go:78-82` and `:461`, `doc.go:310`, `spawn.go:218-262` and `:293-294`, `window.go:80-81`, `render/policy.go:14-23`, `strand.go:62-69`).
All eight are fixed as proposed.
`lyx loom validate-plan` passes on the edited plan with informational findings only.

## Changes

- F1, `_lyx/plan/07-focus-rules.md`: the "ListClientsError" row now seeds a live own-window strand and scripts `list-windows` and `answerWindowPanes` so `relevantWindow` would return its window, and asserts no `select-window` is appended.
  The card says the same fixture without the failure appends the selection.
  The row keeps its warning assertion.
- F2, `_lyx/plan/05-window-order-and-shell.md`: the `doc.go` sentence now rewrites the "@lyx_hidden is display only ... no Go decision reads it" line.
  The new wording: the status bar skips a hidden window, placement and the shell check read the flag to find the last non-hidden window, and no focus rule reads it.
- F3, card 5: callers pass their arguments without `-t`.
  `newWindowLocked` adds `-t` on `exactSessionWindowTarget` only for an empty placement, in the plain form and the psmux retry.
  `TestOpenWindow`'s new row asserts exactly one `-t`.
- F4, `_lyx/plan/06-overview-anchor.md`: `validateDisplay` refuses an overview display that carries a window group, and the refusal names the way forward.
  A `Bound:` sentence is added.
  The direct `validateDisplay` subtest gains the overview-with-`Window` input, which wants the refusal.
- F5, card 7: dropped the `TestWindowOrder` rows for `newAgentWindowSelected` and the `plan:internal/reedengine#TestWindowOrder` Edit bullet.
  `TestWindowOrder_RealTmux` is the only place that pins it.
- F6, card 5: dropped the `TestWindowOrder` hidden-placement row and `hiddenWindowInsertArgs` from that table's subject list.
  `TestOpenWindow`'s "AHiddenWindowOpensAfterTheHighestWindow" now scripts a hidden highest-index window.
- F7, card 5: line 30 now names `spawn.go` as `newWindowLocked`'s home and gives the allowlist reason.
  The later duplicate sentence is removed.
- F8, card 6: `ReplaceStrand`'s claim now says a replacement with an overview strand is refused only while a third strand holds the overview.

## Disputed

None.

## Deferred

None.
