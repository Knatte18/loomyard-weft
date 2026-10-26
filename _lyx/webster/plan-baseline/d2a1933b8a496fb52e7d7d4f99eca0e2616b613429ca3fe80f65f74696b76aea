# Card 20 — batten-cancel-beam

**Create:**
- `plan:internal/battencli#refuseBeamCancel` -> `func refuseBeamCancel(slug, rerun string, deps cancelDeps) error`

**Edit:**
- `internal/battencli#cancelDeps`
- `internal/battencli#hubCancelDeps`
- `internal/battencli#endRunPhase`
- `internal/battencli#cancelFake`
- `internal/battencli#cancelFake.deps`
- `internal/battencli#TestCancelRun`
- `internal/battencli#battenCLI.cancelCmd`
- `internal/battencli/cli.go`

**Intent:**
D11: `lyx batten cancel <beam>`, with or without `--remove`, first runs the hub-scope origin reconcile and refuses while `BeamLandRefusal` refuses over the Bolt it pulled, naming the members to land or cancel first, or, once the pawl run holds the landing hold, only to cancel and move off the beam; when origin is unreachable it refuses with the reconcile's way forward.
A cancel of a beam with no members ends the pawl run's processes and the beam's dobby and orch with them, and `lyx beam create <slug>` starts the run again.

- `cancelDeps` gains `BeamProject func() (bool, error)`, whether the slug's board entry is a beam project, `ReconcileHub func() (fabricengine.OriginReport, error)`, the hub-scope origin reconcile from the prime, `LandRefusal func() error`, `BeamLandRefusal` over the board the reconcile pulled, and `StopBeam func() error`, `orchcli.BeamDown` over the beam's worktree when its pair is here; `hubCancelDeps` fills them, and a nil `BeamProject` reads as no beam; the `LandingHeld` and `LandBlockers` fields below are filled there too, and a nil `LandingHeld` reads as no hold.
- `endRunPhase` calls `refuseBeamCancel` first of all, before the cancel marker or any end: for a beam project it runs `ReconcileHub`, and an error or an unreachable origin refuses with the reconcile's way forward and the cancel's re-run; then `LandRefusal`'s refusal is the cancel's, its blockers naming each member to land or cancel first, and nothing is ended.
  Once the beam's entry reads held (`boardengine.RunHolds`), the pawl run's hold refuses a member's landing, so `refuseBeamCancel` words each member blocker as card 14's `landHaltReason` does instead: for a member with a pair or a run in flight, `lyx batten cancel <member> --remove`, run in the beam's parent worktree, then the `lyx board upsert` move of its `project` off the beam; for a member with no run status, that board move alone; never `lyx batten run <member>`; while the hold is not held, it keeps `BeamLandRefusal`'s text.
  It reads them through two more `cancelDeps` fields, `LandingHeld func() (bool, error)` and `LandBlockers func() (BeamLandBlockers, error)`, which `hubCancelDeps` fills with `boardengine.RunHolds` on the beam's entry and `ReadBeamLandBlockers` (card 13), both over the board the reconcile pulled.
  A beam with no members goes on as any cancel: `runproc.End` of the slug ends the pawl run's batten and driver and the beam's dobby, which run under the beam's slug, and `StopBeam` then stops the beam's orch and watcher, a failure there only warning on the report, since the run is already ended; with `--remove` the pair teardown removes what is left.
- `cancelCmd`'s help names the beam refusal and the re-create, and the package comment in `cli.go` describes the beam cancel.

`TestCancelRun` gains rows over `cancelFake`'s new fields: a beam with members refuses naming them with no marker taken and no end; a beam at `running · Parent-Align` with a member names the member's cancel in the parent worktree and the board move, and no `lyx batten run`; an unreachable origin refuses naming the reconcile's way forward; a beam with no members ends and calls `StopBeam` after the end; a failing `StopBeam` warns and the cancel succeeds; a task never calls the reconcile or the beam seams.

**ImpactSummary:** the cancel gains a pre-check and a post-end stop for beam projects only, through new deps fields its one fake fills; a task's cancel is unchanged.

**Commit:** 20: `lyx batten cancel <beam>` refuses while the beam has members, and stops its orch
