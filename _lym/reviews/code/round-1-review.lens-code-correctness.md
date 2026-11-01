```yaml
lens: code-correctness
outcome: reported
```

**Task:** code-correctness lens. For each function or branch the diff adds or changes, I ran one concrete case against the code in context.

## Per-function list

- **`front.run`, `startOrJoin`, `lockKey`:** Lock free, fresh start, cap 60 s: correct. Lock held with under 30 s of cap left: answers 75 `starting`, correct. Context cancelled: returns 1, then `exitAfterSignal` maps it to 128 plus the signal number, correct.
- **`joinable`:**
  - Expired lease, delivered mark, tree or dirty-set mismatch, or empty tree: no join, correct.
  - Exit file with the abandoned mark: no join, so a fresh run starts and no loop forms, correct.
  - Live pid with a different start reading: no join, correct.
  - A process file read mid-write gives a decode failure and no join, so a live runner is superseded. This is finding C1.
- **`writeNewRun`, `startRun`, `awaitClaim`, `claimByFront`, `failStart`:**
  - The lease is written before the current pointer and before launch, correct.
  - Launch error: the front claims the run itself, expires the lease and exits 1 with stderr, correct.
  - Runner dead before claim: the front's claim succeeds, then `failStart`, correct.
  - Runner claims after the front's create: it gets `ErrExist` and refuses, correct.
  - Front's create fails with `ErrExist`: treated as claimed in time, correct.
  - The first check comes only after a one-second wait, so every fresh start loses at least 1 s. That is allowed by the wake rule and not a defect.
- **`replay`, `copyRunFile`:** Stdout is copied before stderr, and a recorded 75 returns 1 with the added line, correct. A failed copy is only logged and the delivered mark is written anyway. This is finding C2.
- **`chooseCap`, `longWait`, `ranEveryListedTier`, `callSpan`:**
  - No gate key, or zero reruns: short cap, correct.
  - A read error keeps the short cap with a Warn, correct.
  - Fewer than three full calls: false, correct.
  - Even count: mean of the two middle values, correct.
  - A call with a failed build counts through its blocked tiers, correct.
  - Abandoned, carried and empty-scope calls drop out, correct.
  - `WaitStart` zero falls back to `Start`, correct.
- **`runDetached`:** Missing lease, bad request, unknown verb and already-claimed run each refuse before any exit file is written, correct. Stdout and stderr are closed (deferred in `runRequest`) before the exit file is written, correct. For `card-verify`, `request.Args[0]` is read with no length check, so a hand-made request with empty args panics. The runner is internal, so this is a NIT (C5).
- **`runRequest`, `cancelOnPassedLease`:**
  - Signal gives `{128+n, abandoned}`, correct.
  - Passed lease gives `{1, abandoned}` and the abandoned line, correct. The ticker runs independently of the call, so it also covers the slot wait.
  - Natural finish: the watcher is joined, correct.
  - A lease that passes in the gap between the call returning and `cancelCall()` marks a finished run abandoned. This is finding C4.
  - A lease read error counts as a passed lease. This is finding C3.
- **`detachRun.writeLease`, `expireLease`:** `writeLease` only moves the lease later and `expireLease` is the one earlier move, both under the state lock, correct.
- **`endStrayLine`:** The line group is killed only when the runner is not live, correct. It also runs for a normal finished run, where `proc.KillTree` has to check the recorded start time. I did not verify that in `proc`.
- **`frontDecision`, `cappedCall`, `testRequest.words`:**
  - `--wait`, no strand, an inherited slot and a zero cap each run in-process, correct.
  - A resolver fallback still caps, with a Warn, correct.
  - `--tags` is refused by `flagShapeRefusal`, so `words()` leaving it out is safe.
  - The word order is fixed, so equal calls hash to the same key.
- **`registerRunner`:** The runner is registered under the origin slug in a hub, and nothing outside a hub. A front killed between launch and register leaves the runner unregistered, which is a negligible window.
- **`acquireSlot`, `haltExitCode`, `writeRecord`:** Slot waits are uncapped and 75 is no longer produced in-process, correct. `Abandoned` is set from `e.ctx.Err()` and the call id is set on every record, matching the decision record.
- **`runReport`:** Abandoned records are counted apart and skipped for every other total, correct.
- **`cardverify.Runner.runLine`, `acquire`, `execute`:** The strand is stripped from the environment, and the context reaches both the slot acquire and the line, correct. A gate-call line gets the running phase and a slotted line gets the slot phase first, correct. `RunTied` is used only when `Tied` is set, correct.
- **`verifyrun.runTied`, `startAndWaitTied`, `tieToParent`:** The start and the wait run on one locked OS thread, so the parent-death signal follows the thread that started the child. `started` runs after the start and before the wait, and the thread is unlocked before the goroutine exits, so it is not killed. Correct on Linux, with no tie elsewhere.
- **`runCard`, `writeLine`:** The job is held and released, the phase is written and the line tree recorded, correct. An unreadable plan gives the refusal in stderr and exit 1, correct.
- **`runCardVerify` front path:** The key uses `card.ID()` and the worktree path, so `4` and `4-<slug>` join one run, correct.
- **`battencli.commandEnv`:** The strand entry is removed from a clone of the environment, correct. A grep of `verifyrun.Run` and `RunTied` callers shows `verifytree`, `cardverify` and `battencli` only, and all strip the strand.
- **`shuttleengines.KeepWarm`, `baseRole`:**
  - `implementer-2` gives `implementer`, correct.
  - A hyphenated role without a numeric suffix stays whole, correct.
  - An unparsable or empty strand, or a config failure, gives the fallback (`DefaultAwaitCap` for both caps, `MaxReruns` 0), correct.
- **`ValidateKeepWarm`, `checkKeepWarmCap`, `KeepWarmCaps`:** A cap under 60 s, at or above the lifetime, over 55 m, or an empty or unparsable value each error with `lym config claude`, correct.
- **`shuttlecli.awaitCap`:** An explicit `--cap` wins, no worktree gives the default, and no strand gives the fallback cap, correct.
- **`gateslot.renderGateCall`, `AgentGateCall`:** The call has no `--wait`, correct.

## Findings

- **C1: LOW, design.** `detachrun.go:157-162` and `detachfront.go:656`.
  - **Problem:** `claim` creates `process.json` exclusively, then writes its content. The front's claim wait and a second front both treat the file's existence as the claim, and the second front takes the lock right after the first releases it. A read in the gap sees an empty file, fails to decode, and `joinable` returns false. A live, just-claimed runner is then superseded.
  - **Fix:** Write the process record to a temporary file and link or rename it with an exclusive create, or have `readJoinState` retry a decode failure on a present process file.
- **C2: LOW, design.** `detachfront.go:503-514`, `replay`.
  - **Problem:** The delivered mark is written even when copying stdout or stderr failed. The result is lost, the next call starts a fresh run, and it costs one extra run, never a verdict. The focus asks that the mark be set only after a successful copy.
  - **Fix:** Have `copyRunFile` return an error, and write the mark only when both copies succeeded.
- **C3: LOW, design.** `detachrun.go:144-148` and `cancelOnPassedLease`.
  - **Problem:** `leaseHolds` returns false on a read error, and the runner's once-a-second loop uses it. A transient lock or read error on the lease file cancels a healthy run as abandoned.
  - **Fix:** In the loop, cancel only when the lease is read and past, or absent. Keep the strict form only for the startup refusal.
- **C4: LOW, design.** `detachrun.go:322-331`.
  - **Problem:** `leasePassed` is read after the call returned. A lease that passes between the call's return and `cancelCall()` marks a run that finished with a real verdict as abandoned, and the verdict is never replayed.
  - **Fix:** Record whether the call returned before the watcher fired, for example by setting `leasePassed` only when it cancelled a context that was still live. Mark the run abandoned only when the call context was cancelled before the call returned.
- **C5: LOW, design.** `detachfront.go:35` and `answerRunGoing`/`replayedBusyLine`, plus `detachrun.go:70` and `detachrun.go:279-285`.
  - **Problem:** The message and replay text say "gate test" and "the run is still going" for a `card-verify` run. An abandoned exit also answers "still going" with the dead run's stale phase file. Separately, `readRequest` does not check that `card-verify` has an argument.
  - **Fix:** Use the verb in the text, and say the run was cut short and the same command starts a new one. Add an `Args` length check in `readRequest`.
- **C6: LOW, design.** `detachfront.go:300` (no cleanup).
  - **Problem:** A grep shows `detachDirName` is used only to build the key directory. Nothing prunes finished run directories, which hold the full stdout, stderr and runner log of every run. The gate directory grows without bound, while `WriteRunRecord` prunes its own files after 14 days.
  - **Fix:** When a front starts a run, delete run directories under the key directory that are older than 14 days or expired and delivered, skipping the current run and any with a live runner.

## Tool calls

1. One Bash grep, three queries: whether a `--tags` flag exists (it is refused, so no issue), whether `detachDirName` has a pruning user (none), and every `verifyrun.Run`/`RunTied` caller (all strip the strand). The reading manifest held none of these.
