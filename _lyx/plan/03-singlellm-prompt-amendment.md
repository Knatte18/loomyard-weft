# Card 3 — singlellm-prompt-amendment

**Edit:**
- `internal/shedadapters#SingleLLMProducer`
- `internal/shedadapters#NewSingleLLMProducer`
- `internal/shedadapters#NewSingleLLMProducerGated`
- `internal/shedadapters#SingleLLMProducer.Call`
- `internal/shedadapters/singlellm_test.go`
- `internal/shedadapters/doc.go`
- `internal/loomshed#NewPlanDirRotator`
- `internal/loomshed/planwrite_test.go`

**Intent:**
Widen `SingleLLMProducer`'s fresh-spawn preparation so it can append text to the prompt of the session it prepares for (decision record D4).

Change the `prepareFreshSpawn` field and both constructors' parameter from `func() error` to `func() (string, error)`: the string is an amendment appended verbatim to the end of the composed prompt.
In `Call`, on the respawn branch only, after the preparation returns, append the amendment to `spec.Prompt` on the spec handed to `p.shuttle.RunGated` and to no other spec.
The attach branch is unchanged: it never runs the preparation, and `AttachGated` receives the spec as composed.
An empty amendment leaves the prompt byte-identical; a nil preparation behaves exactly as today.
A preparation error still aborts before the archive step and before any spawn.
The amendment is append-only by construction, since the preparation returns text rather than receiving the spec.
Update the struct, constructor and `Call` doc comments, and the `prepareFreshSpawn` paragraph in `doc.go`, to describe the amendment.

`loomshed.NewPlanDirRotator` returns `func() (string, error)` so the Plan-Write wiring keeps compiling; in this card its closure always returns an empty amendment, and card 6 fills it in.
Update `planwrite_test.go`'s rotator calls to the new closure shape and assert the amendment is empty.
The other `NewSingleLLMProducer`/`NewSingleLLMProducerGated` callers pass an untyped `nil` and need no change.

Tests in `singlellm_test.go`:

- Update the existing preparation tests (the respawn-path ordering test and the preparation-error test) to the new closure shape.
- Respawn branch: a preparation returning a non-empty amendment makes `fakeShuttle.gotSpec.Prompt` equal the composed prompt followed by the amendment, and the preparation runs exactly once.
- Attach-found branch: the preparation never runs and `gotAttachSpec.Prompt` equals the composed prompt.
- Nil preparation, and a preparation returning an empty amendment: `gotSpec.Prompt` is byte-identical to the spec source's prompt.
- A preparation error aborts with `RunGated` never called and the output files not archived.

**ImpactSummary:** the preparation seam's function type changes; its only non-nil supplier is `loomshed.NewPlanDirRotator` (updated here) wired by `shedrecipe`'s Plan-Write entry, and nil-passing callers are untouched.
