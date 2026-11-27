---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/finalize.go#gateUnlocked
    summary: The lock-free merge, conflict session, gate, fixer and fix review now run before Finalize's owner checks, so a run another machine owns or origin ended or stopped commits and pushes work unchecked, since the pre-push check skips Finalize because it judges its owners itself (PATTERN-owner-machine).
    fix: "In gateUnlocked, right after the held read and before parentMergeTip, run the owner judgment the locked pass runs: call fz.ownerChecks(ctx, held) and return its ending when non-nil, so a run owned elsewhere is handed over and a run whose origin ended, stopped or is unreachable halts before any merge into the task worktree. Keep the owner checks in landUnderLock as they are, since ownership can change while the lock-free gate runs. Add rows to TestFinalize_GatesWithNoLockAndLocksToLand, or a new table in finalize_owner_test.go at bound 2, where Reconcile reports the run owned elsewhere and where it reports the run ended, asserting the halt and that the resolver recorded no merge."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/parentalign.go#parentMergeTip
    summary: With the push skipped or no upstream, every lock-free pass takes the parent lock for its tip read with front=false, so a run returning after a lost race or a fixer queues behind every waiting landing before it queues again at the front, against Call's "each later pass queues at the front".
    fix: "Add a front bool parameter to parentMergeTip and pass it to guard.ensure in place of the literal false. ParentAlign.Call passes false. gateUnlocked takes a front parameter and Call passes its loop's front value to it. Update parentMergeTip's doc comment to say the read takes the lock at the front of the queue when front is set."
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: contracts/recipes/darn-recipe.yaml:305
    summary: "The Finalize comments in the darn, sampler, tapestry and hardener recipes still say Finalize takes the parent lock first and merges, gates and runs the fixer under it, and the darn and sampler headers say it merges the parent under the lock it takes first."
    fix: "Reword the Finalize row comment in contracts/recipes/darn-recipe.yaml:305, sampler-recipe.yaml:261, tapestry-recipe.yaml:528 and hardener-recipe.yaml:176 to: Finalize merges the parent into the task branch and runs the landing gate and the merge fixer with no parent lock held, then takes the lock, lands only while the parent is still the tip it gated, fast-forwards the parent and releases the lock on every way out. Reword the headers at darn-recipe.yaml:16 and sampler-recipe.yaml:18 to: Finalize merges the parent itself, before it takes the parent lock to land. Reword tapestry-recipe.yaml:52 to: Finalize is the only row that lands on the parent, under the parent lock."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/landingshed/finalize_unlocked_test.go#TestFinalize_GatesWithNoLockAndLocksToLand
    summary: No row drives the lock-free pass against a parent behind its upstream, so the order of syncParent's fast-forward and the gatedTip compare in step 1b4b is unpinned; every row stubs ReadParentTips with ParentEqual.
    fix: "Add a row to TestFinalize_GatesWithNoLockAndLocksToLand whose ReadParentTips stub answers Relation ParentBehind with Upstream u1 and Local p0, and whose merger's FastForwardPair moves the head HeadSHA returns to u1. Assert the run lands, that the resolver's tips are [\"u1\", \"\"], and that FastForwardPair ran once before the landing merge."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/landingshed/finalize_unlocked_test.go#TestFinalize_GatesWithNoLockAndLocksToLand
    summary: No test drives a failing gate, a fixer or a fix review in the lock-free pass, so the new freeLock rule that a process holding no lock frees nothing, the merge record's MergedTip the fixer reads, and the landing that follows are uncovered; the existing fixer tests all run at bound 0.
    fix: "Add a row, or a test beside it in finalize_unlocked_test.go built on newFinalizeGateFixture with FinalizeLostRaces 2, whose first verify fails and whose fixer turn fixes the tree. Assert that the lock was not held at the fixer's spawn, that the merge record read at the spawn has MergedTip p1, that the run lands, and that the lock does not outlive the call."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/finalize.go:442
    summary: Held is read in gateUnlocked before the owner checks' reconcile, which can fast-forward the task branch; a pass whose held turns true under the lock still compares the head with gatedTip, and a mismatch counts a lost race that was no race.
    fix: "In landUnderLock, guard step 1b4b with the held value read after the owner checks: change `if fz.gatedTip != \"\"` to `if fz.gatedTip != \"\" && !held`, so a parent that already holds the task's work goes on to the locked merge and gate without counting a lost race."
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/finalize.go:454
    summary: configChangeNotice now always runs after the lock-free merge, so ReadConfigChanges' merge-base is the merged parent tip and the notice's base values are the parent's; step 1c's comment and the ConfigChanges doc in deps.go still claim the diff is taken against the parent as the task forked from it.
    fix: "Read the notice once in Call before the loop, after the status commit, as notice := fz.configChangeNotice(), and pass it into landUnderLock in place of the call at step 1c. Reword the step 1c comment to say the notice was read before the first merge of the parent, and reword deps.go:262 to: The finalize step calls it once, before its first merge-in, so the diff is taken against the fork point."
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/landingshed/finalize_unlocked_test.go#TestFinalize_GatesWithNoLockAndLocksToLand
    summary: The new error and halt paths have no row, a failed held read in gateUnlocked, an error or lock reason from parentMergeTip in Finalize, and a failed HeadSHA read at step 1b4b, nor does any row assert that the locked pass after a lost race queues at the front.
    fix: "Add an error table to finalize_unlocked_test.go at bound 2 with rows where ParentHoldsTaskWork errors (want an owner-check halt opening with OwnerHaltPrefix), where ReadParentTips(true) errors (want a returned error naming Finalize), and where the merger's HeadSHA errors after the first call (want a returned error naming the parent branch). Add a row to the main table, with a moved parent, that records the Front flag of each lock acquire through a waiter seam or the lock store's queue and asserts the second locked acquire is at the front."
  - id: F9
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/landingshed/finalize_unlocked_test.go:415
    summary: No test shows that the locked pass after a lock-free gate skips the verify of the already verified tree; the fake gate verifies every time, so the rows expect a verify under the lock and would pass if the locked merge changed the tree and re-ran the whole gate under the lock.
    fix: "Make the fake gate in this test answer StatusSkipped for a tree it already passed, keyed on the task head the fixture reports, or count only verifies that ran, and change wantLockAtVerify to [false] for the unmoved row and [false, false] for the moved row, so a gate run under the lock fails the test."
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/landingshed/finalize.go:61
    summary: "The Finalize type doc still says it merges the parent branch into the task worktree under the parent lock, and freeLock's doc says the run then takes the lock again and runs its locked pass again, though Call runs the lock-free pass first while races remain."
    fix: "Reword finalize.go:61 to: Finalize is the ShedProducer that merges the parent branch into the task worktree and gates it, then lands the task branch on the parent pair's own worktree under the parent lock. Reword the third sentence of freeLock's doc at finalize.go:542 to: The run then runs its passes again, the lock-free pass first while lost races remain, and takes the lock again at the front of the queue."
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/loomcli/prepush.go:89
    summary: skipsCheck's doc exempts a run at Finalize because Finalize judges origin and its owners itself, which the diff leaves untrue for the lock-free pass until F1's fix lands.
    fix: "Once F1's fix runs the owner checks before the lock-free merge, leave the sentence as it is; if F1 is fixed another way, reword prepush.go:89 to name where Finalize judges its owners before any commit it makes."
---

The decision record holds the board entry alone, with no added entries, and its `## Idea` is the measuring stick.
Every Idea point is built: the lock-free merge and gate in `gateUnlocked`, the tip compare at step 1b4b, the release and re-run on a moved parent, and the `landing.finalize_lost_races` fallback.
The change is in scope, and the summary's claims hold against the code bar the front-of-queue claim F2 qualifies.
The `two-way` merge-danger call holds: the new key is additive, and an older binary only warns on it.

### [BLOCKING] Lock-free pass runs before the owner checks

**Location:** `internal/landingshed/finalize.go#gateUnlocked`

**Issue:** `Call` now runs `gateUnlocked` before `landUnderLock`, and `landUnderLock` is where the owner checks, the pull-request decision, the beam land check and the parent-pair checks run (`finalize.go:309-317` against `finalize.go:399-438`).
Before the change each of those halted before Finalize touched the task worktree.
Now a run whose origin ended or stopped, or that another machine owns, merges the parent, spawns the conflict session, runs the 4-to-12-minute gate and spawns the fixer and fix review first.
The pre-push check skips a run at Finalize "which judges origin and its owners itself" (`internal/loomcli/prepush.go:89,108`), so those commits reach origin with no owner verdict.
`PATTERN-owner-machine` allows pushing a run only on its owner machine.
The parent-pair and pull-request halts that now also come after the lock-free pass only cost a wasted gate, so the fix targets the owner judgment alone.

### [MEDIUM] Tip read queues at the back on later passes

**Location:** `internal/landingshed/parentalign.go#parentMergeTip`

**Issue:** With the push skipped or no upstream, `parentMergeTip` takes the parent lock with `front` false to read the local tip.
A run coming back after a lost race or a fixer then waits behind every queued landing, and then waits again at the front for its locked pass.
That contradicts `finalize.go:304` and the summary's "After a lost race the next locked pass queues at the front", and it can add a whole queue wait per pass on a busy parent.

### [MEDIUM] Recipe comments still describe the old lock order

**Location:** `contracts/recipes/darn-recipe.yaml:305`

**Issue:** The Finalize comments at `darn-recipe.yaml:16,305`, `sampler-recipe.yaml:18,261`, `tapestry-recipe.yaml:52,528` and `hardener-recipe.yaml:176` say Finalize takes the parent lock first and merges, gates and runs the fixer under it.
The diff updated `docs/overview.md` and the orch contract, but not these.

### [MEDIUM] Behind-upstream path untested

**Location:** `internal/landingshed/finalize_unlocked_test.go#TestFinalize_GatesWithNoLockAndLocksToLand`

**Issue:** Every row answers `ParentEqual`, so `syncParent`'s fast-forward never runs before the compare at `finalize.go:442-451`.
The common production case, a local parent behind the upstream tip the lock-free pass merged, has no row.

### [MEDIUM] Fixer in the lock-free pass untested

**Location:** `internal/landingshed/finalize_unlocked_test.go#TestFinalize_GatesWithNoLockAndLocksToLand`

**Issue:** Every row passes its gate.
`TestFinalize_GateAndFixer` and the other fixer tests use a fixture that leaves `FinalizeLostRaces` at 0 (`finalize_gate_test.go:46-83`), so they drive only the locked path.
The new `freeLock` rule (`finalize.go:545-552`), the `MergedTip` the fixer reads from the lock-free merge record (`finalize.go:1141`, `1354-1362`) and the landing after a lock-free fix are all uncovered.

### [LOW] Held turning true counts a false lost race

**Location:** `internal/landingshed/finalize.go:442`

**Issue:** `gateUnlocked` reads held before the owner checks' reconcile, which can fast-forward the task branch (`finalize.go:406-423`).
When held turns true under the lock, `syncParent` is skipped and the head is still compared with `gatedTip`.
A mismatch frees the lock and spends one of the lost-race bound for no race.

### [LOW] Config-change notice base moved

**Location:** `internal/landingshed/finalize.go:454`

**Issue:** `ReadConfigChanges` takes `merge-base(parentTip, tip)` (`internal/fabricengine/configchanges.go:76-80`).
The lock-free merge has already merged the parent by step 1c, so the reported base is the parent's value.
Before the change, a recipe with no Parent-Align row saw the true fork point on its first pass.
Step 1c's comment and `deps.go:261-262` still claim the older behaviour.

### [LOW] New error paths and front queueing have no row

**Location:** `internal/landingshed/finalize_unlocked_test.go#TestFinalize_GatesWithNoLockAndLocksToLand`

**Issue:** `finalize_unlocked_test.go:430` stubs `ParentHoldsTaskWork` to succeed, and no row makes `ReadParentTips` or `HeadSHA` fail.
No row asserts that the locked pass after a lost race queues at the front.

### [LOW] Verify skip under the lock is not shown

**Location:** `internal/landingshed/finalize_unlocked_test.go:415`

**Issue:** The fake gate verifies on every call, so the rows expect a verify with the lock held (`wantLockAtVerify` `[false, true]`).
The skip that keeps the lock to seconds (`finalize.go:194-195`) is the change's purpose, and a regression that re-ran the gate under the lock would pass.

### [LOW] Finalize type and freeLock docs stale

**Location:** `internal/landingshed/finalize.go:61`

**Issue:** The type doc still says Finalize merges the parent under the parent lock.
`freeLock`'s doc says the run goes straight back to its locked pass, though `Call` runs the lock-free pass first while races remain (`finalize.go:310-316`).

### [LOW] Pre-push exemption rationale

**Location:** `internal/loomcli/prepush.go:89`

**Issue:** The exemption's stated reason, that Finalize judges its owners itself, is untrue for the lock-free pass as the diff leaves it.
F1's fix restores it.

## Rejected

- Merge record overwritten by the locked up-to-date merge (focus 3): a locked pass after a fixer did the same before the change, and the fixer reads only `MergedTip`, written by the lock-free pass it follows.
- `parentGuard` passed by value into `parentMergeTip`: the type holds no mutable state (`parentguard.go:16-22`).
- A hub config without `finalize_lost_races`: the template default of 2 fills it, as for the other landing keys.
- Parent-pair and pull-request halts that now follow a lock-free gate: they cost a wasted gate but commit nothing a halt should have prevented beyond what F1 covers.
- Unrequested scope: the scope lens traced every change to an Idea point or to an edit one forces.
