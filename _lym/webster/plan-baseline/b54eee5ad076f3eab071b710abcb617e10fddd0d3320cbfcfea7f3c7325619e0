# Card 15 — beam-await-drained

**Create:**
- `plan:internal/pawlengine#DrainMarkerName` -> `const DrainMarkerName = "drain-posted"`
- `plan:internal/pawlengine#awaitRun.noticeDrain` -> `func (r *awaitRun) noticeDrain()`

**Edit:**
- `internal/pawlengine#Deps`
- `internal/pawlengine#Paths`
- `internal/pawlengine#awaitRun.wake`
- `internal/pawlengine#NewBeamAwait`
- `internal/pawlengine/doc.go`
- `internal/hubgeom#PawlPaths`
- `internal/shedrecipe#requirePawl`
- `internal/shedrecipe#beamAwaitEntry`
- `internal/shedrecipe#validPawlDeps`
- `internal/shedrecipe#simpleEntryCases`
- `internal/loomcli#pawlDeps`
- `internal/testkit/envkit#FullEnv`
- `internal/battencli#intentionallyNil`
- `internal/pawlengine_test#TestBeamAwait`
- `internal/pawlengine_test#world`
- `internal/pawlengine_test#newWorld`
- `internal/pawlengine_test#world.deps`
- `internal/hubgeom#TestDobbyGeometry`

**Uses:**
- `plan:internal/hubgeom#Autonomy`
- `internal/battencli#ReadBeamLandBlockers`
- `internal/battencli#BeamLandBlockers.Empty`
- `internal/pawlengine#awaitRun.notify`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/hubgeom/doc.go`
- `internal/shedrecipe/doc.go`
- `internal/loomcli/doc.go`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`

**Intent:** D6: while the beam's orch is autonomous, Beam-Await posts one `[beam drained] <slug>` line when the beam's work is done, on its existing wakes.

Seams and paths:

- `Deps` gains `Autonomous func() bool`, the beam's orch's autonomy read at call time;
  `HoldsCommits func() (bool, error)`, whether the beam holds commits beyond its parent;
  and `WorkRemains func() (bool, error)`, whether any member task of the beam is open, drafts included, or any child pair of the beam exists.
- `Paths` gains `DrainMarker`, a file of its own beside the pawl markers; `DrainMarkerName` joins the names in `landrequest.go`, and `hubgeom.PawlPaths` fills it under the beam's `.lym/beam/` directory.
- `NewBeamAwait` requires the three seams and `Paths.DrainMarker` itself; the field stays out of `Deps.pathsSeams`, so no other pawl producer requires it.
  `beamAwaitEntry` lists the seams, and `requirePawl`'s field map knows them.
- shedrecipe's test fixture `validPawlDeps` stubs the three seams and sets `Paths.DrainMarker`,
  and the BeamAwait row of `simpleEntryCases` names the four new fields in its `validatedFields`.
- `pawlDeps` wires `Autonomous` as `hubgeom.Autonomy(location, slug)` (the beam's own worktree is its orch's),
  `HoldsCommits` as the beam pair's `HeadHoldsCommitsNotIn("refs/heads/" + origin.ParentBranch)` with the origin `fabricengine.ReadOriginFor(prime, slug)` reads,
  and `WorkRemains` as `ReadBeamLandBlockers(prime, slug)` not `Empty()`, returning its error;
  `beamMembers` skips only done tasks, so it counts every non-done member, drafts included, and every child pair.
- `envkit.FullEnv` stubs the three seams and sets the marker path; `battencli`'s `intentionallyNil` gains `Pawl.Autonomous`, `Pawl.HoldsCommits` and `Pawl.WorkRemains` with their siblings' reason.

The notice:

- `wake`, after its entry and when no land request is found, calls `noticeDrain` before it returns.
- `noticeDrain` posts nothing while `Autonomous()` reads false.
  Otherwise, when `WorkRemains` reads true it removes the drain marker, so a member that opens again lets the next drain post again.
  When `HoldsCommits` reads true, `WorkRemains` false and `ReadyStands` false, and no drain marker stands, it posts `[beam drained] <slug>` through `notify` and writes the marker once the post reached an orch.
  A seam that fails is logged and decides nothing on that wake.
- Beam-Await alone writes and removes the drain marker; the await marker's writers leave it alone,
  so a re-entry of the row and a dobby-down notice cycle post no second line.
- The line only informs: it opens, merges or readies nothing.
  A takeover or a new pawl run starts without the marker, so it may post one further line for a drain already posted.
- `doc.go` describes the drain notice and its marker.

Tests:

- `world` gains `autonomous`, `holdsCommits`, `workRemains` and their error fields; `newWorld` sets the drain marker path; `world.deps` wires the seams.
- `TestBeamAwait` (tier0) gains rows:
  the first wake with autonomy on, commits held, no work and no ready record posts exactly one `[beam drained] <slug>` and writes the marker;
  later wakes, a re-entry of the row, and a dobby-down notice sent and cleared post no further line;
  work that opens again removes the marker, and the next drain posts a second line;
  no commits beyond the parent, a standing ready record, remaining work, and autonomy off each post none;
  a failed post writes no marker, so the next wake posts again;
  a failing `HoldsCommits` and a failing `WorkRemains` each post nothing and leave the marker as it stood.
- `TestDobbyGeometry`'s `PawlPaths` check adds `DrainMarker` to the paths it wants directly under the dobby's directory.

**ImpactSummary:** Beam-Await gains three seams and one marker, registered in the recipe entry, wiring, envkit and batten's waiver list; the other pawl producers and the await marker's behaviour are unchanged.
