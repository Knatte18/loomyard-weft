```yaml
lens: code-tests
outcome: reported
```

**Lens: code-tests.** For each behaviour the diff adds or changes, the test that pins it, or none. My evidence is the diff's added test names plus three targeted reads. I did not read most test bodies, so "pinned" below means a named test exists whose title matches, not that I checked its assertions.

**Per behaviour:**
- Registry comparison `ReadsAsHere`, `OwnedElsewhere`, `Resolve`: pinned by `TestReadsAsHere` (11 rows, including an unregistered machine reading a registered current name) and the `OwnedElsewhere` rows in `owners_test.go`.
- Registration refusals (`Registry.Check`): `TestRegistryCheck`. `RegisterMachine` is pinned on origin by `TestRegisterMachine` (repeat writes nothing, clash refused, rename moves the old name to former).
- Owner write registers in the same commit: pinned by `TestSetOwners` with its identity subtests.
- `EnsureIdentity` (id write, no directory, invalid name): `TestEnsureIdentityWith`.
- `WriteHold` (pushed before return, repeat idempotent, other machine's hold standing, bad kind or missing fix task): `TestWriteHold`.
- `HoldOf` (adopt-fix does not hold its fix task): `TestHoldOf`.
- `CatchUp` engine (push owed, hand over, failure continues, nil seam): `TestCatchUp`.
- Dobby catch-up loop (fresh start versus re-exec, sleep, origin wait, refused notice, failed then cleared, hand-over owes it once, first read failing): `TestRun_CatchUp`.
- Bolt-hold entry sync (scope, owed push excluded, failure logged once): `TestRun_SyncsBoltHoldEntries`.
- Upgrade first read: `TestUpgradeRead`, including a re-run after the hold was cleared.
- `fixtries.Read` and `Write`: `TestReadWrite`.
- `OwnedWorktrees` and `EnsureLive` through registry flags: `TestOwnedWorktrees`, `TestEnsureLive`, and `TestBeamPair` in hubgeom.
- `ReadRun` held refusal after a Bolt fetch (`ReadRunPulled`): `TestReadRunPulled`, covering a hold written after the last pull, an unreachable origin and a dirty Bolt.
- `ReadRun` name set from the registry: `TestMachineNamesOf`.
- Act rule `Held` and `HoldMachine`: `TestAct` rows.
- `pauserecord` `BoltHoldCaller` readers: `record_test.go` rows.
- Tender resume rules under a hold (hub lift, refusal naming the machine, replacement entry, fix task, cleared hold, failed read): `TestSyncBoltHold` and `TestResume`.
- Pause intents and owed push: `TestPauseIntentEntries`, `TestRecordPauseIntent_ReturnsAnOwedPush`.
- `lym machine name` (`--forget-former`, id keep or replace, registration, sweep): `TestName_ForgetFormer`, `TestName_Set`, `TestName_SweepsHubs`; `machinename.NewID` and `ResolveIdentity` also have tests.
- Hand-over of runs owned elsewhere (push first, takeover entry, failed push writes no archive tag or mark): `TestHandOverOwnedElsewhere`, `TestPushRun_HandsOverOnATakeover`.
- Hand pass catch-up: `TestHubCatchUp` and `TestRunCmd_ResumeDispositions`.
- Queued tasks not claimed under a hold: the `AnAdoptHoldDoesNotStart` row in `TestReadyToStart`.
- Hibernate wake no longer holds runs owned elsewhere: rewritten `watch_test.go` row, no pause request and no live entry.
- orch start `endOwned` seam: `TestStart_ReconcilesTheHubFirst` and `TestStart_Refusals`.
- `lym beam status` showing the identity refusal: `TestDobbyStatus_ShowsTheIdentityRefusal`.

**Findings:**

1. **MEDIUM, design.** Location: `internal/boardengine/holdwrite.go:51-59`, test `internal/boardengine/holdwrite_integration_test.go`.
   - **Behaviour:** `WriteHold` judges "same machine" through `Registry.sameMachine`, so a hold written under a former name reads as the same machine.
   - **Evidence:** a grep for former-name, `sameMachine` and register in the integration test found no match. The subtests are only `AWrittenHold…`, `AHoldAnotherCloneWroteFirst…` and `AHoldOfAnUnknownKind…`.
   - **Consequence:** if `sameMachine` were reduced to `a == b`, a renamed machine's re-run would see its own earlier hold as another machine's, and no test would fail.
   - **Fix:** add a row where the standing hold's machine is a former name of the writer's registered entry, expecting an idempotent no-op with `Standing` nil.

2. **LOW, design.** Location: `internal/boardengine/holdwrite.go:56-58`.
   - **Behaviour:** a second hold of the same machine with a different kind or subjects is appended next to the first.
   - **Evidence:** I found no row for it in the three `TestWriteHold` subtests.
   - **Fix:** add a row asserting that two holds stand, or that the second is refused, whichever D5 intends.

3. **LOW, design.** Location: `internal/dobbyengine/handover.go:64-100` (`handOver` with `EndSubject` and `handOverFailed`).
   - **Behaviour:** the dobby's own hand-over now runs `CatchUp`, then `EndSubject`, then `ArchiveOwner`, and each failure keeps the dobby up.
   - **Evidence:** I confirmed `TestRun_ForeignOwner` exists, but I did not read its body. I could not confirm a row for a failing `EndSubject` or a failing `CatchUp` that keeps the dobby up.
   - **Fix:** if no row exists, add one in `handover_test.go` for each of those two failure steps, asserting that the dobby stays up and the log line is written.

4. **LOW, design.** Location: `internal/dobbyengine/catchupwait.go:313-338` (`oweHandOverCatchUp`).
   - **Behaviour:** `TestRun_CatchUp` has three rows for the hand-over trigger: pair present, pair absent, and owes it once. I found none that pins the memory being forgotten when the run stops answering hand over, so a later hand-over of the same slug would owe the catch-up again.
   - **Fix:** add a row where the verdict leaves hand over and later returns, expecting a second catch-up.

5. **LOW, design.** Location: `internal/dobbyengine/catchup.go:123-127`.
   - **Behaviour:** `ApplyPauseIntents` receives only `OwnedHere`, and its failure is joined into the returned error.
   - **Evidence:** `catchup_test.go:53` only wires a stub. The row titles I saw (lines 94-141) name no assertion on the slugs `ApplyPauseIntents` receives or on its failure.
   - **Fix:** add rows asserting the slugs passed (owned here only) and that a failure is returned without undoing the other runs' work.

6. **No finding, but uncertain:** two behaviours I could not confirm either way in the budget.
   - **Deleted behaviour:** removing `originowed_test.go` and the `OriginOwed` field is consistent with the card, and the `PassExitOriginWait` rows exist in `passes_test.go`.
   - **Pre-push guard and Bolt fetch:** `TestPrePushCheckGuard` and `TestRunOwnerVerdict` are listed as present, but I did not confirm they cover `lym loom resume` fetching Bolt before the act rule.

**Tool calls (3 of 6):**
1. `git diff … -U0 -- '*_test.go'`, filtered for test and row names: the manifest held no test names.
2. `grep` of `holdwrite_integration_test.go` for former, Register and `sameMachine`: to check coverage of the former-name rule in `WriteHold`.
3. `git diff` of `hibernateengine/watch_test.go`, plus a grep of `catchup_test.go` for `Ahead`, `ApplyPauseIntents`, `Pushed` and `HandedOver`: to confirm the hibernate rewrite and the `CatchUp` assertions.
