# Card 5 — loom-preflight-batcher

**Edit:**
- `internal/loomshed#NewLoomPreflight`
- `internal/loomshed#loomPreflightProducer`
- `internal/loomshed#loomPreflightProducer.Call`
- `internal/loomshed/doc.go`
- `internal/shedrecipe#loomPreflightEntry`
- `internal/loomshed/loompreflight_test.go`
- `internal/loomshed/cancellation_test.go`
- `internal/loomshed/gatefindings_test.go`
- `internal/loomrecipe/shape_test.go`
- `internal/shedrecipe/entries_simple_test.go`
- `cmd/lyx/testdata/test-budget.yaml`
- `contracts/specs/refusal-spec.md`

**Uses:**
- `plan:internal/batcher#ErrRetiredKey`
- `internal/loomshed#batchifierReasonPrefix`
- `internal/loomshed#batchifierWayForward`

**Intent:** Loom-Preflight fails a stale `batcher.yaml` before any LLM row runs instead of five rows into the run (D11).
`NewLoomPreflight` takes the told config base directory (the worktree's anchor path, the same path the Batchifier row and webster's wiring load `batcher.yaml` from) as a new parameter, kept on the producer; `loomPreflightEntry` passes `env.AnchorPath`, validated with the registry's existing absolute-root check.
`Call` runs `batcher.Active` on it after a passing seed check, the same load and load mode the Batchifier row makes, and returns `Stuck` with a reason that opens with `batchifierReasonPrefix`, carries the load error and names `batcher.yaml`.
When the error wraps `batcher.ErrRetiredKey` the way forward is `run "lyx config reconcile --apply", then re-step`; otherwise it is the Batchifier row's own `batchifierWayForward`.
A seed failure is reported first and unchanged, so the batcher check never masks it.
Bound: this validates `batcher.yaml` only, never another module's config; it reads and never writes.
`loompreflight_test.go`, whose existing tests each pin one seed outcome, gains `TestLoomPreflight_Call_BatcherConfig` (`test-budget.yaml` raises `internal/loomshed` by one), a table over a valid config (Done), a profile carrying `master_base` (Stuck with the reconcile way forward) and a malformed profile (Stuck with the fix-and-re-step way forward); the other call sites gain the new argument, and `simpleEntryCases` in `entries_simple_test.go` adds `AnchorPath` to the `LoomPreflight` row's `validatedFields`.
`refusal-spec.md`'s loom section gains the Loom-Preflight batcher row with both ways forward, and the loomshed package doc names the check.

**ImpactSummary:** Adds a constructor parameter to NewLoomPreflight, threaded from its one registry entry; every test constructing it gains the argument.
