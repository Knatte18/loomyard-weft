# Card 18 — batten-cancel-core

**Create:**
- `plan:internal/battencli#cancelDeps` -> `type cancelDeps struct`
- `plan:internal/battencli#cancelReport` -> `type cancelReport struct`
- `plan:internal/battencli#cancelRun` -> `func cancelRun(ctx context.Context, slug string, remove bool, deps cancelDeps) (cancelReport, error)`
- `plan:internal/battencli#childReachedParent` -> `func childReachedParent(batten shedengine.Status, battenFound bool, child shedengine.Status, childFound bool, pastMerge bool) bool`
- `plan:internal/battencli#liveOnWindows` -> `func liveOnWindows(records []runproc.Record, createdAt func(pid int) (time.Time, bool)) []runproc.Record`
- `plan:internal/battencli#windowsLiveRefusal` -> `func windowsLiveRefusal(slug string, records []runproc.Record) error`
- `plan:internal/battencli#cancelNotice` -> `func cancelNotice(report cancelReport) string`
- `plan:internal/battencli#TestCancelRun` -> `func TestCancelRun(t *testing.T)`
- `plan:internal/battencli#TestChildReachedParent` -> `func TestChildReachedParent(t *testing.T)`

**Edit:**
- `internal/battencli#stoppedRefusal` -> `func stoppedRefusal(slug, act, reentry string, entry fabricengine.EntryOutcome, branches []fabricengine.BranchOutcome) error`
- `internal/battencli#resumeRefusal`

**Uses:**
- `internal/battencli#heldCancelMarker`
- `internal/battencli#cancelMarkerReading`
- `internal/battencli#cancelMarkerRefusal`
- `internal/pairteardown#CancelRequest`
- `internal/pairteardown#CancelResult`
- `internal/pairteardown#CancelLockWait`
- `internal/fabricengine#PairParts`
- `internal/fabricengine#DiscardResult`

**Intent:**
Write the cancel's whole decision and order, D2 to D9, as `cancelRun` over injected seams in `cancel.go`, so every branch is reached by an untagged test with fakes; card 19 wires the production seams and the verb.

`stoppedRefusal` takes the act it refuses (`resume` or `cancel`) and the re-entry it names, so the cancel's stopped refusal reads `cannot cancel <slug>: …; then run "lyx batten cancel <slug>" again` and never sends the caller to `lyx batten run`; its one caller, `resumeRefusal`, passes `resume` and today's `then re-run lyx batten run <slug>`, so the resume's refusal reads as today.

`cancelDeps` holds one function field per substrate, each documented; nil fields are not allowed (the production builder fills every one, and the test fills every one):
`HubPaused` (a live hub-scope pause entry covers the prime), `ExemptGate` (returns the restore), `TaskWorktreePresent`, `Reconcile` (the judge-only run-scope reconcile), `PullBolt`, `Owner` (this machine and the slug's effective owner on the local Bolt), `ReadStatus` (batten's status and found), `BoardStatus` (the slug's local board entry status and found), `RecordsPresent` (the records run directory exists), `ChildReachedParent` (D2's landed reading), `ReadMarker`, `TakeMarker`, `GOOS`, `LiveRecords`, `CreatedAt`, `PairParts`, `SessionLive`, `EndForCancel`, `TakeBattenLocks` (batten's prime-side driver lock and run lock, each waited for at most `pairteardown.CancelLockWait`, returning the release or the busy path; a busy report holds no lock, so when the run lock is busy it releases the driver lock it already took before returning), `StopTackler` (reads `shedrun.TacklerMarker` under the run's scratch and stops the strand it names; returns its guid or empty), `RemovePair`, `TaskBranchOnOrigin`, `DiscardTaskBranch`, `RemoveRecords` (takes batten's status lock, runs `removeRunRecords`, releases it before returning), `ClearClaim` (clears a run-form status; reports whether it cleared and whether an entry was found), `LiftPauses` (lifts the slug's own live slug-scope entries and returns their scopes), `SpawnPush` and `Notify`.

`childReachedParent` is D2's landed reading over what card 19's seam reads: batten's status, the child's status read from the task worktree, and `battenshed.FinalizePastMerge`'s answer for the child.
It is true for a child Finalize past its parent-side merge, a Finalize Done in the child's history, or batten past Run-Shed with the child done.
With the task worktree gone the child reads not found, so only batten's own reading decides: batten whose current producer is Worktree-Teardown, which runs only after Land, reads reached whatever the child reads, since its work landed and only its teardown is left.

`cancelRun` runs D3's steps, each a no-op when already done:

1. A live hub pause refuses: `a hub pause covers the prime; run "lyx tender resume", then "lyx batten cancel <slug>" again`.
   Then `ExemptGate`, restored on return, so no live pause holds the cancel's network calls while it holds batten's locks (as `lyx fabric remove` does; the push and board sync it spawns keep their own gates).
2. With the task worktree present, `Reconcile`; absent, `PullBolt` then `Owner`, with no prime claim.
   An unreachable origin (the report's `Unreachable`, or a pull error wrapping `boardengine.ErrOwnerOriginUnreachable`) makes the cancel end-only (below), after `Owner` is read from the local Bolt, as D2's offline bound says: owned by another machine there refuses with the owned-elsewhere wording below, with nothing ended.
   Any other reconcile or pull error refuses with the cause, `fix it, then run "lyx batten cancel <slug>" again`.
   Owned elsewhere (the reconcile's `EntryOwnedElsewhere`, or the Bolt owner another machine) refuses with the cancel's own wording: `<slug> is owned by machine <owner>; run "lyx adopt <slug>" on this machine first, or "lyx batten cancel <slug>" on <owner>`.
   A stopped entry refuses through `stoppedRefusal(slug, "cancel", "then run \"lyx batten cancel <slug>\" again", …)`.
   An ended entry whose `How` is `done` while `RecordsPresent` refuses: `<slug> was landed by another machine; run "lyx cleanup --apply" from the prime, whose reconcile tears it down`; an ended entry with no records here, and an entry absent from the board, go on.
3. `ReadStatus` and `ChildReachedParent`; work that reached the parent refuses: `run "lyx batten run <slug>" to finish the landing, then "lyx batten cleanup --apply"`.
   Then the nothing-left reading: no records, no run-form status on the local board entry (`BoardStatus`), no left `PairParts`, no live session, no live record and no marker, and with `remove` no task branch on origin either, refuses `nothing of <slug> is left to cancel; check it with "lyx board list"`, or, without `remove` and with a local task branch left, `run "lyx batten cancel <slug> --remove", which archives and deletes its task branch`.
4. `ReadMarker`: a writer alive refuses with `cancelMarkerRefusal`; otherwise `TakeMarker` (taking a dead writer's marker over).
   On Windows, `liveOnWindows(LiveRecords(), CreatedAt)` drops every record whose pid's process was created after the record's `RegisteredAt` (and keeps a record whose creation time cannot be read); any left refuses with `windowsLiveRefusal`, naming each pid and role with its `taskkill /PID <pid> /T /F` line and the cancel again, and releases the marker, removing it only when this cancel wrote it fresh.
5. `EndForCancel` with `Keep: !remove` and an `AfterEnd` that runs `TakeBattenLocks` and, on Windows, lists again through `liveOnWindows` and refuses with `windowsLiveRefusal` (releasing the locks it took); then `StopTackler`.
   A refusal here leaves the marker, with nothing removed or cleared.
   In end-only mode the cancel stops here: it releases batten's locks, posts its notice naming `lyx batten cancel <slug>` once origin answers, logs a Warn, leaves the marker and returns a report with `EndOnly` set, with or without `remove`.
6. With `remove`, `RemovePair` (Force, Snapshot, Remote, LeaveClaim) runs whatever `PairParts` reads, since only `Remove` reads the pair's branches on origin; an error wrapping `fabricengine.ErrPairNotFound` reads as done, which is how nothing of the pair being left is read, and any other refusal returns naming its own way forward and then `lyx batten cancel <slug> --remove` again.
   Then, while a task branch is left locally or `TaskBranchOnOrigin` reads one, `DiscardTaskBranch`; a failure refuses naming the cancel with `--remove` again.
7. `RemoveRecords`, which holds batten's status lock only for its own call, so a `lyx hibernate request` made meanwhile waits on that lock no longer than step 7.
8. `ClearClaim`.
9. Batten's locks are released, `LiftPauses`, `SpawnPush` when records were committed, `Notify(cancelNotice(report))` (a failure only warns and is recorded on the report), and the marker removed last.

`cancelReport` carries the slug, the state and row step 3 read (empty when no status was readable), and what was ended, stopped, removed, cleared and lifted, the snapshot and archive tags, whether the push was spawned and the notice posted, `EndOnly` and `WayForward` (for a kept pair, `run "lyx batten cancel <slug> --remove" before rerunning it`).
`cancelNotice` renders one line: the slug, the state and row or `state no longer readable`, the pair removed or kept with the rerun step, or, end-only, the cancel to run once origin answers.

`TestCancelRun` is one table over fakes that record their calls; each row asserts the report, the refusal text and the calls made, and every refusal row asserts no end, removal, clear or lift was called:
a blocked run with no pair (pull, owner, tackler stopped, records removed, claim cleared, push, notice; no reconcile);
a live run with the pair present (reconcile judge-only, `EndForCancel` with `Keep`, the kept-pair way forward);
`remove` on a live run (`RemovePair` with Force, Snapshot, Remote and LeaveClaim, `DiscardTaskBranch`, claim cleared although batten read `running`);
paused, awaiting and failed runs; a seed with no status; a claim with no records;
each refusal: hub pause, reached parent, ended-done with records, owned elsewhere present and absent, stopped branch (naming `lyx batten cancel some-slug` and not `lyx batten run`), reconcile error, pull error, Windows live record at the first list, Windows live record appearing only at the re-list in `AfterEnd` (batten's locks released, marker left), nothing left with and without a left task branch, live marker writer, an `EndForCancel` refusal (marker left, no `RemovePair`, `RemoveRecords`, `ClearClaim` or `LiftPauses` call), a refused `RemovePair`, a failed discard;
a Windows refusal at the first list over a fresh marker (released with removal) and over a taken-over marker (released without removal);
an ended entry with no records here and `remove` (removed, no refusal); an entry absent from the board (no claim to clear); `remove` with no local part left (`RemovePair` still called, its `ErrPairNotFound` read as done);
unreachable origin from the reconcile and, separately, from a pull error wrapping `boardengine.ErrOwnerOriginUnreachable`, each with and without `remove` (end-only, marker left, nothing removed or cleared, notice naming the cancel);
unreachable origin from the reconcile and, separately, from the pull error, each with the local Bolt's owner another machine (the owned-elsewhere refusal, no `EndForCancel` call);
`TakeBattenLocks` reporting busy once and then succeeding (the second call made with no lock held from the first);
a re-run after the records were removed (notice says the state is no longer readable); `RemovePair` returning `ErrPairNotFound` read as done; a gone writer's marker taken over; a failing `Notify` (the cancel succeeds, the report records the notice unposted);
a Windows record created after its registration dropped with no `taskkill` line; the gate exemption taken before the first network seam and restored at the end; and `RemoveRecords` returning before `ClearClaim` is called.
`TestChildReachedParent` is a table: a child Finalize past its merge, a child Finalize Done and batten past Run-Shed with the child done each read reached; batten at Worktree-Teardown with the child not found reads reached; batten at Run-Shed with the child running, batten blocked at Worktree-Create with no child, and no status at all read not reached.

**ImpactSummary:** `stoppedRefusal` gains two parameters its one caller, `resumeRefusal`, fills with today's text, so the resume's refusals are unchanged; everything else is new code in `cancel.go`.
