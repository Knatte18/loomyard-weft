# Card 11 — shedrecipe-multillm-entry

**Create:**
- `plan:internal/shedrecipe#multiLLMEntry` -> `func multiLLMEntry(name string, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `plan:internal/shedrecipe#multiLLMSeat` -> `func multiLLMSeat(i int, cfg Config, env Env) (seatengine.Seat, error)`
- `plan:internal/shedrecipe#resolveSeatModel` -> `func resolveSeatModel(entry string, i int, spec string, reg modelspec.Registry) (model, effort, version string, err error)`
- `plan:internal/shedrecipe#parseSegment` -> `func parseSegment(entry, raw string) (segmentcolor.Segment, error)`
- `internal/shedrecipe/entries_multillm_test.go`

**Edit:**
- `internal/shedrecipe#registry`
- `internal/shedrecipe#shedrecipeAllowedImports`
- `internal/shedrecipe#minimalEntries`
- `internal/shedrecipe_test#coverageGuardAllowedUnreachableEngines`
- `internal/shedrecipe/doc.go`
- `//contracts/specs/shed-recipe-spec.md`
- `//cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shedrecipe#singleLLMEntry`
- `internal/shedrecipe#resolveGateSpec`
- `internal/shedrecipe#configMapList`
- `internal/shedrecipe#resolveUnderRoot`
- `internal/shedrecipe#requireSeam`
- `internal/modelspec#Parse`
- `internal/modelspec#Registry.Resolve`
- `internal/segmentcolor#Segments`
- `plan:internal/seatengine#Table.Validate`
- `plan:internal/seatengine#ReservedMarkers`
- `plan:internal/shedadapters#NewMultiLLMProducer`

**Intent:** Register the `MultiLLM` engine (D2, D12, PATTERN-shed-recipe-registry) in `entries_multillm.go`, one map entry and no `init()`.
`multiLLMEntry` reads the row keys `role`, `segment`, `seats` and the optional `gates`, `timeout_min`, `interactive` and `tokens`, rejects an unknown row key, requires the absolute `WorktreeRoot`, `AnchorPath` and `StencilsDir` roots and the `Seats` seam through `requireSeam`, and builds a `seatengine.Table`: the role prefix, `parseSegment`'s segment checked against `segmentcolor.Segments`, `resolveGateSpec`'s gate, the timeout in minutes, the interactive flag and the tokens as table values.
`multiLLMSeat` reads one `seats` element from `configMapList`: `name`, `stencil`, `model`, `inputs` and `outputs`, and the optional `skills` and `values`, rejecting an unknown seat key; inputs and outputs are resolved under `WorktreeRoot` with `resolveUnderRoot`; `resolveSeatModel` parses the model-spec string with `modelspec.Parse` and resolves it against `env.Models`, so the seat carries the spec's model, effort and version and an unknown alias is a construction error naming the seat.
A `tokens` or `values` name that collides with `seatengine.ReservedMarkers`, and an empty value, are refused as the `SingleLLM` row refuses them, and `Table.Validate` against `env.StencilsDir` runs at construction, so an unreadable stencil, an unresolvable include and a nested include fail before any call (PATTERN-stencil-ownership's eager probe).
The entry returns `shedadapters.NewMultiLLMProducer(name, table, env.Seats, env.Now)`.
`shedrecipeAllowedImports` admits `internal/seatengine` and `internal/segmentcolor`; `minimalEntries` gains the `MultiLLM` row over a chair and one advisor with stencils the test writes; `coverageGuardAllowedUnreachableEngines` tolerates `MultiLLM`, since no loom row reaches it in this task.
`entries_multillm_test.go` is one table-driven construction test covering an unknown row key, an unknown seat key, a nil `Seats` seam, an unknown model alias, a bad segment, a reserved or empty token and value, an absolute or escaping path, and each `Table.Validate` refusal reaching the caller, plus one composed-table test pinning that a valid row yields the resolved model triple, absolute inputs and outputs, the segment, the gate, the timeout and the table values on the table the fake seat runner records.
`doc.go` describes the `MultiLLM` row and its seam beside the existing paragraph, and `shed-recipe-spec.md` adds `Env.Seats` and `Env.Models` to its live-seams list and one paragraph on the row's keys, so the recipe spec and the package doc agree.
Raise the `internal/shedrecipe` budget row by the tests this card adds.

**ImpactSummary:** One new registry key with its own tests; the import allowlist widens by two leaf-side packages, and no existing entry's behaviour changes.
