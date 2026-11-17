```yaml
lens: row-validity
outcome: reported
```

Lens: row-validity. I checked whether each planned test row can reach the branch it claims and would fail if the code were wrong, and whether any new test repeats an existing one.

**Findings**

1. **BLOCKING — Card 4, claude engine tests.** Adding the row breaks two tests in `internal/shuttleengine/claudeengine` that the card does not list.
   - `TestBuildSettings_NoAgentHookAndForkGuard` (`settings_test.go:598`) sorts Bash entries by each row's `Steer` (`:632-646`). The new row falls into the `default` bucket, so the assertions "want none" (`:659`) and "want exactly the webster fork guard" (`:663`) fail.
   - `TestBuildSettings_MatchesGolden` (`settings_test.go:1105`) compares against `testdata/settings-golden.json`, which holds every rendered row (`headless` appears 32 times). The golden needs the new row.
   - Fix: add both tests and the golden file to the card. In the first, give the new row its own bucket with a "want exactly one" assertion.

2. **BLOCKING — Card 2, `TwoClonesAdoptingTheSameBeamEndWithOneOwnerAndTheLoserRefusesNamingTheWinner`.** The row cannot put the other clone's push between this clone's pull and its push.
   - `ownerFixture.pushFromOther` (`ownerset_integration_test.go:120`) pushes at once. The existing race row reaches the window only through `OwnerRequest.BeforePush` (`:497`).
   - `WriteAdopt` goes through `commitBoardWriteAndPush` (`registrysetter.go:166`), which has no hook. `AdoptRequest` as listed has no `BeforePush`.
   - As written, the other push lands before the pull, so the row tests a plain "another machine's hold" refusal and never reaches the rejected-push replay or the re-read that names the winner. The D6 criterion "two machines adopting at once end with one owner" then goes untested.
   - Fix: give `AdoptRequest` a `BeforePush` seam, with nil meaning nothing runs, threaded through to the push.

3. **MEDIUM — Card 1, the new `TestFabricReplay` rows.** `TestFabricReplay` (`replay_test.go:183`) builds every row from `replayRow`, whose four sides are `[]Task` only. Nothing in it can carry `holds` or `machines`, so none of the six rows can state its input.
   - Fix: list `internal/boardengine#replayRow`, and the helper that renders a side's `board.json`, as edited targets. Give them holds and registry fields for each side, with nil meaning the key is absent.

4. **MEDIUM — Card 1, `AnAdoptCommitMeetingAnotherMachinesNewHoldIsRefusedWhole`.** The card names neither the commit's input nor the counter-case.
   - The all-or-nothing claim fails only if the commit also changes an entry and the prime owner that origin did not touch. Without those, a partial apply would pass unnoticed.
   - No row covers a commit that leaves holds unchanged while origin gained a hold, which must still replay. That is the "a commit touching neither replays unchanged" branch.

5. **MEDIUM — Card 9, `TestAdoptIntegration_SecondCloneTakesOver`.**
   - `AdoptAs` now runs `HoldCheck` over `hubHoldCheckDeps`, which calls `gatecli.RunGate` on the hub's `adopt` key inside the test binary. `RunGate` has no `testing.Testing()` guard; only `detachfront.go:156` has one.
   - The card states no fake gate config for the fixture and no expected check outcome. The row's final assertions depend on the hold being cleared, which needs a green gate.
   - Fix: name the gate config the fixture seeds, as `TestHubCheckDeps_StaleStampRunsTheTunedFinalizeTiers` does, and the check outcome the row expects.

6. **MEDIUM — Card 2, `TestHoldSetters`.** `FixHold` and `ClearHold` write the same `holds` list through the same commit path as `WriteHold`, and `TestWriteHold` (`holdwrite_integration_test.go:17`) already has the fixture.
   - The four rows belong in `TestWriteHold`, or in `TestWriteAdopt`, which builds the hold they act on. A fourth top-level test over the same fixture repeats the setup.
   - This one is a judgement call: the setters are separate functions, so their own test is defensible.

7. **LOW — Card 5, `TestUnderOwnHold`.** The rows `AHoldOfThisMachineCoversThePrime` and `...CoversABeam` need a hold whose `Machine` reads as the test process's identity, but the card does not say how the test gets that identity. Without a stated identity, "this machine" versus "another machine" is ambiguous and the covers rows could pass for the wrong reason.

8. **LOW — Card 8.** `runHoldCheck` gets new tier-0 rows, but nothing reaches `hubHoldCheckDeps` itself. The card leaves to the implementer how `DropBoltHold` picks its scope ("the scopes of the live `BoltHoldCaller` entries"). The D6 criterion "no bolt-hold entry on the adopting machine when the adopt returns" therefore rests on an untested wiring of `SyncBoltHold`.

9. **LOW — Card 7, `TestHubRunSlugs`.** It is a new top-level test, not a repeat: `DobbyPairSlugs` has no test of its own. It would be cheaper as a row in `TestHubCatchUp` (`battencli/catchup_integration_test.go:30`), which already builds a hub with runs.

**Tool calls**
- grep for the test functions and fixtures the cards name: my context lacked their locations and packages.
- `settings_test.go` function list, lines 500-516 and 625-672: I needed to see where the per-row count and bucket assertions sit.
- grep for the settings golden file and `TestFabricReplay`'s row shape: my context lacked both.
- `gatecli` test guards and `ownerFixture`'s race mechanism (`BeforePush`, `pushFromOther`): my context lacked them.
- `DobbyPairSlugs` tests, envkit's machine identity, and the `runCheck` end-row assertions: I needed them for the card 7, card 5 and card 8 findings.
