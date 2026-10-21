---
round: 1
focus:
  - "internal/fabricengine (doc.go, mutation.go, remove.go, export_test.go, livestate_mutationoracle_test.go): the parent's commit wiring `.scratch` as a link meets the task's dirtiness gate and set-aside path handling; settle whether the gate and the remove/mutation paths still agree on what counts as dirty or removable once `.scratch` is a link."
  - "internal/hubreconcile (walk.go, ensure_integration_test.go) and internal/cleanupcli: the parent's per-machine config layer and `.scratch` link changes meet the task's reconcile walk and cleanup flow; settle whether both sides still assume the same layout and layer order."
  - "internal/testkit/envkit/envkit.go and cmd/lyx/uncontainedwrite_test.go: both sides edited the env kit and the uncontained-write scan; settle whether the merged helpers still isolate the environment the way each side's tests assume, and whether the parent's shell-guard redirect-write rules and the scan still agree."
  - "PATTERN.md and the changed pattern/ files (config-layers, config-strictness, durable-vs-ephemeral-state, friction-capture, hub-wide-config, leaf-packages): settle whether the task's code follows the parent's new machine config layer and per-name-map invariants, and whether the task's PATTERN edits (the F4 restoration and overview cut) dropped or contradicted anything the parent added."
  - "internal/battencli/cli.go and internal/loomcli: settle whether the parent's changes to config layers or landing deps still match the calls the task's code makes, such as renamed or re-signatured symbols that merged without a conflict."
  - "internal/shedrecipe and the lens config module move: settle whether the task's recipe fixtures and lookups still resolve against the parent's relocated lenses and fans config."
  - "Test economy across the overlapping test files: settle whether a task test and a parent test now cover the same behavior, and whether any interaction behavior, such as a dirty gate over a linked `.scratch`, has no test."
---

Focus is ordered by where both sides edited the same files, then by shared invariants.
