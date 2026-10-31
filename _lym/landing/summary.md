# Pawl becomes a loom recipe with a project decision record

Pawl now declares the loom contract, so `lym loom` serves a beam's life like any task recipe, with the members' runs in the writer's place.
Until now pawl had no decision record: its Code-Review perch compared against a snapshot of the project's board entry, and `lym loom decision add` could not change a beam's scope.

## What changes

- **Pawl recipe.** `contracts/recipes/pawl-recipe.yaml` declares `contract: loom` and gains a `Record-Write` row between `Preflight` and `Beam-Up`.
  The Code-Review perch is gone, so pawl has no review round of its own; a member's run reviews its own change before it lands in the beam.
  `loomrecipe.NewPawl`, `PawlRouting` and the pawl special cases in `loomcli` are removed, and pawl runs through the shared `loomrecipe.New` and `IsLoom` path.
  `darnengine`'s board-entry snapshot, which only the old perch used, is removed.
- **Project record.** `Record-Write` writes `_lym/project/<beam-slug>/decision-record.md` and its support log from the project entry's body verbatim under `## Goal`, every other section empty, and commits them.
  The record holds only what binds the whole project (goal, bounds, member constraints); it is no log of what landed, and a member landing never touches it.
  It is keyed by the beam's slug because a pair forked from a beam inherits every record the beam carries.
  `Record-Write` writes only a missing file, so a resume keeps the entries `decision add` added.
- **`lym loom decision add <beam-slug>`.** On a beam's run (a recipe with a `Beam-Await` row) the verb records a scope change in the project record.
  The beam's own orch runs it: the nearest live orch from the beam itself.
  Its notice reaches the live agents of the beam's runs and of every member pair under the beam.
  The way-forward texts for a missing or differing record now name the record's actual path.
- **Board.** `lym board upsert` refuses a change to a beam project's body once its run is past `Preflight` (`boardengine.ProjectRecordHeld`, `ProjectRecordLockedError`), naming `lym loom decision add <beam-slug>`.
  The refusal matches `ErrRunLocked`.
  `Record-Write` joins the hold-free producers, so members can still land in the beam until `Parent-Align`.
- **Members read the record.** The Discussion-Write lead and the darn writer prompts gain an optional `project_directive` marker.
  `loomengine.ProjectRecordDirective` fills it from the new `loom-directive-project-record` stencil, with one bullet per project record above the pair, nearest beam first.
  The records are read live from each enclosing beam's worktree, so a decision recorded after a member forked still reaches its next spawn.
  A pair under no beam keeps the marker empty and its prompt bytes unchanged.
- **Beam Describe.** The beam's description now appends the project record's added entries verbatim under `## Project decisions`.
- **Orch role.** The shipped orch role says a beam's scope change goes through `decision add` on the beam.
- `PATTERN.md`, `docs/overview.md` and the package docs are updated to match.

## Design decisions

- The project record is separate from a member's discussion record, so neither a member's record nor an enclosing beam's shares a path with it.
- The board hold is judged from the run status alone, since `boardengine` reads no record file.
  A run sent back to `Preflight` with a goto lifts the hold, and `Record-Write` keeps the record it finds, so a body edit made in that window never reaches the record.
- The pawl recipe's rows are pinned against the hold-free producer lists by the recipe's own test, since `boardengine` imports no recipe package.

## Check by hand

- `decision add <beam-slug>` on a beam (`internal/loomcli/decision.go`: `decisionParent`, `decisionReceivers`) was found under-tested in review.
  Tests were added afterwards; run it against a live beam with a member run and confirm the notice reaches the member's agents.
- `withProjectDirective` (`internal/loomcli/projectrecord.go`) was found under-tested in review.
  Confirm a member's Discussion-Write and darn writer prompts contain the project directive under a live beam, and are unchanged under none.
- Not run: the full pawl life end to end (`Record-Write` through `Beam-Down`) against a real beam with members, and the landing gate over the merged tree.
