---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lym/plan/08-dobby-worktree-gone.md:57
    summary: The after-tick-1 Sleep row passes with the top-of-iteration worktree-gone check deleted, so no test pins that exit
    fix: >-
      In card 8's `TestRun_WorktreeGoneExits` row "a worktree removed after tick 1 has ended", set the harness `stopAt` to at least 3 so the tick seam's cancel cannot mask a second tick,
      and add three assertions: the harness's recorded ticks hold exactly one entry (`len(h.ticks) == 1`), the worktree path does not exist after `Run` returns, and the existing `StopStrand` count of one.
      Then replace the row's closing clause "so the exit at the top of the iteration, not the one after a tick, ended the loop" with the sentence that a second tick, or a recreated `.lym/beam` stub at the removed path, fails the row.
      Keep the `CheckBinary` assertion only as a check that nothing re-execs; it does not tell the two exits apart.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lym/plan/08-dobby-worktree-gone.md:36
    summary: The after-tick state read has a race branch no named row reaches and a read-error clause that contradicts the loop
    fix: >-
      Rewrite the kept-tidy paragraph of card 8's Intent so the loop, after each tick, first reads the record through `state.ReadJSONNoMkdir`, then tests `worktreeGone` once.
      When `worktreeGone` reports true, the loop discards the read and its error and exits through `exitWorktreeGone` with the earlier `keptTidy`.
      When it reports false and the read failed with an error other than a decode failure, the loop logs it through `d.failures` as a tick error is logged, keeps the earlier `keptTidy` and goes on, and drop "ends the loop as a tick error does", since a tick error never ends the loop.
      When it reports false and the read succeeded, `keptTidy` takes the record's `Tidy`, nil included.
      Add to `TestRun_WorktreeGoneExits` a row where tick 1's hook replaces `state.json` with a directory while `.git` stays present: `Run` neither exits nor changes `keptTidy`, and a later removal stops the earlier `Tidy`'s strand.
      The existing rows that remove the worktree inside a tick then reach the read-fails-and-gone branch, so say in the paragraph that they do.
  - id: F3
    severity: LOW
    class: decision
    location: _lym/plan/11-gate-adopt-key.md:40
    summary: A hub gate file seeded in the same pass from the prime or legacy files gets no adopt until a later binary build
    fix: >-
      In card 11's Intent, replace the sentence "A hub `gate.yaml` that this pass seeds from the prime or from legacy files gains `adopt` at the next reconcile, which finds it present" with a stated choice.
      Either run `m.MigrateHubBase` in `ReconcileHubWideAt`'s absent-file branch over the bytes `sparseSeed` returns for the prime and legacy seeds, and add a row to `TestReconcileHubWideAt_SeedsAdoptFromATunedFinalize` where an absent hub `gate.yaml` seeded from a prime copy tuning `finalize` to `[0, 1]` holds `adopt` with the same value,
      or state in the Intent and in `internal/gateslot/doc.go` that such a file runs the template's `adopt` tiers until the next binary build, since `hubreconcile.Ensure` is gated on the build stamp and the seeding pass leaves it current.
      Both shapes edit only targets the card already lists.
---

### [MEDIUM] The after-tick-1 Sleep row does not pin the top-of-iteration exit

**Location:** `_lym/plan/08-dobby-worktree-gone.md:57`

**Issue:** The focus asked whether some row removes the worktree outside a tick and wakes the loop by one watch event with no probe due, so that deleting the top-of-iteration check fails a test.
The row exists: `Deps.Sleep` on its first call removes the worktree and sends one value on `Watch`.
Its mechanics hold against `internal/dobbyengine/loop.go` and `loop_test.go`.
The first `Sleep` call runs in `daemon.sleep`'s timer goroutine and does not advance the fake clock.
The wake-floor `Sleep` that follows is the second call and advances it by `wakeFloor`, one second.
That is under the harness's ten-second probe, so no probe is due on iteration 2.
The row's assertions do not tell the two exits apart, though.
With the top-of-iteration check deleted, iteration 2 runs a tick: `daemon.tick` calls `UpdateState`, which recreates `.lym/beam`, and the `Tick` seam fires.
After that tick the post-tick check sees `.git` still gone and exits through `exitWorktreeGone` before `CheckBinary`.
`Run` then returns nil, `StopStrand` has run once with the kept `Tidy`'s guid, and `CheckBinary` has run once, after tick 1.
Every assertion the row names holds without the check it claims to pin.
A smaller defect compounds this: the row does not set `stopAt`, and if it were 2 the tick seam's cancel would end the loop on tick 2 before the post-tick check, so the row would fail for an unrelated reason.
The decision record's acceptance criterion "A beam dobby woken by a file event, not a probe tick, after its worktree is removed still exits, and a tidy held in `State.Tidy` is stopped" has this row as its only carrier.
What separates the two exits is observable: the tick count, and a recreated `.lym/beam` stub at the removed path.
The fix adds those assertions.

### [MEDIUM] The after-tick read has an unreached race branch and a contradictory error clause

**Location:** `_lym/plan/08-dobby-worktree-gone.md:36`

**Issue:** The card prescribes that "a read that fails while `worktreeGone` then reports true, as when the removal lands between the marker test and the read" exits through `exitWorktreeGone` with the earlier `keptTidy`.
The card names no input that reaches this branch and no named row does.
The row that removes the worktree during tick 1 has `worktreeGone` true at the loop's first post-tick marker test, so the loop skips the read.
The other rows leave `.git` present for the whole read.
A removal landing between two statements of the loop cannot be injected without a seam the card does not add.
The rubric's reachability bullet flags a guard branch with no named input or test that reaches it.
Reading first and testing the marker once, as the fix orders them, makes the same behavior reachable.
Removing the worktree inside the tick makes the read fail on the missing directory, and the marker test then reports gone.

The same paragraph says "any other read error ends the loop as a tick error does".
`loop.go` handles a tick error by `d.failures.Fail(... "the loop goes on" ...)`, so a tick error never ends the loop.
The sentence contradicts itself, and the card's rows name no input for the branch either.
The focus's third question asks whether the card's reads recreate a removed directory or write under an untouched path.
The answer is no.
`state.ReadJSONNoMkdir` calls no `MkdirAll`, `exitWorktreeGone` reads no record, and every append goes through `Deps.Log`, which creates no directory.
The one write is `lock.AcquireReadLock`'s lock file, and the read runs only while the marker is present, so the lock file sits in a directory that exists.
The fix fixes the error clause and names a row that reaches it: `state.json` replaced by a directory makes `ReadJSONNoMkdir` return `ErrRead` with the marker present.

### [LOW] A seeded hub gate file gets no adopt in the seeding pass

**Location:** `_lym/plan/11-gate-adopt-key.md:40`

**Issue:** The focus asked whether card 11 states which callers of `MigrateConfig` gain the adopt key and whether a test pins that choice.
It does.
The Intent lists the local-layer file, `sparseSeed` and `configreg.PrepareRunSettings` as staying unseeded and gives the one hub-base call site.
`TestMigrateConfig`'s new row pins that `MigrateConfig` never seeds `adopt`, which covers all three by construction.
`TestReconcileHubWideAt_SeedsAdoptFromATunedFinalize` pins the hub call site.
That settles the focus question as not a defect.

The seed does leave one hub-base path out.
`ReconcileHubWideAt`'s absent-file branch builds the file through `sparseSeed` from a prime copy or legacy files, and the card's step runs only in the present-file branch.
The card says such a file "gains `adopt` at the next reconcile, which finds it present", but `hubreconcile.Ensure` returns after one stamp read once the build stamp names the running build, and the seeding pass leaves the stamp current.
A hub file seeded from a prime copy that tunes `finalize` therefore reads the template's `adopt: {tiers: [0]}` until a later binary build.
Decision 9 writes `adopt` "into a hub `gate.yaml` that tunes `finalize`, whenever the file holds no `adopt`", and the parent's amendment confines the seed to the hub base file's reconcile, of which this branch is a part.
The case is narrow, a hub with no gate file yet, and no check runs today on such a hub, so LOW.
The card must either carry the case or state the bound; the sentence it has says neither.

## Rejected

- Card 11's seed copying a `finalize` equal to the template, then `DropTemplateEqualNames` dropping both on the same pass, was checked and is not a defect: the template's `finalize` and `adopt` values are equal, so a retained `finalize` always differs from the template and so does its `adopt` copy, and a second pass is a no-op.
- Card 13's `ReadCheck` and card 15's `ReadCheckView` creating a directory or a lock file for an absent record was checked and is not a defect: both stat `check.json` first, and `lym beam status`'s help text claim of one write holds because card 15's test asserts the adopt scratch directory stays absent.
- The unnamed state-lock path for `check.json`, beside `check.lock` the verb's own lock: not raised, since a reuse of `check.lock` for both would hang `TestRunCheck` at its first record write and the rows already catch it.
