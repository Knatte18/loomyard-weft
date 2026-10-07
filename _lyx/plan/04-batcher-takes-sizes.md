# Card 4 — Batcher takes a size source

**Edit:**
- `internal/batcher/batcher.go`
- `internal/batcher/identity.go`
- `internal/batcher/identity_test.go`
- `internal/batcher/registry_test.go`
- `internal/websterengine/runlevel.go`
- `internal/websterengine/runlevel_test.go`
- `internal/webstercli/beginbatch.go`
- `internal/webstercli/awaitbatch.go`
- `internal/webstercli/recordbatch.go`
- `internal/webstercli/recoverbatch.go`
- `internal/webstercli/rebaseline.go`
- `internal/webstercli/validate.go`

**Create:**
- `plan:internal/batcher#Identity` -> `func Identity() Batcher`

**Uses:**
- `plan:internal/batcher#DiskSizes`
- `plan:internal/batcher#SizeSource`

**Intent:**
The `Batcher` interface takes the plan, the cards and a size source, and each batch carries the profile that formed it and its estimate (Decision 9).
Behavior is unchanged: identity is still the only batchifier.

- `Batcher.Batch(plan *planparser.Plan, cards []planparser.Card, sizes SizeSource) ([]Batch, error)`.
  `cards` is a contiguous run of `plan.Cards` (the whole list, or a rebaseline's tail later); `plan` carries the glyph language the estimator needs.
- `Batch` gains `Profile string`, the `Name()` of the batcher that formed it, and `Estimate float64`, its segment cost, zero for the identity batchifier, which ignores `plan` and `sizes`.
- `Identity()` returns the identity batchifier without a registry lookup; webster's fallback for a state without a recorded partition (card 8) needs it whatever profile is active.
- Every caller passes `batcher.DiskSizes(<worktree root>)` from the root it is already told: `Run` passes `deps.Geom.WorktreeRoot`, the webstercli verbs `c.geom.WorktreeRoot`.
  A `Batch` error is refused as raw I/O, naming the plan and "re-run the verb" as the way forward (the `raw I/O` row of `contracts/specs/refusal-spec.md`'s webster section).
- `identity_test.go` asserts the profile name and zero estimate on each batch; `registry_test.go`'s probe batcher and `runlevel_test.go`'s direct `Batch` call follow the new signature.

**ImpactSummary:** every `Batcher.Batch` caller (webster's `Run` and six webstercli verbs) and the two in-package test batchers change signature; output batches are identical to today's.
