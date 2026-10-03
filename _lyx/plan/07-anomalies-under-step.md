# Card 7 — anomalies-under-step

**Edit:**
- `internal/loomcli/arm.go`
- `internal/loomcli/cli.go`
- `internal/loomcli/halt.go`
- `internal/loomcli/selfreport.go`
- `internal/loomcli/halt_test.go`

**Uses:**
- `internal/loomcli#detectAndFileAnomalies`
- `internal/loomcli#observeEntry`
- `internal/loomcli#recordStepHandoff`
- `internal/loomengine#DetectAnomalies`

**Intent:**
Tier 1 anomaly filing runs under `step` as under `run` (operator addition relayed by the hub; see the overview), so a driven run files its escalation-to-human, bounce-budget-exhausted, producer-hard-failure, recurring-finding and crash-resume anomalies with no operator in the loop.

`loomPostRun`'s inline `selfreportDeps` literal moves into `(c *loomCLI) anomalyDeps(ctx context.Context, runErr error) selfreportDeps`, which both verbs call.
`loomCLI` gains a `fileIssue` field of `selfreportDeps.FileIssue`'s type, the filing seam a test fakes; `anomalyDeps` uses `selfreportengine.CreateIssue` when it is nil, so production wiring needs no change.

Entry observation: `loomPreStepUnmarked` sets `c.entryObservation = observeEntry(...)` with the same arguments `loomPreRun` passes, right after its early run-lock probe finds the lock free and before the bootstrap, so a step refused as busy observes nothing.
The clean-handoff marker `loomPostStep` writes after every completed step is what keeps this from filing a crash-resume on every step: each step's observation consumes the previous step's marker, so only a step whose predecessor died mid-producer (no marker written) observes a crash-resume.

Detection: `loomAfterStep` calls `detectAndFileAnomalies(c.anomalyDeps(ctx, stepErr))` first, before the halt note and reflection, on every path the hook sees, mirroring `loomPostRun`'s order.
Its three skips are unchanged and need no step-specific code: the knob off returns before any read, a busy `stepErr` returns before any read, and a cancelled context files only a crash-resume observed at entry.
Since the step's final status is re-read from the status file, a `blocked` or `failed` step files its halt anomaly at that step; the filed-title marker keeps a run that stays blocked across several steps, or rediscovers the same open ledger entry on every step, at one issue per title.

What this lets through, and the bound: Tier 1 now files public issues from every driven step with no approval, per the operator's decision; it is bounded by the existing `selfreport` knob, the busy and cancelled-context skips, and the filed-title marker's one-issue-per-title dedupe, and it files only through `selfreportengine.CreateIssue`.
Ownership against D2 stays as card 6 states it: Tier 1 files the halt event, and the halt note it shares the halt with tells the reflection not to refile it.

The `detectAndFileAnomalies` doc comment and the `selfreportDeps.Ctx`/`RunErr` field docs stop speaking of `shed.Run` alone and name the step error too.

`halt_test.go` gains cases driving `loomAfterStep` with a counting fake on `c.fileIssue`:

- a `blocked` step with the knob on files one escalation anomaly and writes the halt note carrying the ownership line;
- a second `blocked` step at the same halt files nothing new;
- a busy `stepErr` files nothing and observes nothing;
- a step whose predecessor completed (marker present) observes no crash-resume, and one whose predecessor wrote no marker files one;
- the knob off files nothing under step.

**ImpactSummary:** `internal/loomcli` only: `loomPreStepUnmarked` gains the entry observation, `loomAfterStep` gains the detection call, and `loomPostRun`'s deps literal becomes a shared helper with identical values.
