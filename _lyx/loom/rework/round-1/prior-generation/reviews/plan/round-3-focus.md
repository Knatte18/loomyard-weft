---
round: 3
focus:
  - "Cards that move or remove a string literal a scan allowlist pins, such as `new-window` in `internal/reedengine/window.go` against `windowCreationAllowlist` in `panebin_enforcement_test.go`: settle whether each card that changes where the literal lives also targets the allowlist, since `RequireNoStale` fails on an entry matching no file."
  - "Every sentence of `internal/reedengine/doc.go` that describes OpenWindow's argv, own windows taking focus, where removal sends a viewer, or the geometry-pins button list: settle whether the card whose code falsifies it names it for a rewrite."
  - "Card 5 and card 6 tests of window placement helpers (`TestWindowOrder`, the `overviewWindowInsertArgs` rows) beside the `TestWindowOrder_RealTmux` rows: settle whether any pure row pins a placement a real-tmux row already pins, per `PATTERN-test-economy`."
  - "Card 5's `OpenWindow` change under a nil `LoadState` result (no reed.json) and `TestOpenWindow`'s exact `wantArgv`: settle whether the card states both."
  - "Card 6's overview window-group refusal, which fires on `Display.Window` non-empty or `WindowSize` non-zero: settle whether a named test reaches each clause."
---

Round 3 re-checks the areas where round 2 found defects, so a fresh reviewer verifies the fixes rather than trusting them.
