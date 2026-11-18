```yaml
lens: code-correctness
outcome: reported
```

I reviewed under the code-correctness lens: each function and branch the diff changes, run against concrete inputs. I found two problems in `internal/fabricengine` and nothing wrong in the tender or test changes.

**Cases run**

- **`coalescePush`** (coalesce.go:47): lock acquire fails → the deferred `leaveGate` runs and the error is returned. Lock taken → admitted until the lock is released. Correct.
- **`Bolt.pullFirst` / `Bolt.Pull`** (bolt.go:269-303): run with a covering pause and an uncovered caller. `Pull` takes the board write lock at bolt.go:273, then `pullFirst` calls `Admit` at bolt.go:301 and waits at the gate while that lock is held. **Not correct** (finding 1).
- **`coalesceFlagged`** (pendingflags.go:151-163):
  - First acquire returns `err` or `!ok` → leaves the gate and returns `(false, err)` or `(false, nil)`, as before the diff. Correct.
  - A step fails → the lock is released in `runHeld`, then the gate is left, and `(true, err)` is returned. Correct.
  - Loop goes back to acquire after a step pushed, in a deferring child with a covering pause → `Admit` fires the deferral, which puts back every owed flag, including sides this call already pushed. The state is harmless, but the log text and the doc are wrong (finding 2).
- **`pushDeferral`** (pendingflags.go:63-75):
  - `owed` holds only the records side → only that flag is marked and `sides=[records]` is logged. Correct.
  - `markPending` fails → a warning is logged and the exit still runs. Same as before the diff. Correct.
- **`coalescePushSides` step** (coalesce.go:177-186):
  - Records side not landed → the code side is skipped. Correct.
  - Records landed with `beforeWarp==""` → the code side is skipped. Correct.
  - The merged `if` matches the old nesting once the removed `pushed` writes are taken out. Correct.
- **`lockWeftForReplay`** (pull.go:498-518): write lock fails → gate left. Push lock fails → write lock released, then gate left. Success → the release frees push, write, then gate. Correct. The callers at pull.go:413-421 hold no other lock.
- **`retryBoltPush`** (pushclassified.go:811): try-lock not held → returns deferred and the deferred `leaveGate` runs. Correct. The caller holds no board write lock, since this function try-acquires it itself.
- **`takeoverBranch.land`** (takeoverwrite.go:173): correct within the function. It runs under `TakeOverRun`'s teardown locks (takeoverwrite.go:107-118); I leave that to the handler, who is already weighing the teardown lock.
- **`coveredSessions`** (pause.go:236-255):
  - `Sessions` returns an error wrapping `reedengine.ErrNoSession` (reed's `Status` returns `requireSessionLocked`'s `noSessionError` unwrapped, lifecycle.go:1294, and `shuttlePair.Live` returns it raw, deps.go:476) → the pair goes in `noSession` and the loop goes on. Correct.
  - Any other error → `(nil, nil, wrapped error)`. Correct.
  - A pair with no session readings → `Sessions` returns `(nil, nil)` and lists nothing. Correct.
- **`Pause` with a skipped pair** (pause.go:175-179): notes are appended to `Skipped` and the grace loop breaks at once (`covered` is empty, nothing busy). `goProducers` may also list the same pair as a go-producer, since `Running()` reads only the status file. Consistent.
- **`Revive`** (resume.go:479-483): same as `coveredSessions`. Correct.
- **`Status` / `sessionStatuses`** (resume.go:625-716):
  - Prime has no session → it appears in `skipped` under the prime's slug. Correct.
  - Another error → returns `(nil, nil, err)` and `Status` returns it. Same as before the diff. Correct.
  - Success → `Skipped` is never nil, so JSON prints `[]`. Correct.
- **`noReedSessionNote`** (report.go:136): the reason is the fixed prefix plus `err.Error()`. Correct.
- **`TestCoalescePushBothAt_HeldByAPauseWithoutThePushLock`**: `pushLockFree` opens a new flock handle (doc.go:1801), so a lock held in the same process reads as busy. A wait at the git push, after the lock is taken, would fail the in-Sleep check. Correct.
- **`TestSync_DeferrableExitsUnderACoveringPause`**, plain covered row: `Defer` with no deferral stored → the gate waits, `Sleep` lifts the pause → one wait, then the board error. Correct.

**Findings**

1. **BLOCKING, design.** `internal/fabricengine/bolt.go:301` (`Bolt.pullFirst`, reached from `Bolt.Pull` at bolt.go:269-278).
   - The gate is passed after `Pull` has already taken the board write lock (bolt.go:273, `acquireWriteLock` at bolt.go:78). A pause therefore holds a board pull at `Admit` while it holds a lock every other board writer waits on.
   - Callers that reach it include battencli/catchup.go:161, beamcli/dobbydeps.go:516, hubpull.go:101, adoptcli/caller.go:49 and machinecli/cli.go:129.
   - This contradicts the goal's rule that "a pause hold never waits while holding a shared lock". It also contradicts the summary and doc.go:1245, which say the gate is passed "before it takes its first lock".
   - Evidence: bolt.go:283 ("The caller holds the board write lock"), bolt.go:273-277 and bolt.go:301.
   - Fix: admit in `Bolt.Pull` after its `SkipGit` return and before `acquireWriteLock`, and stay admitted through `pullFirst`.
2. **LOW, consistency.** `internal/fabricengine/pendingflags.go:61-62` (`pushDeferral` doc), and the matching doc.go:1305 text.
   - Both say the deferral fires "before the push lock is taken, when no side of the call has been pushed". `coalesceFlagged` calls `Admit` on every loop pass (pendingflags.go:152), so on a pass after a step has pushed, a covering pause fires the deferral in a detached child.
   - The child then puts back flags for sides it already pushed and logs "exiting with the commits left local". The state is harmless (the next push finds nothing to push), but the claim and the log line are wrong on that path.

Tool calls:
1. `bolt.go:278-330` and reed `Status`: the manifest held `pullFirst` only from the diff, not its caller-holds-write-lock contract, and not how reed's `Status` returns the no-session error.
2. `bolt.go:255-279` and `Bolt.Pull` callers: to confirm `Pull` takes the board write lock before `pullFirst`, and to find who reaches it.
3. `pull.go:395-440` and `Bolt.acquireWriteLock`: to check whether `lockWeftForReplay`'s caller holds another lock, and the board write lock path.
