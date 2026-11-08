---
round: 1
focus:
  - "internal/shuttleengine/codexengine/command.go and config.go: settle whether the launch line under danger-full-access is correct (flags dropped, `-a never` kept) and whether an invalid or empty sandbox_mode value fails loudly or falls back to workspace-write."
  - "internal/configengine/layers.go and internal/configcli: settle whether stacking the machine layer between worktree and local for a per-worktree module with a machine key leaves other modules' layering and error paths unchanged, and whether a local override of a machine key is refused."
  - "Scope against the board entry: the entry asks for a sandbox_mode key and, optionally, a bwrap preflight; settle whether the diff makes changes beyond those (the config-layer change, configreg test rewrite) that the entry does not license."
  - "pattern/PATTERN-config-layers.md and docs/overview.md: settle whether the doc and pattern edits match the layering the code now implements."
  - "Tests (prepare_test.go, configreg_test.go, configcli_test.go, layers_test.go): settle whether the rewritten configreg test lost coverage, and whether each new test duplicates an existing one."
---

Seed from the diff stat and commit messages; the decision record holds only the board entry under `## Goal`.
