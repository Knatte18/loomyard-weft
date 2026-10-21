# Card 8 — gate-tier-verb

**Create:**
- `internal/verifyrun#ShellFor`
- `internal/gatecli#tierRun`
- `internal/gatecli#planTierRuns`
- `internal/gatecli#renderTierCommand`
- `internal/gatecli#runTier`
- `internal/gatecli#tierResult`
- `internal/gatecli#runGuardStep`
- `internal/gatecli#TestPlanTierRuns`

**Edit:**
- `internal/verifyrun#run`
- `internal/verifyrun#QuoteArg`
- `internal/gatecli#Command`
- `internal/gatecli#newCommand` -> `func newCommand(goBinary string, samplePeriod, slotWait, preTierCeiling, timeoutUnit time.Duration) *cobra.Command`
- `internal/gatecli#testRequest`
- `internal/gatecli#runTest`
- `internal/gatecli#acquireSlot` -> `func acquireSlot(ctx context.Context, out io.Writer, pool *gateslot.Pool, location *lyxcwd.Location, request testRequest, tier string, count int) (*gateslot.Lease, int)`
- `internal/gatecli#answerSlotBusy`
- `internal/gatecli#guardScanRefusal`
- `internal/gatecli#runGateHelper`
- `internal/gatecli#runStandIn`
- `internal/gatecli#standInRecord`
- `internal/gatecli#TestGateTest_Scenario`
- `internal/gatecli#TestGateTest_RefusesBeforeResolvingTheHub`
- `internal/gatecli/doc.go`
- `docs/overview.md`

**Delete:**
- `internal/verifyrun#shellFor`
- `internal/gatecli#goTestArgs`
- `internal/gatecli#missingPackageDirs`
- `internal/gatecli#isRelativePackageArg`
- `internal/gatecli#isDirectory`
- `internal/gatecli#refusePackages`

**Uses:**
- `internal/gatecli#buildRoot`
- `internal/gatecli#runTiedChild`
- `internal/gatecli#watchdogApplies`
- `internal/gatecli#newWatchdog`
- `internal/gatecli#newCounter`
- `internal/gatecli#resolveHub`
- `internal/gatecli#absoluteDir`
- `internal/gatecli#catchTerminatingSignals`
- `internal/gatecli#exitAfterSignal`
- `internal/gatecli#exitCodeOf`
- `internal/gatecli#startGate`
- `internal/gatecli#runGate`
- `internal/gatecli#eventually`
- `internal/impactset#ModuleGuardTests`
- `internal/gateslot#Pool.AcquireSlots`
- `internal/gateslot#Lease.InheritEnv`
- `internal/gateslot#WriteRunRecord`
- `internal/gateslot#TierTimeoutLine`
- `internal/gateslot#TierSummaryLine`
- `internal/hubforge#CopyHub`

**Intent:**
`lyx gate test` stops being a `go test` wrapper and runs the hub's tiers (D2), with explicit targets and `--all` as its scope forms in this card; card 9 adds the computed, `--base` and `--card` forms.

The verb is `lyx gate test [-C <dir>] [--tier N]... [--card <id>]... [--base <rev>] [--all] [--guards] [--wait] [<targets>...] [-- <args>]`.
`--tags` stays a registered flag only to be refused before any slot, naming `--tier`.
`--card` and `--base` are registered here and refused as not yet supported until card 9 wires them; no Go site or stencil names either form before card 9.
`planTierRuns`, pure over a loaded `gateslot.Config` and the request, builds the run list: the `--tier` values, default `[0]`, ascending and deduplicated, each resolved through `Config.Tier`; a listed tier another listed tier `covers` is dropped and named as covered on stderr; with explicit targets each run's `{targets}` is those targets verbatim and its directory the `-C` directory, else the tier's `dir`; with `--all`, or with no scope form under this card, `{targets}` is the tier's `all_targets`.
The flag-shape refusals need no config and run first in `runTest`, before hub resolution: `--tags`, naming `--tier`; `-C` with any scope form but explicit targets, naming explicit targets; and more than one scope form.
`planTierRuns` then refuses, before any slot: words after `--` for a tier whose command lacks `{args}`, naming the tier and `lyx config gate`; an undeclared tier, a hub with no `tier0`, and a `covers` entry or a `guard_tier` naming a tier the hub does not declare, each naming `lyx config gate` (D1's checks at use).
Every refusal that names `lyx config gate` ends with the role clause of the overview's shared decision, so a fork that meets one reports status: FAILED rather than editing config.
`renderTierCommand` fills `{targets}`, `{args}` and `{parallel}` with each target and each arg quoted through `verifyrun.QuoteArg`; a command without `{targets}` runs whole whatever the scope and says so on stderr.

Each tier runs in turn, stopping at the first failing tier with its exit code.
A tier first takes its slots through `AcquireSlots` with its `slots` count and a holder naming the tier, writing the wait record while it waits as today, and releases them before the next tier; inside an inherited slot (`gateslot.InheritEnv` naming a held slot of the pool) it acquires nothing, whatever it asks for.
Without `--wait` the slot wait keeps the keep-cache-warm cap and answers `slot_free: false` with `SlotBusyExit` as today; `--wait` lifts the cap, so the call blocks until its slots free.
The tier's `timeout_min`, counted once its slots are held, bounds every invocation of the tier and its `prebuilt_lyx` build; a tier that outlives it has its process tree killed through `runTiedChild`'s context and fails with exit code 1 and `gateslot.TierTimeoutLine` on stderr and in its log.
The work before the first tier is bounded by `preTierCeiling`, 10 minutes in production, failing with a timeout line naming the pre-tier step and the way forward to run the same command again, which a transient hang such as a stalled `go list` passes.
`timeoutUnit` is one minute in production; a test passes a short one so a `timeout_min: 1` tier times out in milliseconds (`PATTERN-test-speed`).
The command runs through `verifyrun.ShellFor`'s shell as a tied child in the run directory, with this process's environment less the strand-name and trace-id variables (as today), less an inherited `PrebuiltLyxEnv`, plus the tier's `env` with `{parallel}` filled, plus the lease's `InheritEnv` and the enclosing-gate pid.
With `prebuilt_lyx` the tier first runs today's `lyx` prebuild at the tier's `parallel`, inside its slots and timeout, only where the worktree root holds `cmd/lyx`, and exports `PrebuiltLyxEnv`.
The CPU watchdog applies as today; it watches `.test` processes only, so a `vstest` tier is bounded by its `timeout_min` alone.
Output goes through the tier's counter (card 7) to stdout and into the run's log buffer; after the tier, the verb prints `gateslot.TierSummaryLine` to stderr, naming tier, scope form, counts, wall seconds, exit code and log path, and writes the run record and log through `gateslot.WriteRunRecord` under `RunLogDir(Dir(boardDir))` inside a hub or `RunLogDir(WaitDir(worktreeRoot))` outside every hub; a write failure is logged and decides nothing.
The record's slug is the one the pair's fabric origin record names, empty for the prime and outside every hub.
Zero discovered passes.

`--guards` keeps its meaning under the `go` scoper: after the listed tiers pass it runs the guard step, `runGuardStep`, over `impactset.ModuleGuardTests` of the module at the run directory: one run of `guard_tier`'s command with the guard packages as `{targets}` and `-run '^(<names>)$'` after any words from `--` as `{args}`, acquiring `guard_tier`'s slots after the last listed tier released its own, under its `timeout_min` and `prebuilt_lyx`, logged as the guard step.
It is skipped when the call ran `guard_tier`, or a tier covering it, whole.
A guard step that is not skipped refuses a `guard_tier` command lacking `{targets}` or `{args}`, naming `lyx config gate`; a scan error refuses before any slot with `guardScanRefusal`'s file, line and way forward, reworded to name the guard step instead of `--guards`; `--guards` under the `none` scoper is refused naming the scoper.
A module with no marked test runs no guard step and logs so.

Inside a hub the config is `gateslot.LoadConfig` over the hub; a hub with no tier is refused naming `lyx config gate`.
Outside every hub, in a git worktree whose hub carries no board, the worktree's own `gate.yaml` is read through the same loader with only the anchor told, and the run is unslotted; with none, the call is refused naming `lyx config gate` (card 2).
A directory in no git worktree has no worktree layer to read, so it is refused naming `lyx config gate` run in a git worktree.

`goTestArgs` and the package-directory check go, since targets pass verbatim to a command lyx does not spell; `isDirectory` and `refusePackages` go with them.
`shellFor` becomes the exported `ShellFor` so the gate and `verifyrun.Run` start one shell the same way.
`doc.go` is rewritten for the tier verb, its refusals, slots, timeouts, counters, log, inheritance, watchdog and exit codes, with loomyard's tier block from D12 as the documented example; `docs/overview.md`'s gate module rows describe the tier runner.

Bound: explicit targets and `--all` change only which targets a tier runs over, never the slot rules; `--wait` lets a caller block one call as long as the hub is busy, and costs only that caller's prompt cache.
A tier command is a hub-file-only key (card 2), so no worktree or task layer injects one inside a hub.

Tests: the new `TestPlanTierRuns` covers the pure run list: the default `[0]`, ascending order, a covered tier dropped, explicit targets with `-C`, a tier's `dir` as the run directory without `-C` and `.` by default, `--all` taking `all_targets`, the guard step planned after the listed tiers and skipped when the call runs `guard_tier` whole or a tier covering it, and each refusal of `planTierRuns` with its named way forward and role clause: words after `--` for a tier without `{args}`, an undeclared tier, no `tier0`, a dangling `covers` entry, a dangling `guard_tier`, a non-skipped guard step whose `guard_tier` command lacks `{targets}` or `{args}`, and `--guards` under the `none` scoper.
`TestGateTest_RefusesBeforeResolvingTheHub` swaps the package rows for the flag-shape refusals, which run before hub resolution and need no config: `--tags` naming `--tier`, `-C` with `--all`, and `--all` with explicit targets.
`TestGateTest_Scenario` seeds `gate.yaml` with tiers whose command runs the stand-in through `{targets} {args}` and keeps its rows recast as tier runs: exit code passthrough, environment drops, slot busy without `--wait`, the wait with `--wait` lasting past the keep-cache-warm cap the helper sets and passing once the holder releases, inheritance, `-C` from outside the hub, signals, SIGKILL, prebuild and its failure, and the `--guards` rows through `guard_tier`; its "a real go test runs in the nested module" row now runs a `go test -json` tier over a package with only a `TestMain` and asserts the pass with zero discovered.
The outside-every-hub row moves into a standalone git worktree with no board: with its own `gate.yaml` it runs unslotted and writes its record under that worktree's `.lyx` gate `runs` directory, and without one it is refused naming `lyx config gate`; a cwd in no git worktree is refused the same way.
It gains rows for: a two-tier call stopping at the first failing tier; a `slots: all` tier holding every slot; a command without `{targets}` running whole under explicit targets; a tier whose `env` sets `GOFLAGS: "-trimpath -p={parallel}"` reaching the stand-in as `-trimpath -p=3` for a `parallel: 3` tier; a `timeout_min` tier killed with the timeout line; a pre-tier ceiling expiry naming the re-run; the stderr summary line and the run record and log written under the board; a hub with no tier refused.
`standInRecord` and `runStandIn` record the stand-in's argv as the tier command passes it, and `runGateHelper` passes the short ceiling and timeout unit.

**ImpactSummary:** the verb's whole contract changes; every Go site still runs its own command until cards 10 to 15 render tier calls, and the help text, doc and overview move with it.
