```yaml
lens: row-validity
outcome: reported
```

Row-validity review of the landing-queue plan: whether each planned test row reaches the branch it claims, fails when the code is wrong, and is needed.

**MEDIUM**

1. **Card 5, `TestFinalize_MergesWithNoLockAndGatesUnderTheLock`: the "parent that moved before the lock" row cannot produce its state.**
   - The deleted table moves the parent through `movesAt`, which is keyed to verify calls (`internal/landingshed/finalize_unlocked_test.go:72-73`).
   - Once card 5 lands, no verify runs before the lock, so `movesAt` cannot move the parent "before the lock".
   - The row's expectations equal the unmoved row's: `p1` with no lock, `""` under it, one verify under the lock. So it fails on nothing the unmoved row does not already catch.
   - It needs a hook that moves the parent during the lock-free merge or the lock wait. The card does not list one.

2. **Card 5: a new top-level test replaces an existing one rather than rewriting it.**
   - The card deletes `TestFinalize_GatesWithNoLockAndLocksToLand` and creates `TestFinalize_MergesWithNoLockAndGatesUnderTheLock` holding "the kept rows of the deleted table".
   - That is the same test with its lost-race column dropped. It belongs under `Edit` on the existing test (renamed if wanted), not as a delete plus a create.

3. **Card 4: the code-side push hold-off in `pushClassified` has no row.**
   - Rows exist for `readLocalBranch` (`TestRunCommitsAhead`), `RefusedPushStands` and `parentHoldsTaskWorkAt`.
   - None reaches the new "marker stands → push nothing, return nil" branch on the code side, nor shows the records side still pushes.
   - If the branch were missing or wrong, no planned test would fail.

4. **Card 4: a prescribed refusal no row reaches (rubric "Reachability of a prescribed refusal").**
   - `parentHoldsTaskWorkAt` refuses "a marker that names no existing branch" with an error naming the marker.
   - The Intent names no input that reaches it and no test that does.

5. **Card 3: branches with no row.**
   - `Fabric.MergesClean` has no row in any of the card's three tests. Card 7 relies on it for D8's "would need a conflict session" precheck.
   - `MoveCodeTo` refuses a detached head; no row reaches it.
   - `BuildLandingChain` errors on a non-SHA base or task head, naming the slug; no row reaches it.

6. **Card 6, D9 criterion: `RetiredKeys` gaining `finalize_lost_races` is not pinned.**
   - The planned assertion is that a seeded file holding `finalize_lost_races` "still loads with the template's values".
   - Without the `RetiredKeys` entry, a key gone from the template is still an unknown key that loads with a warning. So the row passes whether or not the card adds the entry.
   - The entry's one effect, the local and machine layers refusing the key, has no row.
   - `internal/configengine/layers_test.go:373` shows how such a refusal is asserted.

7. **Card 10, `TestFinalizeQueue_ResumedDetached`: the clean-with-marker row cannot reach the closure it claims.**
   - The row says the worktree is restored "after the shed restore the loomcli closure runs".
   - That closure is set in `internal/loomcli#loomCLI.newShed` (`internal/loomcli/arm.go:302`) and is unreachable from `package landingshed_test`.
   - Calling `fabricengine.RestoreDetached` directly repeats `TestRestoreDetached`'s first row (card 4).
   - D5's "resumed at any row … put back on its task branch first" needs a loomcli-level row over `newShed`'s `Restore`. No card lists one.

8. **Card 7, `TestFinalizeQueue_ActsOnItsOutcome`: the "task branch gained a commit after it was marked landed" row does not reach the `actOnOutcome` branch it targets.**
   - `Call` runs `joinQueue` before `landUnderLock`.
   - Card 1's `Join` already clears a `landed` outcome when the task head moves.
   - So under the lock the entry has no outcome, and the "landed with a task tip past the entry's head" case in `actOnOutcome` is never reached by this row.
   - The row passes whether or not that branch is right. Only the parent-reset row reaches the clear-and-lead path.

**LOW**

9. **Card 7, tier0 conflict row: a new fake is not listed.**
   - The row needs a resolver fake that reads the lock store's record during `Resolve`.
   - Neither `tipRecordingResolver` nor `recordingResolver` does that, and the card lists no new fake under Create.

10. **Card 7, tier0 trail row: the row passes on the wrong branch.**
    - `finalizeGateFixture` wires no `PullRequest` seams (`internal/landingshed/finalize_gate_test.go:47-86`).
    - A row that sets only `Mode` to `trail` leaves `RecordPath` empty. `pullrequest.ReadRecord` may then fail.
    - That takes `joinQueue`'s "fails to read" exit, which also writes no entry. The row passes even if the trail decision were wrong.
    - The row needs a valid record path and lock.

11. **Card 7: the "backstop seam error keeps the mark" row repeats card 1.**
    - Card 1's `TestStore_Backstop` already pins that a `WriteLanding` error keeps the mark.
    - Card 7's row should assert only what is new: `Call` returns the error and no merge runs.

12. **Card 8, `TestFinalizeQueue_LandsABatch`: two rows lack listed support, and one repeats card 7.**
    - "A parent moved during the gate" names no mechanism that moves the parent while the shell gate runs, such as a gate command that commits in the parent pair.
    - The `WriteLedgerLanding` fake that reads the lock record is not declared.
    - "Each member's own Finalize ends Done with its `RecordLanding`/`LandLocally` commit equal to its landing commit" repeats card 7's landed-woken row for members. Only the leader's own commit is new there.

13. **Card 8, `TestDarnTreeScope_DiffsTheDetachedChainCommit`: should be a row.**
    - `TestDarnTreeScope_ClassifiesAtEachCall` (`internal/loomcli/darntreescope_integration_test.go:29`, tier1) already drives `darnTreeScope`. The new case belongs there as a row, not as a new top-level test.
    - It pins behaviour no card changes, since D5 keeps `darnTreeScope` reading HEAD. It holds only as a regression guard.

14. **Card 10, `TestShed_ResumeRestoreRunsBeforeAnyRow`: should be rows.**
    - `TestLockedCheck_RefusesUnderTheRunLock` (`internal/shedengine/lockedcheck_test.go:17`) is already a table over `Run` and `Step` with subtests per hook outcome.
    - The restore hook's cases fit there as subtests, not as a new top-level test.

15. **Card 9: the fixer row's fake and one claim go unpinned.**
    - "The fixer fake runs a second member's Finalize" needs a `FixerSeats` fake that builds and calls another `Finalize`. The card does not list it.
    - No row asserts "no gate runs twice on one tree" (a gate-run count for the culprit leader). That claim is the one behaviour the new `runFixer` entry point exists for.

16. **Other branches with no row:**
    - Card 1: `Backstop`'s "member with no entry is written to the ledger alone".
    - Card 4: `CheckoutBranch`'s dirt refusal.
    - Card 3: `NewestLandingOf`'s 500-commit cap.
    - Card 11: the "backstop error releases the lock and is returned" path.

Tool calls:
- First call: a grep for whether the named existing tests and `newShed` exist and their tiers, plus `FinalizeLostRaces` uses in `finalize_unlocked_test.go`. My context had only glyph names, not definitions or tiers.
- Second call: reads of `finalizeGateFixture`'s seams, the shape of `TestLockedCheck_RefusesUnderTheRunLock` and `TestDarnTreeScope_ClassifiesAtEachCall`, and configengine's retired-key tests. My context lacked these fixture bodies.
