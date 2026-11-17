# Card 9 — batch-attempt-strand

**Create:**
- `internal/weaverengine#StrandRunner`
- `internal/weaverengine#attemptSpec`
- `internal/weaverengine#attemptLaunch`
- `internal/weaverengine#batchGateCalls`
- `internal/weaverengine#batchGateSpec`
- `internal/weaverengine#attemptVerdict`
- `internal/weaverengine#classifyAttempt`
- `internal/weaverengine#StrandStopError`
- `internal/weaverengine#stopAttempt`

**Edit:**
- `pattern/PATTERN-gate-slot.md`
- `pattern/PATTERN-test-speed.md`
- `internal/gateslot/doc.go`
- `cmd/lym#planVerifySites`

**Extends:**
- `internal/weaverengine/doc.go`

**Uses:**
- `internal/runplan#ParseReport`
- `internal/gateslot#MainPackagesFlag`
- `internal/shuttleengine#Runner.StartGated`
- `internal/shuttleengine#Runner.ProbeGated`
- `internal/shuttleengine#Runner.StopStrand`

**Intent:** One batch attempt as one cold gated strand, and how its end is judged (D4, D6).

- `StrandRunner` is `StartGated`, `ProbeGated` and `StopStrand`, the three `*shuttleengine.Runner` methods Weaver drives; a test fakes it.
  The resume takes `ProbeGated`'s handle, not `AttachGated`, so it can deliver decisions before it waits (card 13).
- `attemptSpec` builds the spawn: `OutputFiles` the attempt's report path alone; `ModelChoice` the rung's choice, built only by `shuttleengine.NewModelChoice` (card 2); `ConfigRole` `agentconfig.RoleWebster`, the implementer role's skills; `Skills` the hub's language skills; `Role` `BatchStrandRole`; `Segment` `segmentcolor.Webster`; `Round` the batch label; a reed pane with no operator (autonomous).
- `batchGateCalls` returns the batch's own gate calls:
  one `card` call per card of the batch, in card order, the hub's `card` gate call over that card (`GateCall("card", <card id>)`);
  then one `batch` call over every card of the batch (`GateCall("batch", <card ids>...)`), with `" --" + gateslot.MainPackagesFlag` appended when `last` is set, so it also runs the module's main packages.
  A gate config without a `card` or `batch` key refuses with the config's way forward.
- `batchGateSpec` is the strand's must-pass gates, so the agent fixes its own gate failures before it ends (D4):
  one entry per call `batchGateCalls` returns, in its order, whose closure runs that call;
  then the row's `gates` entries unchanged.
  Each Weaver entry runs its call through `verifytree.Verify` in the geometry's gate slots with the geometry's pause hold, with the gate config's re-prompt budget.
- The verify-site docs and tripwire name Weaver's batch gates beside Webster's:
  `PATTERN-gate-slot.md`'s Verified-tree site list names the Weaver `card` and `batch` strand gates;
  `PATTERN-test-speed.md`'s Gate keys table names Weaver beside Webster on the `card` and `batch` rows;
  `gateslot/doc.go`'s tier plan gains Weaver's per-card and per-batch rows;
  `planVerifySites` gains the weaverengine file that declares `batchGateSpec`.
  Card 11 adds the plan-level verify's site and the `webster` key's Weaver use.
- `classifyAttempt` judges a waited result into one verdict:
  a parsed `OK` report with every gate passed is `recordable`;
  a `FAILED` report, a gate failing after its re-prompts, `OutcomeTimeout`, and a result with no report file are `failed`, with the cause;
  a `NEEDS_PARENT` report is `needs_parent` with its `need`;
  `OutcomeDied`, `OutcomeAPIError` and a wait error from reed are `lost`, a machinery loss;
  a report that does not parse is `failed`, naming the parse error.
  The attach's not-found answer never reaches it: the resume judges that itself (card 13).
- `stopAttempt` stops the strand through `StopStrand`; a failure returns a `*StrandStopError` whose text names the guid and `lym reed remove <guid>` as the way forward.

These unexported helpers are tested through `Run`: card 12's `TestRun` holds each verdict's and the gate spec's rows.

**ImpactSummary:** Adds to the new weaverengine package's doc, names Weaver's batch gates in two pattern docs and gateslot's tier plan, and adds one plan-verify site to the tripwire's list; no existing symbol's behaviour changes.
