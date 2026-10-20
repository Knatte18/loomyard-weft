---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    origin: lens:affected-set
    location: _lyx/plan/07-land-weft-stop.md
    summary: Card 7 retypes LandDeps.Pull but misses two literals that fill it, internal/shedrecipe newTestEnv and internal/testkit/envkit FullEnv, which stop compiling
    fix: In card 7 add `internal/shedrecipe#newTestEnv` and `internal/testkit/envkit#FullEnv` to **Edit:**, give both the new Pull type returning an empty stop, and replace the sentence calling landFixture.deps and newFakeReceiver the two test literals with one naming all four; in 00-overview.md append `./internal/shedrecipe ./internal/testkit/envkit` to the `lyx gate test` verify line.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:guard-reach
    location: _lyx/plan/04-records-replay.md
    summary: finalizeEntered counts current_producer Finalize as entered, so the run-scope reconcile at Finalize entry never reads the run as ended and D8's Finalize ended Stuck with Halt can never fire, letting a run landed on another machine land again here
    fix: In card 4 change finalizeEntered so current_producer Finalize alone does not count as entered; count only a Finalize history row, a prime batten status at Land or later or done, or evidence Finalize itself writes once it has pushed its done mark (name the field or record and the card that writes it). In card 22 add a Finalize row over a real status file whose current_producer is Finalize with no history row and an entry done on origin, which is Stuck with Halt, and keep the row for a Finalize re-entered after its own done mark, which proceeds; update card 4's TestRecordsReplay row that sets current_producer Finalize with no history row to expect the entry ended.
  - id: F3
    severity: BLOCKING
    class: design
    origin: lens:way-forward
    location: _lyx/plan/21-batten-resume-reconcile.md
    summary: The ended way forward of cards 21 and 22 names lyx batten cleanup --apply, which never touches a pair and removes only a done slug's records, so the ended run's pair and record stay
    fix: In card 21 (resumeRefusal) and card 22 (reconcileVerdict) replace `lyx batten cleanup --apply` with `lyx cleanup --apply` run from the prime, whose hub reconcile tears down the ended entry through D6 (card 20); in card 22 keep pausing the run first; update the ended rows of TestRunCmd_ResumeDispositions, TestMergeIn_RunsStatusCommitThenLockThenSyncThenMergeAndVerify and TestFinalize_StuckReasons to assert the new way forward.
  - id: F4
    severity: BLOCKING
    class: scope
    origin: handler
    location: _lyx/plan/15-origin-reconcile-judge.md
    summary: lsRemotePair runs each attempt under a 20-second deadline, but gitexec bounds remote commands only by the package-level 10-minute remoteDeadline and has no per-call deadline, and no card targets gitexec
    fix: In card 15 add to **Create:** `plan:internal/gitexec#RunWithin` -> `func RunWithin(args []string, cwd string, deadline time.Duration) (string, error)` and `plan:internal/gitexec#runCoreWithin` -> `func runCoreWithin(args []string, cwd, stdin string, deadline time.Duration) (stdout, stderr string, exitCode int, err error)`, and to **Edit:** `internal/gitexec#runCore`, which calls runCoreWithin with remoteDeadline; state that lsRemotePair calls RunWithin with the attempt deadline, the gate hold still coming before the deadline; in 00-overview.md append `./internal/gitexec` to the verify line.
  - id: F5
    severity: BLOCKING
    class: design
    origin: handler
    location: _lyx/plan/09-push-retry-and-record.md
    summary: Card 9 injects the gone backoff's sleep as the package-level var goneBackoffSleep with SetGoneBackoffSleepForTest, which PATTERN-test-speed forbids and which forces TestPushFailureClasses off t.Parallel for a time seam
    fix: In card 9 drop `plan:internal/fabricengine#goneBackoffSleep` and `plan:internal/fabricengine#SetGoneBackoffSleepForTest` from **Create:**, add `internal/fabricengine#SyncOptions` to **Edit:** with a new `BackoffSleep func(time.Duration)` field (nil sleeps the PushWithRetry delays), have pushClassified read it from the caller's opts, and reword the test paragraph so TestPushFailureClasses sets opts.BackoffSleep (restoring the origin URL inside it) instead of the setter.
  - id: F6
    severity: MEDIUM
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/13-remove-local-mode.md
    summary: commitPendingRecords(try) cannot try-acquire the weft write lock, since CommitRecordsPaths takes it with a blocking acquire inside and a pre-acquire would block on the second flock, so the planned ErrPairBusy row has no edit producing it
    fix: In card 13 add to **Create:** `plan:internal/fabricengine#commitRecordsPathsHeld` -> `func commitRecordsPathsHeld(rec *Mutations, weftPath, anchorRel string, relPaths []string, msg string, opts SyncOptions) (sha string, committed bool, err error)` (CommitRecordsPaths' body without the lock) and to **Edit:** `internal/fabricengine#CommitRecordsPaths`, which takes the lock and calls it; state that commitPendingRecords with try true takes the weft write lock by one try-acquire itself, a held lock wrapping ErrPairBusy, and calls commitRecordsPathsHeld.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lyx/plan/13-remove-local-mode.md
    summary: Remove's deferred failure path calls respawnPairPush, an origin step, and card 13 moves that body into removePair without excluding it in local mode, so a failed local removal of an ended entry pushes its weft
    fix: In card 13's local-mode bullet list add that removePair skips respawnPairPush in local mode, leaving a surviving worktree's commits local for the next reconcile, and add a TestRemoveLocal row where a step after the teardown lock fails (through an existing hook) and the weft bare is unchanged.
  - id: F8
    severity: MEDIUM
    class: scope
    origin: lens:guard-reach
    location: _lyx/plan/09-push-retry-and-record.md
    summary: The gone and rewound checks and the failure record cover only pushClassified's callers; Publish's task-branch push through Fabric.PushBranch and PushWarpRebaseFreeAt pushes a task branch with an upstream unchecked, though D10 puts every fabric push and every task-branch push through the ls-remote
    fix: In card 9 add `internal/fabricengine#PushWarpRebaseFreeAt` to **Edit:** and state that it pushes through pushClassified on the code side, so its push runs the gone and rewound checks and records a failure; add a TestPushFailureClasses row where PushBranch over a task branch rewound on origin is not pushed and writes a rewound record.
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lyx/plan/18-reconcile-seams.md
    summary: HubSeamOptions fills HubPaused when the seams are built, before the reconcile's ls-remote waits at the gate for an operator pause's lift, so a driver the lift resumed still reads not live and the reconcile moves a live pair, against D4
    fix: In card 18 make SeamOptions.HubPaused a `func(slug string) (bool, error)` that HubSeamOptions fills with a read of the pause record at each call, state that LivePair reads it when called, and add a TestHubSeamOptions row where an entry lifted after HubSeamOptions returned reads not paused.
  - id: F10
    severity: MEDIUM
    class: scope
    origin: lens:affected-set
    location: _lyx/plan/05-bolt-pullfirst-replay.md
    summary: cmd/lyx TestStencilSeeding_HubScenario asserts the Bolt divergence skip and its Warn naming git pull --rebase, which card 5 replaces, and no card updates it though ./cmd/lyx is in verify
    fix: In card 5 add `cmd/lyx#TestStencilSeeding_HubScenario` to **Edit:** and state that its divergence step is re-read against the replay as TestEnsure_UnpushedHubWideCommitOverMovedUpstreamLeavesStampAbsent is, keeping a conflicting row that asserts the new Warn naming the path and `lyx board sync`.
  - id: F11
    severity: MEDIUM
    class: design
    origin: lens:way-forward
    location: _lyx/plan/01-failed-push-record.md
    summary: FailedPushWayForward ignores the record's repo, naming lyx fabric push for a Bolt record, which that verb cannot push; gone names lyx cleanup --apply, which leaves a still-local gone branch and its record in place; and the not_fast_forward text fits the records side only, not card 9's code-side rejection
    fix: In card 1 make FailedPushWayForward depend on side and class; a Bolt record's deferred, transport and auth name `lyx board sync` from the prime; a code-side not_fast_forward names merging the remote task branch in the task worktree, as commit-records' pushFailureWayForward does; gone names `lyx cleanup --apply` only for an entry ended on origin, and otherwise names the deleted branch and leaves the decision to the operator, with lyx fabric remove as the step that clears it; add one TestFailedPushRecord row per side and class pair.
  - id: F12
    severity: MEDIUM
    class: design
    origin: lens:way-forward
    location: _lyx/plan/06-pull-replay-reflog.md
    summary: Several new stops name no step that can succeed; card 6's weft conflict and card 1's not_fast_forward re-run the same replay; card 5's divergedDetail names settling the paths and lyx board sync, which meets the same conflict; the diverged state names only the conflicting paths; the rewound stops name ResetPairCode's way forward, whose text tells a machine that ran no reset to re-run one
    fix: Name runnable allowed commands in each; for a records conflict (cards 1, 5, 6, 16 and the Summary of card 15), the two choices with commands, keeping origin's side by `git reset --keep origin/<branch>` in the weft or board worktree, or keeping this machine's by settling the named paths then `lyx fabric commit`/`lyx board sync`; for a diverged code branch (card 15 Summary), merging the remote task branch in the task worktree; for rewound (cards 6, 9 and 15), `git reset --keep <origin head>` in the task worktree then the plain push, or the ours merge D4 names, spelled out instead of cited; assert each line in the card's existing test row.
  - id: F13
    severity: MEDIUM
    class: design
    origin: handler
    location: _lyx/plan/03-board-replay-seam.md
    summary: FabricReplay{} is a zero value with no board Config, but Apply must render README.md and the design files with the board's outputs and apply the unknown-label refusal with its vocabulary, both of which boardengine.New takes from Config
    fix: In card 3 state where Apply gets the board config, for example by loading it from the request's board directory through boardengine.LoadConfig or OpenHub with the hub path, and that the store it loads carries that vocabulary and RenderToDisk uses those outputs; add a TestFabricReplay row where the unknown-label refusal uses a label known only through the board config.
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/09-push-retry-and-record.md
    summary: PushWithRetry retries only a (failed) or cannot-lock refusal, so a connection refusal returns at its first attempt; the transport row's after the backoff and the Intent's existing backoff's failure, class transport describe a backoff that does not run
    fix: In card 9 reword the transport bullet to say a gitexec.IsTransportFailure error is recorded as class transport at once, and an ErrPushRetriesExhausted error after the existing backoff, and change the test row to a push to the refused URL writes a transport record.
  - id: F15
    severity: LOW
    class: design
    origin: lens:way-forward
    location: _lyx/plan/17-origin-reconcile-ended.md
    summary: An ended entry reported busy outside a wake hold (orch start, lyx cleanup) carries no way forward, since card 18's force case applies only to a wake hold or an earlier snapshot
    fix: In card 15's Summary and card 17's ended-busy outcome name the way forward, pausing the pair with `lyx tender pause <slug>` and re-running `lyx cleanup --apply`, or name that the removal waits for the driver to end, and assert it in TestReconcileOrigin_Ended's busy row.
  - id: F16
    severity: LOW
    class: design
    origin: lens:way-forward
    location: _lyx/plan/21-batten-resume-reconcile.md
    summary: Card 21's reconcile-error way forward re-runs lyx batten run, which repeats a non-transport error such as an auth refusal, and card 22's stopped Stuck names no verb that re-enters the row once the stop is settled
    fix: In card 21 make the reconcile-error way forward name fixing the cause the error names and then re-running `lyx batten run <slug>`; in card 22 append the row re-entry verb, `lyx loom resume` in the task worktree, to the stopped Stuck's reason, as preflightshed's reconcileRefusal does.
  - id: F17
    severity: LOW
    class: scope
    origin: handler
    location: _lyx/plan/09-push-retry-and-record.md
    summary: Card 9 says D5's replay replaces pull-rebase on every fabric push path, but Fabric.PushWeft and PushWarpAt still push through gitrepo.PushCoalesced, whose rejected push runs git pull --rebase
    fix: In card 9 either add `internal/fabricengine#Fabric.PushWeft` and `internal/fabricengine#PushWarpAt` to **Delete:** (neither has a production caller), or state that both are left on PushCoalesced because no production path reaches them.
  - id: F18
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/plan/09-push-retry-and-record.md
    summary: writeHubRules is to branch on the failed-push record's class from bolt.Push's error, and TestWriteHubRules' fake returns a rejection carrying a recorded class, but no card creates an error type or accessor that carries the class
    fix: In card 9 name how writeHubRules reads the class, either an exported error type such as `plan:internal/fabricengine#PushFailedError` with a Class field added to **Create:**, or a ListFailedPushes read of the board branch's record after the push fails.
  - id: F19
    severity: NIT
    class: consistency
    origin: handler
    location: _lyx/plan/08-push-auth-class.md
    summary: Card 8 says PushRebaseFree returns ErrPushAuthRefused bare, as ErrPushRejected is, and also that git's last error line is kept on a wrapping error that unwraps to it
    fix: In card 8 drop returned bare, as ErrPushRejected is returned bare, and say the error wraps ErrPushAuthRefused with git's last error line, matched with errors.Is.
---

### [BLOCKING] Card 7 misses two literals of the retyped LandDeps.Pull

**Location:** `_lyx/plan/07-land-weft-stop.md`

**Issue:** `internal/testkit/envkit/envkit.go:389` (`FullEnv`) and `internal/shedrecipe/fixture_test.go:199` (`newTestEnv`) both set `Pull: func(context.Context) error { return nil }` in a `battenshed.LandDeps` literal.
Card 7 changes the field's type and edits only `landFixture.deps` and `newFakeReceiver`, calling them the two test literals.
After card 7, `envkit` and every test binary importing it stop compiling.
Neither package is in the verify line, so the plan's own verify would not see it.

### [BLOCKING] Finalize's ended check can never fire

**Location:** `_lyx/plan/04-records-replay.md`, `_lyx/plan/22-landing-reconcile.md`

**Issue:** Card 4's `finalizeEntered` counts a pair status whose `current_producer` is `Finalize` as Finalize entered.
Shed persists `current_producer` as the next row before it calls that row, so the status always reads `Finalize` when `Finalize.Call` runs (`internal/landingshed/finalize.go:187-197`).
At Finalize entry, the run-scope reconcile therefore drops the slug from the ended set every time.
D8's `ended` Stuck with Halt at Finalize is reached only through card 22's fake report.
That D8 check exists for one case: a run that another machine landed while this machine was between Merge-In and Finalize.
With the check dead, this machine goes on to land the run a second time, which is the stale-view harm the task fixes.

### [BLOCKING] The ended way forward does not remove the pair

**Location:** `_lyx/plan/21-batten-resume-reconcile.md`, `_lyx/plan/22-landing-reconcile.md`

**Issue:** `battencli.Cleanup` says it never touches a pair or a branch, and that it deletes only a done slug's records run directory (`internal/battencli/cleanup.go:3-5`, `:88`).
A run that ended on another machine is not done in this machine's batten status, so `lyx batten cleanup --apply` skips it.
The pair stays, and the resume refusal and the Halt keep recurring.
The removal that works is D6's teardown inside the hub reconcile, which `lyx cleanup --apply` runs (card 20).

### [BLOCKING] The 20-second per-attempt deadline needs a gitexec change no card makes

**Location:** `_lyx/plan/15-origin-reconcile-judge.md`

**Issue:** D8 fixes a 20-second deadline per ls-remote attempt, injectable.
`gitexec.runCore` bounds every remote subcommand by the package-level `remoteDeadline` of 10 minutes and offers no per-call deadline (`internal/gitexec/gitexec.go:44-46`, `:198-205`).
PATTERN-fabric-git keeps raw git out of fabricengine, so card 15 cannot bound the call inside its own targets.
Card 15 has no gitexec target, and `./internal/gitexec` is not in verify.
The fix's targets and verify line passed `lyx loom validate-plan --overlay`.

### [BLOCKING] The gone backoff's sleep is a package-level var

**Location:** `_lyx/plan/09-push-retry-and-record.md`

**Issue:** PATTERN-test-speed: "A production interval, timeout or clock that a test would otherwise wait out is a struct field or a parameter, never a package-level `var`, so tests keep `t.Parallel`."
Card 9 creates `var goneBackoffSleep func(time.Duration)` and a `SetGoneBackoffSleepForTest` swap, and runs `TestPushFailureClasses` without `t.Parallel` partly because of that swap.
The repo's own precedent is a per-handle field, `gitrepo.Repo.pushSleep` (`internal/gitrepo/gitrepo.go:63-64`).
The fix passed `lyx loom validate-plan --overlay`.

### [MEDIUM] The try-acquire of the weft write lock has no planned home

**Location:** `_lyx/plan/13-remove-local-mode.md`

**Issue:** `commitPendingRecords` reaches the weft write lock only through `CommitRecordsPaths`, which takes it with a blocking `lock.AcquireWriteLock` (`internal/fabricengine/commitweftpaths.go:65-73`).
A `try` flag on `commitPendingRecords` cannot change that acquire.
A pre-acquire inside `commitPendingRecords` would block on the second flock of the same file.
`CommitRecordsPaths` is not among card 13's targets.
The fix passed `lyx loom validate-plan --overlay`.

### [MEDIUM] A failed local removal respawns the pair's push

**Location:** `_lyx/plan/13-remove-local-mode.md`

**Issue:** `Remove` defers a `respawnPairPush` for any failure after the teardown lock while a worktree survives (`internal/fabricengine/remove.go:177-185`).
That respawn spawns the pair's detached push (`internal/fabricengine/teardownlock.go:118-130`), which is an origin step.
Card 13 moves the body into `removePair`, and its list of what local mode skips leaves the respawn out.
D6 bounds the local mode to no origin step, and the ended entry's local-only commits are the very ones it keeps under a local tag.

### [MEDIUM] Publish's task-branch push escapes the gone and rewound checks

**Location:** `_lyx/plan/09-push-retry-and-record.md`

**Issue:** Publish pushes the task branch through `p.deps.PushBranch()`, then `Fabric.PushBranch`, then `PushWarpRebaseFreeAt` (`internal/landingshed/publish.go:143`, `internal/fabricengine/fabric.go:215`).
None of these go through `pushClassified`.
D10 classifies every push fabricengine runs, and runs the ls-remote before every push of a task branch with an upstream.
D4 bounds the next push against restoring a rewind's dropped commits.
A rewind on origin after Merge-In's reconcile is therefore pushed back by Publish, and Publish's failure writes no record.

### [MEDIUM] HubPaused is read before the gate wait

**Location:** `_lyx/plan/18-reconcile-seams.md`

**Issue:** Cards 19 and 20 build the seams, then call `ReconcileOrigin`.
Under an operator's hub pause, that call's ls-remote waits at the network gate until the lift (`internal/gitexec/gitexec.go:198-202`).
Card 18 says `HubSeamOptions` fills `HubPaused` from the pause record.
Filled once, that value still reads paused after the lift.
A driver the lift resumed then reads not live, and the reconcile moves its branches, which D4 forbids.

### [MEDIUM] TestStencilSeeding_HubScenario asserts the old divergence behaviour

**Location:** `_lyx/plan/05-bolt-pullfirst-replay.md`

**Issue:** `cmd/lyx/stencilseed_integration_test.go:278-285` asserts that the board head stays on the board commit and that the log carries the divergence Warn naming `git pull --rebase`.
Card 5 turns most of these divergences into replays and removes that text.
Card 5 updates the two other tests asserting the Warn, but not this one.
`./cmd/lyx` is in verify, so this test fails at the plan's own gate.
The fix passed `lyx loom validate-plan --overlay`.

### [MEDIUM] FailedPushWayForward's table has dead ends

**Location:** `_lyx/plan/01-failed-push-record.md`

**Issue:** `lyx fabric push` opens a pair fabric and pushes through `PushPairAnchored` (`internal/fabriccli/weft_verbs.go:200`, `:283`), so it cannot push Bolt.
For a gone branch whose entry is not ended, the reconcile reports the branch `gone`.
Card 17 clears only records whose branch is no longer local, so `lyx cleanup --apply` leaves both the record and the branch.
Card 9 records a code-side rejection as `not_fast_forward`, but the line for that class talks about another machine writing the same records paths.

### [MEDIUM] Several stops name no step that can succeed

**Location:** `_lyx/plan/06-pull-replay-reflog.md`, `_lyx/plan/01-failed-push-record.md`, `_lyx/plan/05-bolt-pullfirst-replay.md`, `_lyx/plan/15-origin-reconcile-judge.md`, `_lyx/plan/09-push-retry-and-record.md`

**Issue:** The decision record's constraints require every stop and failed-push class to name an allowed way forward.
These do not:
- After "the operator decides which machine keeps the run", a re-run of `lyx fabric pull` or `lyx fabric push` replays the same commits onto the same origin and stops again.
- `lyx board sync` pushes through `retryBoltPush`'s replay (`internal/boardengine/sync.go:53`), which meets the same cherry-pick conflict.
- `Summary` gives `diverged` only its conflicting paths.
- The rewound stops cite `ResetPairCode`'s way forward.
  That text, `remoteDivergenceWayForward` (`internal/fabricengine/destroy.go:1476-1479`), tells the operator to re-run a reset this machine never ran, and no card spells out D4's "rewind the local branch the same way" as a command.

PATTERN-no-denied-recovery permits `git reset --keep`, so each of these stops has an allowed command it can name.

### [MEDIUM] FabricReplay has no board config

**Location:** `_lyx/plan/03-board-replay-seam.md`

**Issue:** `boardengine.New` takes the outputs and the label vocabulary from `Config` (`internal/boardengine/board.go:27-40`).
`boardCriticalSection` gives the store that vocabulary and passes those outputs to `RenderToDisk` (`board.go:75`, `:118`).
Card 3 installs a zero `FabricReplay{}` and says only that `Apply` loads a Store from origin's `board.json` and re-renders.
It never says where the outputs come from, or the vocabulary that the unknown-label refusal class needs.

### [LOW] The transport class has no backoff

**Location:** `_lyx/plan/09-push-retry-and-record.md`

**Issue:** `PushWithRetry` retries only a `(failed)` or `cannot lock ref` refusal (`internal/gitrepo/pushretry.go:74-96`).
A `connection refused` push returns at its first attempt, so the row's "after the backoff" describes a state that never occurs.

### [LOW] An ended busy entry outside a wake hold has no way forward

**Location:** `_lyx/plan/17-origin-reconcile-ended.md`

**Issue:** At orch start or `lyx cleanup`, an ended entry whose driver is live is reported ended with the busy flag.
Card 18's force case does not cover it, and nothing names how to end that driver, so a re-run stays busy.

### [LOW] Resume and landing stops lack their last step

**Location:** `_lyx/plan/21-batten-resume-reconcile.md`, `_lyx/plan/22-landing-reconcile.md`

**Issue:** Card 15 returns an auth refusal or a missing remote as an error that no retry settles, so re-running `lyx batten run <slug>` alone repeats it.
Card 22's stopped Stuck names the state's way forward but not the verb that re-enters the row.

### [LOW] Two fabric push paths keep pull-rebase

**Location:** `_lyx/plan/09-push-retry-and-record.md`

**Issue:** `Fabric.PushWeft` (`internal/fabricengine/recordsgit.go:323-336`) and `PushWarpAt` (`internal/fabricengine/spawn.go:206-221`) still push through `PushCoalesced`.
Its rejected push runs `git pull --rebase` (`internal/gitrepo/push.go:72-115`).
Card 9's intent claims the replay replaces pull-rebase on every fabric push path.
Neither function has a production caller.

### [LOW] writeHubRules has no way to read the class

**Location:** `_lyx/plan/09-push-retry-and-record.md`

**Issue:** Card 9 makes `writeHubRules` branch on the failed-push class of the error `bolt.Push` returns.
No card creates an error that carries a class, and card 1's record API is keyed by hub, repo and branch.

### [NIT] Card 8 calls the auth error both bare and wrapped

**Location:** `_lyx/plan/08-push-auth-class.md`

**Issue:** Card 8 says the error is returned "bare, as `ErrPushRejected` is returned bare", and in the same sentence that it is kept "on a wrapping error that unwraps to it".

## Rejected

- way-forward lens, the claim that the `ours` merge after a rewind restores the dropped content: D4 names that merge as a way forward itself, so only the missing runnable command is kept, in F12.
- The notice route and board replay installed process-wide rather than wired by `fabriccli` (D1): the plan's Shared Decisions record why, the test-build cycle through `orchcli`'s `hubforge` tests, and D1's substance, a `NotifyPrime` route reaching the detached child, is carried.
- Card 11 picking `reconcileHubWide`'s `bolt.Push` while the 2026-10-17 trace shows a pair-weft push: commit `f431a941` (2026-10-18) made the status seams spawn detached, so the remaining synchronous records push in Preflight is the one card 11 names.
- guard-reach lens, card 2's hub-mismatch branch reachable only from tests: it is a guard over a process-global seam, and no defect follows from it.
- CLI shape: cards 10, 19, 20 and 21 add envelope keys and change no command, flag or `Use`, so the audience bullet of `pattern/PATTERN-cli-cobra.md` is not engaged.
- Card order: no card's `Uses` names a symbol a later card creates or re-signs. Cards 3–6 against 9–10, 15–18 and 19–23 all point backward.
- Test economy: cards 15–17 test three distinct behaviours of one entry point, and cards 19–23 extend the existing tests of their verbs or rows. The new status tests of card 10 have no existing status test to join in `tendercli` or `fabriccli`.
