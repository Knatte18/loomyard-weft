```yaml
lens: way-forward
outcome: reported
```

Way-forward lens: each refusal, stop and recovery step the cards add, checked for a next step that works from the state it leaves.

**BLOCKING**

**1. Card 9 / Card 13, D10 push stop: the recorded failed push names a `git merge` across the marker change.**
- Card 9 has `retryWeftPush` return an error wrapping `ErrAnchorChange`, and says `pushClassified` "records it as the push's failure". Card 13 does the same for a standing move record.
- That error is not one of `classifyFailure`'s cases. So `pushClassified` joins it with the first rejection (`errors.Join(pushErr, retryErr)`, `internal/fabricengine/pushclassified.go:134-140`).
- `classifyFailure` then matches `gitrepo.ErrPushRejected` in the join and records `PushFailureNotFastForward` (`pushclassified.go:225-226`).
- That class's way forward is `recordsConflictWayForward` (`internal/fabricengine/failedpush.go:217-221`), which says to run `git reset --keep origin/<branch>` or `git merge origin/<branch>`.
- A hand merge brings in the upstream's marker and content at the new anchor with no move of `.lyx`, junctions or launchers. That leaves exactly the half-rewired pair D10 exists to prevent, and drops the `lyx fabric anchor <slug> --set` way forward the card prescribes.
- No card adds a failure class or way forward for the anchor stop. No planned row asserts the recorded `FailedPush`'s way forward, since `TestPushFailureClasses` only asserts that the push fails with `ErrAnchorChange`.

**MEDIUM**

**2. Cards 9, 10, 11 and 17, D10 stops for a pair with a live run: the named `--set` is refused, and so is the cancel that refusal names.**
- The pull stop (card 9), the drop-local refusal (card 10) and the origin-reconcile `anchor_stopped` way forward (card 11) all name `lyx fabric anchor <slug> --set <anchor>` as the next step.
- Card 17 refuses that `--set` when the worktree's reed state holds a strand, or its run status names an unfinished run, "(wait for it, or `lyx batten cancel <slug>`)".
- A run-scoped origin reconcile, or a pull, of a pair with a run under way is exactly where both hold:
  - Waiting cannot end the run, because its weft pushes and its origin reconcile stop on the same marker change.
  - `lyx batten cancel` refuses "when origin stopped the entry" (`internal/battencli/cancelcmd.go:59-62`), and card 11 makes `BranchAnchorStopped` count in `BranchState.stops`, so the entry reads stopped.
  - Cancel also refuses "when another machine owns the slug", which is the drop-local case of card 10.
- The way forwards loop between `--set` and cancel, and no card names an exit.

**3. Card 12, promote refusal: the named `--set` cannot run for a task with a held run, and its fallback destroys the promote.**
- Card 12 names `lyx fabric anchor <slug> --set <parent anchor>`, then `lyx beam promote <slug>`.
- Promote accepts a task with `entry.RunHeld` (`internal/beamcli/promote.go`, `judgePromoteTask`) and ends that run itself in `promoteSteps`.
- Card 17's `--set` refuses that same task while its run is unfinished or its driver strand is stored, and names `lyx batten cancel <slug>`.
- Cancel "clears the board claim, removes the run's records" (`cancelcmd.go:51-54`). After it, `judgePromoteTask` refuses with "was never started, so there is no pair to keep" (`entry.Owner == "" && !entry.RunHeld`). The pair is then never promoted.

**4. Cards 17 and 19, D7 recovery on a worktree now owned elsewhere: adopt and `--set` point at each other.**
- Card 19's adopt refuses while a move record stands on the named entry's worktree, naming `lyx fabric anchor <worktree> --set <B>`.
- Card 17 runs its owner refusal before `BeginAnchorChange`'s recovery: "the board names another machine as the worktree's effective owner while `B` differs from the marker at the weft's fetched upstream tip", naming `lyx adopt <slug>`.
- Suppose a move here was killed before its commit was pushed, and another machine has since taken the entry. Then `B` differs from the upstream marker. Adopt names `--set`, `--set` names adopt, and neither progresses.
- Card 17 gives the re-run under a standing record no exemption from the owner check.

**5. Card 18, D6 present-pair refusal: no re-run of `beam create` is named, so the pair stays unmarked.**
- `EnsureAnchor` refuses a different anchor naming only `lyx fabric anchor <slug> --set <anchor>`.
- `createSteps` calls it "before the mark", so the refusal leaves the pair without the beam mark, push, workspace regeneration and dobby. The card states no wrapper adding the create's usual rerun clause.
- The `--set` alone completes none of those steps. The way forward should go on to `lyx beam create <slug> --anchor <anchor>` again, which then passes as the equal-anchor case.

**LOW**

**6. Card 17, prime-only refusals: three name no next step.**
- The card states a way forward only for the live hibernate daemon (`lyx hibernate cancel`).
- The live reed session or stored strand anywhere in the hub (`lyx reed down` in each worktree), the live batten run, and the held gate slot (wait for the holder) are listed with none, although the Intent says "each naming its way forward".

**7. Card 19, adopt refusal: no re-run of the adopt is named.**
- The refusal names a change in progress and `lyx fabric anchor <worktree> --set <B>`, but not `lyx adopt …` again afterwards. Card 10's drop-local refusal does name the adopt again, so the two adopt-path refusals differ.

**Per refusal: next step, progresses or not**
- Card 7, empty own marker → `git restore`, or write the marker and commit → progresses.
- Card 8, reconcile pair failures → progress.
- Card 9, pull stop → `--set` integrates → progresses, except for a pair with a live run (finding 2).
- Card 9 / Card 13, push stop → recorded way forward is the conflict merge → does not progress safely (finding 1).
- Card 10, checkout refusal → warp update, `--set`, checkout → progresses.
- Card 10, drop-local refusal → `--set`, then adopt → blocked (finding 2).
- Card 11, origin-reconcile pair stop → blocked for a live run (finding 2). The prime stop progresses.
- Card 12, promote refusal → blocked for a task with a held run (finding 3).
- Card 13, standing move record → wait, or re-run the `--set` → progresses, except under finding 4.
- Card 14, integration conflict → tag, save, `reset --keep`, re-run → progresses.
- Card 15, engine refusals and recovery re-runs → progress.
- Card 16, `EnsureLive` refused while a record stands → progresses.
- Card 17, `--set` refusals → findings 2, 4 and 6.
- Card 18, `EnsureAnchor` different-anchor refusal → finding 5.
- Card 19, adopt refusal → finding 4, plus LOW finding 7.

**Tool calls**
- Read `pushClassified`, `classifyFailure` and `FailedPushWayForward`: my context lacked how an unclassified retry error is recorded and what that record's way forward says.
- Read `lyx batten cancel`'s help: my context lacked what cancel does and when it refuses.
