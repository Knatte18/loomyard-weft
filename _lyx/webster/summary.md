# MultiLLMProducer: seatengine, the fifth adapter and the MultiLLM recipe row

All three batches recorded `done`.

## Batch 01 (head 3068ea714)

Shuttle gained `Spec.QuietHold`, a per-run send lock and `Run.StrandName()`.
`internal/stencil` gained `FillWith` and `IncludeNames` beside `Fill`.
The two seat directive stencils landed under `contracts/stencils/seat`, registered and marker-pinned.
The new `internal/seatengine` package landed with its table, validation, told geometry and handle-shaped shuttle seam.
No deviations were reported.

## Batch 05 (head bffed1776)

`seatengine` gained its prompt and spec composers, `Engine.Run` with the notice sender and stop rule, and `Probe` and `Resume`.
`shedfake.SeatRunner` landed as the shared fake.
`shedadapters` gained `SeatRunner` and `MultiLLMProducer`.
No deviations were reported.

## Batch 10 (head ee0ac707c)

`Env.Seats` and `Env.Models` were added and filled by loom's wiring and the test fixtures.
The `MultiLLM` registry row, a fixture recipe run against fakes and the `llm`-tier smoke landed, and `docs/overview.md` names `seatengine` and the fifth adapter.
Reported deviations (informational): `internal/shedrecipe/seam_enforcement_test.go`, `registry_test.go`, `conformance_test.go` and `testdata/multillm-recipe.yaml` were changed outside the declared file-ops.
