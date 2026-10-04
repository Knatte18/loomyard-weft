---
round: 2
focus:
  - "Card 2 (`_lyx/plan/02-pattern-warp-root.md`) and the `directive-root` Shared Decision in `_lyx/plan/00-overview.md`: in hub mode, `internal/hubgeom/hubgeom.go` (`BurlerGeometry`) and `internal/hubgeom/webstergeom.go` set `WorktreeRoot` from `l.AnchorPath()`, not `l.WorktreePath()`. Settle whether the plan gives burler and webster call sites a value that is the real warp worktree root in a subpath-anchored hub, so `PATTERN.md` resolves at the repo root, and whether a test covers that case."
  - "Card 2: `RenderMasterPrompt` takes both `worktreeRoot` and `anchorRoot`, and `RenderRecoveryPrompt` has a `promptWorktreeRoot` parameter. Settle whether the card states unambiguously which parameter each keeps or drops, and lists every caller and test call that changes."
  - "Card 2 Intent: a whitespace-only `PATTERN.md` yields no directive. Settle whether the Intent states what this skips and what bounds it, per the attack-surface rule."
  - "Cards 14 to 16: settle whether every listed test file still carries a `weft` or `warp` hit after stripping `vocab-exempt` identifiers, and whether card 18's scan boundaries (`internal/configsync` test files, repo-root `sandbox/` versus `tools/sandbox/`) are stated."
---

Targets the areas where round 1 found plan defects.
