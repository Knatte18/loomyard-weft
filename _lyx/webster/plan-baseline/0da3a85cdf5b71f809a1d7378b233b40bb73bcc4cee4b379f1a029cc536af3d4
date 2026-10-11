# Card 8 — land-producer

**Create:**
- `plan:internal/battenshed#LandDeps` -> `type LandDeps struct`
- `plan:internal/battenshed#LandingSettings` -> `type LandingSettings struct`
- `plan:internal/battenshed#DeployResult` -> `type DeployResult struct`
- `plan:internal/battenshed#landProducer` -> `type landProducer struct`
- `plan:internal/battenshed#NewLand` -> `func NewLand(name, slug string, deps LandDeps, primeLock PrimeLock, scratchDir string) shedengine.ShedProducer`
- `plan:internal/battenshed#landProducer.Call` -> `func (p *landProducer) Call(ctx context.Context) (shedengine.Outcome, shedengine.OutputPointer, error)`
- `plan:internal/battenshed#landProducer.acquireLandLock` -> `func (p *landProducer) acquireLandLock(ctx context.Context) (release func() error, stuckReason string, err error)`
- `plan:internal/battenshed#landProducer.waitCleanPrime` -> `func (p *landProducer) waitCleanPrime(ctx context.Context) (stuckReason string, err error)`
- `plan:internal/battenshed#landProducer.newlyReady` -> `func (p *landProducer) newlyReady() (ready []string, stuckReason string)`
- `plan:internal/battenshed#rowNotifier.ready` -> `func (n rowNotifier) ready(ctx context.Context, landed string, slugs []string)`
- `plan:internal/battenshed#readySnapshotFileSuffix` -> `const readySnapshotFileSuffix = "-ready-snapshot"`
- `plan:internal/battenshed#landLockPollBase` -> `const landLockPollBase = time.Second`
- `plan:internal/battenshed#landLockPollCeiling` -> `const landLockPollCeiling = 30 * time.Second`
- `plan:internal/battenshed#primeDirtyWaitBound` -> `const primeDirtyWaitBound = 10 * time.Minute`
- `plan:internal/battenshed#primeDirtyPollInterval` -> `const primeDirtyPollInterval = 5 * time.Second`
- `plan:internal/battenshed#deployTimeout` -> `const deployTimeout = 30 * time.Minute`
- `internal/battenshed/land_test.go`

**Edit:**
- `plan:internal/battenshed#runMarkerSuffixes`
- `internal/battenshed/doc.go`
- `cmd/lyx#unboundedWaitFiles`
- `pattern/PATTERN-daemon-wakeups.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/battenshed#RowNoticeDeps`
- `plan:internal/battenshed#newRowNotifier`
- `plan:internal/battenshed#rowNotifier.failed`
- `plan:internal/battenshed#rowNotifier.send`
- `internal/battenshed#PrimeLock`
- `internal/battenshed#acquirePrimeLock`

**Intent:**
The Land producer of D1, in a new file `internal/battenshed/land.go`, standalone: card 9 registers it and puts its row into the batten recipe.

`LandDeps` carries every outside effect as a seam, so battenshed's import allowlist does not grow and every path and command is told (PATTERN-told-geometry):
`ParentBranch func() (string, error)`, the child's recorded parent branch;
`PrimeBranch func() (string, error)`, the prime's checked-out branch;
`PrimeDirty func() ([]string, error)`, the prime code worktree's uncommitted tracked paths;
`Pull func(ctx context.Context) error`, fabric's pull of the prime;
`Landing func() (LandingSettings, error)`, the hub-wide landing config's `Deploy` and `RequirePRToBase`, read on every Call;
`RunDeploy func(ctx context.Context, command, logPath string, timeout time.Duration) (DeployResult, error)`, which runs the command in the prime with its output to `logPath`, and whose `DeployResult` holds `ExitCode` and `TimedOut`, while an error means the shell could not start;
`ReadyTasks func() ([]string, error)` and `PruneBoard func() error`;
`LandLock PrimeLock`, the hub-level land lock, the same non-blocking acquire shape as the prime lock;
`PauseRequested func() (bool, error)` (nil reads as never paused), `Sleep` (nil is `waitOrCancel`) and `Now` (nil is `time.Now`);
and `Notices RowNoticeDeps`.

`landProducer.Call` runs the steps of D1 in order, each idempotent, so a resume re-runs the row from the top:
1. It takes the land lock for its whole body through `acquireLandLock`, which polls `LandLock.Acquire` without a time limit, from `landLockPollBase` doubling to `landLockPollCeiling`, logging one Info on first contention (PATTERN-daemon-wakeups); a pause request seen between two polls ends the wait with the pause Stuck of the Shared Decision `land-row-pause`, a `PauseRequested` read error ends it with a Stuck naming the land lock (Shared Decision `land-steps-stuck`), and a cancelled context ends it with `cancelErr`.
2. Prime check: an empty `ParentBranch` answer, an origin record carrying no parent, is Stuck naming the prime check, since it cannot be compared with the prime's branch; when `ParentBranch` differs from `PrimeBranch`, pull and deploy are skipped with an Info log and the prime's tree is never read; otherwise `waitCleanPrime` re-reads `PrimeDirty` every `primeDirtyPollInterval` until it is empty, for at most `primeDirtyWaitBound`, honouring a pause, a `PauseRequested` read error (a Stuck naming the prime check) and cancellation as above, and past the bound is Stuck naming the paths.
3. Pull: under the prime lock taken with `acquirePrimeLock`'s existing bounded wait and released right after, `Pull` runs; a refusal, a diverged prime included, is Stuck with fabric's reason, and a spent prime-lock wait is Stuck with its reason. Batten never pushes the prime (PATTERN-push-both-sides).
4. Deploy, outside the prime lock: when the pull ran, `Landing().Deploy` is non-empty and the parent branch is in `RequirePRToBase`, `RunDeploy` runs it with `deployTimeout` and the log `<scratchDir>/<row>-deploy.log`; a parent outside the list logs the skip at Info, and an empty command runs and logs nothing. A non-zero exit, a timeout or a shell that could not start is Stuck naming the exit code or the timeout, the log path and the way forward.
5. Ready snapshot: unless `<scratchDir>/<row>-ready-snapshot` exists, `ReadyTasks`' answer is written to it, one slug per line, before the prune.
6. `PruneBoard` runs.
7. `newlyReady` reads `ReadyTasks` again and returns the slugs not in the snapshot, sorted; when any, `rowNotifier.ready` queues `[batten notice] <slug>: landed; newly ready: <slugs>` once per run, recorded as a `ready` line in the row-notice marker only once queued, and a dropped one is a Warn.
The row then returns Done.

Every failed step, a seam read error included, is Stuck naming the step through `reportStuck`, and every Stuck but a pause calls `rowNotifier.failed` with the step, so a failure halts batten blocked at the row with the pair intact and sends one notice per episode.
A pause sends no notice, since the operator asked for it.
The pause Stuck's reason says the pause was honoured and names the way forward `lyx batten goto <slug> --to Land`, then `lyx batten run <slug>`: shed's blocked persist leaves `pause_requested` set, so a bare `lyx batten run <slug>` would only record the run paused, while the goto clears the flag and leaves the run paused at Land for the run to re-enter the row.
Persisting the snapshot before the prune keeps the newly ready set right across a restart after the prune, and the land lock makes one landing's snapshot see every earlier landing's prune, so a task freed by another landing's prune is named exactly once.
`readySnapshotFileSuffix` joins `runMarkerSuffixes`, so a fresh seed clears it.

`cmd/lyx#unboundedWaitFiles` lists `internal/battenshed/land.go`, and `pattern/PATTERN-daemon-wakeups.md`'s list of unbounded waits names the Land row's land-lock wait.
`internal/battenshed/doc.go`'s opening names the Land producer among the package's producers, and a new paragraph states its steps, the two locks and what each holds, the prime check, the dirty-prime bound, the deploy bound, the ready snapshot, the notices, and that a pause during its waits halts the row blocked rather than paused, since the row has no `on_stuck` route to yield through, with `lyx batten goto <slug> --to Land` then `lyx batten run <slug>` as the way back into the row.

Bound: the row only lands a run whose Run-Shed returned Done (card 9 wires it so); the deploy runs only from it, only in the prime, only after a pull, only for a parent branch in `require_pr_to_base`, and only under the land lock; the dirty-prime wait is bounded, and the land-lock wait is bounded by pause and cancellation only.

Tests, in `internal/battenshed/land_test.go`, with fake seams, a fake clock and a recording Notify:
a table test of the landing dispositions: parent equal to the prime's branch pulls, deploys once and prunes with no notice; another parent skips pull and deploy, never reads `PrimeDirty`, and prunes; a parent outside `RequirePRToBase` pulls and skips the deploy; an empty deploy runs nothing;
a table test of the failing steps, each Stuck with the pair untouched and one notice naming the step: a non-zero deploy and a timed-out one (both naming the log path), a deploy shell that cannot start, a pull refusal, a spent prime-lock wait, a `ReadyTasks` error at the snapshot, a prune failure, a `ReadyTasks` error at the read after the prune (whose re-entry, with the snapshot present, then sends the ready notice), a read error from each of `ParentBranch`, `PrimeBranch`, `PrimeDirty` and `Landing`, a `PauseRequested` read error during the land-lock wait and during the dirty-prime wait, an empty `ParentBranch` answer, a `LandLock.Acquire` error, and a ready-snapshot marker that cannot be written or read back (its scratch path occupied by a directory); with a re-entry at the same history length sending no second notice;
a test of the dirty-prime wait: clean within the bound lands, dirty past it is Stuck naming the paths, and a pause during it is Stuck with no notice and a reason naming the goto and run way forward;
a test of the land-lock wait: it polls until the lock frees with sleeps doubling from the base to the ceiling, the prime lock is held only around the pull and is free during the deploy, a pause ends it with no notice, and a cancelled context is a hard error;
a test of the ready notice: a landing that empties a dependant's set sends one notice naming it, one that readies nothing sends none, a restart after the prune with the snapshot present sends it, a re-entry after it was queued does not repeat it, and a task already in the snapshot is not named.
The new top-level tests raise `internal/battenshed`'s row in `cmd/lyx/testdata/test-budget.yaml`.

**ImpactSummary:** a new, not yet registered producer plus one marker suffix, the daemon-wakeups scan list and its background; no existing producer changes.
