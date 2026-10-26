# Card 21 — beam-create-starts-pawl

**Create:**
- `plan:internal/beamcli#refuseCreateHere` -> `func refuseCreateHere(deps createDeps, acting, slug string) error`

**Edit:**
- `internal/beamcli#createDeps`
- `internal/beamcli#createResult`
- `internal/beamcli#create`
- `internal/beamcli#judgeCreateEntry`
- `internal/beamcli#createSteps`
- `internal/beamcli#hubCreateDeps`
- `internal/beamcli#createCmd`
- `internal/beamcli#createWorld`
- `internal/beamcli#newCreateWorld`
- `internal/beamcli#createWorld.deps`
- `internal/beamcli#TestCreate`
- `internal/beamcli#TestCreate_StepFailureNamesTheRerun`
- `internal/beamcli_test#TestCreate_RunsFromABeamAndReachesTheVerbsOwnRefusal`
- `internal/beamcli/doc.go`
- `docs/overview.md`

**Delete:**
- `internal/beamcli#requireParentBeamPair`
- `internal/beamcli#startDobby`
- `internal/beamcli#startBeamDobby`

**Intent:**
D10: `lyx beam create <slug> [--anchor <dir>]` records the beam and starts its pawl run, whose Worktree-Create makes the pair (card 18) and whose Beam-Up marks it and starts its dobby and orch (card 15); the create no longer makes, marks or pushes a pair itself.

- `refuseCreateHere` runs first: the verb runs from the beam's parent worktree, the prime or the parent beam `ParentBeam` names, and refuses elsewhere naming that worktree's path and the same command there; when the parent beam's pair is absent on this machine, so that path does not exist, it names instead `lyx beam create <parent>`, or `lyx adopt <parent>` when the parent's branch is on origin (the kept `BranchOnOrigin` seam), then the same command from the parent's worktree; `createDeps` gains `Acting`, the acting worktree's name, which `createCmd` fills from the name `runVerb` already passes, so `create`'s signature is unchanged.
- `judgeCreateEntry` takes an open project entry, refuses a note, a task and a done project as today (card 1's wording), refuses a beam another machine owns naming `lyx adopt <slug>`, and refuses when the parent reads owned by another machine, naming `lyx adopt <parent subject>` (`lyx adopt root` for the prime), through `Owners.Effective` and `Owners.AdoptSubject`.
- `createSteps`, each skipped when done: claims the root for this machine when the board records no root owner (`SetOwners` with `Prime`); records this machine as the beam's owner with the beam flag and `recipe: pawl` through the owner setter (`Beam: true`), a write lost to another machine refused naming `lyx adopt <slug>`; then starts the batten run as a detached first pass through `battencli.StartFirstPass(prime, slug, FirstPassOptions{Anchor: anchor})`, driven on by the parent's dobby, and the envelope names `lyx batten status <slug>`.
  A pawl run in flight, a batten status file not done, is reported live and starts nothing; a flagged beam with no run status, left by a first pass a covering pause refused or that died, gets its first pass again; a legacy beam with a marked pair and no pawl run gets its pawl run, Worktree-Create finding the pair complete and Beam-Up finding it marked.
  A first pass whose envelope is not ok, such as one refused for a covering pause, is the verb's error carrying that envelope's error, and leaves no run.
- `createDeps` keeps `PairPresent` and `BranchOnOrigin` for `refuseCreateHere`, loses the pair add, mark, anchor, push, workspace and dobby seams and gains `RunStatus(slug)` (the batten status file's state, found or not) and `StartRun(slug, anchor)`; `hubCreateDeps` fills them, and `createResult` reports the root claim, the owner write, the run started or live, and the first pass's envelope.
- `createCmd`'s help and `beamcli/doc.go`'s create section describe the steps and the bound: a first pass refused for a covering pause leaves no run and the same `lyx beam create <slug> [--anchor <dir>]` after the lift starts it, and a flagged beam left by a first pass that died takes the same `--anchor` again, since its anchor reached no run record; dobby never starts a beam project.

Tests: `TestCreate` rows reach: a create from a worktree that is not the parent refused naming the parent's path; a nested beam whose parent beam has no pair here refused naming `lyx beam create <parent>`, and `lyx adopt <parent>` when its branch is on origin; an owner write lost to another machine refused naming `lyx adopt <slug>`; a root claim another machine wins, after which the parent reads owned elsewhere and the create is refused naming `lyx adopt root`; a beam another machine owns naming `lyx adopt <slug>`; a parent owned elsewhere naming `lyx adopt root` and, for a nested beam, the parent beam; a fresh project claiming root, recording owner, flag and pawl, and starting the run with its anchor; a pawl run in flight reported live with no start; a flagged beam with no run status started again; a refused first pass erroring with its envelope.
`TestCreate_StepFailureNamesTheRerun` covers the owner write's and the start's failures naming the re-run; `TestCreate_RunsFromABeamAndReachesTheVerbsOwnRefusal` runs from the parent beam.
`docs/overview.md`'s beam line says create starts the pawl run.

**ImpactSummary:** the create verb's steps are replaced inside beamcli; its three removed helpers have no caller outside this file, and every row of its tests is on this card.

**Commit:** 21: `lyx beam create` records the beam and starts its pawl run
