# Card 11 — anomalies-under-step

**Edit:**
- `internal/loomcli/cli.go`
- `internal/loomcli/arm.go`
- `internal/loomcli/halt.go`
- `internal/loomcli/halt_test.go`
- `internal/loomcli/step_test.go`

**Uses:**
- `internal/loomcli#detectAndFileAnomalies`
- `internal/loomcli#observeEntry`
- `internal/selfreportengine#CreateIssue`
- `internal/loomengine#LoomSelfreportFiled`
- `internal/loomengine#LoomStepHandoff`
- `internal/shedengine#ErrShedBusy`
- `internal/testkit/shedfake#Shuttle`

**Intent:**
Tier 1 anomaly filing (`detectAndFileAnomalies`) runs under `step` as under `run`, so a driven run files its escalation-to-human, bounce-budget-exhausted, producer-hard-failure, recurring-finding and crash-resume anomalies with no operator in the loop (PR-Gate F1, operator addition (a)).
Today only `loomPostRun` calls it, and every driven run uses `step`.

Filing seam: `loomCLI` gains `fileIssue func(title string, body *string, labels []string) (string, int, error)`, matching `selfreportengine.CreateIssue`.
`newLoomCLI` assigns `selfreportengine.CreateIssue`, except under `testing.Testing()`, where it assigns a filer that returns an error and files nothing, as `suppressWatchdogSpawn` does for watchdog spawns, so no test binary files a real issue.
Tests that pin filing set their own fake.

`arm.go`:

- A new `(c *loomCLI) anomalyDeps(ctx context.Context, runErr error) selfreportDeps` builds the deps `loomPostRun` builds today, with `FileIssue: c.fileIssue`; `loomPostRun` calls `detectAndFileAnomalies(c.anomalyDeps(ctx, runErr))`.
- A new `(c *loomCLI) observeStepEntry()` sets `c.entryObservation` through `observeEntry` with the same arguments `loomPreRun` passes.
  `loomPreStepUnmarked` calls it last, after the busy probe, the seed-and-commit bootstrap and `reed.Up()` have all succeeded, just before it returns.
  It sits there so a step that `PreStep` refuses, busy or at the bootstrap, neither observes nor consumes the step clean-handoff marker, and only a step that goes on to `BuildShed` spends it.
  It still precedes `shed.Step`, so the previous step's completed aftermath matches that marker rather than reading as a crash.
  For a seeded run the bootstrap leaves the status file untouched (`loomshed.Seed` returns `ErrSeedExists`), so the later observation reads what an entry observation would, and it follows the ownership check as `loomPreRun`'s does.
  `loomPreStep`'s doc names the observation and its placement.

`halt.go`:

- `loomAfterStep` calls `detectAndFileAnomalies(c.anomalyDeps(ctx, stepErr))` first, on every path, ahead of the halt note and reflection.
  The existing skips hold unchanged: the knob off, `shedengine.ErrShedBusy` and a cancelled context (filing only a lone crash-resume) each file nothing more.
- `haltNote` gains `TierOneFiled bool`, and `reflectHalt` sets it from `c.cfg.Selfreport`, so both hooks set it alike.
  When it is set, `writeHaltNote` adds the line `Tier 1 anomaly filing files the halt event itself; file only the lyx problems behind it.` after the reason block.
- The file's header comment states which path owns which filing: Tier 1 files the halt event, and the reflection files only the lyx problems behind it, so one halt is not filed twice.

What this lets through, and the bound: a driven step now files public issues through `selfreportengine.CreateIssue` with no operator in the loop, per the operator's decision; it files only what `detectAndFileAnomalies` detects, behind the `selfreport` knob, deduplicated by the filed-title marker, and never on a busy step or for a step `PreStep` or `BuildShed` refused.
Two residuals stay, and they are the ones `run` already has after the same failures.
A step refused at `BuildShed`, or one whose `shed.Step` error left the run `running` (a status read failure, an unknown `current_producer`, a persist failure), has spent the clean-handoff marker without `PostStep` writing a fresh one, so the next step files one `crash-resume` issue, titled by producer and history length so it files once.
With the knob on, the halt note's Tier 1 line is written even when this pass's filing of the halt failed (card 10 keeps it pending) or was skipped by a context cancelled after `shed.Step` returned; a failed filing is retried from the pending list by the next Tier 1 pass, a skipped one is detected again by the next pass only while the run is still halted, and neither is filed if no later `run` or `step` runs in that worktree.
The `fileIssue` seam lets a test replace the filer; production code assigns it only in `newLoomCLI`.

`halt_test.go`: `haltFixture` gains a recording fake on `c.fileIssue` that appends to an event log the shuttle fake's `DuringRun` also appends to, and the Tier 1 cases turn `cfg.Selfreport` on.

- The generic `step` body (`stepEnvelopeOver`) over a producer that returns Stuck files one `escalation-to-human` issue naming the halted producer before the reflection spawns, its halt note carries the Tier 1 line, and the envelope's `friction` reads `reflected`; stepping the same halt again files nothing new.
- The generic `step` body over a producer that returns an error files one `producer-hard-failure` issue before the reflection spawns.
- `loomAfterStep` with a crash-resume entry observation on `c.entryObservation` and a `running` result files one `crash-resume` issue, writes no halt note and spawns nothing.
- The real handoff sequence: the generic `step` body runs twice, with `PreStep` set to a hook that calls `c.observeStepEntry()`, over a shed whose first row returns Done and leaves the run `running`, the status seeded once rather than on each call as `stepEnvelopeOver` does today.
  The second step files no `crash-resume`; the same two steps with the clean-handoff marker removed between them file one.
- `loomAfterStep` with a busy error, with the knob off, and with a cancelled context and no crash files nothing.
- With the knob off the halt note carries no Tier 1 line.
- `loomPostRun` on `RunBlocked` with the knob on files through `c.fileIssue` and writes the Tier 1 line.

`step_test.go`'s `TestStepCmd_BusyRefusal_BeforeBootstrap` turns the knob on and seeds a step clean-handoff marker, and also asserts that the busy refusal leaves the marker in place and `c.entryObservation` zero.

**ImpactSummary:** confined to `internal/loomcli`: one new seam field set in `newLoomCLI`, the entry observation added at the end of `loomPreStepUnmarked` through one new helper, `loomAfterStep` gaining the Tier 1 call, and one optional line in the halt note; `loomPostRun`'s filing moves onto the seam with unchanged behaviour.
