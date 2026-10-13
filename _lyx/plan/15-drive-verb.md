# Card 15 — drive-verb

**Create:**
- `internal/loomcli#loomCLI.driveCmd`
- `internal/loomcli#driveVerb`
- `internal/loomcli#loomCLI.goDriverDepsFor`
- `internal/loomcli#spawnDetachedDrive`
- `internal/loomcli#driveEnvelopePath`
- `internal/loomcli#TestDriveVerb`
- `internal/loomengine#PathsOutsideRunRecords`
- `internal/loomengine#TestPathsOutsideRunRecords`

**Edit:**
- `internal/loomcli#loomCLI`
- `internal/loomcli#newLoomCLI`
- `internal/loomcli#Command`
- `internal/loomcli/doc.go`
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/loomcli#goDriver.run`
- `internal/shedrun#DriverLock`
- `internal/battenshed#PublishRetrySpent`
- `internal/shedverbs#RunUntilStop`
- `internal/shedverbs#RunGoto`

**Intent:**
D2: the Go driver is the hidden role verb `lyx loom drive`, holding a driver lock of its own for its whole life, so a settling Go driver reads as live while the run lock stays free between steps.

`driveCmd` registers `drive` in `Command`.
It is `Hidden: true` with the `clihelp.AudienceInternal` annotation (`PATTERN-cli-cobra`), `Use: "drive"`, `cobra.NoArgs`, a lower-case `Short` ("drive this worktree's go-seeded loom run, settling its stops by rule (spawned by lyx loom start)") and a `Long` saying `lyx loom start`, `lyx loom resume` and a hand-over spawn it.
It is armed through the full `wire`, as `step` is.
Its body calls `driveVerb` with `shedrun.DriverLock`, the stale files `shedrun.ParkMarker` and `shedrun.ResumeReport`, and a `run` that builds `goDriverDepsFor` and calls `goDriver.run`.

`driveVerb`:
1. Creates the lock's directory and takes the driver lock without blocking.
2. A held lock refuses at once with a message naming the way forward: a Go driver already drives this run; `lyx loom status` shows the run, and the driver log shows the live driver. So a second Go driver never stacks.
3. Otherwise it removes each stale file, ignoring an absent one (D2), runs `run`, and releases the lock exactly once, whether `run` released it early for a hand-over or not.
Bound: the verb steps only through `shedverbs.RunUntilStop` and `RunGoto`, and its one goto is the matching verify record's target, once per run.
Its spawns are capped by `tackler_max_spawns` and `arbiter_max_spawns` per run, plus one successor per stop key and build, over a clean tree only.
It moves nothing while a pause covers the slug, every move it makes is a settle record and a stop report, and the driver lock caps it at one driver per run.

`goDriverDepsFor` wires the card 12 to 14 seams over the receiver:
- stepping and goto: `Step` over `shedverbs.RunUntilStop(ctx, c.spec, []string{"loom", "step"}, attempt, &buf)`, `Goto` over `shedverbs.RunGoto`, and the status and run-lock reads;
- records and envelopes: `SaveEnvelope` writing to `driveEnvelopePath`, a sibling of `driverReportPath` named `drive-envelope-<timestamp>-<4-hex>.json` under the drive-reports directory, and `SettlePath` = `shedrun.SettleRecordsFile(loomengine.LoomFrictionDir(location))`;
- escalation and verify reads: `EscalationPending` wraps `arbiterDeps.PendingEscalation`'s bouncer read and drops its brief path; `VerifyRecord` reads `verifytree.ReadPublishFailure` over `verifytree.NewPaths(location.WorktreePath(), verifytree.Dir(location.AnchorPath()))` and maps `Row`, `Head`, `Target` and `WrittenAt` onto `battenshed.ChildPublishFailure`, as battencli's `readChildPublishFailure` does; `Head` through `fabricengine.Open(location).HeadSHA`, and `VerifyRows` holding `loomshed.NameMergeIn` and `loomshed.NamePublish`;
- batten: `Watch` through `readBattenWatch` over `shedrun.BattenWatchedMarker`, and `BattenRetrySpent` through `battenshed.PublishRetrySpent`;
- pause: `Pause` through `hubgeom.PauseGeometry` and `pauserecord.ReadCoverage`, and `PauseHold` through `pauserecord.WaitUncovered`;
- hand-back: `Park` with `Notify` over `orchcli.NotifyPrime`, `OrchRecorded` reading the prime's orch state (`orchengine.LoadState(orchcli.PrimePaths(prime)).Strand != ""`, as batten's `orchStrandRecorded` does) and `loomCommitStatusDeps`; `NewReportPath` through `driverReportPath`; `CommitRecords` through `commitRecordsVerb`;
- the D13 note: `WriteFrictionNote` writing a `go-driver-paused` note through `friction.NotePath` naming the stop, the producer and the envelope path, a no-op when Tier 2 is off;
- agents: `Tackler` over `tacklerDeps` with `loomengine.ResolveTackler`, the driver log, the steps directory's trace copy and the pause record's covered time; `ConsumeTackler`; `Arbiter` over `arbiterDepsFor`; `OpenAttempts` over the two attempt directories;
- the worktree: `Snapshot` from `HeadSHA`, `CurrentBranch` and `DirtyPaths` with a SHA-256 per path, the tracked read card 7 names; the hand-over's `DirtyPaths` from `fabricengine.Open(location)`'s `Status`, which lists tracked-modified, staged and untracked paths per side, each entry joined to `location.WorktreePath()` and passed through `loomengine.PathsOutsideRunRecords`;
  the weft side's entries are joined to the warp worktree too, since its records reach the warp worktree through the `_lyx` junction and the records directories that filter drops resolve there;
- the hand-over: `OnDiskBuild` over `buildvcs.OfFile` of `os.Executable()` resolved once, `BuildChanged` over `shedverbs.LastStepBuildChanged(c.spec.StepsDir, ...)`, `ReleaseLock` as `release`, and `SpawnSuccessor` over the new `spawnGoDriver` seam.

`loomengine.PathsOutsideRunRecords`, in `internal/loomengine/config.go` beside the durable-directory accessors, drops every absolute path under the run's own records directories: `shedrun.RunDir(l, runID)`, `LoomDurableDir(l)` and `LoomReviewsDir(l)`, the directories `statusCommitPathspec` stages.
The hand-over's clean-tree check here and batten's in card 21 share it, so the two judge one tree the same way.

`loomCLI` gains `spawnGoDriver func() (int, error)`, set in `newLoomCLI` to a closure over `spawnDetachedDrive(c.location)`.
`spawnDetachedDrive` starts `os.Executable()` with `loom drive` in the task worktree, detached through `proc.Detach`, with stdout and stderr appended to `loomengine.LoomDriverLog`.
It logs `loom: spawned detached Go driver` with the pid and the log path (`PATTERN-spawn-observability`), reaps the child in a background goroutine as start's inline go arm does today (so a fast-exiting driver never reads alive as a zombie), and returns the pid.
Card 16 points start's and resume's go arms at the same seam.

`doc.go` gains a `# Go driver` section (the verb, the lock, the closed rule table, settle records, hand-back, notices, hand-over, pause hold, moved runs, the empty-reason note) and a `# Tackler` section (spawn, verdict, causes, deadline, attempts, consume, tampering, the case brief, and that a pair's session end ends a live tackler strand with the pair's other strands, since it is an interactive strand in the pair's reed session).
Its `# Arbiter` section says the Go driver calls the same path in-process and passes the case brief.

`refusal-spec.md` gains a loom-section row `drive: a Go driver already drives the run`, a transient: "`lyx loom status` shows the run; the live driver settles it".

Test `TestDriveVerb` in `drive_test.go`, a table over a temp lock path:
- with the lock held by the test, it refuses with the way forward, never calls `run`, and leaves the stale files;
- with the lock free, it removes the park marker and the resume report, calls `run` once, and leaves the lock free afterwards;
- a `run` that released early for a hand-over still leaves the lock free with no double-release error.
`cmd/lyx/clitree_test.go` already checks the new command's audience, `Use`, `Short` and `Long`.
Test `TestPathsOutsideRunRecords` in `internal/loomengine/config_test.go`, a table over a synthetic location: a path under the run directory, the loom durable directory and the reviews directory are each dropped; a code path and a path whose name only shares a prefix with a records directory are kept.
`test-budget.yaml`'s `internal/loomcli` and `internal/loomengine` rows each rise by one.

**ImpactSummary:** one new hidden verb registered in loom's tree and one new seam field on `loomCLI`; no existing verb's behavior changes until card 16 points start's go arm at the verb.
