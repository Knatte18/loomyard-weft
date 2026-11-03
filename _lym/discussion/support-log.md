# Support log — batten-remote-order

## Interview

### Turn 1 — exploration (autonomous)

Read the board entry, the touching notes `fabric-remote-async` and `landed-run-left-paused`, `lym loom peers` (no peer touches batten or fabric), and the PATTERN backgrounds for remote-refs-lock, push-both-sides, pair-teardown, fabric-git, owner-machine and run-process-owner.
Three Sonnet subagents traced each bug in the code; their findings were checked against `internal/battencli/resumereconcile.go`, `internal/fabricengine/teardownlock.go`, `internal/fabricengine/remove.go`, `internal/fabricengine/weftwiring.go`, `internal/fabricengine/cleanup.go`, `internal/battencli/cancel.go`, `internal/fabricengine/originreconcile.go` and `internal/landingshed/finalize_owner.go`.

Findings that shaped the design:

- Bug 1: the run-scope reconcile errors on an absent pair worktree whatever its mode, and the resume turns that into a bare pre-run refusal, so no status write happens and dobby re-plans a pass for the `running` status forever. `Topology.Add` already adopts a live pair.
- Bug 2: the teardown never pushes the weft branch; it relies on detached pushes, and a detached push that starts while the removal holds the teardown lock exits without pushing. The tag push is raw and not retried on "missing necessary objects". The tackler stencil says nothing about checking a remote.
- Bug 3: every remote deletion of the cancel is synchronous; its spawned push is the prime's records push and deletes nothing, so the board's guess that the background push deleted the new run's weft branch does not match the code. The weft branch's remote delete is unleased, and `Add` checks no in-flight work of the slug.

### Turn 2 — design picks (autonomous)

No operator; every pick is in the question ledger as an auto-pick.

### Turn 3 — lens review

All six forks returned a report.

- bound-breaker + interactions: transient `Add` refusals (unreachable origin, D7 lock wait) became a permanent failed run — fixed: D2 fails only on a permanent answer; transient ones refuse the pass as today.
- interactions: `Add` forks a new pair when not live, so D1 could not be adopt-only — fixed: D1 uses a new adopt-only option with a not-live sentinel.
- walkthrough + existing-paths + bound-breaker: `Add` refuses an unmarked local warp branch without `AdoptLocal`, and a stale registration blocks it — fixed: D1 prunes stale registrations and sets `AdoptLocal`; a surviving records worktree stays a permanent refusal, stated in D1's bound.
- bound-breaker: D1's owner check on the local board could re-create a foreign pair — fixed: the run-scope reconcile now tolerates an absent pair and judges the entry against origin before `Add`.
- bound-breaker: D2's cancel way forward at Land would release a landed entry — fixed: at Land the way forward is `lym fabric add <slug> --adopt-local` then `lym batten run <slug>`.
- existing-paths + walkthrough: `WriteFailedStop` neither commits nor writes on a Busy lock, and writes nothing for a non-`running` status — fixed: D2 commits and spawns the status push, Busy is transient, a non-`running` status keeps its state.
- interactions + existing-paths + bound-breaker: a weft branch kept by the lease after a not-fast-forward push reads as live — fixed: D4 archives a diverged origin tip too, and D6 leases on the origin tip read under the lock.
- interactions + bound-breaker: the cleanup sweep sat outside the lock and the PATTERN text — fixed: D7 and the PATTERN entry include the sweep with one try per slug.
- existing-paths: D4's push under the pause gate — fixed: D4 states it passes the same gate as the tag push already does.
- existing-paths + bound-breaker: the coalesced push carries the code branch and writes failed-push records — fixed: D4 pushes the weft branch alone with no record, and the note says so.
- bound-breaker: "refused" undefined in D4 — fixed: any branch-push failure is logged and never fails the removal.
- walkthrough: the "missing necessary objects" cause unpinned — fixed: added to Open risks.
- existing-paths: a pair gone during continue passes — fixed: a continue pass with an absent task worktree is refused, so the next pass is a resume.
- interactions: a push of the slug dropped while `Add` holds the lock — rejected: no pair of the slug exists while `Add` holds it, so no push of it is lost.
- leanness: Constraints narrative, duplicate open risk, long rationales, pair-teardown and `fabric-remote-async` constraints, auto-mode repeats, duplicate notes — fixed: narrative moved here, rationales cut to what the plan writer needs, the two constraints dropped or moved into D4's bound, auto-mode assumptions point at the ledger, notes trimmed.
- leanness: D3's reached-parent claim unsupported — fixed: D3's rationale cites `childReachedParent`'s behaviour.
- board-scope: D2 writes `failed`, not a hand-back — rejected: the effect the entry wants holds, and Q4 records why.

## Rejected alternatives

- One lock serialising all of a slug's remote work, fetches and origin judges included: broader than the three bugs need, and the reconcile's judges already hold the remote-refs lock for their own window.
- Making `lym fabric remove` clear the batten run record: fabric would write batten's records; naming `lym batten cancel` gives the same result with the record's owner doing the write.
- A blocked hand-back from the pre-run for an unrecoverable pair: the driver has not started, so there is no report or park marker path to use; a failed stop ends dobby's retries the same way.
- Pushing the weft branch and tag in one atomic push: the branch push may be refused while the tag must still go out, so the two stay separate.
- Making a new run wait on the cancel's spawned push: that push is the prime's records push and cannot delete a slug's branch, so waiting on it orders nothing.
- A run id in branch names: changes every pair's naming and every reader of it, for a race the lock and leases close.
- A liveness probe before `Add` instead of an adopt-only option: the probe would run outside `Add`'s teardown lock, so the answer could change before `Add` acts.
- Leasing the weft delete on the local tip: a remover behind origin would keep a tip no archive tag covers, which `Add` reads as live.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): One ordering rule or a fix per bug? — Per-bug fixes, joined by one per-slug order for pair creation, removal, discard, sweep and pushes (D7) and leased deletes (D6). Rationale: the three bugs share remote ordering only in part; bug 1 is a recovery gap.
- Q2 (auto-pick): Recover a gone pair or only stop? — Recover a live pair through `Add` (D1), stop otherwise (D2). Rationale: the board's expected behaviour, and the workaround it names.
- Q3 (auto-pick): Which pairs may D1 re-adopt? — Only a live one by `Add`'s rule, under an adopt-only option, after an origin judge of the entry. Rationale: after `lym fabric remove` the archive tag covers the weft tip, and an owner verdict must read origin.
- Q4 (auto-pick): What does D2 write? — A committed `failed` status through `shedengine.WriteFailedStop`, a way forward by row, and one parent notice. Rationale: the driver never started, so a failed stop is the status writer available, and dobby stops on it.
- Q5 (auto-pick): Fabric remove leaving the record? — Name `lym batten cancel` in the way forward and in fabric remove's warning (D3).
- Q6 (auto-pick): Archive order? — Synchronous weft push before the tag, an extra tag for a diverged origin tip, and a retry on "missing necessary objects" (D4).
- Q7 (auto-pick): Tackler? — A stencil rule to run `git ls-remote` before judging a commit absent (D5).
- Q8 (auto-pick): Cancel race? — Leased weft deletes (D6) and the teardown lock in `Add`, the discard and the sweep (D7); the trigger is not pinned, recorded under Open risks.
- Q9 (auto-pick): PATTERN? — Add one entry for the per-slug order and leased deletes.
- Q10 (auto-pick): D1 at which rows? — Every row but Worktree-Create and Worktree-Teardown, Run-Shed and Land included, with D2's way forward differing at Land.
