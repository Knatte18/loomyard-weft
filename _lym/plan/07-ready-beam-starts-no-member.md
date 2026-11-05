# Card 7 — ready-beam-starts-no-member

**Edit:**
- `internal/battencli#beamForkPrecondition` -> `func beamForkPrecondition(slug, beam string, beamEntry boardengine.Task, beamFound, pairPresent bool, branchOnOrigin func() (bool, error), ready func() (bool, error)) error`
- `internal/battencli#beamForkParent`
- `internal/battencli#TestBeamForkPrecondition`
- `internal/battencli#TestWorktreeCreate_ForksFromTheBeam`
- `internal/battencli#dobbyTaskWait`
- `internal/battencli#TestDobbyLines`
- `internal/dobbyengine#Deps`
- `internal/dobbyengine#State`
- `internal/dobbyengine#daemon.startReady`
- `internal/dobbyengine#WaitingTask`
- `internal/dobbyengine#WaitingTask.Ready`
- `internal/dobbyengine#BuildView`
- `internal/dobbyengine#TestBuildView`
- `internal/dobbyengine#TestStartReady_RefusalsAndBackoff`
- `internal/dobbyengine/doc.go`
- `internal/beamcli#hubDobbyDeps`
- `internal/beamcli/doc.go`

**Create:**
- `plan:internal/dobbyengine#ReadyHoldState` -> `type ReadyHoldState struct`
- `plan:internal/dobbyengine#daemon.holdReadyBeam` -> `func (d *daemon) holdReadyBeam(ctx context.Context, tasks []TaskView) bool`

**Uses:**
- `internal/pawlengine#ReadyFor`
- `internal/loomengine#BeamReadyPath`
- `internal/loomengine#BeamReadyLockPath`

**Intent:** From the ready record on, no member run starts under the beam (D2): the beam's dobby starts none, and the member fork every member run passes refuses one.

- `beamForkPrecondition` gains `ready`, which reads whether the beam's ready record names this beam. After the pair-present and `RunHolds` checks, a ready record naming the beam refuses with a `*battenshed.PreconditionError` whose Unmet says `<slug>` belongs to beam `<beam>`, which is flagged ready to land, so no member starts in it, and whose WayForward names `lym batten cancel <slug> --remove`, as the dobby's notice does, since ready is final and the member cannot land into this beam. A `ready` error refuses too, with Unmet naming the error and the same way forward after fixing it. Its doc says the beam's own landing run holds the beam from Finalize on, not from Parent-Align. `beamForkParent` passes a closure that resolves the beam's pair (`lymcwd.ResolveWorktree(fabricengine.WorktreePath(prime, beam))`) and calls `pawlengine.ReadyFor(loomengine.BeamReadyPath(..), loomengine.BeamReadyLockPath(..), beam)`; it is read only once the pair is present.
- Reaching input: a member released under a beam whose `lym beam ready` wrote the record while the beam's run waits at Beam-Await, so `RunHolds` reads the beam as holding nothing; a hand `lym batten run <member>` reaches Worktree-Create's fork precondition and is refused.
- `dobbyengine.Deps` gains `BeamReady func() (bool, error)`; nil, as on the prime and in a harness that tells none, holds nothing, and `holdReadyBeam` gates on it alone. `State` gains `ReadyHold *ReadyHoldState`, whose `Notified []string` lists the members already named this episode.
- `daemon.startReady` calls `holdReadyBeam` first: with `BeamReady` true it starts no task and returns true, and for each open member of tasks, every task whose parent is this worktree, that is not done and that has no run listed, draft or held by a dependency included, that `Notified` lacks, it posts one notice through `Deps.Notice` naming the member, that it was filed after the beam was flagged ready, that it blocks the landing, and `lym batten cancel <member> --remove`, recording the slug only once the post succeeded, so a failed post is retried next tick. With `BeamReady` false it clears `ReadyHold` and returns false, so a later episode notices again. A `BeamReady` error starts nothing this tick, is logged once per episode and changes no record.
- `WaitingTask` gains `ReadyHold bool`; `BuildView` sets it on every waiting task while the record carries `ReadyHold`, and `WaitingTask.Ready` reads false for it, so `lym beam status` shows the hold. `dobbyTaskWait` renders it as `the beam is flagged ready: lym batten cancel <slug> --remove`.
- `internal/dobbyengine/doc.go`'s slot rule reads Finalize for Parent-Align: a pawl run takes a slot from Finalize on.
- `beamcli.hubDobbyDeps` fills `BeamReady` for a beam's dobby from `hubLandDeps(prime).Ready(slug)` and leaves it nil for the prime.
- Bound: between `lym beam ready` and the accept, the dobby's start hold and the fork precondition together refuse every member start, by the dobby or a hand `lym batten run`, and card 15's refusal of open work stops the ready itself while a member works, so nothing lands into a ready beam. A failing re-check at the accept still lifts the hold and restarts the dobby as today, and the dobby's start hold still applies while the record stands.

Tests: `TestBeamForkPrecondition` (tier 0) gains a ready-true row (refused naming `lym batten cancel <member> --remove`), a `ready` error row (refused naming the error) and a ready-false row (passes), and its existing rows pass a `ready` reading false. `TestWorktreeCreate_ForksFromTheBeam` (tier 1) gains the subtest `MemberOfABeamFlaggedReadyIsRefused`: a hub beam whose pair holds a ready record naming it, and a `lym batten run` of a released member refused at Worktree-Create naming the cancel, so `beamForkParent`'s closure is driven. `TestStartReady_RefusalsAndBackoff` (tier 0) gains rows over a harness whose `geom` is set to a beam's after `newLoopHarness` builds it: a released member, a draft member and a dependency-held member with `BeamReady` true: nothing is spawned, one notice names each and `lym batten cancel <member> --remove`; a second tick posts nothing; a failed post is retried next tick; `BeamReady` false then starts the released member and clears the record; a `BeamReady` error spawns nothing; its other rows keep passing with `BeamReady` nil. `TestBuildView` gains a row with `ReadyHold` in the record, and `TestDobbyLines` a row rendering the hold.

**ImpactSummary:** one re-signed battencli helper with one production caller and its test, one new dobby seam and state field filled by beamcli's dobby wiring, and the waiting view's new reason; a prime dobby and a beam not flagged ready behave as today.

**Commit:** 7: A beam flagged ready starts no member run
