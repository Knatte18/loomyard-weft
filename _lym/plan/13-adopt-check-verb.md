# Card 13 — adopt-check-verb

**Edit:**
- `internal/adoptcli#Command`
- `internal/fabricengine#AdoptScratchDir`
- `internal/lymdirs#AdoptDirName`

**Create:**
- `plan:internal/adoptcli#checkRecordName` -> `const checkRecordName = "check.json"`
- `plan:internal/adoptcli#checkOutName` -> `const checkOutName = "check.out"`
- `plan:internal/adoptcli#checkLockName` -> `const checkLockName = "check.lock"`
- `plan:internal/adoptcli#checkGateKey` -> `const checkGateKey = "adopt"`
- `plan:internal/adoptcli#checkProgram` -> `const checkProgram = "ADOPT"`
- `plan:internal/adoptcli#CheckRecord` -> `type CheckRecord struct`
- `plan:internal/adoptcli#CheckTier` -> `type CheckTier struct`
- `plan:internal/adoptcli#ReadCheck` -> `func ReadCheck(recordPath string) (CheckRecord, bool, error)`
- `plan:internal/adoptcli#checkDeps` -> `type checkDeps struct`
- `plan:internal/adoptcli#runCheck` -> `func runCheck(ctx context.Context, deps checkDeps) (CheckRecord, bool, error)`
- `plan:internal/adoptcli#hubCheckDeps` -> `func hubCheckDeps(prime *lymcwd.Location) checkDeps`
- `plan:internal/adoptcli#checkCmd` -> `func checkCmd() *cobra.Command`
- `plan:internal/adoptcli#TestRunCheck` -> `func TestRunCheck(t *testing.T)`
- `plan:internal/adoptcli#TestHubCheckDeps_StaleStampRunsTheTunedFinalizeTiers` -> `func TestHubCheckDeps_StaleStampRunsTheTunedFinalizeTiers(t *testing.T)`
- `plan:internal/adoptcli_test#TestCheckCmd_FromABeamExitsZeroWhileOneRuns` -> `func TestCheckCmd_FromABeamExitsZeroWhileOneRuns(t *testing.T)`

**Extends:**
- `internal/adoptcli/doc.go`

**Uses:**
- `plan:internal/gatecli#RunGate`
- `plan:internal/gatecli#TierObserver`
- `internal/adoptcli#suiteLogName`

**Intent:**
Decision 10: a hidden `lym adopt check` runs the hub's `adopt` gate in-process, records its progress in `check.json` and reports each tier and its end to the nearest live orch of the prime.

Files, all under `fabricengine.AdoptScratchDir` beside `suite.log`, never-tracked `.lym` files named only here beside their sibling (PATTERN-durable-vs-ephemeral-state): `check.json`, the progress record, written through `internal/state` with its own state lock; `check.lock`, the verb's own lock; `check.out`, which card 14's spawn points the check's stdout and stderr at. The doc comments of `fabricengine.AdoptScratchDir` and `lymdirs.AdoptDirName` name the adopt check's record, output and lock beside the suite's log.

`CheckRecord` holds the check's `PID`, `StartTicks` (`proc.StartTime`, as `runproc` keys its records), `Start`, the `Gate` key, `Tiers` (each a `CheckTier`: tier, outcome, passed, failed, skipped and wall seconds as it ends), `Running` (a pointer to the tier in flight, nil between tiers), `State` (`running`, `green` or `failed`), `EndedAt`, `FailedTier` (a pointer) and `Error`; every field `omitempty` where it can be absent, and an unknown key reads (PATTERN-tolerant-records). `ReadCheck` stats `check.json` first and reports false without taking a lock when it is absent, so a read on a hub whose adopt scratch directory does not exist yet creates nothing and fails nothing; otherwise it reads the record through `state.ReadJSONNoMkdir`, which creates no directory; a record that does not decode reads as absent with one `logger.Warn`, as `dobbyengine.ReadState` does, so a damaged file never blocks the next check from writing a fresh one.

`checkDeps` carries the paths, `Now`, `PID`, `StartTicks`, `Reconcile func() error`, `RunGate func(ctx context.Context, log io.Writer, observer gatecli.TierObserver) int` and `Notice func(line string) error`.

`runCheck`, in order, returning the final record and whether this check ran:
1. Takes `check.lock` with `lock.TryAcquireWriteLock`; when it is held, returns at once having written nothing: no `suite.log`, no `check.json`, no post.
2. Holding it, truncates `suite.log` by creating it, and writes a fresh running record.
3. Calls `Reconcile`; a failure ends the record `failed` with `Error` naming the error and the way forward `fix the cause, then run lym fabric reconcile in the prime and lym gate test --gate adopt`, posts that as the end notice, and runs no tier, so no check goes green on the template's tiers.
4. Calls `RunGate` with `suite.log` as its log and an observer: `Started` sets `Running` and rewrites the record; `Ended` appends the tier, clears `Running`, rewrites the record and posts `adopt check: tier<N> <green|failed>, passed <p> failed <f> skipped <s>, <wall>s`.
5. Ends the record `green` on exit 0, else `failed` with `FailedTier` the first failed tier, or, when the gate ended before any tier did, as on a refused key, no `FailedTier` and `Error` naming the gate's exit code. Either way it clears `Running`, since a tier that refuses after `Started` returns without `Ended`. It posts one end notice: green naming the tiers and `suite.log`, failed naming the failing tier, or the exit code when no tier ended, `suite.log` and the rerun `lym gate test --gate adopt` in the prime.
A post that fails is logged through `logger.Warn` and changes nothing.

`hubCheckDeps` fills the seams over the prime: `Reconcile` is `hubreconcile.Ensure` over `hubgeom.ReconcileGeometry(prime)`, an absent board-level lym dir reading as an error, since an adopt runs only in a hub; `RunGate` is `gatecli.RunGate` over the prime's anchor with key `adopt`, the log as both writers; `Notice` is `orchcli.NotifyFrom(prime, checkProgram, line)`, a post with no orch strand an error.

`checkCmd` is the hidden `check` verb, audience internal, no arguments: it resolves the prime from the working directory through `lymcwd.Resolve` and `battencli.ResolveBattenPrime`, as `lym adopt` does, so it runs from anywhere in the hub, runs `runCheck` and prints an ok envelope with the record's `state`, or `already_running: true` when the lock was held.
`Command` adds it and sets `Args: cobra.ArbitraryArgs` on `adopt`, so a standalone `RunCLI` still reaches `adopt <subject>`.
Bound: the hidden verb adds an entry point that runs the hub's gate, only the `adopt` key, in the prime; a beam named `check` cannot be named to `lym adopt` and is adopted through `--all`; the gate slots do not serialize two checks, the lock does, so one check at a time writes the record, the log and the posts.

Tests:
- `TestRunCheck` (tier0, temp directory, a fake `RunGate` that drives the observer through scripted tier records and returns a scripted code, recording `Notice`): two green tiers record both with their counts, clear `Running`, end `green` with `EndedAt`, and post two tier lines and one green end notice; a run whose tier1 fails ends `failed` with `FailedTier` 1 and an end notice naming tier1, `suite.log` and `lym gate test --gate adopt`; with a `Notice` that fails, the record still holds every tier and the end state; a failing `Reconcile` ends `failed` naming its error, never calls `RunGate`, and posts one notice; a check started over an existing green `check.json` and a non-empty `suite.log` writes a fresh record with its own pid and leaves the log empty before the gate writes; `ReadCheck` on a directory that does not exist reports false with no error and the directory still absent afterwards; with `check.lock` held by the test, `runCheck` reports it did not run, never calls `RunGate`, leaves `check.json` and `suite.log` byte-identical and posts nothing; inside the fake gate after `Started`, `ReadCheck` reads `running` with that tier in flight; a fake gate that returns 1 with no tier ended ends `failed` with no `FailedTier`, an `Error` naming exit 1, and an end notice naming it and `suite.log`; a fake gate that calls `Started` for tier0 and returns 1 without `Ended` ends `failed` with no `Running`; a `check.json` holding `{not json` reads as absent through `ReadCheck` and the check writes a fresh record over it.
- `TestCheckCmd_FromABeamExitsZeroWhileOneRuns` (tier1, `adoptcli_test`, a `hubforge` hub with one beam pair): with `check.lock` under the hub's adopt scratch directory held by the test, `RunCLIIn` from the beam's worktree with `check` exits 0 with an ok envelope carrying `already_running: true`, and writes no `check.json`; that resolves the prime from a beam, whatever the cwd.
- `TestHubCheckDeps_StaleStampRunsTheTunedFinalizeTiers` (tier1, a `hubforge` hub whose hub `gate.yaml` declares tier0 and tier1, tunes `finalize` to `[0, 1]` and holds no `adopt`, with its build stamp made stale): `runCheck` over `hubCheckDeps` with `RunGate` replaced by a fake that loads the hub's gate config and records `TierList("adopt")` reads `[0, 1]`, and the hub file then holds `adopt`.

`doc.go` gains a section on the check verb, its files, its lock, its record and its posts.

**ImpactSummary:** A new hidden subcommand and its files; `adopt` itself only gains the subcommand and arbitrary-args parsing, so its envelope and steps are unchanged until card 14.

**Commit:** 13: Add the hidden adopt check that runs the adopt gate and reports per tier
