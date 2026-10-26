# Card 10 — orch-beam-up

**Create:**
- `plan:internal/orchcli#BeamUpOptions` -> `type BeamUpOptions struct`
- `plan:internal/orchcli#BeamUpReport` -> `type BeamUpReport struct`
- `plan:internal/orchcli#BeamUp` -> `func BeamUp(location *lyxcwd.Location, opts BeamUpOptions) (BeamUpReport, error)`
- `plan:internal/orchcli#BeamDownReport` -> `type BeamDownReport struct`
- `plan:internal/orchcli#BeamDown` -> `func BeamDown(location *lyxcwd.Location) (BeamDownReport, error)`
- `plan:internal/orchcli#wireCLI` -> `func wireCLI(c *orchCLI, location *lyxcwd.Location) error`
- `plan:internal/orchcli#orchCLI.beamUp` -> `func (c *orchCLI) beamUp(opts BeamUpOptions) (BeamUpReport, error)`
- `plan:internal/orchcli#orchCLI.startOrch` -> `func (c *orchCLI) startOrch(index func() (string, error), note string) (BeamUpReport, error)`
- `plan:internal/orchcli#orchCLI.stopOrch` -> `func (c *orchCLI) stopOrch() (removed bool, strand string, err error)`
- `plan:internal/orchcli#beamUpRefusal` -> `func beamUpRefusal(slug, here, owner string, moveWayForward string, held bool, status string) error`
- `plan:internal/orchcli#TestStart_ResumesTheRecordedSession` -> `func TestStart_ResumesTheRecordedSession(t *testing.T)`
- `plan:internal/orchcli#TestBeamDown_StopsOrchThenTheDobbyByRole` -> `func TestBeamDown_StopsOrchThenTheDobbyByRole(t *testing.T)`
- `plan:internal/orchcli#beamFacts` -> `type beamFacts struct`
- `plan:internal/orchcli#orchCLI.beamDown` -> `func (c *orchCLI) beamDown() (BeamDownReport, error)`
- `plan:internal/orchengine#SourceResume` -> `const SourceResume = "resume"`

**Delete:**
- `internal/orchcli#orchCLI.startCmd`
- `internal/orchcli#previousSessionHint`
- `internal/orchcli#withContinueWaysForward`
- `internal/orchcli#refuseBeamStart`
- `internal/orchcli#checkBeamStart`
- `internal/orchcli#orchCLI.startDobbiesNotes`
- `internal/orchcli#TestRefuseBeamStart`
- `internal/orchcli#TestStart_ResumesFromHandoffFile`
- `internal/orchcli#TestStart_AdoptLaunchesResumeSpec`
- `internal/orchcli#TestSmokeOrch_Adopt`
- `internal/orchcli/smoke_adopt_large_test.go`
- `internal/orchengine#RenderAdoptPrompt`
- `internal/orchengine#adoptStencilName`
- `internal/orchengine#SourceAdopt`
- `internal/orchengine#SourceFlag`
- `contracts/stencils#OrchTemplateAdopt`
- `contracts/stencils/orch/orch-template-adopt.md`

**Edit:**
- `internal/orchcli#orchCLI`
- `internal/orchcli#commandOf`
- `internal/orchcli#ensureDobbies` -> `func ensureDobbies(location *lyxcwd.Location) (dobbyengine.StartEntry, error)`
- `internal/orchcli#orchCLI.claimPrimeNotes`
- `internal/orchcli#orchCLI.pauseOwnedRuns`
- `internal/orchcli#stoppingWatcherHint`
- `internal/orchcli#withdrawWayForward`
- `internal/orchcli#orchCLI.orchReceiver`
- `internal/orchcli#orchCLI.stopCmd`
- `internal/orchcli#startLockHeldHint`
- `internal/orchcli#orchCLI.statusCmd`
- `internal/orchcli#indexWayForward`
- `internal/orchcli#orchCLI.watchCmd`
- `internal/orchcli#startHarness`
- `internal/orchcli#newStartHarness`
- `internal/orchcli#startHarness.run`
- `internal/orchcli#TestStart_EnsuresTheRunsStrandBeforeTheOrch`
- `internal/orchcli#TestStart_NoStrandLaunchesAndSpawnsWatcher`
- `internal/orchcli#TestStart_LiveStrand`
- `internal/orchcli#TestStart_NeverAttaches`
- `internal/orchcli#TestStart_DeadStrandRemovedBeforeStart`
- `internal/orchcli#TestStart_FreshLaunchResetsAbandonedPhase`
- `internal/orchcli#TestStart_Refusals`
- `internal/orchcli#TestStart_ReconcilesTheHubFirst`
- `internal/orchcli#TestStart_StarterErrorFailsAndSavesNoState`
- `internal/orchcli#TestStart_StartsTheDobbiesFirst`
- `internal/orchcli#TestStop`
- `internal/orchcli#fakeStarter`
- `internal/orchcli#fakeStarter.StartSession`
- `internal/orchcli#TestStatus_RestartsADeadWatcher`
- `internal/orchcli#TestWatcherIndexSource`
- `internal/orchcli#TestOrchIntegration_Refusals`
- `internal/orchcli#TestSmokeOrch_OneFullCycle`
- `internal/beamcli#createSteps`
- `internal/beamcli#TestCreate`
- `internal/orchcli#liveFixture.startOrch`
- `internal/orchengine#ChooseStartPrompt` -> `func ChooseStartPrompt(stencilsDir, rolePath string, s State, exists func(string) bool) (prompt, source string, err error)`
- `internal/orchengine#TestChooseStartPrompt`
- `internal/orchengine#TestRenderFiles_WriteWithoutLeadingComment`
- `internal/orchengine#TestRenderPointers_MultiLineOverrideFails`
- `internal/orchengine#TestRenderPointers_OneLineNamingTheirPaths`
- `contracts/stencils#entries`
- `cmd/lyx#TestExitCode_HelpPaths`
- `cmd/lyx#newRoot`
- `contracts/stencils/orch/orch-template-start.md`
- `contracts/stencils/orch/orch-template-role.md`
- `internal/orchcli/doc.go`
- `internal/orchengine/doc.go`
- `docs/overview.md`

**Intent:**
D1–D4's orch half: `lyx orch start`, with `--continue`, `--handoff` and `--adopt`, goes; its steps move into `BeamUp`, which `lyx beam start` (card 11), adopt (card 24) and Beam-Up (card 15) call, and `BeamDown` is the stop `lyx beam stop` and Beam-Down call.

- `wireCLI` is `commandOf`'s pre-run wiring, factored out: the prime-or-beam refusal, the orch, shuttle and reed config, the reed engine, the runner, the paths and every seam, filled onto `c` for `location`.
  The pre-run calls it for every verb but the bare group, and no orch verb reconciles the hub's config any more: `hubreconcile.EnsureOrWarn` leaves the pre-run, and `lyx beam start` calls `hubreconcile.Ensure` itself (card 11).
- `BeamUpOptions` holds `Prepare bool`, `Index func() (string, error)` and `Here string`; a nil `Index` runs `<os.Executable()> help index` through `runHelpIndex`; an empty `Here` reads `machinename.Name`.
  `BeamUp(location, opts)` builds an `orchCLI` over `wireCLI` and runs `beamUp`.
- `orchCLI.beamUp`, in order:
  1. With `Prepare`, the hub-scope origin reconcile from the prime (`reconcileHub`), the failed-push read, the prime claim when the board records no root owner (`claimPrimeNotes`, now run in the prime and in a beam alike), and the owner pause of each owned_elsewhere entry (`pauseOwnedRuns`, its warnings naming `lyx beam start`), as `start` did; without it none of these runs.
  2. The refusals, before any dobby, window or orch start, read from the local Bolt the reconcile just pulled: a beam another machine owns, naming the owner and `lyx adopt <slug>` (`lyx adopt root` for the prime); an anchor move standing on the worktree or the prime, naming its way forward through `fabricengine.MoveInProgress`, as `EnsureLive` reads it; a beam whose pawl run holds the landing hold (`boardengine.RunHolds` on its entry), naming `lyx batten status <slug>`.
     `beamUpRefusal` is the pure judgment over the read facts; it replaces `refuseBeamStart`, and `checkBeamStart` is gone with it.
     The facts come through the receiver's `readBeamFacts func() (beamFacts, error)` field, which `wireCLI` fills from the local Bolt, `machinename`, `fabricengine.MoveInProgress` and the worktree's board entry; `beamFacts` holds this machine's name, the beam's owner, a standing move's way forward and the entry's hold and status, and a nil field, as a hand-built receiver leaves it, refuses nothing.
  3. The beam steps: the receiver's `startDobby func() (dobbyengine.StartEntry, error)` field, which replaces `startDobbies` and which `wireCLI` fills with `ensureDobbies`, starts this worktree's dobby alone through `dobbyengine.EnsureLive(hubgeom.DobbyStart(prime), []string{worktree})`, a missing dobby started on its own, and a nil field starts none; `ensureRunsStrand` ensures the beam window, a failure only warning as today; `startOrch` starts the orch and its watcher.
  A refusal returns its error with the report gathered so far; `BeamUpReport` carries the worktree, the dobby outcome and detail, the orch's action, strand, prompt source and abandoned session, the reconcile report, the failed pushes, the owner pauses, the mutations with `partial`, and the warnings.
- `startOrch` keeps `start`'s strand logic, start lock, withdraw rounds and watcher-stop wait, and changes the launch (D2):
  a live strand with a live watcher is left alone; a live strand with a dead watcher gets a watcher; a missing or dead strand is relaunched, resuming the session `State.SessionID` records with `ResumeSessionID` and the reload pointer (`orchengine.RenderReloadPrompt`), source `SourceResume`.
  When that resume fails to start, its transcript gone or its process not coming up live, it falls back to a fresh launch of the prompt `ChooseStartPrompt` picks, never to an error, and the report's abandoned session names the id it gave up.
  With no session recorded it launches fresh from the last completed handoff note, else from the start stencil, through `ChooseStartPrompt`, and the reconcile note `Prepare` gathered joins a fresh prompt.
  A fresh orch otherwise comes only from `lyx orch refresh` on a live orch, since `lyx orch stop` keeps the recorded id.
  Bound: the orch resumes the context the recorded id names; after a reed server rebirth in clear mode that is the pre-clear context, as `--continue` resumed it.
- `orchengine.ChooseStartPrompt` loses `handoffFlag` and `SourceFlag`; `RenderAdoptPrompt`, `adoptStencilName`, `SourceAdopt` and the `orch-template-adopt` stencil, its embed and its `entries` row go with `--adopt`.
- `BeamDown(location)` builds the receiver over `wireCLI` and runs `beamDown`, which stops the worktree's orch strand through `stopOrch`, `stopCmd`'s body under the start lock, after which the watcher exits on its own, and then, when the receiver's `dobbyLive func() (bool, error)` field reads the dobby's lock held, ends it through its `endDobby func() error` field; `wireCLI` fills them with `dobbyengine.Live` over the worktree's `hubgeom.DobbyGeometry` and with `runproc.EndRole(fabricengine.RunProcDir(hub), owner, runproc.RoleDobby, os.Getpid())`, owner the beam's slug or `runproc.HubOwner` for the prime, so no other process of the slug, the pawl run's batten, driver and agents included, is signalled; a nil `dobbyLive` stops no dobby.
  Each part stops only when live, so it is idempotent; `BeamDownReport` says what stopped.
  `stopCmd` keeps its envelope over `stopOrch`.
- Every message naming `lyx orch start` names `lyx beam start` instead (`stoppingWatcherHint`, `withdrawWayForward`, `orchReceiver`, `pauseOwnedRuns`, `startLockHeldHint`, `statusCmd`'s help and restart failure, `indexWayForward`, `watchCmd`'s no-strand refusal, `stopCmd`'s lock refusal), under fb5c10c3's rule that messages name the full command; `newRoot`'s comment on the installed owner pause names `lyx beam start`.
  `beamcli.createSteps`' way forward, which card 1 kept, names `lyx beam start` in the beam's worktree, which card 11 makes bring up the orch, and `TestCreate`'s assertion of it follows; card 21 then rewrites create.
- The start stencil's comment and the role stencil's line on starting dobbies name `lyx beam start`, which brings up the worktree's dobby and orch.

Tests, over the start harness, whose `run` now calls `beamUp` with `Prepare` and decodes the report instead of executing a cobra verb:
- `TestStart_ResumesTheRecordedSession`: a recorded session id is resumed with the reload pointer and source `resume`; a resume whose start fails falls back to a fresh launch from the last note, reports the abandoned id and leaves no error; with no id recorded and no note it launches the start stencil.
  `fakeStarter` gains `resumeErr`, returned only for a spec whose `ResumeSessionID` is set, so the resume fails and the fresh launch after it succeeds.
- `TestStart_Refusals` drops the flag rows and gains, through the harness's `readBeamFacts`, a beam another machine owns refused naming `lyx adopt`, the prime owned elsewhere naming `lyx adopt root`, a standing anchor move naming its way forward and a held beam naming `lyx batten status`, each with no dobby, window or orch started.
- `TestStart_StartsTheDobbiesFirst` drives the harness's `startDobby` field and asserts it runs once, before the window and the orch; `TestStart_ReconcilesTheHubFirst` keeps its reconcile, claim and pause assertions under `Prepare` and gains a row with `Prepare` false that reconciles, claims and pauses nothing.
- `TestBeamDown_StopsOrchThenTheDobbyByRole` drives `beamDown` over the start harness with fake `dobbyLive` and `endDobby`: it removes the recorded strand, then ends the dobby once; a dead dobby is not ended; a second call stops nothing. That `EndRole` ends the dobby record and spares a driver record of the same slug is card 2's `TestEnd` row.
- `TestStatus_RestartsADeadWatcher`'s failing-spawn row and `TestWatcherIndexSource` assert `lyx beam start` where they asserted `lyx orch start`.
- `TestOrchIntegration_Refusals` drops `start` from its verb loop, and its start case calls `BeamUp` from the same non-prime worktree and asserts the same prime-or-beam refusal; `TestSmokeOrch_OneFullCycle` starts its orch through `BeamUp` instead of `orch start --no-attach`.
- The other `TestStart_*` tests, `TestStop`, `TestChooseStartPrompt`, the two render tests and `TestExitCode_HelpPaths` (its `orch start --help` row now `beam start --help`) follow the removed flags and verb; `TestSmokeOrch_Adopt` and the large-session adopt smoke file test `--adopt` and go; `liveFixture.startOrch` calls `BeamUp`.

The docs: `orchcli/doc.go` replaces `start` with `BeamUp` and `BeamDown` and drops `--continue`; `orchengine/doc.go`'s "Adopting a running session" section goes and "Continuing after a reboot" says a start resumes the recorded session or falls back to the note; `docs/overview.md`'s orch line lists `stop|status|refresh|distill|withdraw|rules`.

What it lets through, bounded: `BeamUp` with `Prepare` false skips the reconcile, the claim and the owner pauses, and its callers, adopt after its own reconcile and Beam-Up inside the run, each already judged origin; the refusals of step 2 still run.

**ImpactSummary:** the orch module loses its start verb and gains two exported entry points over the same internals; `ensureDobbies`' re-sign has one caller here, and every test, stencil and doc that named the removed flags is on this card.

**Commit:** 10: Orch start moves into BeamUp and BeamDown; drop `lyx orch start`
