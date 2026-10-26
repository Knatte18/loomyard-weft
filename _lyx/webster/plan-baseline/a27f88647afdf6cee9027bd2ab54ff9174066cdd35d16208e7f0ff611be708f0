# Card 17 — pawl-wiring

**Create:**
- `internal/hubgeom#PawlPaths`
- `internal/loomcli#pawlDeps`
- `internal/loomcli#pawlRecheckLines`
- `internal/loomcli#pawlLandBlockers`
- `internal/loomcli#stopBeamDobby`
- `internal/loomcli#TestPawlRecheckLines`

**Edit:**
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#landingDeps`
- `internal/loomcli#codeReviewOff`
- `internal/loomcli#writeBoardEntrySnapshot`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWriteBoardEntrySnapshot`
- `internal/hubgeom#TestDobbyGeometry`
- `internal/loomcli/doc.go`

**Intent:**
Wires the pawl recipe's rows over the hub, inside the beam pair the pawl run's child drives.

- `hubgeom.PawlPaths(beam)` composes the land request, dobby-stop marker and await marker paths under the beam worktree's `.lyx/beam/` at its own anchor, through `lyxdirs.DotLyxDirName` and `lyxdirs.BeamDirName` as `DobbyGeometry` composes the dobby's, so the dobby, `lyx beam land` (card 22) and Beam-Await read one directory.
- `loomCLI.wire`, for a pawl run, keeps the BeamDescribe seams and fills `env.Pawl` with `pawlDeps(location, slug)`:
  `RunID` is the child seed's `started_at` (`shedrun.ReadSeed` of the run, the pawl run's seed id); `SetStatus` writes `boardengine.RunStatus("running", producer)` on the beam's entry through `Board.SetRunStatus`, and `EntryProducer` reads the entry's producer half; `OwnedElsewhere` reads the beam's effective owner from the local Bolt against `machinename.Name`; `MarkBeam`, `SpawnPush` and `RegenerateWorkspace` call `fabricengine.MarkBeam`, `fabricengine.SpawnPairPush` and `fabricengine.RegenerateHubWorkspace` over the prime; `Steps` is `orchcli.BeamUp(location, {Prepare: false})`; `StopDobby` is `stopBeamDobby`, which ends the beam's dobby through `runproc.EndRole` by its `RoleDobby` record when `dobbyengine.Live` reads it held; `StartDobby` is `dobbyengine.EnsureLive` naming the beam alone; `DobbyLive` is `dobbyengine.Live` over the beam's `hubgeom.DobbyGeometry`; `Recheck` reads `battencli.ReadBeamLandBlockers`, `battencli.BeamLandRefusal`, the beam's owner and its parent's owner and returns `pawlRecheckLines`; `WriteSnapshot` is `writeBoardEntrySnapshot` followed by a commit of the snapshot through `fabricengine.CommitAnchoredPaths`, since the perch reads it from the committed tree; `Notify` is `orchcli.NotifyOrchAt` on the beam's own worktree signed `PAWL`; `Down` is `orchcli.BeamDown(location)`; `Watch` watches the request's directory through `internal/fswatch`; `Now` and `Sleep` are the wall clock and a context sleep.
- `pawlRecheckLines` words the failed re-check: each blocker of `BeamLandRefusal`'s text; a beam this machine does not own, naming `lyx adopt <slug>`; a parent another machine owns, naming `lyx adopt <parentSubject>` (`lyx adopt root` for the prime); empty when all pass.
- `writeBoardEntrySnapshot` serves both recipes: it reads the run's own entry, a beam's project entry for pawl, with `darnengine.BoardEntrySnapshot`.
- `landingDeps` for a pawl run keeps `ConflictReroutes` false, `LargeMergeLands` true and `RerouteReviewed` nil, since pawl's Finalize declares no `on_reroute` and a spent or over-cap fixer halts for the orch; it fills `LandBlockers` with `pawlLandBlockers`, mapping `ReadBeamLandBlockers` onto `landingshed.LandBlocker`, and `ParentWorktree` with the beam's recorded parent worktree; loom and darn leave both unset.
- `codeReviewOff` reads the Code-Perch's budget for a pawl run, which has Parent-Align now, and keeps answering false for darn.

`TestWire_BeamRecipe` asserts, for a pawl seed: no `Pawl` seam is nil, the routing's nine rows, and the landing deps' `LandBlockers` set with `ConflictReroutes` false, `LargeMergeLands` true and `RerouteReviewed` nil.
`TestPawlRecheckLines`: no blockers and both owners here give no line; a member blocker, a beam owned elsewhere and a parent owned elsewhere each give their line, the prime's naming `lyx adopt root`.
`TestDobbyGeometry` asserts, beside the dobby's own paths, that `PawlPaths` puts every path it composes in the same `.lyx/beam/` at the beam's anchor, its sub-anchor row included.
`TestWriteBoardEntrySnapshot` gains a pawl row writing the beam project entry's snapshot.
`loomcli/doc.go` describes the pawl wiring.

**ImpactSummary:** fills the pawl rows' seams and two Finalize seams for pawl runs only; loom and darn wiring is unchanged, and `codeReviewOff`'s pawl answer is the only reading that changes for an existing recipe name.

**Commit:** 17: Wire the pawl rows: Beam-Up's steps, Beam-Await's seams, the perch's answer key and Finalize's re-check
