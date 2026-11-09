# Make the eight longest serial internal/fabricengine tests parallel

`fabric-serial-phase-tests` · Task · enhancement, fabric, testkit · running · Drift-Expect

Depends on: [`testmap-verb`](design-testmap-verb.md)

## Seen (orch, 2026-11-09)

`docs/research/speed/fabricengine-shards-2026-11-09/report.md`, measured on OSL-1046 under its normal load: 46 of the package's 400 top-level tests are not marked parallel, and the serial chain they form is 143 s, 59.5 percent of the 242 s one-process wall.
The eight longest of them take about 84 s of that chain.
Several are serial because they set a package-level variable the production code reads: a push notifier (`installedPushNotifier`, `failedpush.go`), an origin call hook (`originCallHook`, `originreconcile.go`) and a gc threshold (`gcAutoThreshold`, `housekeeping.go`).
`docs/research/speed/fabricengine-per-test-times-2026-11-07.md` names the reason for several of the others.

## This task

Make these eight top-level tests parallel, and only these:

- `TestPushFailureClasses`
- `TestAnchorChange`
- `TestReconcileOrigin_Judge`
- `TestStoreHousekeeping`
- `TestWorktreeSeam_Events`
- `TestReconcileOrigin_Ended`
- `TestReconcileOrigin_Moves`
- `TestReconcile_MigratesMarkerlessWefts`

First read each one and name why it is serial: a package-level variable it sets, a process environment variable or working directory, or a shared path.
Where the cause is package-level state, the state moves into a parameter the test passes, such as a field of `SyncOptions` or of the options type the code path already takes, and the production path reads it from there.
The package-level variable goes away when no test needs it; one that stays must not be read by any path a parallel test reaches, or two parallel tests would see each other's value.
Every other test that sets the same variable moves with it in this task, so no test is left setting state a parallel test can read.
A test whose cause cannot be removed without a larger change stays serial, and the task names it and the cause in its landing note instead of changing more.

## Limits

- Nothing outside `internal/fabricengine` and its tests changes, except a caller of a function whose signature the task changes.
- No change to what any production path does; only where its inputs come from.
- No test is dropped or loosened, and no test outside the eight changes, except the ones that set the same package-level state.
- The tier 1 gate on `./internal/fabricengine` is green, run with `-race` if the gate offers it, since a missed shared state shows there.

## Measuring

The task does not bench.
A later run of `docs/research/speed/fabricengine-shards-2026-11-09/serial-share.sh` on the one-process run's events gives the new phase 1 against the 143 s above.
The figure is from OSL-1046, where Cortex XDR slows every new binary and file; another machine will show a smaller serial phase.
