# Pawl joins the loom contract with a project record and no review round

Pawl, a beam's life, becomes a loom recipe. It gets a decision record that holds only what binds the whole project, and `lym loom decision add <beam-slug>` becomes the one way the project's scope changes. Pawl also loses its own Code-Review round, since each member's run reviews its own change.

## What changes

- `contracts/recipes/pawl-recipe.yaml` declares `contract: loom` and `verify_source: plan`.
  It now reads `Preflight → Record-Write → Beam-Up → Beam-Await → Parent-Align → Beam-Describe → PR-Review → Finalize → Beam-Down`.
  Beam-Await is the build row, because the members' runs make the change while it holds the run, so `shedrecipe.LoomBuildEngines` adds `BeamAwait`.
  The Code-Review perch is gone.
- `loomrecipe.NewPawl`, `PawlRouting` and `ServedByLoom` are gone: pawl builds and routes through `loomrecipe.New` and `Routing`, and `lym loom` serves loom runs only.
- **The project record.** Pawl's Record-Write writes the record and a support log from the project entry's body, the same way darn's does, and commits them.
  They live under `_lym/project/<slug>/` (`loomengine.ProjectDecisionRecord` and related functions), not `_lym/discussion/`.
  A pair forked from a beam inherits the beam's records, so a shared path would collide with each member's own record and with an enclosing beam's.
  In `loomcli`, `runRecordOf` picks a run's record from the recipe's rows.
  A recipe with a Beam-Await row gets the project record, and every other recipe gets the discussion record.
  This choice feeds the Record-Write deps, the record history, the commit, the arbiter prompt, the landing deps and `decision add`.
- **The board lock.** `boardengine.ProjectRecordHeld` is true for a beam project whose run is past Preflight.
  For such a project, `checkRunLocks` refuses a body edit with a `ProjectRecordLockedError` that names `lym loom decision add <slug>`.
  Title, brief, labels and membership still take edits as before.
  `holdFreeProducers` adds Record-Write.
- **`decision add` on a beam.** The verb resolves the project record and lets the beam's own orch (the nearest live orch from the beam itself) record the call.
  Its notice goes to the beam's live runs and to every live run in the member pairs `battencli.ReadBeamLandBlockers` lists under the beam.
  The restore way forwards name the record's real path inside `_lym`.
- **Members read the record.** A new stencil, `loom-directive-project-record`, is rendered by `loomengine.ProjectRecordDirective`.
  It fills an optional `project_directive` marker in the Discussion-Write lead's prompt and the darn writer's prompt.
  The directive lists the record of every beam above the pair, nearest first.
  The beams come from the board through `boardengine.ParentBeam`, and each record is read in that beam's own worktree, so a decision recorded after the fork reaches the next spawn.
  The marker shares the pattern directive's line, so a run under no beam gets a byte-identical prompt.
- **Beam-Describe** takes a `ProjectDecisions` seam and adds the record's `Added after Discussion` entries, verbatim, under `## Project decisions` after the commit bullets.
- The board entry snapshot is removed:
  - `darnengine.BoardEntrySnapshot`, `loomengine.DiscussionBoardEntry`, `writeBoardEntrySnapshot`, the `WriteSnapshot` seam of `pawlengine.Deps` and PR-Review's `BoardEntryPath`;
  - the rubric's third answer-key form and the PR-Review stencil's mention of it.
- The orch role stencil says a beam's scope change goes through `lym loom decision add <beam-slug> --by parent`, run by the beam's own orch.
- `PATTERN.md`: `PATTERN-loom-contract` drops "bar pawl until it joins", and a new `PATTERN-project-record` entry states the route.
  The package docs of `loomrecipe`, `loomcli`, `boardengine`, `beamcli`, `landingshed` and `pawlengine` and `docs/overview.md` are updated.

## Design decisions

- The record is written by pawl's own Record-Write row, not by `lym beam create` directly, because the verb makes no pair.
  The run it starts makes the pair and writes the record before Beam-Up brings the beam up.
- Joining the contract also moves two existing loom behaviors onto pawl:
  - a fixer's escalate halt at Finalize goes to the arbiter first (`EscalateJudgedFirst`, `RestepHold`);
  - the arbiter prompt and the landing deps read the project record.
  `docs/overview.md` says so.
- A beam's project record declares no PATTERN changes of its own, so pawl's rows get no `PatternRecordPath`.
- `projectRecordPaths` and `memberPairs` resolve other worktrees through `lymcwd.NamedWorktree`, which spawns no git.
- No retired-row mapping is added for the removed perch rows, because renaming a run's history onto Beam-Describe would misreport it.

## Check by hand

- In-flight pawl runs started before this change have no project record.
  For one parked at Beam-Await, run `lym loom goto --to Record-Write`, then `lym loom resume`.
  Until then, the board refuses its body edits and `decision add` refuses naming that same goto.
- A pawl run parked on a removed perch row (`Code-Bouncer`, `Code-Burler-Review` or `Code-Burler-Fix`) needs `lym loom goto --to Beam-Describe`, then `lym loom resume`.
- Hub stencil copies need `lym stencil sync` for the new directive, the `project_directive` markers, the rubric, the PR-Review stencil and the orch role.
