---
round: 2
focus:
  - "Check in `internal/shuttleengine/codexengine/home.go` and `codexengine.go` whether the engine learns the weft writable root by reading the anchor's `.lym` and `_lym` link targets (`fslink.IsLink`, `fslink.PointsTo`) or is handed an absolute path as told geometry, and whether `PATTERN-cwd-resolution` and the decision's weft-checkout writable root are met either way."
  - "Check whether `Prepare` tests in `internal/shuttleengine/codexengine/prepare_test.go` pin the launch shapes outside a linked worktree: real `.lym` and `_lym` directories, a plain clone, and a `.codex/rules` directory under an anchor that differs from the worktree root."
---

The entries name where to look and which question to settle; they cap no severity.
