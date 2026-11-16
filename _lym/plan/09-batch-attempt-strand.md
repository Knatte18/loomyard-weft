# Card 9 — batch-attempt-strand

**Create:**
- `plan:internal/weaverengine#StrandRunner` -> `type StrandRunner interface`
- `plan:internal/weaverengine#attemptSpec` -> `func attemptSpec(in attemptLaunch) shuttleengine.Spec`
- `plan:internal/weaverengine#attemptLaunch` -> `type attemptLaunch struct`
- `plan:internal/weaverengine#batchGateSpec` -> `func batchGateSpec(geom Geometry, cards []planparser.Card, last bool, rowGates shuttleengine.GateSpec) (shuttleengine.GateSpec, error)`
- `plan:internal/weaverengine#attemptVerdict` -> `type attemptVerdict struct`
- `plan:internal/weaverengine#classifyAttempt` -> `func classifyAttempt(result shuttleengine.Result, waitErr error, reportPath string) attemptVerdict`
- `plan:internal/weaverengine#StrandStopError` -> `type StrandStopError struct`
- `plan:internal/weaverengine#stopAttempt` -> `func stopAttempt(runner StrandRunner, guid string) error`
- `plan:internal/weaverengine#TestClassifyAttempt` -> `func TestClassifyAttempt(t *testing.T)`
- `plan:internal/weaverengine#TestBatchGateSpec` -> `func TestBatchGateSpec(t *testing.T)`

**Extends:**
- `internal/weaverengine/doc.go`

**Uses:**
- `internal/runplan#ParseReport`
- `internal/shuttleengine#Runner.StartGated`
- `internal/shuttleengine#Runner.ProbeGated`
- `internal/shuttleengine#Runner.StopStrand`

**Intent:** One batch attempt as one cold gated strand, and how its end is judged (D4, D6).

- `StrandRunner` is `StartGated`, `ProbeGated` and `StopStrand`, the three `*shuttleengine.Runner` methods Weaver drives; a test fakes it.
  The resume takes `ProbeGated`'s handle, not `AttachGated`, so it can deliver decisions before it waits (card 13).
- `attemptSpec` builds the spawn: `OutputFiles` the attempt's report path alone; `ModelChoice` the rung's choice, built only by `shuttleengine.NewModelChoice` (card 2); `ConfigRole` `agentconfig.RoleWebster`, the implementer role's skills; `Skills` the hub's language skills; `Role` `BatchStrandRole`; `Segment` `segmentcolor.Webster`; `Round` the batch label; a reed pane with no operator (autonomous).
- `batchGateSpec` is the strand's must-pass gates, so the agent fixes its own gate failures before it ends (D4):
  one `card` entry per card of the batch, in card order, whose closure runs the hub's `card` gate call over that card (`GateCall("card", <card id>)`);
  then one `batch` entry over every card of the batch, with the module's main packages when `last` is set;
  then the row's `gates` entries unchanged.
  Each Weaver entry runs its call through `verifytree.Verify` in the geometry's gate slots, with the gate config's re-prompt budget.
  A gate config without a `card` or `batch` key refuses with the config's way forward.
- `classifyAttempt` judges a waited result into one verdict:
  a parsed `OK` report with every gate passed is `recordable`;
  a `FAILED` report, a gate failing after its re-prompts, `OutcomeTimeout`, and a result with no report file are `failed`, with the cause;
  a `NEEDS_PARENT` report is `needs_parent` with its `need`;
  `OutcomeDied`, `OutcomeAPIError` and a wait error from reed are `lost`, a machinery loss;
  a report that does not parse is `failed`, naming the parse error.
  The attach's not-found answer never reaches it: the resume judges that itself (card 13).
- `stopAttempt` stops the strand through `StopStrand`; a failure returns a `*StrandStopError` whose text names the guid and `lym reed remove <guid>` as the way forward.

`TestClassifyAttempt` is tier0: one row per verdict above (an `OutcomeTimeout` with no report file stands for the no-report case), and a failed stop's text naming the guid and `lym reed remove`.
`TestBatchGateSpec` is tier0, over a gate config fake: a two-card batch yields two `card` entries, then `batch`, then the row's `drift` entry, in that order; the last batch's `batch` call carries the main packages and another batch's does not; a config lacking `card` and one lacking `batch` each refuse naming the config's way forward.

**ImpactSummary:** Only adds to the new weaverengine package's doc; no existing symbol changes.
