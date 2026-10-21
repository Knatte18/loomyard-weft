---
round: 2
focus:
  - "In internal/gateslot (config.go, template.yaml), internal/configreg/configreg.go and internal/yamlengine/layers.go: settle whether a hub-side `lyx config gate --set gates.<name>=...` leaves every other gate's list resolving to the template list after load, given that the parent writes sparse hub bases via `SetSparse` and the task declares `gates` in the gate module's registry entry."
  - "In internal/yamlengine/layers.go, `Merge` and `DropTemplateEqualNames`: settle whether a per-name map descends per key on merge and sparse write, and whether `tiers` and `tier_parallel` still behave as whole-unit open maps under the sparse base."
  - "In internal/configcli/configcli_test.go: settle whether a test drives a hub `--set gates.*` through the sparse write and loads with `gateslot.LoadConfig`, asserting the set gate and an unset gate."
  - "In internal/hubreconcile/ensure_integration_test.go, `preMigrationBase`'s darn case and `assertBoardStrippedOnce`: settle whether the planted darn value is valid under the task's darn load, and whether a hub file holding a retired key (darn `verify`, gate `go_parallel`, landing `publish_verify`) beside a template-equal unit has the retired key and the equal unit removed and a tuned unit kept by the walk strip."
---

Both areas are interactions of the parent's sparse hub-base writes with the task's retired and reshaped config keys.
