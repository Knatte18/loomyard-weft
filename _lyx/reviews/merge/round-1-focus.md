---
round: 1
focus:
  - "internal/webstercli/wiring.go, internal/burlercli/wiring.go, internal/loomcli/wiring.go: the parent changed `languageconfig.Load` to return a struct with `Skills` and `VerifyTiers`, and the task removed the `reedUp` ctx/watch parameters in the same files. Settle whether every wiring site still compiles to the meant semantics: does the burler wiring use `VerifyTiers` where the parent's prompts need it, and does the standalone `reedUp` closure at each site match the task's new `func() error` contract?"
  - "internal/webstercli/recoverbatch.go and internal/webstercli/cli.go: the parent added `verifyTiers` and `VerifyTiers` to `recoverDeps`/`beginDeps`, and the task changed the `reedUp` call in recover-batch. Settle whether the recover-batch path no longer carries the removed `watch=false` asymmetry the old comment described, and whether any comment or doc in these files still refers to a watcher or the `watch` parameter."
  - "CLAUDE.md and docs/overview.md: the task removed the #673 workaround and rewrote the reed rows; the parent also edited overview.md around the shuttle and loom rows. Settle whether the merged text is consistent about default anchor `own-window` and about verify tiers, and whether CLAUDE.md still matches the merged docs."
  - "Parent-added fork and recovery prompts that render verify tiers (internal/websterengine/render.go, template_test.go) against the task's placement vocabulary: settle whether any parent stencil or test still names `below-parent`, panes in the shell window, or a layout that the task removed."
  - "Test economy: in internal/webstercli/wiring_test.go, settle whether the parent's `wantTiers` assertions and the task's wiring tests cover the same wiring twice, and whether the merged `reedUp` behavior in recover-batch has any test."
---

Seed focus from the overlapping files and packages; no prior round exists.
