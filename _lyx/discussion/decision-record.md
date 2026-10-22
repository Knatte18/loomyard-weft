# Decision record — batten-cancel-verb

## Goal

One verb, `lyx batten cancel <slug>`, ends a batten run for good from the hub's prime, whatever its state: before or after its pair exists, live, paused, blocked, failed or awaiting.
It ends the run's processes and strands, clears the board claim, removes batten's records and scratch, and with `--remove` also removes the pair and its task branches.
`lyx batten run` refuses a dirty source worktree before it writes anything, and a dirty source or a left task branch met later at Worktree-Create halts with its way forward and never reaches a tackler.
On Windows, where `runproc.End` signals nothing, a live run is refused at once until its processes are ended by hand (D4).

## Scope

In:

- `internal/battencli`: the verb, the cancel-marker check and the dirty-source check on every auto-seeding verb, the start's refusal of a left local part of the pair (D5), the pre-drive check in its pre-run hooks' seeding closures and installed in shed (D3), and the cancel notice.
- `internal/shedrun`: the cancel marker's path and its lock path.
- `internal/proc`: a Windows read of a process's creation time by pid, for D4's reused-pid test.
- `internal/pairteardown`: a cancel phase beside `EndSession`, a removal mode that leaves the claim to its caller, and the snapshot taken before an origin-mode removal.
- `internal/fabricengine`: a removal that archives the task branch's local and origin tips and deletes the task branch locally and on origin whether or not its work is landed (D5).
- `internal/battenshed` and `internal/fabricengine`: a typed dirty-source error from the pair add, and Worktree-Create's precondition halt for it and for an existing or leftover task branch.
- `internal/shedengine` and `internal/sheddriver`: the precondition reason prefix and its hand-back row, and the locked pre-drive check `Run` and `Step` call once they hold the run lock (D3).
- Way-forward texts that tell a caller to abandon a run with `lyx board set-status` now name `lyx batten cancel <slug>`: `boardengine`'s run-lock refusal and `pairteardown`'s held-gone settle warning.
- Docs: the package docs of the packages above, the orch role stencil's run-handling line, and `docs/overview.md` where it lists batten's verbs.

Out:

- The board entry's body, scope, owner and every field but its status.
- `lyx batten cleanup` and `lyx cleanup`, which keep skipping non-done runs.
- The child run's records inside a kept pair; they leave with the pair.
- A new batten or shed state such as `aborted`: the records are removed, so nothing would read one.
- Hub-scope pause entries.
- The `ly:cli` skill, whose index `lyx help index` generates from the binary.

## Decisions

### D1. The verb is `lyx batten cancel <slug> [--remove]`, run from the prime

Rationale: it sits beside `run`, `pause`, `goto` and `cleanup` in batten's tree, and the task slug and issue name it "cancel".
The verb applies `RefuseUnlessPrime` first, so a pair agent cannot reach it.
The orch may call it: the orch is the operator's delegate, and the 2026-10-21 incident was an orch stop.
Bound: it acts on the one slug named; an omitted or reserved slug refuses, as batten's other verbs do.
Prime-only is today's `RefuseUnlessPrime` rule; `orch-any-worktree` widens it for every batten verb at once.

### D2. The run's state chooses the path; no flag chooses refuse or abort

Every non-terminal state is ended: running, paused, blocked, failed, awaiting, a seed with no status, and a run-form board claim with no records.

The owner and origin reading depends on whether the pair's task worktree is present:

- Present: the run-scope origin reconcile (PATTERN-origin-reconcile), run with `ReconcileScope.JudgeOnly`, so it moves no ref of the still-live pair (`internal/fabricengine/doc.go:83-87`, `originreconcile.go:34-37`).
- Absent, whatever the status reads (none, Worktree-Create with no pair, or any state whose pair is gone): no reconcile, since the run scope errors on a slug with no pair worktree (`internal/fabricengine/originreconcile.go:574-576`).
  The verb pulls Bolt and reads the effective owner from it, as batten's start check does (`refuseStartOwnedElsewhere`, reached from `ownerPreflight` in `internal/battencli/owner.go:194-205`), reusing only that check's pull and owner read:
  it makes no `claimPrimeOwner` write, so a refusal changes nothing, and its refusals carry the cancel's own wording, never `startRefusal`'s `lyx batten run` way forward.

The verb refuses, with nothing changed, in these cases:

- The child's work has reached its parent: a Finalize past its parent-side merge (`battenshed.FinalizePastMerge`), a Finalize Done in the child's status history, or batten past Run-Shed with the child done.
  Way forward: `lyx batten run <slug>` to finish the landing, then `lyx batten cleanup --apply`.
- The reconcile reads the run's own entry ended, the entry reads done on the Bolt tip the reconcile fetched, and batten's run record is still present here: another machine's landing reached it, since the ended set drops every run this machine's own landing reached (`internal/fabricengine/doc.go:76-80`).
  Way forward: `lyx cleanup --apply` from the prime, whose reconcile tears the ended run down, since it tears down only an ended run with a batten run record here.
  An ended entry with no batten run record here, such as a kept pair whose entry the caller marked done after an earlier cancel (D7), does not refuse: the cancel goes on to the removal.
- The entry is owned by another machine: way forward `lyx adopt <slug>` on this machine, or the cancel on the owner machine.
- The reconcile reads a stopped branch: the refusal names the branch's own way forward, then the cancel again.
  A judge-only reconcile reads no dirty or busy branch, since only a move adds those states.
- The reconcile or the Bolt pull errors for any cause but an unreachable origin: way forward fixing the cause, then the cancel again.
- A live hub-scope pause entry covers the prime when the verb starts: way forward `lyx tender resume`, then the cancel again.
- On Windows, a process is registered live under the slug (D4).
- Nothing is left to cancel: no records run directory, no run-form status, no task or weft worktree, no local weft branch and no launcher entry of the pair, no reed session of the slug, no live process registered under the slug and no cancel marker (D3), and with `--remove` no task branch of the slug locally or on the warp origin either.
  Way forward: check the slug with `lyx board list`.
  When no `--remove` was given and a task branch of the slug is left locally, a local ref read with no network call, the way forward is `lyx batten cancel <slug> --remove` instead.

The reconcile's verdict does not say why an entry ended, so the verb reads the entry on the fetched Bolt tip to split it.
An entry absent there left the board, as on 2026-10-18 when an entry was re-scoped off it, and does not refuse: the cancel goes on, and D7's clear finds no entry.

Past the hub-pause check the verb exempts its own network gate from every live pause for its duration, as `lyx fabric remove` does (`internal/fabriccli/doc.go:56`), so a hub pause put in place meanwhile, by `lyx hibernate request` among others, never holds its network calls while it holds batten's locks.
Its detached children, the records push and the board sync, keep their own gates.

An unreachable origin, met by the reconcile or the Bolt pull, makes the cancel end-only: the verb ends the run (D3 steps 4 and 5) with a Warn, releases batten's locks, posts its notice and stops there, with or without `--remove`.
It removes no pair, no records and no claim, lifts no pause entry, and leaves the cancel marker, so no start resumes the run; the envelope and the notice name `lyx batten cancel <slug>` once origin answers as the way to finish.
Bound: offline the owner comes from the local Bolt, which can predate another machine's adopt, and stopped branches go unread.
Ending this machine's own processes of such a run is accepted, since an adopted run's driver lives on the owner machine; removing records on the shared prime records branch and clearing the claim are not, since both would sync onto the owner's live run, so they wait for origin.
`--remove` cannot run offline either way, since `Remove` stops at its archive push with everything in place.

Rationale: the operator decision of 2026-10-22 makes one verb end a run whatever its state; the refusals left protect landed work and other machines' runs.
Bound: the verb can end a working agent mid-turn and a step mid-write; it cannot end work that reached the parent or a run origin reads as owned elsewhere.

### D3. Order of the cancel, each step a no-op when already done

1. The prime, slug and hub-pause checks, then the network-gate exemption.
2. The owner and origin reading, then its refusals (D2).
3. Read what later steps need before anything is removed: batten's status and its state and row, and the child's landed reading and its refusal.
4. Write the cancel marker under its lock.
   On Windows, then list the slug's live registered processes and refuse on any (D4); that refusal releases the marker's lock, and removes the marker only when this cancel wrote it fresh, so nothing is left changed and a marker taken over from an interrupted cancel keeps barring starts.
5. End the run (D4), which leaves batten's prime-side driver lock and run lock held by the verb, then read batten's in-flight tackler marker and stop the strand it names (D4).
   An unreachable origin (D2) then runs only step 9's lock release and notice, and leaves the lift, the records push and the marker removal.
6. With `--remove`, remove the pair (D5).
7. Under batten's prime-side status lock, taken here and released after, remove batten's records run directory and empty its scratch beside the lock files, then commit the records side (D6).
8. Clear the board claim (D7).
9. Release batten's locks, lift the slug's own live pause entries (D8), spawn the records push, post the notice (D9), and remove the cancel marker last.

The cancel marker is a `shedrun` path outside the run directory and the scratch directory, so step 7 leaves it; it records the verb's pid and start time, which the refusals below name.
Its writer holds an exclusive lock on a `shedrun` lock path beside it from before writing it until after removing it, and that lock, never the pid, is the writer's liveness:
the OS releases it when the writer dies, so a reused pid, on Windows where no start time is read as elsewhere, never makes a dead cancel read alive.
A reader that finds the marker try-acquires that lock shared: held means the writer is alive, and free means it is gone if the marker still exists once the shared lock is held, and that no cancel runs if the writer removed it meanwhile.
Batten's pre-run, `armAt`, reads the marker first, ahead of the start owner check (`refuseStartOwnedElsewhere`, `internal/battencli/arm.go:377-383`) and beside D10's dirty check, so before the board pull, the `claimPrimeOwner` write and `armSeed`, which writes the seed and registers the process before any batten lock is taken.
It refuses every verb that drives or resumes the run, `run`, `step` and `goto`, adopt's restart among them, while the marker exists:
with its writer alive the way forward is to wait for the cancel to end, and with its writer gone it is `lyx batten cancel <slug>` to finish the interrupted cancel.
The cancel verb itself runs outside `armAt`'s seed gate, which would refuse a slug whose seed is gone, and reads the marker on its own: a live writer refuses a second cancel with the same wait, and a gone writer's marker is taken over, the verb taking its lock exclusive, waiting at most the run-lock bound for a reader's shared probe to let go.
A second gate, the pre-drive check batten installs, re-reads the cancel marker and the seed.
With the marker present it refuses with the ways forward above, and with the seed gone it refuses as a run with no seed, naming `lyx batten run <slug>` for a fresh start; either way it writes nothing.
It has two call sites.
The first is batten's pre-run hooks, `battenPreRun` and `battenPreStepUnmarked` in `internal/battencli/arm.go`, which run after `armAt` and before shed's `Run` or `Step` takes the run lock, and for an absent status clear the run's scratch markers (`battenshed.ClearRunMarkers`) and seed `Worktree-Create` running through `state.UpdateJSON`, `battenPreRun` without touching the run lock and `battenPreStepUnmarked` after only probing and releasing it.
The check runs inside that seeding closure, under the status lock, and the marker clear moves into the closure behind it, so a refused seeding writes no status and clears no marker.
The second is batten's shed `Run` and `Step`, once they hold `shedrun.RunLock` and before they read or write status, for a resume whose status was already present.
A batten that passed `armAt` before the marker was written is then a registered process, which step 5's `End` reaches, a holder of batten's locks, which D4 step 2 reaches, or an unregistered one still in the pre-run hook, such as `lyx batten step`, which registers no process:
a seeding closure that runs before step 7 takes the status lock finds the marker, written at step 4, one that runs after finds the seed step 7 removed, and the marker stands until step 9 removes it last, so no status file is written between the records removal and the marker removal;
a hook that found a status file goes on to shed, whose busy refusal meets the run lock the verb holds until step 9 and whose `Run` or `Step` check meets the marker or the missing seed after.
A batten that seeded its status before step 4 had it removed at step 7, and its own `Run` or `Step` meets the run lock or the second call site.
Steps 5 to 9 run under batten's prime-side driver lock and run lock, so no batten resume can drive the run until the records and the claim are gone.
A run that survives the end stops the verb after step 5 with nothing removed or cleared, and the marker left.
Rationale: every step re-reads its own state, and the marker outlives every step but the last, so a cancel interrupted anywhere finishes on a re-run, notice included.

### D4. Ending the run: owned processes, batten's locks, strands, loop, session

On Windows, step 4 of D3, once the marker is written, lists the slug's live registered processes with `runproc.List` and refuses at once when there are any, naming each pid and role with its `taskkill /PID <pid> /T /F` line, since `runproc.End` there returns with nothing signalled (`internal/runproc/end.go:110-112`).
A batten that registered before the marker is so listed and refused, and one that had not yet reached `armSeed` meets the marker.
`runproc.List` there keeps a record with an empty start time while its pid lives (`internal/runproc/doc.go`), so the verb drops, as gone, every listed record whose pid's process was created after the record's registration time, read through `internal/proc`'s Windows creation-time read, and prints no `taskkill` line for it.
Bound: a pid reused before the record registered cannot happen, since the registrant was alive at registration; a clock moved back between the registration and the reuse can still make a reused pid read live, and its refusal then names a foreign process.
With none listed, the end below runs as on Linux, and a batten lock holder no record names meets step 2's held-lock refusal.
After step 2's lock takes below, in the phase or where the verb runs steps 1 and 2 alone, and before step 3's strand stop and any removal, the cancel on Windows calls `runproc.List` for the slug again, applies the same creation-time filter, and refuses with each pid, role and `taskkill` line, then `lyx batten cancel <slug>` again, when any record remains, leaving the marker as D3 says for a run that survives the end.
On Linux the survivors error `runproc.End` returns does this work.

The split follows the pair's task worktree, as D2's does.
With the task worktree present the end is a new `pairteardown` phase beside `EndSession`, which runs, in order:

1. While holding the prime records side's combined write lock (`.weft/weft.write.lock`, which every committing `Fabric.Commit` takes, per `internal/fabricengine/doc.go`), and the pair's own, `runproc.End` for the slug, sparing the caller: batten, the loom driver, the step loop and the pair's pushes.
   The write locks are waited for at most `pairteardown`'s run-lock bound, and a spent bound refuses naming the busy lock and the cancel again as the way forward.
   They are released as soon as `End` returns, so no ended process died inside a records commit.
   While they are held, other callers that try-acquire them, such as the records push retry, the coalesced push and the origin reconcile moves, read them busy and retry by their own rules.
2. The pair's driver-lock probe, as `EndSession`'s owned-process end makes it, then the caller's after-end hook, in which batten takes its prime-side driver lock and run lock (`shedrun.DriverLock` and `shedrun.RunLock` of the prime), each waited for at most the run-lock bound.
   A held lock in either is a batten started meanwhile or another taker of the same lock, such as `lyx batten cleanup`, which try-acquires the run lock of each slug it scans (`internal/battencli/cleanup.go:187`): the hook reports it busy, and the phase runs step 1 once more, under the same write locks, then this step again.
   A lock still held then refuses naming the held lock's path, with a batten of the slug and `lyx batten cleanup` among its possible holders, and the cancel again as the way forward.
   Any other hook error stops the phase.
3. Shuttle's stop verb for every live strand of the pair's session, the driver strand of an `llm` child and every arbiter and producer strand among them.
4. The pair's step loop kill and loop-lock take, as `EndSession`'s loop end does.
5. The wait for the pair's own run lock, the one `EndSession`'s run-lock wait waits for, bounded by the run-lock bound, now after the end.
6. The pair's reed session end.
7. Without `--remove`, the loop's pid file mark is withdrawn and the loop lock released, so the kept pair holds no teardown mark.

With the task worktree gone but a reed session or another part of the pair left, the phase runs steps 1, 2, 3 and 6: step 1 takes the pair's write lock only while its weft worktree stands, step 2 makes no pair driver-lock probe, and step 6 ends the reed session by name.
Steps 4, 5 and 7 are skipped, as `EndSession` skips the loop seizure and the run-lock wait for a gone task worktree (`internal/pairteardown/teardown.go:206-263`).
Only a slug with no reed session and no part of a pair left has the verb run steps 1 and 2 alone, with the prime's write lock and no pair driver-lock probe.
In every case, once the phase or those two steps return, the verb reads batten's in-flight tackler marker, `shedrun.TacklerMarker` under the run's `ScratchDir`, and stops through shuttle the strand it names (`StrandGUID`) in the prime's session.
The read follows the end and the lock take, so a tackler the driver had recorded before the end is read, and none is spawned after it; step 7 of D3 empties the scratch only after.
The driver's tackler call writes the attempt record and the in-flight marker only after `Shuttle.Start` returns (`internal/sheddriver/tackler.go`, the `run.StrandGUID` call and `recordStartedTackler`), so a driver ended inside that window leaves a live tackler strand no marker or attempt record names; see Open risks.

The phase skips the quiet wait, the removal refusal probe and the pre-end run-lock wait a skipped teardown runs, since a cancel ends the holder instead of waiting for it.
A process registered under the slug still alive after `End`, read from `End`'s survivors error on Linux and from the re-list above on Windows, or the pair's driver lock still held, refuses with the survivors' pids and roles, or the held lock's path, and the way forward `lyx tender status` and `ps` on Linux, or the lock path and Task Manager on Windows, as `pairteardown` names them; batten's prime-side driver lock or run lock still held refuses as step 2 words it.
Rationale: no Go driver, batten or `llm` driver strand is left alive when the session ends, so no drive-report writer starts, no `Run <slug> stopped` notice goes, and nothing revives a strand; that closes the 2026-10-21 incident's loop by order.
Bound: a step killed mid-write may leave the pair's working files half-written; a kept pair keeps them for inspection, and `--remove` snapshots them (D5).

### D5. `--remove` removes the pair and its task branches through pairteardown, forced with a snapshot

After the end, `RemovePair` runs with Force, Snapshot and Remote set, so the ordinary origin steps run.
It runs only while some part of the pair is left: the task or weft worktree, the warp or weft branch locally or on origin, or the launcher entry.
An error wrapping `ErrPairNotFound` (`internal/fabricengine/remove.go:153`, passed through by `RemovePair`) reads as done, so a `--remove` re-run after step 6, or `--remove` on a run that never had a pair, goes on to step 7.
Today `RemovePair` takes the snapshot only for a local removal (`internal/pairteardown/teardown.go:347`), and a forced `Remove` discards uncommitted changes outright (`internal/fabricengine/remove.go:126-128`).
So `pairteardown` changes: with Snapshot in origin mode, the pair's uncommitted paths are kept under the local snapshot tag before `Remove` runs, and a refused snapshot removes nothing.
The removal then pushes the weft archive tag and removes the weft branch on origin, as every origin removal does.

`Remove` keeps an unlanded task branch, locally through the destructive gate and on origin through the landed check (`internal/fabricengine/remove.go:148` and `:683`), and a fresh Add then refuses both: an existing local warp branch as `ErrBranchExists`, and an origin branch not reachable from the HEAD it forks from (`internal/fabricengine/remoteleftover.go:157-161`).
So the cancel's removal also keeps the task branch's local tip and its observed origin tip, each under its own `archive/<slug>/<tip>` tag pushed to the warp origin unless one tip contains the other, when the containing tip's tag alone holds both, then deletes the task branch locally and on origin, leased to the observed tip, under the cancel's own authority, whether or not its work is landed.
A failed tag push removes no branch and is a refusal naming the cancel with `--remove` again.
This archive-and-delete runs whenever a task branch of the slug is left, locally or on origin, also when `RemovePair` ran not at all or read `ErrPairNotFound`;
a cancel that finds only such a branch left, as after `lyx fabric remove` of a kept pair, which keeps the unlanded branch (`internal/fabricengine/remove.go:148-152`), runs only the archive-and-delete, and its later steps find nothing to do.

It runs in a mode that leaves the board claim to its caller, so the claim is cleared only at step 8, under batten's locks.
A refusal `RemovePair` raises after the session end leaves the run ended with its records and claim intact; it names its own way forward, then the cancel with `--remove` again.
Without `--remove` the pair, its branches and its child records stay, with no session, no process and no teardown mark.
Batten's Worktree-Create adopts a present, complete task worktree without calling fabric's add (`taskWorktreeComplete`, `internal/battencli/wire.go:822-829`), so a later start would drive the kept pair with its old child records.
With the task worktree gone and the weft side left, a later add trips on or adopts that side: a left weft worktree meets the add's bare `weft worktree directory already exists` refusal (`internal/fabricengine/add.go`, the `weftTarget` check), which no precondition mapping reads, and a left weft branch is adopted, so the new run inherits the old run's records branch.
So batten's pre-run, on a start (no status file), refuses when any local part of the pair that the add would trip on or adopt is left: the task worktree, the weft worktree (`fabricengine.RecordsWorktreePath`) or a local weft branch, read locally as the removal's left-part read reads them, with no network call.
It names `lyx batten cancel <slug> --remove`, and runs beside D10's dirty check and before the seed, the board pull and the claim.
Bound: a pair made by hand with `lyx fabric add` before its first `lyx batten run` is refused the same way.
With only a task branch left, as after `lyx fabric remove`, the start reaches fabric's add, which refuses the left task branch with `*ErrBranchExists`, or an origin task branch with an `*ErrRemoteLeftover` naming it;
Worktree-Create maps both to D11's precondition prefix with the same way forward, so the driver hands back and no tackler is spawned, and that cancel removes the branch and the new run's records and claim.
So the way to rerun is `lyx batten cancel <slug> --remove` first; the envelope and the notice of a cancel that kept the pair say so.
Rationale: the board entry asks for the pair as an option; keeping it by default keeps the work for inspection.
Bound: `--remove` discards nothing uncommitted without a local snapshot tag, and deletes no task branch tip, local or origin, without an archive tag on the warp origin holding it; it never removes a pair whose run here has work that reached the parent (D2).
An ended entry's pair with no batten run record here is removed even when its work landed elsewhere, since nothing here is left to finish and every tip is archived.

### D6. Records and scratch removal reuses batten cleanup's per-slug removal

The removal of the records run directory, the scratch emptying beside lock files and the records commit are the step `lyx batten cleanup` runs for a done slug, called with the cancel's own decision instead of cleanup's done check.
`cleanupSlug` today takes batten's run lock by try-acquire and its status lock by blocking acquire on fresh `gofrs/flock` handles (`internal/battencli/cleanup.go:187-206`, `internal/lock/lock.go:27-38`), which would conflict with the verb's own held locks in the same process.
So the split separates both the lock-taking and the done judgement from the removal: the removal takes no lock of its own and runs under its caller's held locks, cleanup's run lock and status lock, or the cancel's driver, run and status locks (D3 step 7).
Cleanup's own judgement and lock-taking stay unchanged, and it still skips non-done runs.
Rationale: one removal step, so the two verbs cannot drift on what a run's records are.
Bound: it removes only the named slug's run directory and scratch contents; lock files, the cancel marker, other slugs and the pair's own records are untouched.

### D7. The board claim is cleared, and only the status

A run-form status (`boardengine.IsRunStatus`) is cleared through `boardengine` by the verb itself, in both modes.
The write saves Bolt locally, and the board's detached sync pushes it (`internal/boardengine/board.go:53`), so the verb's envelope reports the local write and not the push.
A done status, a non-run status and an absent entry are left alone, so the write is idempotent and a re-run makes it again when an earlier one failed.
A failed board push surfaces through that sync's failed-push record and creel notice (PATTERN-push-failure-visible).
After the clear the entry reads ready again; keeping a cancelled task from being started, by a hand start or by the planned automatic starts of `umbrella-daemon` and `batten-ready-overlap`, is the caller's own step: a re-scope, a dependency or a status.
Rationale: the claim is the run's board lock, so clearing it lets the entry take edits again; body, scope and owner stay the caller's.

### D8. The slug's own pause entries are lifted after the end

Each live slug-scope pause entry naming the slug is lifted by identity through `tenderengine`'s `Lift`, after the end and the removal.
With no sessions left the lift sends no resume line; a kept pair gets the push its pause deferred, and a removed pair none.
Rationale: a stale slug pause would hold a later run of the slug at its InnerRun row.
Bound: hub-scope entries and entries of other slugs are never lifted.

### D9. One cancel notice, and the records push

After the claim clear, the verb spawns the records push as cleanup does and posts one line to the orch through creel: the slug, the state and row step 3 read, and whether the pair was removed or kept, with D5's rerun step for a kept pair.
An end-only cancel (D2) posts its line too, naming the cancel once origin answers.
On a re-run whose records were already removed, the line says the state is no longer readable.
A failed notice only warns; the verb's JSON envelope carries the same report: what was ended, stopped, removed, cleared and lifted, and any snapshot and archive tag.
The notice goes even when the orch itself ran the verb.

### D10. Every auto-seeding batten verb refuses a dirty source before it writes anything

On a start (no status file) and on a resume whose status is still at Worktree-Create with no pair, the pre-run of every auto-seeding verb (`battenAutoSeedVerbs`: `run` and `step`) reads the source worktree's tracked uncommitted paths and refuses when there are any.
The source worktree is the one fabric's pair add forks from, which is the prime today.
The probe is the one the add uses: `internal/fabricengine/dirtiness.go` reads tracked paths only (`--untracked-files=no`), and `trackedDirtyPaths` returns them verbatim.
The refusal lists the paths and the way forward: commit or revert them in the source worktree, then the same verb again.
It runs before the seed, the board pull and the claim, and in the caller of any path that opens a window, before the window opens, so the orch reads the refusal from its own call.
A resume past Worktree-Create runs no such check.
Rationale: the operator decision of 2026-10-20; batten owns the precondition, so the orch need not check it.
Bound: untracked files do not refuse, matching the add.

### D11. Worktree-Create halts on a dirty source or a left task branch with a precondition reason, never a tackler

Fabric's pair add returns a typed dirty-source error carrying the tracked dirty paths instead of a bare message.
Worktree-Create maps it to a Halt Stuck whose reason starts with a new shed constant, the precondition prefix, and names the paths and D10's way forward.
It maps fabric's existing-task-branch refusals, `*ErrBranchExists` and an `*ErrRemoteLeftover` naming the task branch, to the same prefix with the way forward `lyx batten cancel <slug> --remove` (D5).
The Go driver's rule table gains a row beside the bounce-budget-exhausted row: a blocked stop whose reason, or status error, starts with the precondition prefix hands back, ahead of every hand-over, arbiter and tackler row.
Rationale: the source can turn dirty between D10's check and the add, a task branch a cancel kept is left after its pair is removed by hand, and today a blocked stop no other row matches goes to the tackler (`internal/sheddriver/doc.go`), as on 2026-10-20; neither is a state a tackler can settle.
Bound: the row only hands back and never resumes; only a producer that sets the prefix reaches it, and Worktree-Create sets it only for the typed dirty-source error and the two typed task-branch refusals.

### Added after Discussion (arbiter, 2026-10-22): A start re-reads the cancel marker after its board pull

Decision: On a start (no status file), armAt reads the cancel marker again after refuseStartOwnedElsewhere's Bolt pull and before claimPrimeOwner writes the claim, and once more in armSeed right before WriteSeed, refusing as D3's first marker read does. The first read before the pull stays.

Rationale: D3 reads the marker once, before the pull. A start that passed that read and whose network pull spans the cancel's steps 4 to 8 then writes the claim after step 8 cleared it and the seed after step 7 removed it, and the seeding closure's refusal leaves both behind. Re-reading after the pull and before each write leaves only a local write's span, which ends before the cancel's step 7, which comes after the end and the strand stops.

## Constraints

- PATTERN-no-denied-recovery: the dirty-source way forward says "commit or revert" and names no `git stash`.
- PATTERN-refusal-way-forward binds the new shed constant, the driver row and Worktree-Create's reason through `internal/shed*`; batten's own refusals name a way forward too, as its existing ones do.
- PATTERN-shed-run-directory: the run directory, scratch, lock, tackler-marker and cancel-marker paths come from `shedrun`.
- PATTERN-sole-parsers: the precondition prefix is read through the same stop and status reading the budget-exhausted row uses.
- The `--window` window is created with `remain-on-exit off` (`internal/reedengine/doc.go`), so it closes when the ended batten exits and needs no step of its own.

## Auto-mode assumptions

- D1: the name `cancel`, with no `abort` or `abandon` alias.
- D2: no confirmation flag for a live run; an unreachable origin ends the run with a Warn and leaves the rest for a re-run.
- D5: the pair kept by default, and `--remove` deletes the unlanded task branch after archiving its tip.
- D3: the cancel marker's liveness read from a lock its writer holds, and a second marker gate in the pre-run hooks' seeding closures and behind the run lock.
- D4: a Windows record whose pid's process was created after its registration read as gone.
- D5: a start refused on a left task worktree, weft worktree or local weft branch, naming the cancel with `--remove`.
- D9: the notice posted when the orch is the caller.

## Open risks

- A raw git child of an ended process may leave an `index.lock` in a records repository if it ran outside the combined write lock; the next commit there then fails and names the lock.
- Processes an older binary started are not registered; the driver-lock and run-lock re-check catches the driver, and other such processes stay for `lyx tender reap`.
- A tackler strand whose driver was ended between `Shuttle.Start` and the marker write is named by no marker, and by no attempt record, which gets its strand only then; the cancel does not stop it.
  Its prime-side name, `<shortname>:tackler` or its numbered form (`internal/agentname`), carries no slug, since the prime's reed geometry leaves `NameSlug` empty (`internal/hubgeom`), so a list of the prime session's strands cannot tell it from another run's tackler, and stopping every tackler there would end other runs' work.
  With its driver gone no deadline ends it, since shuttle classifies the deadline in the waiter's `Wait`; it stays, and can still act on the prime, until its pane is closed by hand.
  Bound: the window is the two file writes after `Shuttle.Start` returns.

## Acceptance criteria

- A blocked run with no pair, as on 2026-10-20: the cancel reads the owner from the pulled Bolt with no reconcile, stops its tackler strand, removes its records run directory and scratch contents, commits and pushes the records side, clears the board status, and the runs overview no longer lists it.
- A live run with a Go child mid-step and an agent turn running: batten, the driver, the loop and the pair's session end; no drive report is written, no stop notice is posted, one cancel notice is posted; the pair stays with no session and no loop mark; the claim is cleared and the records are removed.
- A live run with an `llm` child driver: the driver strand is stopped before the loop end, and no drive report or stop notice follows.
- `--remove` on a live run with uncommitted pair paths: the pair is removed with a local snapshot tag holding them, the task branch is gone locally and on origin with archive tags on the warp origin holding its local and origin tips, and the claim ends cleared although batten's record read `running`.
- A paused, an awaiting and a failed run each end the same way.
- A `lyx batten run <slug>` or `lyx batten step <slug>` started during a cancel refuses on the cancel marker before the board pull, writing no seed and no claim, and a batten already past that check is ended or refused as a survivor before anything is removed, or refused by the pre-drive check (D3) with nothing written.
- A `lyx batten step <slug>` that passed the marker check before the marker was written and is still in batten's pre-run hook, unregistered and holding no lock, refuses in the hook's seeding closure or at the run lock, writing no status, no seed and no claim; one whose hook runs after the marker's removal meets the missing seed in the closure before any status write.
- Each D2 refusal is reached with nothing ended, removed or cleared, and its message names its way forward.
- An unreachable origin, with or without `--remove`, ends the run with a Warn, removes and clears nothing, and leaves the marker; the cancel once origin answers finishes it.
- A run whose entry left the board is cancelled, with no claim to clear.
- On Windows a live registered process refuses at once with its pid and `taskkill` line; a process registered after that list and still live at the re-list after the lock takes refuses with its pid, role and `taskkill` line; in both cases nothing is removed or cleared.
- A `lyx hibernate request` made during a cancel neither holds the cancel's network calls nor waits on its status lock past step 7.
- A cancel interrupted after any step completes on a re-run, the claim clear, the pause lift and the notice included; a `--remove` cancel interrupted after step 6 reads the pair's absence as done and finishes, as does `--remove` on a run that never had a pair; after a full cancel with `--remove`, a re-run refuses as nothing to cancel.
- After a cancel with `--remove`, `lyx batten run <slug>` starts fresh; after one without, it does so once `lyx batten cancel <slug> --remove` has run.
- A `lyx batten run <slug>` after a cancel that kept the pair refuses in the pre-run on the present task worktree, naming `lyx batten cancel <slug> --remove`, with no seed and no claim.
- A cancel without `--remove` over a slug whose task worktree was already gone with its weft worktree or weft branch left, followed by `lyx batten run <slug>`: the run refuses in the pre-run naming `lyx batten cancel <slug> --remove`, with no seed and no claim.
- With the kept pair's task worktree and weft side gone and its task branch left, `lyx batten run <slug>` halts Worktree-Create with the precondition reason naming the same cancel, and the driver hands back with no tackler spawn; that cancel then removes the branch and the new run's records and claim.
- A kept pair whose entry the caller then marked done is removed by `lyx batten cancel <slug> --remove`, with no ended refusal.
- After a cancel that kept the pair and a `lyx fabric remove <slug>`, a plain `lyx batten cancel <slug>` refuses as nothing to cancel naming `lyx batten cancel <slug> --remove`, which archives and deletes the left task branch, locally and on origin, and the rerun then starts fresh.
- On Windows a registered record whose pid's process was created after the record's registration reads gone, with no `taskkill` line; a cancel marker whose writer died reads as an interrupted cancel whatever process now holds its pid.
- The slug's own pause entry is lifted and a hub entry is left.
- `lyx batten run` and `lyx batten step` on a dirty source, `run` plain and in a window, refuse naming the paths: no seed, no claim, no window, no tackler; a resume at Worktree-Create with no pair refuses the same way; untracked files alone do not refuse.
- A source dirtied after the start check halts Worktree-Create with the precondition reason, and the driver hands back with no tackler spawn.
- `lyx batten cleanup` still skips non-done runs.

## PATTERN changes

- `PATTERN-pair-teardown` — changes: Every teardown of a pair goes through `internal/pairteardown`, which ends the pair's owned processes and then its reed session before any worktree is removed; its driver checks are skipped for a pair that is paused, ended, owned by another machine, or done locally with batten no longer running, never its run-lock wait while the task worktree is present, which then runs before the owned processes are ended; a batten cancel's phase instead ends the owned processes and strands first and waits for the run lock after, and its removal leaves the board claim to the cancel, which clears it.

## Notes for the plan writer

- Cleanup's per-slug step is `cleanupSlug`; split its lock-taking and its done judgement from its removal rather than copying it, so the removal runs under whichever caller holds the locks (D6).
- The precondition prefix belongs beside `shedengine.ReasonBounceBudgetExhausted`, and its rule row beside `budgetExhausted` in the driver rules.
- The batten tackler's in-flight marker, `shedrun.TacklerMarker` under the run's `ScratchDir`, carries the strand to stop; it is read at D3 step 5, after the end and the lock take, and before step 7 empties the scratch.
  It covers a tackler the driver had recorded, not one started in the window before `recordStartedTackler` (Open risks).
