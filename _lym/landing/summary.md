# Default darn's Code-Review reviewer to opus[high] in every profile

A darn run has no Discussion or Plan review, so its Code-Review perch is the only in-run review.
That reviewer has to judge whether the run did what the entry asks, not only whether the diff is correct.
It took the hub's `code_reviewer_model` default, `sonnet[medium]`, and in a recent run it let incomplete work through.
Hubs worked around this per task with `run_settings`.

Every darn block in `internal/runprofile/template.yaml` now sets `loom.code_reviewer_model: "opus[high]"`.
That covers `speed`, `light`, `solid`, `wide` and `docs`.
A darn run on any profile, or on none, gets an Opus reviewer without a per-task override.
Tapestry keeps the hub's Code-Review model, since its Discussion and Plan reviews already check the scope.
A task's own `run_settings` still layer on top and can override the model.

Closes #130

## Design decisions

- The model is set in the profile blocks, not in `internal/loomengine/template.yaml`, so only darn changes.
  The `pawl` and `sampler` blocks stay on the hub's model.
  `TestTemplateLadder` now checks the sampler block's rounds separately from its reviewer model, since the two no longer match darn's `loom` map.
- `speed`'s darn block carries the model although its Code perch has 0 rounds.
  The setting only takes effect when a task raises the rounds, for example with `lym loom circling continue`.
- `docs` keeps its solo reviewer through `code_reviewer_fan: ""` and gains the Opus model.
- The change reaches existing hubs through the seed-only reconcile in `internal/configsync`.
  A profiles file the operator has not edited gains `code_reviewer_model` on its darn blocks.
  A file where the operator set their own `code_reviewer_model` keeps it, and the difference is listed.
  Two new `configsync` cases pin both outcomes.
- The `internal/runprofile/doc.go` module doc and the template header comment describe the new default.

## To check

- Run `lym` on a hub whose profiles file was seeded from the earlier template and confirm the reconcile adds `code_reviewer_model` to the darn blocks.
  The tests cover this on fixtures, and no real hub was upgraded.
