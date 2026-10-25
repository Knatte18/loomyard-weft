```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it, or "none". Test economy is applied where I could see the tests.

**Limit of this report.** I opened no test bodies. The map below comes from test and subtest names in the diff plus the manifest's reading. I can say a test exists and what it claims. I can't say it fails on a regression, so every "pinned" below is by name only.

## Behaviour → pinning test

Pinned by name:

- **Yield outcome and loop end** (cards 8 to 10):
  - Persists nothing, ends the run: `internal/shedengine/yield_test.go` `TestStep_YieldPersistsNothing` (Step and Run rows).
  - Envelope `continue` false on a yield: `internal/shedverbs/step_test.go` and `loop_test.go` rows (+14 and +15 lines in the stat, bodies not opened).
  - Driver exit `ExitYield` with no rule, report or budget: `internal/sheddriver/drive_test.go` and `driverules_test.go` rows (+7 and +9 lines, thin).
- **InnerRun one check per pass, memory record** (card 13): `TestInnerRun_MemoryOutlivesTheProcess` in `innerrun_memory_test.go`, plus subtests "ACheckThatEndsNothingYieldsWithAnEmptyPointerAndNoSleep" and "ARunningChildWithNothingToDoYieldsAndLeavesBattensStatusUntouched".
- **Spawn-hold yield** (card 14): `TestInnerRun_SpawnHoldYields`.
- **Respawn cap** (card 15): `TestInnerRun_RespawnCapStopsWithoutProgress`.
- **API-error revive** (card 16): subtests "TheFirstSightRevivesAndTheEpisodeStillStandingAtTheNextProbeNotices" and "AnEpisodeThatEndsAfterTheReviveNoticesNothing".
- **Failed revive sends the driver-dead notice**: subtest "AFailedReviveOfAHaltedChildSendsTheDriverDeadNoticeAfterTheHaltsOwn".
- **Idle-bound release across passes**: subtest "AnIdleOwnersParentLockIsReleasedOnThePassAfterTheIdleBound".
- **Idle-watch snapshot** (card 11): `TestIdleWatch_RestoreKeepsTheWait`.
- **Episode-set snapshot** (card 12): a `logger/episode_test.go` addition (+23 lines, name not seen).
- **Records-stop notice on a successful landing** (cards 17, 42): `TestLand_RecordsStopNotice`.
- **Land and Teardown failure notices after the stop chain** (card 18): rows in `internal/battenshed/notice_test.go`, `rownotice_test.go` and `stuck_test.go`. These are inferred from the stat, not seen.
- **Draft rule, migration and key** (cards 5, 6): `TestDraftStatus` and `TestDraftMigration`, plus `replay_test.go` rows.
- **Cancel and claim settlement write draft, start guard refuses draft** (card 7):
  - Claim settlement: the pairteardown row "BoardClaimIsLeftDraftWhenBattenNeverSeededTheRun".
  - Start guard: `startguard_test.go` (+35 lines, name not seen).
  - Cancel: `cancelcmd_test.go` (+5 lines, thin).
- **Dobby planner** (cards 32, 33):
  - Pass kind decisions: `TestPlanPass`, rows include cancel marker, owned-elsewhere lift, covered lift.
  - Exit handling: `TestDrivePasses_ExitHandling`.
- **Dobby loop** (card 32): `TestRun_LockOwnerAndWakeups`, `TestRun_TickFailuresLogOncePerEpisode`, `TestNextWait`.
  - Subtest "a sleep longer than a minute between ticks is recorded in the state and owes every run a full pass" covers the sleep watch.
  - Subtest "a recorded sleep owes every run a full pass" also covers it, and the two look overlapping.
- **Owned-elsewhere handling** (card 34):
  - Owner pause stands until its lift: subtest "a covering owner pause stops every pass, a restart's included, until its lift, which runs a full pass whose exit 0 clears the mark".
  - Dobby lifts a stale owner pause itself: subtest "a local Bolt that reads this machine again lifts the owner pause and runs a full pass, and one still reading the other machine lifts nothing".
  - Pause lift while a pass is in flight: subtest "a lift while a pass is in flight gives the run its pass once that pass exits".
- **Ready starts, backoff, `max_runs`** (card 35): `TestReadyToStart` (rows "AbsentStatusStarts", "DraftDoesNotStart"), `TestNextStartTry`, `TestStartReady_RefusalsAndBackoff`.
- **`EnsureLive` and live read** (cards 27, 37): `TestEnsureLive`, `TestLive`, `TestDobbyGeometry`.
- **Board tidy** (card 36): `TestCountLandingsAndTidy` (five-landing spawn, restart, first start, 0 off, deadline) and `TestTidyPrompt`.
- **Dobby state record**: `TestState_RoundTripKeepsUnknownKeys`.
- **Config**: `TestLoadConfig`.
- **Owned-elsewhere typed exit** (card 26): `TestOwnedElsewhere`.
- **Continue-pass refusals** (card 30): `TestBattenPreRun_ContinuePass`.
- **Origin-owed marker** (card 29): `TestOriginOwedMarker`.
- **Dobby reading by the start guard**: `TestDobbyReading`.
- **Arbiter**:
  - Shed-layer call, prompt and spec: `shedarbiter` `TestPrompt`, `TestSpec`, `TestCall_*`, `TestVerdictGate`, `TestConsumeAttempt`.
  - Batten eligibility, including the pause-honoured hand-back row: `TestEligibility_BattenRule` and `TestCheck_RefusesLandingHalts`.
  - Key copy into `darn.yaml`: `TestReconcile_CopiesLoomArbiterKeysIntoDarn`.
- **Key-level config notice** (card 41): `TestChangedKeys` and `configchanges_integration_test.go`, plus finalize rows.
- **Pair-teardown liveness read** (card 40):
  - Registered batten, or a live dobby with batten `running`: the `fakeSubstrates` table, with a "CancelMarkerIsNotAlive" row.
  - `reed down` sparing `RoleDobby`: `TestRegisterDobby` and `runproc/end_test.go` changes.
- **Adopt refusals before any mutation** (card 39): `adopt_test.go` (+107 lines, names not seen).
- **Agent name scoped program** (card 1): `agentname_test.go` changes.
- **Creel scoped program** (card 2): `creelcli/goclient_test.go` (+40 lines).
- **Orch starts dobbies, Warn on failure** (card 38): `orchcli/startdobbies_test.go`.

## Findings

**F-T1 — MEDIUM, design.** `internal/dobbyengine/passes.go` (`planPass`, `passExited`) and `loop.go` (`tick`): the debug log of a yield is a Constraint ("a yield at Debug"), but no test I could see asserts the level or the line.
- The logs are in `sheddriver/drive.go` ("sheddriver: drive: the step yielded") and `innerrun.go` (`once`).
- I found no assertion on either Debug line.
- Evidence: `drive.go:217-221`, `innerrun.go` `once`.
- This is "a behaviour the diff adds with no coverage", or the PATTERN-shed-driver "A yield is logged at Debug" line has no check.
- I'm unsure of it because I did not open `drive_test.go`'s +7 lines.
- Lower to LOW if any test captures the log.

**F-T2 — LOW, design.** Respawn-cap reset on a status write, and the third revived api-error episode in a row with no status write: no named row.
- Only `TestInnerRun_RespawnCapStopsWithoutProgress` and the two revive subtests are visible.
- The reset and the third-episode trip (`apiRevives`) are separate behaviours from the first-sight revive.
- Evidence: `innerrun.go` `Call` (`baseline.After(p.respawnStatusTime)`), `innerrun_memory.go` `APIRevives`.
- If the cap test covers the reset as a row, this is not a finding.

**F-T3 — LOW, consistency.** `internal/dobbyengine/start_test.go` and `starts_test.go` are both 207 lines against 212 and 214 lines of source.
- They hold distinct functions by name (`TestEnsureLive` and `TestLive` against `TestReadyToStart`, `TestNextStartTry` and `TestStartReady_RefusalsAndBackoff`), so not a duplicate.
- The sleep-watch behaviour is named in two subtests (loop test "a sleep longer than a minute…" and passes test "a recorded sleep owes every run a full pass"): check for overlap.
- Evidence: loop_test.go subtest list.
- Raise only if the bodies assert the same thing.

**F-T4 — LOW, design.** Behaviours with no named test:
- Re-exec is not a start, and a wake owed across a re-exec is kept.
  - `Run`'s pid check in `recordStart` has no named row (`TestRun_LockOwnerAndWakeups` may hold it).
  - Evidence: `loop.go:150-161`.
- Dobby's `beamOwnedElsewhere` exit on a probe tick is probably in `TestRun_LockOwnerAndWakeups`.
- Batten `--continue-pass` skipping the pre-run pause refusal is probably in `TestBattenPreRun_ContinuePass`.

I found none of these:
- A test of the fswatch-open-failure fallback to the timer.
- A test of the `max_runs` cap itself, as opposed to `ReadyToStart` rows (`TestReadyToStart` may carry it).
- A test of `internal/configsync/movekeys.go` and `configreg/keymove.go` outside `TestReconcile_MovesKeysAcrossModules`, which probably covers them.

**No stencil-prose test findings.** I saw no new assertion on static stencil prose among the listed names.

**Deleted tests, checked for coverage loss.**
- Deleted: `loomcli/arbiterprompt_test.go`, `loomengine/driver_test.go` (−64), `battenshed/teardown_test.go` (−72) and `windowsite_integration_test.go`.
- `shedarbiter/prompt_test.go` replaces the first.
- The Teardown notice-after-chain behaviour moved to notice and rownotice tests.
- I can't confirm the other deletions kept their behaviours covered.
- `TestSpec`/`TestPrompt` in shedarbiter cover the moved code.

## Tool calls

1. Grepped test function names across the new and changed test files, because my context held no test bodies or names.
2. Grepped added subtest and row names in the diff's `*_test.go` files for revive, yield, lift, wake, draft, tidy and teardown behaviours, because the first grep showed only top-level names.
