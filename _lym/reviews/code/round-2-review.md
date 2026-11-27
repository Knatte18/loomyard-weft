---
verdict: APPROVED
prior:
  - id: F1
    status: resolved
    answer: gateUnlocked runs fz.ownerChecks(ctx, held) after the held read and before readConfigBeforeMerge and parentMergeTip; landUnderLock keeps its own owner checks; TestFinalize_LockFreePassStops rows for a run owned elsewhere and a run origin ended assert the halt and no merge of the parent.
  - id: F2
    status: resolved
    answer: parentMergeTip takes front and passes it to guard.ensure; ParentAlign.Call passes false; Call passes its loop's front to gateUnlocked; TestFinalize_QueuesAtTheFrontAfterALostRace pins the queue place of the push-skipped tip read.
  - id: F3
    status: resolved
    answer: The Finalize row comments and the darn, sampler and tapestry headers now describe the lock-free merge, gate and fixer before the lock, and the fallback under the lock; the added fallback sentence matches Call.
  - id: F4
    status: resolved
    answer: The behind row drives ParentBehind with a fast-forward that moves the head to u1, and expects tips u1 then empty and fast-forward before merge.
  - id: F5
    status: resolved
    answer: TestFinalize_FixerInTheLockFreePass runs at bound 2 with a failing first verify and asserts no lock record and merged tip p1 at the fixer's spawn, the landing and no lock left after.
  - id: F6
    status: resolved
    answer: Step 1b4b now compares only when gatedTip is set and held is false, with held read again after the owner checks.
  - id: F7
    status: resolved
    answer: The fixer's alternative holds; the changes and base come from the read before the call's first merge, the parent values from a read under the lock after the sync, and the deps.go and step 1c comments say so.
  - id: F8
    status: resolved
    answer: TestFinalize_LockFreePassStops covers a failed held read, a paused lock wait for the tip read, a failed tip read and a failed head read under the lock; TestFinalize_QueuesAtTheFrontAfterALostRace asserts the front flag of each wait.
  - id: F9
    status: resolved
    answer: The fake gate answers skipped for a tree it already passed and wantLockAtVerify counts only fresh verifies, while gateAskedUnderLock still requires the locked pass to call the gate.
  - id: F10
    status: resolved
    answer: The Finalize type doc and freeLock's doc now describe the lock-free pass and the re-run of both passes.
  - id: F11
    status: resolved
    answer: With the owner checks before the lock-free merge, skipsCheck's sentence that Finalize judges origin and its owners itself holds unchanged.
---

This incremental round found no new BLOCKING defect on a changed hunk, and every prior finding is resolved.
`lym gate test ./internal/landingshed` passed on this tree: 864 passed, 0 failed, 3 skipped.

## Focus verdicts

- **Owner and origin before the lock-free pass.**
  `gateUnlocked` reads whether the parent holds the work, then runs `ownerChecks`, then the config read, the tip read, the merge, the gate and any fixer or fix review.
  `landUnderLock` still runs the pull-request step, the owner checks, `landCheck`, `syncParent` and `recheckLock`.
  `skipsCheck`'s doc in `internal/loomcli/prepush.go` holds: a commit the lock-free merge or fixer makes follows Finalize's own owner judgment.
  Each lock-free halt has a row in `TestFinalize_LockFreePassStops`.
- **Queue place.**
  Every acquire after a lost race or a locked-pass fixer queues at the front, the push-skipped and no-upstream tip read included, since both reach `guard.ensure(ctx, producer, front)`.
  `ParentAlign.Call` passes `false`, as before.
- **Held read and the gatedTip compare.**
  The compare uses the held value read after the owner checks and the head read after `syncParent`.
  A parent that came to hold the work skips the compare and goes to the locked merge and gate, so no lost race is counted.
  A task-branch fast-forward by the reconcile changes the tree, not the parent head, so it counts no race, and the locked gate then verifies the changed tree.
  Not a defect.
- **Config-change notice.**
  The changes and their base come from `ReadConfigChanges` before the call's first merge, so the base is the fork point or a parent tip an earlier row merged.
  The parent values come from a second read under the lock, and a file the second read no longer reports is left out.
  The `ConfigChanges` doc in `deps.go` and the step 1c comment state this.
- **Tests.**
  The behind parent, the lock-free fixer, the new halts and errors, the front-of-queue re-acquire and the skipped verify under the lock each have a test that fails on a regression.
- **Recipe comments and docs.**
  The recipe comments, the type doc, `freeLock`'s doc and `docs/overview.md` match the order the code runs.

## Rejected

An incremental round raises only BLOCKING findings, so these observations below BLOCKING are recorded here and not raised:

- A fixer that runs in the lock-free pass is followed, in the same Call, by an acquire at the Call's first `front` value, `false` on a fresh entry.
  A Finalize resumed from the same attempt record queues at the front through `returningFromFixer`.
  The queue place is harmless, but the two disagree.
- `TestFinalize_FixerInTheLockFreePass` asserts no lock record at the fixer's spawn.
  That also holds for a fixer in the locked pass, since `freeLock` releases the lock before the spawn, so the assertion does not pin which pass ran the fixer.
  `TestFinalize_GatesWithNoLockAndLocksToLand` pins which pass gates.
- The pull-request halts and `landCheck`'s beam-member halt now come after a whole lock-free merge, gate and possible fixer, where they used to come before any merge.
  This costs gate time and possibly fixer attempts, but nothing wrong lands.
- `docs/overview.md`'s list of landing keys near `landing.finalize_fix_review_rounds` does not name `landing.finalize_lost_races`.
  The same paragraph names the key above that list.
- `internal/loomrecipe/testdata/tapestry-rows-before-motif.yaml` still says Finalize takes the parent lock first.
  It is a fixture of an older recipe, so this is not a defect.
