```yaml
lens: row-validity
outcome: reported
```

Row-validity review of the 25-card plan: whether each planned test row reaches its branch and would fail if the code were wrong, and whether it is needed.

**Findings**

1. **BLOCKING** (card 22 and card 18; D4, D5): the zero value of the new `dobbyengine.Deps.Upgrade` has no stated meaning, and beam dobbies would never catch up.
   - Card 22 fills `Upgrade` "for the prime's dobby only", so every beam dobby carries a zero `UpgradeDeps`.
   - `catchUp` "runs `UpgradeRead` before `CatchUp` until one run of it succeeds, keeping the catch-up owed while it fails".
   - Card 18's rule for its own seams is "A nil seam is a wiring error returned at once". If the same rule holds for `UpgradeDeps`, every beam dobby keeps its catch-up owed forever, so it spawns no pass and starts no task (card 21's gate).
   - No row in `TestUpgradeRead` or `TestRun_CatchUp` drives a zero `Upgrade`.
   - Fix: state that a zero `Upgrade` skips the first read, and add that row to `TestRun_CatchUp`.

2. **MEDIUM** (cards 21 and 18; nil seams in hand-built receivers): `Deps.CatchUp` and `Deps.SyncBoltHold` get no stated zero-value meaning.
   - Card 21 owes the catch-up at every fresh start. A zero `CatchUpDeps` hits card 18's wiring error.
   - Every loop test (`TestRun_LockOwnerAndWakeups`, `TestRun_TickFailuresLogOncePerEpisode`, `TestRun_Heartbeat` and the `handover_test.go` set) runs through `newLoopHarness`. Card 21 lists that harness under Edit but does not say whether it fills `CatchUp` fakes. If it does not, all of those tests stop spawning passes.
   - Fix: state the zero meaning (nil means no catch-up, as nil `PullBolt` means no pull), or say that `newLoopHarness` fills succeeding fakes.

3. **MEDIUM** (card 14; D5 criterion "`lym loom resume` refuses a run under a hold another machine wrote since the worktree's last Bolt pull"): the planned row cannot reach the pull.
   - `TestStartAndResume_RefuseARunOwnedElsewhere` is tier0 (`internal/loomcli/ownercheck_test.go:34`) and drives the `runVerdict` seam with a fake `act` verdict (lines 39–45).
   - The new `boardengine.PullBolt` sits inside `hubRunVerdict`, which that fake replaces. Deleting the pull would still pass the row, and a tier0 test cannot hold a second clone.
   - Fix: move the row to a tier1 test that drives `hubRunVerdict` over a hub whose origin holds a hold pushed from a second clone, and name that test and its helper on the card.

4. **MEDIUM** (card 14; the same criterion for `lym tender resume <slug>`): the planned row has neither a second clone nor a run.
   - `TestPauseAndResume_WriteLiftAndRefuseOverTheHub` (tier1) builds one `hubforge.CopyHub` (`internal/tendercli/cli_test.go:49`). It has no second clone to push the hold, and no batten record for `pairSlug`.
   - The card lists no helper that pushes a hold from another clone and none that seeds the run's record, so the row cannot reach the `Verdict` seam's pull followed by a held refusal.

5. **MEDIUM** (cards 7 and 13; `TestReadRun` rows): the fake the new rows need is not listed.
   - `TestReadRun` builds its reader through `runFakes.reader()`, which fills `owner:` from `f.owner`, `f.adoptSubject` and `f.elsewhere` (`internal/pairteardown/runread_test.go:58,76`).
   - Card 7 replaces the `owner` seam with `owners`, `identity` and `parentOf`, and card 13's hold rows need `Owners.Holds` from that seam.
   - Neither card lists `internal/pairteardown#runFakes` or `internal/pairteardown#runFakes.reader`, which must gain those fields. Without them the D3 and hold rows have no input and the package does not compile.

6. **MEDIUM** (card 7; batten's owner comparisons): batten's identity path is unplanned.
   - Batten's owner checks read this machine's name through `ownerSeams.Machine` (`internal/battencli/owner.go:29-30`, filled at `:53`) and through `battenCLI.thisMachine` (`:103`).
   - Card 7 edits `refuseStartOwnedElsewhere`, `refuseOwnedElsewhereOnLocalBoard`, `ownerExtras` and `otherMachineOwning` to compare through the registry. It names neither the seam that supplies an identity, nor `thisMachine`, nor the test fake `fakeOwners`, which has only `machine string` (`internal/battencli/owner_helper_test.go:12-14`).
   - Card 16's held rows in `TestRefuseStartOwnedElsewhere` run on that fake as well.

7. **LOW** (cards 3 and 7; test economy): two tests would pin the same registry rule.
   - Card 7 adds `TestOwnersEffective` rows for "a registered former name read here and elsewhere" and "an unregistered machine reading a registered current name".
   - Since `OwnedElsewhere` is `owner != "" && !registry.ReadsAsHere(...)`, those rows repeat card 3's `TestReadsAsHere` rows.
   - Fix: keep the empty-owner row in `TestOwnersEffective` and leave the registry cases to `TestReadsAsHere`.

8. **LOW** (card 20; test economy): `TestCatchUpRun` should not be a new top-level test.
   - It covers `battenCLI` pre-run dispositions: refusal, exit code, the hand-over exit, and going on to the reconcile.
   - `TestRunCmd_ResumeDispositions` already table-tests the dispositions of that same pre-run, and the card already edits it. The catch-up rows belong there.

9. **LOW** (card 6; `TestSetOwners` refusal row): the row has no named way to seed the registry.
   - The row needs origin's registry to hold another machine's entry.
   - `newOwnerFixture` seeds entries and a prime owner only (`internal/boardengine/ownerset_integration_test.go:40-75`). It has a second clone (`otherDir`) but no registry.
   - The card names neither `RegisterMachine` from the other clone nor a registry seed.

10. **LOW** (card 13; `TestAct` row): the new row is close to an existing one.
    - "A held landing run not past its parent-side merge waits" is near the existing `HeldWaits` row (`internal/runfsm/act_test.go:63-68`).
    - It adds value only if it sets `State: Landing` with `PastParentMerge: false`; the card should say so, or drop the row.

**Tool calls**

1. Grepped where the named tests live and their `tiergate.Require` tiers, plus the `TestReadRun` fake and `TestTriesStore`: my context had the test names but not their files or tiers.
2. Read the `runFakes` fields and `reader()`, and the head of `TestStartAndResume_RefuseARunOwnedElsewhere`: needed to judge whether the rows in cards 7, 13 and 14 can reach their branch.
3. Grepped which dobbyengine tests build `Deps` and read the head of `TestRefuseStartOwnedElsewhere`: needed to judge the nil-seam reach of cards 21 and 22, and card 16's fake.
4. Read the head of `TestPauseAndResume_WriteLiftAndRefuseOverTheHub`, the `CopyHub` use, the `TestWaitUncovered` line rows and the `TestName_Set` asserts: needed for the rows of cards 14, 12 and 10.
5. Read `newOwnerFixture` and `fakeOwners` and grepped batten's `Machine` seam: needed for the rows of cards 4–6 and the batten identity gap in card 7.
