# Card 7 — dobby-start-named-beams

**Create:**
- `internal/dobbyengine#OwnedWorktrees`

**Edit:**
- `internal/dobbyengine#EnsureLive` -> `func EnsureLive(deps StartDeps, worktrees []string) (StartReport, error)`
- `internal/dobbyengine#StartDeps`
- `internal/dobbyengine#ensureOne`
- `internal/dobbyengine#TestEnsureLive`
- `internal/hubgeom#DobbyStart`
- `internal/orchcli#ensureDobbies`
- `internal/beamcli#startCmd`
- `internal/adoptcli#adoptDeps`
- `internal/adoptcli#adopt`
- `internal/adoptcli#hubDepsAs`
- `internal/adoptcli#world.deps`
- `internal/adoptcli#TestAdopt`
- `internal/fabriccli#anchorDeps`
- `internal/fabriccli#hubAnchorDeps`
- `internal/fabriccli#startDobby`
- `internal/fabriccli#anchorFake.deps`
- `internal/fabriccli#TestChangeAnchor`
- `internal/battencli#ensureParentDobby` -> `func ensureParentDobby(prime *lyxcwd.Location, slug string, ownerElsewhere func(parent string) (string, error), landingHeld func(parent string) (bool, error), live func(dobbyengine.Geometry) (bool, error), spawn func(dobbyengine.Geometry) error) error`
- `internal/battencli#battenCLI.ensureDobby`
- `internal/battencli#battenCLI`
- `internal/battencli#TestStartFirstPass`
- `internal/dobbyengine/doc.go`

**Intent:**
D1: every `EnsureLive` caller starts only the beams it names, and none of them, nor batten's start of a run's parent dobby, starts a dobby for a beam whose pawl run holds the landing hold (D9).

- `EnsureLive(deps, worktrees)` considers only the named worktrees, the prime's name or beam slugs, the prime first when named.
  A named prime with no recorded owner is claimed first, as today; a named prime another machine owns is refused with `owned by <owner>; lyx adopt root takes it over here`; a named beam another machine owns is refused naming `lyx adopt <slug>` with its owner, instead of being left out; a named beam whose landing hold stands is refused naming `lyx batten status <slug>` before any spawn.
  A named slug that is no local beam pair gets no refusal of its own: every caller names a worktree it resolved itself, `BeamUp` its own, `--all` `OwnedWorktrees`, Beam-Up and Beam-Await their beam, and the anchor move the dobbies it ended, so after card 24 no caller names an unpaired beam.
  `StartDeps` gains `LandingHeld func(worktree string) (held bool, err error)`, read in `ensureOne` after the anchor-move check; a nil one reads as no hold, and a read error is the worktree's failure.
- `OwnedWorktrees(deps)` lists the prime's name when this machine owns the prime or none is recorded, then every local beam pair not owned elsewhere, the set `EnsureLive` covered before; it is what `lyx beam start --all` names (card 11) and what the callers this card does not narrow pass.
- `hubgeom.DobbyStart` fills `LandingHeld` from the local board: the worktree's board entry, read through `boardengine.ReadTasks`, held by `boardengine.RunHolds`; the prime is never held.
- The callers: `orchcli.ensureDobbies` and `beamcli.startCmd` pass `OwnedWorktrees`, until cards 10 and 11 replace them; adopt's `StartDobbies` seam becomes `func(worktrees []string) error` and `adopt` names the adopted beams and the prime's name when root is adopted; the anchor move's `EnsureDobbies` seam becomes `func(worktrees []string) error` and `startDobby` names the worktrees `dobbyWorktrees` says the move ended, a beam's own move included, so its beam branch no longer spawns through the `StartDobby` seam, which goes from `anchorDeps` and `hubAnchorDeps`, and a held beam's dobby is never restarted by a move.
- `ensureParentDobby` takes `landingHeld` and, when the parent beam's landing hold stands, spawns nothing and returns a refusal naming `lyx batten status <parent>`; its owner refusal names `lyx adopt root` for the prime; its failure way forward names `lyx beam start <parent>`, the parent's own beam, or `lyx beam start root` when the parent is the prime, instead of every dobby this machine owns; `battenCLI` gains a `landingHeld func(parent string) (bool, error)` field beside `parentOwner`, which `battenCLI.ensureDobby` passes on, and a nil field reads the hold over the local board through `boardengine.RunHolds`.

Tests: `TestEnsureLive` passes its worktrees and gains rows: a beam not named is not started; a named beam another machine owns is refused naming `lyx adopt`; a named beam whose hold stands is refused naming `lyx batten status` and spawns nothing; a failing `LandingHeld` read is that worktree's failure and the others still start; a nil `LandingHeld` starts the beam; the prime's refusal names `lyx adopt root`.
`TestStartFirstPass` gains a row whose `landingHeld` field reports the parent beam held, which spawns no dobby and names `lyx batten status`, and its prime-owner row asserts `lyx adopt root`; its `FailedDobbySpawnWarnsAndKeepsThePass` row, whose parent is the prime, asserts `lyx beam start root`.
`TestAdopt` asserts the started worktrees are the adopted beams and the prime when root is adopted; `TestChangeAnchor` asserts a prime move restarts only the dobbies it ended and a beam's move names that beam alone to `EnsureDobbies`.

Bound: a dobby left unstarted for a held beam is started again by `lyx beam start` once the hold lifts, which only Beam-Await's own writes do (card 15).

**ImpactSummary:** one re-signed export with four production callers and two test fakes, all on this card; the new hold read touches only the local board.

**Commit:** 7: Dobby starts name their beams and skip a beam its landing holds
