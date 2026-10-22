---
round: 1
focus:
  - "contracts/stencils/orch/orch-template-role.md: does the replaced small-task rule drop, weaken or contradict any other orch rule, and does it match the entry's wording (a task needing fewer rounds names a profile, never hand-written run_settings)?"
  - "internal/runprofile/template.yaml: is the `light` profile's `darn` block valid under the profile schema and loader, and does it sit beside the loom values in the expected shape?"
  - "Every place that states or pins the old hard cap of 4 (docs, doc.go, config.go comments, tests, other templates or stencils): is any left stale after the change?"
  - "internal/darnengine/config_test.go, internal/loomcli/circling_test.go, internal/loomcli/wiring_darn_test.go: do the changed pins still test what they did, and is any behavior the diff adds left uncovered?"
  - "_lyx/landing/summary.md claims: does the diff bear out each one, including that the loomyard hub's own darn.yaml is not touched by the diff?"
---

Seed focus from the board entry and the committed diff; the diff is small, so the focus is on stale references and the stencil rule.
