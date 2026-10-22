---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/04-board-owner-writes.md:55
    summary: TestSetOwners names no input for the diverged pull refusal and has no row for the dirty pull refusal the Intent prescribes
    fix: In card 4's `TestSetOwners` paragraph, replace "a diverged pull refusing naming its detail" with two rows that each name their input. Diverged - a local commit of a non-board file such as `notes/c.md` made in the board repo while a second clone pushes a different `notes/c.md` to origin, so the pull's replay conflicts and the setter refuses with `SkipDetail` and writes nothing. Dirty - a tracked file such as `notes/x.md` left edited and uncommitted in the board repo while a second clone pushes a change to it, so the pull's fast-forward is blocked and the setter refuses with the dirty detail, which names `lyx board sync`, and writes nothing. Do not use a `board.json` conflict as the diverged input, because the board replay merges per entry and per field and never conflicts there. Alternatively drop "or `dirty`" from the Intent's refusal sentence. No target or verify line changes, so every plan check admits the fix.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/04-board-owner-writes.md:47
    summary: The push-side transport failure mapped to ErrOwnerOriginUnreachable has no TestSetOwners row, because the unreachable-URL row stops earlier at the pull
    fix: Add one `TestSetOwners` row for the push-side transport branch - `BeforePush` runs `git remote set-url origin http://127.0.0.1:1/board.git` in the board repo after the pull's fetch has succeeded, so the first push fails on transport (git's connection error is not a retryable refusal, so no real backoff runs and the branch needs no sleep seam), and `SetOwners` returns `ErrOwnerOriginUnreachable` wrapped with the push's detail, with the owner commit kept local and not reported in `Recorded`. Reword the existing unreachable row to say it reaches the pull's `fetch_failed` skip, so the two rows are not read as covering one branch. No target or verify line changes, so every plan check admits the fix.
---

### [MEDIUM] Diverged and dirty pull refusals have no reaching input

**Location:** `_lyx/plan/04-board-owner-writes.md:46` and `:55`
**Issue:** The Intent prescribes two refusals from the pull skip: "a `diverged` or `dirty` skip refuses naming its detail".
The test paragraph carries one row for them, "a diverged pull refusing naming its detail", with no input, and none for dirty.
`pullFirst` in `internal/fabricengine/bolt.go` reaches `BoltSkipDiverged` only through a replay conflict, and `replayBolt` merges `board.json` per entry and per field with origin-wins replay fields, so a tester who conflicts the board file reaches no refusal.
`internal/fabricengine/boltpullfirst_integration_test.go` shows the inputs that do reach them: a local and a remote commit on the same non-board path for diverged, and a dirty tracked file on a path upstream changed for dirty.
The row has to name such an input, and dirty needs a row or its mention dropped.

### [MEDIUM] Push-side transport failure has no reaching row

**Location:** `_lyx/plan/04-board-owner-writes.md:47` and `:55`
**Issue:** The Intent maps "a transport failure" of the push to `ErrOwnerOriginUnreachable` wrapped.
The only unreachable-origin row uses `http://127.0.0.1:1/board.git` as the origin URL, which makes `PullThenCommitWritten`'s fetch fail first, so it reaches the `fetch_failed` skip and `SetOwners` never pushes.
No row reaches the `PushRecorded` transport branch (`classifyFailure` in `internal/fabricengine/pushclassified.go`, `PushFailureTransport`).
The `BeforePush` seam, which runs between the commit and the first push, is the route to it: it can repoint the board repo's origin after the pull succeeded.

## Settled by this round

The focus directive's three questions, with the verdicts:

- **Deferred push retried once after `Sleep`, then retried-and-lands or still-ahead refusing naming `lyx board sync`:** reached.
  `BeforePush` pushes another entry's commit from a second clone, so the lock-free first push in `pushClassified` is rejected, and holds the board write lock with `lock.TryAcquireWriteLock`.
  `retryBoltPush` try-acquires the same lock through its own `flock` handle, so it fails and reports a deferral, `PushRecorded` returns nil, and the local branch is still ahead of its stale tracking ref.
  The `Sleep` stub releasing the lock on its first call lets the second `PushRecorded` replay and land; keeping it makes the second call defer again and `SetOwners` refuse.
  Neither case waits a real second, and `lyx board sync` exists as a command.
- **Competing write replayed so origin's owner stands and the slug lands in `Lost`:** reached.
  `BeforePush` runs after `PullThenCommitWritten` released its lock and pulled, so its second-clone push makes the first push a rejection, `retryBoltPush` takes the free lock and replays under card 2's origin-wins `owner_machine` rule, and the post-push `ReadOwners` re-read reports origin's owner.
- **`BeforePush` and `Sleep`:** not a defect.
  The card states both are nil in production (`Sleep` reading `time.Sleep`), that `BeforePush` is the test's only route to the branch because the pull sits immediately before the commit, and that `Sleep` is injected, which is what `PATTERN-test-speed` asks.
  Neither skips a guard or lets anything through, so the attack-surface question has nothing to bound.
- **Cards 9 and 24 resting on `Lost`:** the path is exercised by a real setter in card 4, not only against a fake `Set`.
  The competing-write row covers the post-push re-read, and the "owned elsewhere skipped" row covers the pull-time `Lost` that card 9's claim refusal also names.
