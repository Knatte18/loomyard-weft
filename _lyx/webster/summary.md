# Twill, the self-forking Discussion writer

Batch 01 (`01-shuttle-await-poll`, cards 1–6) finished at `40fd8d693`.
It added the shared capped file poll in `internal/shuttleengine` and rewired `await-review` onto it.
It added the `lyx shuttle await <path>... [--cap]` verb and the `premises`, `board-fit` and `twill-discussion` lenses.
It added the host-neutral twill directive and write-scope stencils, the `internal/twilldirective` package that renders them, and the two optional twill markers in the discussion template, threaded through the composer and chair table.

Batch 07 (`07-twill-config-value`, cards 7–10) finished at `3e3eb0aa1`.
It made `discussion_producer: twill` load, with the unknown-value error naming all three values.
It added the ready marker's path accessor under `.lyx` and its removal.
It made `DiscussionSpec` render the twill block, resolve the fan and set fork subagents for `twill`.
Go now removes a stale ready marker before a fresh Discussion-Write spawn.

Reported deviations (files outside the declared file-ops, all informational):
- Batch 01: `internal/shuttleengine/await.go`, `internal/shuttlecli/await.go`, `internal/twilldirective/twilldirective.go`, `internal/burlerengine/template.yaml`, `internal/loomengine/prompt.go`, `internal/loomengine/discussiontable_test.go`, `internal/loomengine/prompt_test.go`, `contracts/stencils/stencils.go`, `contracts/stencils/markers_test.go`, `contracts/stencils/claims_test.go`, `docs/overview.md`.
- Batch 07: `internal/loomengine/twill.go`, `internal/loomengine/discussion_test.go`, `internal/loomengine/discussionpath_test.go`, `internal/loomengine/config_test.go`, `internal/shedrecipe/recipe.go`, `internal/shedrecipe/fixture_test.go`, `internal/loomcli/wiring.go`.

Neither batch needed recovery.
