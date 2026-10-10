# Card 7 — step-progress-compact

**Edit:**
- `internal/shedverbs#StepEnvelope`
- `internal/shedverbs/step_test.go`
- `internal/shedverbs/doc.go`

**Create:**
- `internal/shedverbs#compactProgress`

**Uses:**
- `internal/shedengine#Progress`
- `internal/shedverbs#statusCmd`

**Intent:** The compact envelope of D7: the step envelope's `progress` drops `remaining` and every empty key, while the status envelope keeps `remaining`, since the driver names producers from it.
`compactProgress` maps a `Progress` to `step`, `steps` and `name`, omitting a zero `step` or `steps` and an empty `name`, and returns nil for a nil progress.
`StepEnvelope` stores `compactProgress(progress)` under `progress`; its key set and every other key are unchanged, and `statusCmd` keeps the full `Progress` value.
`step_test.go`'s identity-and-progress test asserts the compact shape, and the field-mapping table gains a row for a nil progress staying null.
`doc.go` states the two progress shapes.

**ImpactSummary:** Only the value under the step envelope's `progress` key changes; no Go reader of the step record decodes `remaining`.
