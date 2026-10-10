# Plan round 3 — fixer report

Each finding was checked against the code and the decision record, and all four hold; each is fixed in the card's Intent.

## Fixed

- **F1** (`02-stencil-include-form.md`): `FillWith` now rewrites each bare include action's pipeline to the root variable `$`, not dot, so a block receives the value map `Execute` was given, including inside a `{{with}}` or `{{range}}` body.
  `TestFill` gains a row in which a bare include inside a `{{with}}` body renders its marker from the value map.
- **F2** (`02-stencil-include-form.md`): `IncludeNames` now strips the template's leading comment before parsing, as `TopLevelMarkers` does (`internal/stencil/stencil.go`, `stripLeadingComment`).
  This means a seat block whose header spells its own include action reports no include of itself.
  The `IncludeNames` test gains that row.
- **F3** (`09-shedadapters-multillm.md`): the chair outcome mapping is now one table-driven test.
  Each row takes its expected mapping from `SingleLLMProducer` driven through `shedfake.Shuttle` and asserts that `MultiLLMProducer` over a one-chair table maps the outcome the same way.
  The separate N = 1 comparison test is dropped.
  The card's budget line already sizes the budget row by the tests the card adds, so it needed no change.
- **F4** (`01-shuttle-quiet-hold-send-lock.md`): the send-lock test's case beside the gated Wait now also asserts that the result's `Gate.Attempts` equals the attempts of the same run with no concurrent send.
  `Gate.Attempts` is already read by the existing gate tests, for example `internal/shuttleengine/gate_test.go`.

## Disputed

None.

## Deferred

None.
