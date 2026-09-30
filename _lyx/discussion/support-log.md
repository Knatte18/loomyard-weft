# Support log — batten-hub-ready

## Interview

The session ran autonomously; no operator answered.
Each turn below is a question the design had to settle, with the answer taken from the codebase or picked under auto mode.

1. **Who resumes the child after `lyx loom approve`?**
   Codebase: `approve.go` writes the approval record and deliberately never resumes the run.
   Since `operator-surface`, a bootstrap-launched ly-drive removes its own strand after its stop report, so nothing is left to re-step an awaiting child.
   The task goal removes hand-run `lyx loom start --no-attach`.
   Auto-pick: batten's `Run-Shed` resumes the child through its existing `Spawn` seam once it sees an approval record, once per approval identity.
2. **How does batten know an approval is new?**
   Re-spawning every poll while the record exists would launch an LLM driver every 30 s whenever Publish re-parks on a stale approval.
   Auto-pick: a marker under `Run-Shed`'s scratch directory holding the acted-on approval's `approved_at` and `head_sha`.
3. **How does awaiting time stay off the budget?**
   Codebase: `episodeStuckCount` counts every Stuck entry for the row since its last Done; `Awaiting` as an outer outcome would halt the batten run itself.
   Auto-pick: a budget-exempt flag on `OutputPointer`, persisted on the history entry and skipped by the count.
4. **Where do friction notes and drive reports go?**
   Codebase: `LoomFrictionDir` is under `.lyx/loom/`; `driverReportPath` is under `shedrun.ScratchDir`.
   The Durable-vs-Ephemeral invariant mirrors `.lyx` subpaths onto `_lyx`.
   Decision: `_lyx/loom/friction/` (lock stays in `.lyx/loom/`) and `_lyx/shed/<slug>/drive-reports/` (accessor in `shedrun`).
5. **When is the drive report committed?**
   Codebase: ly-drive writes the stop report after its last `lyx shed step`, so no transition commit sees it.
   Auto-pick: a new `lyx loom commit-records` verb, run by the driver as part of its end-of-session command before the strand removal.
6. **Does the weft branch survive teardown?**
   Codebase: batten's `Worktree-Teardown` calls `Topology.Remove(location, slug, false, true)`, which deletes the weft branch locally and on origin; the remote deletion exists so a later create of the same slug can push.
   The task body's premise is therefore wrong: a weft commit is gone after a batten teardown.
   Auto-pick: fabricengine tags the weft tip under `archive/` and pushes the tag before any weft-branch deletion.
7. **Can batten tear down before the driver has committed its report?**
   Yes: batten polls every 30 s and would reach teardown, which kills sessions, while ly-drive is still writing.
   Auto-pick: `Run-Shed` on `done` waits for the driver strand to end, bounded by `driver_exit_grace_s` (default 900).
8. **Testing approach.**
   Unit tests for the new `Run-Shed` arms over the existing fake deps, shedengine budget tests for exempt entries, accessor path tests, pathspec tests, a fabricengine test for the archive tag against the hermetic git fixtures, and one batten integration test covering awaiting → approval → done → teardown.

## Rejected alternatives

- **Orchestrator runs `lyx loom start` after approving.**
  Keeps a hand-run step the goal removes.
- **`lyx loom approve` resumes the run itself.**
  approve deliberately never resumes, so a supervisor stepping the run is never raced; reversing that reopens the race.
- **Re-spawn an awaiting child on every poll.**
  Each spawn launches an LLM driver that re-parks at Publish without a valid approval.
- **`Run-Shed` returns `Awaiting` for an awaiting child.**
  The outer batten run would halt and need a manual resume, the failure this task removes.
- **`Run-Shed` loops internally while the child is awaiting.**
  Breaks the one-check-per-Call design, keeps status stale, and makes `lyx batten pause` wait out the whole approval.
- **A fourth producer outcome (`Waiting`).**
  Widens the three-outcome seam contract across every producer and recipe validator for one row's need; a flag on the Stuck verdict is enough.
- **A separate, larger awaiting budget.**
  An awaiting child waits on a person with no natural bound; any number chosen is arbitrary and still fails an approval left over a weekend.
- **Keep the weft branch on origin at batten teardown.**
  A later create of the same slug refuses its push when the remote branch exists.
- **Copy records into prime's `_lyx` (`main-weft`) at teardown.**
  `main-weft` is local-only and never pushed, so the copy does not leave the machine; it also has batten reach into loom's record layout.
- **Batten's teardown commits the child's records.**
  Covers only the batten route, not a manual `lyx fabric remove`, and still races a driver writing its report.
- **Count the done-wait's Stuck bounces against the running budget.**
  They share one episode with the running watch, so a child that ran close to 12 hours would block batten on budget exhaustion after it is already `done`; the grace window already bounds the wait.
- **Tag the weft tip at the weft-branch deletion inside `Remove`.**
  `Remove` reaches that point after the portal, launchers and both worktrees are gone, so an abort there leaves a half-torn pair a plain re-run cannot finish.
- **Let `force` skip the archive tag.**
  `force` answers dirtiness only, never a live merge record; widening it to skip the archive would let an offline removal silently drop the records the tag exists to keep.
- **Include ly-drive's `repairs/` records.**
  The task names friction notes and the driver report only, and the hub loop reads only those two.

## Review rounds

_No rounds yet._

## Question ledger

- **Resolved (auto-pick):** batten resumes an approved awaiting child, once per approval identity.
  Rationale: removes the last hand-run step; the marker bounds respawns.
- **Resolved (auto-pick):** awaiting bounces are budget-exempt via a flag on the Stuck verdict, not a separate budget.
  Rationale: approval time has no natural bound.
- **Resolved (auto-pick):** friction at `_lyx/loom/friction/`, drive reports at `_lyx/shed/<slug>/drive-reports/`.
  Rationale: mirrors the existing `.lyx` subpaths under the durable tree, per the Durable-vs-Ephemeral and Shed Run-Directory invariants.
- **Resolved (auto-pick):** `lyx loom commit-records` run as the driver's end-of-session command.
  Rationale: the stop report postdates the last transition commit.
- **Resolved (auto-pick):** archive tag on the weft tip before every weft-branch deletion.
  Rationale: the task body assumed the pushed weft branch survives teardown; batten's teardown deletes it on origin.
  The operator should confirm this repair when reviewing, since it corrects the body's premise.
- **Resolved (auto-pick):** `Run-Shed` waits for the driver strand to end after `done`, with a 900 s grace.
  Rationale: prevents teardown from killing the driver before its report is committed.
- **Resolved (auto-pick):** the done-wait's Stuck bounces are budget-exempt, like the awaiting wait.
  Rationale: the grace window bounds the wait, and the running watch may already have spent the budget.
- **Resolved (auto-pick):** the archive tag is pushed whatever `remote` says, and `force` does not skip it.
  Rationale: keeps `force` dirtiness-only; the cost, offline pair removal, is recorded as an open risk for the operator to weigh.
- **Resolved (auto-pick):** friction notes an in-flight run left under the old `.lyx` path at deploy are not reflected, committed or migrated.
  Rationale: deploy is operator-timed and can wait for in-flight runs to finish.
- **Resolved (codebase):** `blocked`, `paused`, `failed` stay halts; a child `blocked` at Publish is not auto-resumed after approval.
  Rationale: the body keeps those three as halts.
- **Open:** history growth in prime's batten status file during multi-day awaits; left to the plan to measure.
- **Open:** whether `lyx loom start --no-attach` on an `awaiting` run launches a driver without any extra flag; the ly-drive skill says so, the plan verifies against `start.go`.
