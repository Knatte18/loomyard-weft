---
round: 2
focus:
  - "decision-record.md D1: settle whether the rule that stacks the machine layer reads the told `Dirs` or the `layerDirs` that `load` in `internal/configengine/config.go` and `Provenance` in `internal/configengine/layers.go` produce after clearing `Anchor` for an anchor with no `_lyx/`; check that every worktree of a hub on one machine resolves the same `go_parallel` and `slots`, and that an acceptance criterion pins the answer."
  - "decision-record.md D3 and D4: settle whether each way forward a gate load failure names works from the session that reads it, given that D4 refuses `lyx config gate --layer machine` from a pair, and whether any escape hatch D3 introduces (removing the machine file) states what it lets through and what bounds it."
  - "decision-record.md D2 and D5: settle what happens to a retired key in the machine file, where `load` validates each layer before `dropRetired` runs and a retired key is admitted only when its recorded category is `machine`; check that D2 and D5 give one outcome, and that the failure message reach for the Board-only hub-file checks (`validateHubFile`, `setHubWide`, `copyStagedToHub` in `internal/configcli/hubwide.go`) is stated."
---

The focus lists where the first round's gating-class finding sits and the decisions around it that the same fix touches.
