# batten: wait through awaiting, keep run records past teardown

The hub orchestrator can now drive a task with `lyx batten run <slug>` from the prime, without hand-running `lyx fabric add`, `lyx loom start --no-attach` or `lyx fabric remove`.
Three gaps blocked that.

## What changes

- **An `awaiting` child is watched, not a hard error.**
  `Run-Shed` (`internal/battenshed`) sleeps `poll_interval_s` and returns Stuck through its self-route.
  The reason carries the child's awaiting reason and the hand-off instruction, `lyx loom approve` in the task worktree.
  `blocked`, `failed` and `paused` children still halt with `haltedChildRemedy`.
- **Batten resumes an approved child once per approval.**
  When the child is `awaiting` and its approval record exists, `Run-Shed` calls `deps.Spawn` and writes a marker holding the approval's `approved_at` and `head_sha`.
  The same approval never spawns twice; a new approval record does.
  The marker is written only after `Spawn` succeeds, so a spawn error stays a hard error and a resume retries it.
  `lyx loom approve` itself is unchanged and still never resumes the run.
- **Awaiting time no longer spends the watch budget.**
  `shedengine.OutputPointer` gains a budget-exempt flag, persisted on the history entry and skipped by `episodeStuckCount`.
  Routing is unchanged.
  `Run-Shed` sets it for an `awaiting` child and for a `done` child's driver-exit wait, never for a `running` child.
  Status files written before the change decode as not exempt.
- **A `done` child is finished only after its driver has ended.**
  `Run-Shed` checks the driver strand through an injected liveness seam and waits up to `driver_exit_grace_s` (default 900, on the `Run-Shed` row of `batten-recipe.yaml`).
  Past the window it logs a warning and returns Done.
  A driver killed with its report uncommitted leaves a dirty weft; `Worktree-Teardown` then blocks and names `lyx loom commit-records` plus a batten resume, never `--force`.
- **Friction notes and drive reports are durable run records.**
  Friction notes move from `.lyx/loom/friction/` to `_lyx/loom/friction/` (archives with them); the reflection lock stays at `.lyx/loom/friction.lock`.
  Drive reports move to `_lyx/shed/<slug>/drive-reports/`, declared by `shedrun`.
  Old-path files are not migrated.
- **Records are committed by Go.**
  The per-transition status commit stages the friction tree, its archive siblings and the drive-reports directory when they hold a file.
  The new verb `lyx loom commit-records` commits and pushes the records at the driver's stop, since the stop report is written after the last step.
  It is a no-op on a clean tree and skips during a merge.
  The ly-drive launch prompt runs it before the strand removal, and the removal still runs if the commit fails.
- **Teardown archives the weft tip.**
  Before `fabricengine` deletes an existing pair's weft branch (`Remove`, `RemovePairBranch`, prune), it tags the tip under `archive/` and pushes the tag.
  Batten's teardown deletes the weft branch on origin, so a weft commit alone would not survive it.
  `Add`'s rollback of a branch it just created is excluded.
  The remove result reports the tag name, and batten logs it.

## Design decisions to review

- The archive tag name is derived from the slug and tip SHA, so a resumed teardown finds it and proceeds.
- `Remove` pushes the tag after its refusals and before its first mutation, so a failed push leaves the whole pair in place and a plain re-run retries.
- `force` does not bypass the archive; it still answers dirtiness only.
  The archive runs whatever `remote` says.
  A weft with no origin skips the tag with a reason.
- The exempt flag is plain data, so the three-outcome producer contract and the `shedengine` import limits hold.
  battenshed gets approval and liveness through injected `InnerRunDeps` seams and imports no loom or landing package.
- Recipe row names and `max_bounces: 1440` are unchanged; the recipe header comment describes the exempt wait and grace.
- Docs, `CONSTRAINTS.md` and the ly-drive skill are updated for the moved paths and the awaiting flow.

## Accepted behavior changes and risks

- `lyx fabric remove`, `RemovePairBranch` and prune now fail while the weft origin exists but is unreachable; they succeeded offline before.
- Archive tags accumulate on the weft origin, one per removed pair; nothing prunes them.
- Each exempt bounce appends a history entry to prime's batten status file, so a long await grows it.
- A run started on the old binary and resumed on the new one splits its friction notes between the old and new paths; the old ones are never reflected or committed.

## To check by hand

- Run one real task through `lyx batten run` with a PR left awaiting for a while, then `lyx loom approve`; confirm the run resumes, tears down, and the notes and drive report are reachable from the `archive/` tag on the weft origin.
- Check `lyx batten status` rendering after a few thousand exempt history entries.
- Confirm a driver that outlives the grace window produces the `lyx loom commit-records` remedy at `Worktree-Teardown`.

The integration suite passed at the final head; unit tests and `go vet -tags smoke` also passed.
