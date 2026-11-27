```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each Decision's mechanism end to end on one concrete case, passing on the data each step actually produces.

**Decisions walked**
- D1 — `markTaskDone` makes a board write at Finalize, and that commit goes to the board target: **broken** (finding 3).
- D2 — an owed pusher push on a flaky origin, with the interval in force at 60 s: **broken** (finding 7).
- D3 — a pair target is refused while its branch is still ahead, so the ledger empties: **broken** (finding 6).
- D4 — a hub with nothing owed, then one owed target, read by the `#()` job: **met**.
- D5 (a) — a local landing with origin unreachable: **broken** (finding 1).
- D5 (b) — a second local landing on a parent whose first landing push is still owed: **broken** (finding 2).
- D5 (c) — a resume after the merge with the landing push owed, read through the local parent: **met**.
- D6 (a) — a `git-transport` stop at Finalize's owner check, with the driver holding the run lock through its wait: **broken** (finding 4).
- D6 (b) — no halt note while the driver waits: **broken** (finding 5).
- D7 — a hand commit H on GitHub, then a rejected pair push, fetch, `resetUnnamedTip`: H is reachable from no local branch or reflog, so the push is refused. A `git pull` of H makes the push a fast-forward, and a rewind's dropped commits sit in the reflog, so the reset is allowed: **met**.
- D8 — the catch-up's push hits a held push lock and gets `ErrPushLockBusy`: **met for the pair**, with a gap (finding 10).
- D9 — a merge-in deletes `proj/` (and `proj/.gitignore` with `bin/`) while `proj/bin/o.dll` stands, including a resume: **broken** (findings 8 and 9).
- D10 — an agent's push fails, and the directive says it needs no action: **met**.
- D11 — not walkable; it names tests only.
- D12 — #218: `runread` already drops the code-tip refusal when the local pair holds the tip: **met**.
- D13 — Worktree-Teardown with origin unreachable sets the push-now mark, leaves the pair standing and ends done; the dobby removes the pair once the tips are on origin and no parent push is owed: **met**.

**Findings**

1. BLOCKING · design · `decision-record.md` D5/D6, Goal and the D5 acceptance criterion
   With origin unreachable, a local landing cannot return done: it waits before it ever merges.
   - `gateUnlocked` runs `ownerChecks(held=false)` before any merge (`internal/landingshed/finalize.go:359`), and `landUnderLock` runs it again (`finalize.go:439`).
   - The reconcile reports `Unreachable` (`internal/landingshed/finalize_owner.go:53`), which D6 turns into a `git-transport` wait.
   - Past that, `syncParent` calls `ReadParentTips(true)`, which fetches first and fails on that fetch (`finalize.go:612`, `internal/fabricengine/parenttips.go:25-28`).
   - So the criterion "returns done while origin is unreachable" and the Goal's "never fails on a transient error" for a landing that needs no GitHub do not hold as designed.

2. BLOCKING · design · `decision-record.md` D5
   A second local landing on a parent whose first landing push is still owed halts as Stuck.
   - The local parent is then ahead of its upstream, and `syncParent` returns Stuck: "the local parent branch … is ahead of its upstream; way forward: push or reconcile" (`internal/landingshed/finalize.go:619-620`).
   - D5 handles only a lagging parent and a diverged one, so other runs cannot keep landing locally during an outage.

3. BLOCKING · consistency · `decision-record.md` D1 vs D5
   The board `done` commit is pushed by the board target, not by the landing target, so it can reach origin before the parent push.
   - `markTaskDone` is a board write (`internal/landingshed/finalize.go:806-813`), and every board write spawns the board sync (`internal/boardengine/board.go:217`).
   - D1 hands that sync to the board's branch target, which pushes every owed board commit.
   - D5's promise that the landing target pushes `done` only after the parent push lands therefore has no mechanism: nothing holds back that board commit.

4. BLOCKING · design · `decision-record.md` D6
   "While it waits the driver holds its run lock" breaks the re-step itself.
   - After the wait, `recheck` probes the run lock with a try-acquire (`internal/sheddriver/drive.go:854`, `internal/battencli/wire.go:193-204`).
   - A lock the driver itself holds reads as held, so the driver exits as `ExitRunningElsewhere` (`drive.go:858-862`).
   - The re-stepped `loom step` also takes the same run lock (`internal/loomcli/wiring.go:400`).
   - The driver's own lock is `shedrun.DriverLock` (`internal/battencli/drive.go:72`), so the record names the wrong lock or a mechanism that cannot work.

5. MEDIUM · design · `decision-record.md` D6
   "The driver writes no halt note" misses where the note is written.
   - The halt note is written in the step process by `loomAfterStep`, including for a transient failed halt (`internal/loomcli/halt.go:2`, `halt.go:37-38`).
   - That happens before the driver reads the envelope or writes a wait record, so every waited stop still leaves a note.

6. MEDIUM · design · `decision-record.md` D3/D4
   A refused target that leaves the ledger produces a false all-clear while its commits are still only local.
   - Once the refused target leaves the ledger, the ledger is empty and the dobby posts "every branch is pushed".
   - `sync:` reads `0m` while the refused branch is still ahead.
   - The dobby tick writes an entry only when it starts a pusher, and a standing refused-push record blocks that start (`internal/runpusher/start.go:122-125`).
   - This misses D4's stated purpose: to show before a machine switch whether work is still only local.

7. MEDIUM · design · `decision-record.md` D2
   D2 does not say that the pusher's existing retry loop is replaced.
   - The pusher's push still runs `PushUntilLanded`, which retries from 1 s doubling to 60 s (`internal/runpusher/pushuntillanded.go:22-39`, `internal/runpusher/run.go:144`).
   - The push also runs `retryTransportPush`'s backoff inside `pushClassified` (`internal/fabricengine/pushclassified.go:192-206`).
   - So "at most once per interval in force" is false unless both loops yield to the ledger's interval.

8. MEDIUM · design · `decision-record.md` D9
   On a resume, the before-merge clean check halts first, so the removal never runs.
   - The leftovers still stand on resume, and `mergeParent` runs `clean "before the merge-in"` before any merge (`internal/landingshed/finalize.go:1344`).
   - The removal is placed only "after its merge-in, and again on every resume past it", so it must also run ahead of that check.

9. MEDIUM · design · `decision-record.md` D9
   The "merge commit's first parent" is not the pre-merge task tip when the merge-in fast-forwards.
   - `MergeCommit` is read as `TaskHead()` after any merge that is not up to date (`internal/landingshed/finalize.go:1369-1373`).
   - In a fast-forward, that head is the parent's own commit, and its first parent is not the task tip.
   - The deletion may also lie several commits back.
   - The merge record already carries `TaskHead` as it was before the merge-in (`finalize.go:1382`), which is the right base.

10. LOW · consistency · `decision-record.md` D8
    The catch-up pushes the prime's records before the pair (`internal/battencli/catchup.go:265-275`).
    - An owed failure there belongs to the prime's target, not "that run's pusher".
    - D8 does not say which pusher starts in that case.

**Tool calls**
- `runLockHeld` in `internal/battencli/wire.go`, the driver lock in `internal/battencli/drive.go`, and `ReadParentTips` in `internal/fabricengine/parenttips.go`: the manifest lacked the run-lock probe and the fetch inside the parent-tip read.
- `PushOwed` in `internal/dobbyengine/catchup.go` and `FinalizePastMerge` in `internal/battenshed/innerrun_owner.go`: needed to walk the catch-up and the resume past the merge.
- `mergeParent`, the merge flags, and `RunCommitsAhead` in `internal/fabricengine/pusherreads.go`: needed to walk D9's merge-in and D1's ahead rule.
- `finalize.go:1365-1385` and the `"merge"` calls in `internal/mergeresolve` and `internal/fabricengine`: needed to see how the merge commit and the merge record are read for D9.
