# Card 9 — interrupted-kind-and-arming-refusal

**Create:**
- `internal/shedverbs#KindInterrupted`
- `internal/shedverbs#KindlessRefusal`
- `internal/shedverbs#ReportArmError`
- `internal/loomcli#hubConfigWayForward`

**Edit:**
- `internal/shedverbs#StepKinds`
- `internal/shedverbs/doc.go`
- `internal/shedcli#shedCLI.resolvePersistentPreRun`
- `internal/shedcli#armFromSeed`
- `internal/shedcli#recipes`
- `internal/loomcli#loomCLI.resolvePersistentPreRun`
- `internal/loomcli#loomCLI.resolveRunID`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli/wiring_test.go`
- `contracts/specs/refusal-spec.md`
- `PATTERN.md`
- `pattern/PATTERN-shed-verb-set.md`

**Rename:**
- `internal/shedverbs#TestStepKinds_IsExactlyFive` -> `internal/shedverbs#TestStepKinds_IsExactlySix`
- `internal/shedcli#TestArmFromSeed_MissingRunRefusalCarriesNoKindField` -> `internal/shedcli#TestArmFromSeed_RefusalEnvelopeKinds`

**Uses:**
- `internal/shedverbs#KindBootstrap`
- `internal/shedrun#MissingSeedMessage`

**Intent:** The kind vocabulary grows to six and a pre-step arming refusal carries a kind and a trace, D4's vocabulary change and D6.
`KindInterrupted` joins the const block as the kind of a child step that ended without an envelope; `StepKinds` lists six, the const block's comment and the renamed test say six, and the test's `want` gains the new constant.
Nothing in this card emits the kind yet; card 11's loop does.
`ReportArmError` lives in `internal/shedverbs`, beside the kind vocabulary, because both step subtrees arm in their own pre-run and `internal/loomcli` cannot import `internal/shedcli`.
It prints a bare `output.Err` envelope for a `KindlessRefusal` or for any verb but `step`, and returns the exit code; for `step` it logs the refusal with `logger.Warn`, then prints an `output.ErrFields` envelope carrying `kind: bootstrap` and `trace_file: logger.TraceFile()`, reported verbatim, so the arming refusal reaches the driver as a `bootstrap` error with a trace holding it instead of a kind-less line.
The `bootstrap` message is the error's text, followed, when that text names no `way forward:` clause of its own, by `; way forward: fix the cause this error names, then run the step again, escalating when the fix lies outside the repair verbs`, so every arming refusal on `step` names its way forward (`PATTERN-refusal-way-forward`), the unknown recipe, the raw seed read and the bare `armAt` and `wire` errors included, while the hub-config refusal keeps its own clause alone.
`KindlessRefusal` wraps an error whose envelope must carry no kind; its `Error` and `Unwrap` pass the inner error through.
In `internal/shedcli`, `armFromSeed` wraps its two kind-less refusals, the missing seed and the unsupported verb, in `KindlessRefusal`, and `resolvePersistentPreRun` reports every arming error through `ReportArmError`; `recipes`' comment says six.
In `internal/loomcli`, `resolveRunID` wraps its missing-seed refusal in `KindlessRefusal`, and `resolvePersistentPreRun` reports every arming error through `ReportArmError`, so `lyx loom step` answers an arming refusal the same way.
The missing-seed refusal stays kind-less on both subtrees, and a missing seed is never re-seeded by a driver.
`wire` wraps a failure of the hub-wide landing config load with `hubConfigWayForward`, so the missing-hub-config refusal names its way forward in the message; the driver's disposition for `bootstrap` is the repair path, whose repairs are a closed list of `lyx` verbs, and a config fix lies outside it, so the driver escalates with the way forward.
The renamed `TestArmFromSeed_RefusalEnvelopeKinds` becomes a table that renders `armFromSeed`'s refusal through `shedverbs.ReportArmError`, the render path the pre-run uses, instead of calling `output.Err` itself: a missing run on `step` prints the two keys `ok` and `error`; an unknown recipe on `step` prints `kind: bootstrap`, a `trace_file` and an error ending in the generic way-forward clause; an error already naming a way forward on `step` keeps its own clause and gains no second one; the same refusal on `status` prints no kind.
`wiring_test.go` gains a row in the default-config wire test's package where the landing config is absent from the hub directory and asserts the error ends in the way forward.
`refusal-spec.md`'s shed section gains the rows `step arming refused` (correctness halt, kind `bootstrap`, way forward: fix the cause the error names, then run the step again, escalating when the fix lies outside the repair verbs) and `hub config missing` (correctness halt, `lyx config reconcile --apply` in the hub, then the step again).
`PATTERN-shed-verb-set.md` states the six kinds and names `KindInterrupted` as the one the loop emits.
`PATTERN.md`'s shed-verb-set line becomes "Adding a run, step, status, pause or goto verb: the generic bodies live in `internal/shedverbs` only, which derives no path and closes the step kinds at six. (test)" with its background link, which card 8's room keeps under the Shared Decision `pattern-overview-cap`, at about 15874 bytes.
`doc.go` states the six kinds and the arming report.

**ImpactSummary:** `StepKinds` gains an entry no code branches on yet; both pre-runs keep every kind-less refusal kind-less and add `kind: bootstrap` only on `step`; `wire` only wraps one error; one overview line grows by a clause.
