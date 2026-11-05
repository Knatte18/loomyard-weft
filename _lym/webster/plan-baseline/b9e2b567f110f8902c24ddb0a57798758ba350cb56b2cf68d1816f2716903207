# Card 8 — retired-move-pawl

**Create:**
- `plan:internal/pawlengine#RetiredMoveDeps` -> `type RetiredMoveDeps struct`
- `plan:internal/pawlengine#MoveOffRetired` -> `func MoveOffRetired(deps RetiredMoveDeps, from, to string, resumed bool) error`
- `plan:internal/pawlengine_test#TestMoveOffRetired` -> `func TestMoveOffRetired(t *testing.T)`
- `plan:internal/loomcli#pawlRetiredMove` -> `func pawlRetiredMove(location *lymcwd.Location, slug string, pawl pawlengine.Deps) func(from, to string, resumed bool) error`

**Edit:**
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli/doc.go`

**Extends:**
- `internal/pawlengine/doc.go`

**Uses:**
- `internal/pawlengine#UpdateLanding`
- `internal/pawlengine#RemoveReady`
- `internal/pawlengine#RemoveLandRequest`
- `internal/pawlengine#Paths`
- `internal/loomengine#BeamLandingPath`
- `internal/loomengine#BeamLandingLockPath`
- `internal/loomengine#BeamReadyPath`
- `internal/loomengine#BeamReadyLockPath`
- `internal/loomengine#LandingDirRel`
- `internal/shedrun#PullRequestRecord`
- `internal/shedrun#PullRequestRecordLock`
- `internal/shedbuild#ShedPaths`

**Intent:** Give a pawl run moved off a retired row (card 1's seam) the side effects D13 names, so an in-flight landing under the old rows comes back to Beam-Await paused for the orch.

- `RetiredMoveDeps` carries `Slug`, `Paths`, `LandingPath`, `LandingLockPath`, `ReadyPath`, `ReadyLockPath`, and the seams `EndSession func() error` (ends a live PR-Review session of the run), `ScratchPullRequest func() (number int, url string, found bool, err error)` (the run's scratch pull-request record as it stands), `PullRequestOpen func(number int) (bool, error)` (whether GitHub reports it open), `BeamHead func() (string, error)`, `RemoveScratch func() error`, `Commit func() error` (commits the beam worktree's landing directory), `SetStatus func(producer string) error` and `Notify func(line string) error`. A nil seam is refused naming the field.
- `MoveOffRetired` runs, each step idempotent so a retried move repeats safely: end the session; read the scratch record and, when one is found, ask `PullRequestOpen`: an open one, and one whose GitHub read fails (logged, so the pull request is never dropped on a transient failure), is carried into the landing record through `UpdateLanding` with the beam's head as both its pushed and announce head; a closed or merged one is not carried; then the scratch record is removed; remove the ready record (`RemoveReady` for the slug), the land request, the dobby-stop marker and the reopen request an older binary wrote; commit the carry and the ready record's removal together through `Commit`; write the entry's board status naming to, which lifts the hold, since to is Beam-Await; post one notice: the run was moved from `from` to `to` and paused, the pull request stays open when one was carried, run `lym beam start <slug>` and `lym loom resume` in the beam's worktree, a second `lym loom resume` when resumed is true (the resume that made the move ends at the pause), then the verbs `lym beam describe`, `lym beam pr open|update|status|close` and `lym beam ready`. Any failed step but the notice is returned with the step named, and card 1's step then persists nothing and names the resume as the way forward, so the next step retries; a failed notice is logged only.
- `pawlRetiredMove` fills the deps over the hub: the paths from `hubgeom.PawlPaths`, `loomengine.BeamLandingPath`/`BeamLandingLockPath` and `BeamReadyPath`/`BeamReadyLockPath`; `EndSession` ends the PR-Review session strand the scratch record names through `runproc` (`PATTERN-run-process-owner`); `ScratchPullRequest` reads `shedrun.PullRequestRecord`; `PullRequestOpen` opens `pullrequest.Open` over the pair's origin and reads the number's state; `BeamHead` reads the pair's HEAD through fabricengine; `Commit` is `fabricengine.CommitAnchoredPaths` over `loomengine.LandingDirRel()` in the beam worktree; `SetStatus` and `Notify` are the pawl deps'.
- `loomCLI.wire` sets `c.shedPaths.RetiredMove` to `pawlRetiredMove` for a beam run (`record.project`) and `RetiredWayForward` to `lym loom resume` for every loom run.
- Bound: only a status naming a retired row moves; a run at Finalize or past it is untouched (card 1), and card 11's Finalize lands through an open pull request the old PR-Review left in its scratch record.

Tests (tier 0): the new `TestMoveOffRetired` drives the function over a temp dir with fake seams: `EndSession` called once; an open scratch pull request is carried into the landing record with the beam's head and the scratch removed; a scratch pull request whose `PullRequestOpen` fails is carried too; a closed one is not carried and the scratch is still removed; the ready record, land request, dobby-stop marker and reopen request are gone; `Commit` ran once after the carry and the removal; `SetStatus` named Beam-Await; one notice names `lym beam start`, `lym loom resume` once, or twice with resumed true, and the pull request when carried; with no scratch record nothing is carried and the rest still happens; a failing `Commit` returns its error naming the step before the status write; a second call after a full one changes nothing on disk. `TestWire_BeamRecipe` asserts the beam run's ShedPaths carries `RetiredMove` and `RetiredWayForward`.

**ImpactSummary:** one new pawlengine function with its deps and one loomcli wiring helper set only for a beam run; no existing behaviour changes for a run on a recipe row.

**Commit:** 8: Move an in-flight pawl run off its retired rows to Beam-Await
