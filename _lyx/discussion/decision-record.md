# Decision record — batten-hub-ready

## Goal

The hub orchestrator drives every task with `lyx batten run <slug>` from the prime, and never hand-runs `lyx fabric add`, `lyx loom start --no-attach` or `lyx fabric remove`.
Three gaps block that today:
an `awaiting` child fails the batten run,
teardown destroys the run's friction notes and drive reports,
and time spent awaiting PR approval exhausts `Run-Shed`'s watch budget.
The target flow is: batten runs in the background;
the child stops `awaiting` at Publish;
the orchestrator reviews the PR and runs `lyx loom approve` in the task worktree;
batten resumes the child by itself, sees `done`, waits for the child's driver to finish, and tears the pair down with the run's records preserved.

## Scope

In:

- `internal/battenshed` `Run-Shed` (`innerRunProducer`): an `awaiting` child is watched, not a hard error; an approved awaiting child is resumed once per approval; a `done` child is finished only after its driver session has ended or a grace window elapses.
- `internal/battencli` wiring for the new `InnerRunDeps` seams (approval read, driver liveness) and `contracts/recipes/batten-recipe.yaml` config for the grace window.
- `internal/shedengine`: a producer can mark a Stuck verdict budget-exempt; exempt history entries are not counted by the bounce budget.
- Friction notes move from `.lyx/loom/friction/` to `_lyx/loom/friction/` (archive siblings with them); the reflection lock stays under `.lyx/loom/`.
- Drive reports move from `shedrun.ScratchDir` to a `drive-reports/` directory under `shedrun.RunDir` (`_lyx/shed/<slug>/drive-reports/`).
- loom's per-transition status commit also stages the friction tree and the drive-reports directory when they hold a file.
- A new loom verb, `lyx loom commit-records`, commits and pushes the run records; the ly-drive launch prompt's end-of-session command runs it before the strand removal.
- `internal/fabricengine`: every path that deletes a pair's weft branch first tags the weft tip under an archive namespace and pushes the tag, so the records survive branch deletion.
- Module docs, `CONSTRAINTS.md` and the ly-drive skill text, updated in the same commit wherever they name the moved paths or the awaiting flow.

Out:

- Deploy (stays with the hub prime after landing).
- Opening VS Code (`batten-open-ide`).
- ly-drive's `<scratch_dir>/repairs/` records and every other `.lyx` scratch file: they stay machine-local.
- `blocked`, `failed` and `paused` children: they stay halts with `haltedChildRemedy`, including a child `blocked` at Publish that the operator then approves.
- Migrating notes or reports already written under the old `.lyx` paths.
- Any change to `lyx loom approve`: it still never resumes the run.

## Decisions

- **An `awaiting` child is watched like a `running` one.**
  `Run-Shed` sleeps `poll_interval_s` and returns Stuck through its existing self-route, with the child's reason and the hand-off instruction (`lyx loom approve` in the task worktree) in the Stuck reason.
  `awaitingChildRemedy` as a hard-error text goes away.
  Rationale: awaiting is a planned hand-off, and the batten run must survive it without a manual resume.
- **Batten resumes an approved awaiting child, once per approval.**
  When the child is `awaiting` and the child's approval record (`loomengine.LoomApprovalPath` of the task worktree, decoded through `landingshed`) exists, `Run-Shed` calls `deps.Spawn` (the idempotent `lyx loom start --no-attach` bootstrap) and records that approval's identity (`approved_at` plus `head_sha`) in a marker under its scratch directory.
  A later Call that sees the same approval identity while the child is still or again `awaiting` does not spawn again; only a new approval record triggers another resume.
  battenshed reads the approval through a new injected `InnerRunDeps` seam, wired in `battencli`, so battenshed gains no loom or landing import.
  Rationale: the orchestrator's only action is `lyx loom approve`; the per-approval marker stops a Publish that re-parks on a stale approval from respawning an LLM driver every poll.
- **Awaiting-wait bounces do not consume the watch budget.**
  `shedengine.OutputPointer` gains a budget-exempt flag; shedengine persists it on the history entry, and `episodeStuckCount` skips exempt entries.
  Routing is unchanged: an exempt Stuck still follows `OnStuck`.
  `Run-Shed` sets the flag only for an `awaiting` child.
  Rationale: approval time is unbounded by design, and running time keeps its 12-hour budget counted across the whole child run minus the awaiting spans; no fourth outcome value, so the three-outcome producer contract holds.
- **A `done` child is finished only after its driver has ended.**
  On a `done` child, `Run-Shed` checks the child's ly-drive strand through a new injected liveness seam.
  While the strand is live and the grace window (`driver_exit_grace_s` on the `Run-Shed` row's config, default 900) has not elapsed, it sleeps and returns Stuck (not exempt) with a reason naming the wait.
  When the strand is gone, it returns Done.
  When the grace window elapses with the strand still live, it logs a warning and returns Done; `Worktree-Teardown`'s session shutdown and `Remove`'s clean-worktree check take it from there.
  The grace window's start is a first-seen-done timestamp marker under the producer's scratch directory.
  Rationale: the driver writes its stop report after the final step, and teardown would otherwise kill it before the report is written and committed.
- **Friction notes live at `_lyx/loom/friction/`.**
  `loomengine.LoomFrictionDir` and `LoomFrictionArchivePrefix` move onto the durable tree (a new durable loom segment accessor alongside the existing `.lyx` one); `LoomFrictionLock` stays at `.lyx/loom/friction.lock`, the mirrored subpath.
  Every consumer (loom, webster, plan, discussion, reflection) keeps calling the same accessors.
  Rationale: the notes are run records, and the Durable-vs-Ephemeral invariant puts tracked content under `_lyx` with its lock at the mirrored `.lyx` subpath.
- **Drive reports live at `_lyx/shed/<slug>/drive-reports/`.**
  `shedrun` gains the accessor for that directory (and its anchor-relative form), since it declares the run directory; `driverReportPath` composes onto it, keeping its timestamp-plus-random filename.
  Rationale: the report belongs to one run, and `shedrun` is the sole declarer of the `shed` segment.
- **The per-transition status commit stages the records.**
  `statusCommitPathspec` adds the `_lyx/loom/` friction tree and the drive-reports directory under the same "holds at least one file" rule it applies to `_lyx/reviews/`.
  Rationale: friction notes are written during producers, so the transition that follows commits them, through the route `_lyx/reviews/` already uses.
- **`lyx loom commit-records` commits the records at the driver's stop.**
  It stages the same records pathspec (status file included), skips while a merge is active exactly as the status commit does, commits through `fabricengine.CommitAnchoredPaths` and pushes through `fabricengine.PushAnchored`.
  A clean tree is a successful no-op.
  The ly-drive launch prompt's end-of-session command becomes `lyx loom commit-records` followed by the strand removal, joined so the removal runs even when the commit fails.
  Rationale: the stop report is written after the last `lyx shed step`, so no transition commit can capture it; the commit stays Go through fabricengine, which the Fabric Git Invariant requires, while the agent only triggers it.
- **Every weft-branch deletion archives the weft tip first.**
  Before `fabricengine` deletes a pair's weft branch (`Remove`, `RemovePairBranch`, and prune's pair removal), it creates a tag on the weft tip named deterministically from the slug and the tip SHA under an `archive/` namespace, and pushes it to the weft origin.
  The tag name is idempotent: a resumed teardown on the same tip finds the tag already there and proceeds.
  A failed tag push aborts the deletion before any branch is removed and surfaces as an error, so batten's teardown halts resumable instead of losing records.
  A weft repo with no origin skips the tag with a reason in the result, as `RemoteSkippedReason` already does.
  The remove result reports the tag name, and batten's teardown logs it.
  Rationale: batten's teardown deletes the weft branch locally and on origin (`remote: true`, needed so a later create of the same slug can push), so a weft commit alone does not survive teardown; a tag keeps the commits reachable without blocking slug reuse.

## Constraints

- **Batten Bookend Invariant:** the new seams are read from the prime; batten never drives from inside the task worktree, and `Worktree-Teardown` still sequences session shutdown before removal in one producer.
- **Fabric Git Invariant:** every commit, tag and push goes through `internal/fabricengine` in Go; the ly-drive agent triggers `lyx loom commit-records` but never runs git itself.
- **Fabric Destruction Chokepoint Invariant:** the archive tag is created ahead of the branch deletion inside the existing chokepoint flow; no destructive primitive leaves `destroy.go`.
- **Durable-vs-Ephemeral State Invariant:** moved records go under `_lyx`; their locks stay at the mirrored `.lyx` subpath; no engine spells `.lyx` or `_lyx` itself.
- **Shed Run-Directory Invariant:** only `shedrun` names the `shed` segment; the drive-reports accessor lives there.
- **Shed Producer-Seam Invariant:** `internal/shedengine` still imports only stdlib, `state`, `lock`; the exempt flag is plain data on `OutputPointer` and the history entry.
- **battenshed seam-enforcement scan:** approval and liveness arrive through injected `InnerRunDeps` fields; battenshed imports no resolver, loom or landing package.
- **CLI/Cobra Invariant:** `lyx loom commit-records` goes through the module `Command()` seam with a `Short` and the help-tree tests.
- **Recipe row names** in `batten-recipe.yaml` stay unchanged (resume depends on them); `max_bounces: 1440` stays, and the file's header comment is updated to describe the exempt awaiting wait and the driver-exit grace.
- **Status history compatibility:** a status file written before the change (no exempt field) decodes and counts exactly as today.
- **Documentation Lifecycle:** module docs, `docs/overview.md` if affected, and `CONSTRAINTS.md` land in the same commit; `manifest/roadmap.md` does not move.

## Auto-mode assumptions

- The body's premise that records "survive teardown on the pushed weft branch" is false for batten, whose teardown deletes the weft branch on origin; the archive tag is the chosen repair (auto-pick, see support log).
- Batten, not the orchestrator, performs the resume after approval (auto-pick): the task goal removes hand-run `lyx loom start`, and the body's "child re-stepped" names no actor.
- "Does not consume the budget" was chosen over "its own larger budget" (auto-pick).
- `lyx loom start --no-attach` on an `awaiting` run launches a driver whose first step resumes the run, as the ly-drive skill states; the plan verifies this against `start.go`.
- The driver strand's display name (`driverStrandDisplayName`) is a sufficient liveness key for the child's driver.
- A grace window of 900 s is enough for a driver to write and commit its stop report.
- Notes and reports under the old `.lyx` paths from runs in flight at deploy time are left in place, not migrated.
- The verb name `lyx loom commit-records` is final unless it collides with an existing verb.

## Open risks

- **History growth during long awaits:** each exempt bounce appends a history entry to prime's batten status file; an approval left for days writes thousands of entries and rewrites the file each time.
  Today's running watch already allows 1440; the plan should check status write cost and `lyx batten status` rendering at a few thousand entries.
- **Stale approval loop:** Publish re-parking `awaiting` on the same approval record relies on the per-approval marker to stop respawns; if Publish rewrites or deletes the record in a way that changes its identity, batten could resume more than once per operator action.
- **Driver never exits after `done`:** past the grace window, teardown's session shutdown kills the driver; an uncommitted report then makes `Remove` (force false) refuse on a dirty weft, halting batten with a failed teardown rather than losing the report silently.
- **Archive tags accumulate** on the weft origin, one per removed pair; nothing prunes them.
- **In-flight runs at deploy:** a run started on the old binary and resumed on the new one splits its friction notes between `.lyx/loom/friction/` and `_lyx/loom/friction/`.

## Acceptance criteria

- `Run-Shed` on an `awaiting` child without an approval record: sleeps, returns Stuck with the exempt flag, reason names the awaiting reason and `lyx loom approve`, and does not spawn.
- `Run-Shed` on an `awaiting` child with an approval record not yet acted on: spawns once, writes the approval-identity marker, and returns Stuck; a second Call with the same approval and the child still `awaiting` does not spawn; a Call after a new approval record spawns again.
- `Run-Shed` on `blocked`, `paused` or `failed` still returns the hard error carrying `haltedChildRemedy`.
- shedengine: exempt Stuck entries route through `OnStuck` and are not counted by `episodeStuckCount`; a row with a budget of N survives any number of exempt Stucks interleaved with fewer than N counted ones, and blocks on the (N+1)th counted Stuck.
- shedengine: a history entry without the exempt field decodes as not exempt.
- `Run-Shed` on a `done` child with a live driver strand returns Stuck until the strand is gone, then Done; with the grace window elapsed and the strand still live, it returns Done and logs a warning.
- `loomengine.LoomFrictionDir` and its archive prefix resolve under `_lyx/loom/`; `LoomFrictionLock` resolves under `.lyx/loom/`.
- `driverReportPath` resolves under `_lyx/shed/<slug>/drive-reports/`.
- `statusCommitPathspec` includes the friction tree and the drive-reports directory only when each holds a file.
- `lyx loom commit-records`: commits and pushes pending records; is a no-op success on a clean tree; skips during an active merge; has a `Short` and appears in the help tree.
- The driver launch prompt's end-of-session command runs `lyx loom commit-records` before the strand removal, and the removal still runs when the commit fails.
- fabricengine `Remove` with a weft origin: an `archive/` tag on the weft tip exists on origin after removal and the result names it; a resumed removal on the same tip is idempotent; a tag push failure leaves both branches in place and returns an error; no origin skips the tag with a reason.
- `RemovePairBranch` and prune archive the weft tip before deleting the branch.
- Integration: a batten run whose child goes `awaiting`, gets an approval record, resumes, reaches `done`, and is torn down ends with the friction notes and the drive report reachable from the archive tag on the weft origin.

## Notes for the plan writer

- Existing awaiting coverage to rewrite: `internal/battenshed/innerrun_awaiting_test.go` asserts today's hard error.
- `innerRunProducer.Call`'s long doc comment carries the full disposition table; it must describe the new awaiting, resume and done-wait arms.
- `Spawn` in `internal/battencli/wire.go` already runs `lyx loom start --no-attach` in the task worktree's `AnchorPath()`.
- `loomcli/run.go`'s `reflectFrictionRow` comment and the ly-drive skill mention the `.lyx/loom/friction/` location by name.
- `driverTeardownCommand` in `internal/loomcli/driverprompt.go` is the end-of-session command to extend.
- `batten-recipe.yaml`'s header comment explains the 12-hour budget and must be updated.
