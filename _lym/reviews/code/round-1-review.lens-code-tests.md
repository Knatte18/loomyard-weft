```yaml
lens: code-tests
outcome: reported
```

Task: for each behaviour the diff adds or changes, name the test that pins it, or none (lens code-tests).

**Scope note:** my context holds the diff pieces for `reedengine`, `shuttleengine`, `claudeengine`, `shuttleengines`, `orchcli` and `orchengine`. I did not read `adoptcli`, `battencli`, `claudecli`, `hubgeom`, `loomcli`, `shell`, `shuttlecli`, `tokencount` or `fabricengine`, so I give no per-behaviour verdict for them.

**Per behaviour: the test that pins it**

- **Reed hold ops:**
  - Held strand skipped by resume planning: `TestPlanResumeLaunches_ThreeLifecycleStates`.
  - End, relaunch under the same GUID, refusal of a removed or unheld strand, and release: `TestEndAndRelaunchStrand_KeepTheGUID` (tier 2).
  - `Status.Held`: the same tier-2 test.
  - "Reconcile leaves a held record" (doc claim): none.
  - `EndStrandProcess` failing after the mark was saved: none.
- **Relaunch record writes under one lock hold, stop-made terminal record left alone:**
  - `ARecordAStopStrandMadeTerminalBefore…`, `AStopStrandBetweenTheMark…`, `ARunStopBetween…` in `TestWait_UsageLimitSwitchRelaunchesInTheRun`.
  - The start-path stop in `TestStart_UsageLimitedLoadTurnRepeatsTheLaunch`.
  - A concurrent write racing `Run.Stop` or `stopRecorded` ("loses neither change"): none.
- **Relaunch step order, record names the new launch before reed relaunches:** none. The Wait rows assert only the final record. Nothing reads the record inside `RelaunchStrand` (`relaunchReed.onRelaunch` could).
- **Relaunch records `PromptOffset`/`LaunchOffset` (events size) and `TranscriptCopied`:**
  - Offsets: only through a seeded record in `TestAttach_RelaunchingRecordPastItsBound`.
  - Fresh relaunch records `TranscriptCopied` false: `startedFresh`.
  - A continue relaunch recording `copied == true`: none (`engine.carried` is never set on a Wait row).
- **Startup wait with events proof off, trust dialog dismissed:** `ATrustDialogOverTheOldSessionsLinesIsDismissed…`.
- **Never-ready pane ends died or timeout:** `APaneThatNeverReachesReady…` (both rows).
- **`Prepare`, check refusal and no-transcript fallback to fresh:** the matching rows. No-transcript fallback in the start-path repeat: none.
- **`relaunchRepeat` of a launch that resumed a session (same session id, copy carried):** none. The only `resumes` start row uses the fresh relaunch.
- **Declined relaunch (mark write fails):** none. Covered by no row: `errRelaunchDeclined` in `relaunchUnderAccount`, `relaunchAfterLoadHold`, `startFreshAfterRefusedResume`, and `switchAccountAtLimit`'s restore of `handledOffset`.
- **End of process fails over a live strand (release hold, clear mark), or over a dead strand (carry on):** both rows.
- **Switch answers false or errors, nothing sent to reed:** `ASwitchAnsweringFalse…`. The switch-error branch of `switchAccountAtLimit`/`switchAtLoadLimit`: none.
- **Bound handling:**
  - Attach past the bound over a held, paneless strand ends died: `AMarkBeforeTheRecordWrite…`.
  - Within the bound attaches as live: `AMarkWithinItsBoundAttachesAsLive`.
  - Past the bound over a live strand clears the mark and types the `PromptLine` once: `AMarkPastItsBoundOverALiveStrand…`.
  - Output files existing ends done, not died: `ARecordWhoseOutputFilesAllExist…`.
  - A Wait beside a mark waits out the bound: `AWaitBesideAMarkWithinItsBound…`.
  - `checkLivenessTick` held and paneless inside the bound counts as no failure: same row.
- **Running-run predicate sites:**
  - Activity and waits: `TestReadAgentActivity` (rows within and past the bound), `TestReadAgentWaits`.
  - Session state: `TestReadSessionStates`.
  - Carry-over: the carryover row.
  - Orphan sweep: `TestSweepOrphans_AgeGuardAndLiveGuid`.
  - Attach: `TestAttach_OutcomeDisposition` and `TestDispositionCandidate_RelaunchingRecord`.
  - `StopStepRuns` within the bound: the `relaunching` row.
  - `StopStepRuns` past the bound: none.
  - `StopOrphanedStepRuns` with the new `runIsLive(rs, now, bound)` (a relaunching run within the bound must not be selected): none.
- **`StopStrand` over a held strand:** `TestStopStrand_RelaunchingRecordStopsAndRemovesTheHeldStrand`.
- **Switch asks, `RefusedResume` and `refusedResume` rules:**
  - Both are pinned by `TestRunner_SwitchAccountOnLimit` and `TestWait_RefusedResumeStartsFresh` (Wait path and `RefusedResume` rows).
  - Load-turn path: `TestStart_RefusedResumeOfALoadTurnStartsFresh`.
  - Switch at a load limit: the first `TestStart_UsageLimitedLoadTurnRepeatsTheLaunch` row.
  - Limited turn end neither counted nor re-sent: the `retries: 2` rows.
- **`checkResume` carry, copy fact on `SessionHeldError`, and the orch retry carrying it:** `TestRunnerStart_Resume_CarriesTheTranscriptFirst` and `TestStart_ResumeOverALiveHolder`.
- **Orch path:**
  - `relaunchOrch` resume and fresh: `TestRelaunchOrch_RestartsOverTheLiveLimitedStrand`.
  - Watcher switch restart and failed resume: `TestWatcher_UsageLimitSwitchRestartsTheOrch`.
  - Fresh restart on a refused resume: `TestWatcher_RefusedResumeStartsTheOrchFresh`.
  - Failed fresh restart, and a `RefusedResume` ask error logging only: none.
  - The `watch.go` closure wiring (`relaunch`, `LoadState`): none.
- **`claudeengine`:**
  - D1 config refusals: `TestLoadConfig`.
  - Account store and merge: `TestMergeUsage`.
  - Pick and `PickAccount`: `TestPickAccount` and `TestClaude_PickAccount`.
  - Resolution and launch export, unset or none: `TestPrepare_ExportsTheResolvedAccount`, `TestBuildLaunchCmd`, `TestBuildResumeCmd` and `TestOpenLaunch`.
  - Switch rules: `TestClaude_SwitchAccount`.
  - Verbs: `TestClaude_RecordStatusLine`, `TestClaude_SetAccount`.
  - Carry: `TestClaude_CarryTranscript`.
  - Registries as one set, in both the prober and the holder: `TestProcessLiveness`, `TestCheckResumable`, `TestSessionHolder`.
  - Scrub: `TestEnvScrub_MatchesTmuxkitServerEnviron`.
  - `NewForWorktree`: `TestNewForWorktree`.
  - Hub-built readers and namers: `TestRecordReaders`, `TestSessionNamers`.
  - Live readers (`OutstandingForks`, `AuditForks`): `TestClaude_LiveReadersReadTheLaunchedDirectory`.
- **`claudeengine` gaps:**
  - The env module's `CLAUDE_CONFIG_DIR` as a known directory for `LocateSessions` or the registries: none.
  - The `sessionProjectDir` tier "no live entry, resolved lacks it, newest known copy": none.
  - `SetAccount` on an engine told no worktree: none.
  - `SessionNameDrift` and `SessionUsage` (they use `sessionProjectDir` but have no launched-directory test): none.
- **`PickAccount` calls `Names()` under the lock** (D2: "a worktree made meanwhile keeps its entry"): none. The rows use a fixed names list, so moving `Names()` outside `updateAccountFile` would pass every row.

**Findings**

1. LOW, `scope`, `internal/shuttleengine/accountrelaunch_test.go` (`TestWait_UsageLimitSwitchRelaunchesInTheRun`). No row proves the record names the new session, `PromptLine`, cleared started flag and offsets before reed relaunches, which D7 step 4 and the acceptance criteria require. Evidence: `relaunchReed.RelaunchStrand` at lines 187-203 never reads the record, and the rows read it only after Wait.
2. LOW, `scope`, `internal/shuttleengine/stepstop.go:78-81`. `StopOrphanedStepRuns` now passes `now` and `bound` to `runIsLive`, and no test pins a relaunching run within its bound being left alone. The `StopStepRuns` past-bound case is also unpinned (`stop_test.go` has only the 1-second mark).
3. LOW, `scope`, `internal/shuttleengine/accountrelaunch.go:258-265`. No row exercises `errRelaunchDeclined` (mark write fails) in any of the Wait, load-hold or refused-resume callers, nor the switch-error branch.
4. LOW, `scope`, `internal/shuttleengine/accountrelaunch.go:341-356`. `relaunchRepeat` over a resumed spec has no test. A resumed start whose load turn hits the limit should resume the same session id with the copy carried (parent decision F2).
5. LOW, `scope`, `internal/orchengine/restart.go:60-90`. No row covers a failing `Fresh` restart or a `RefusedResume` error.
6. LOW, `scope`, `internal/shuttleengine/claudeengine/accountkeeper.go:593-597`. No test fails if the `Names()` reader moves outside the account file's lock (acceptance D2).
7. LOW, `scope`, `internal/shuttleengine/claudeengine/transcriptread.go:452-489`. The env-module directory as a known directory is untested for `LocateSessions` and the registries. The `sessionProjectDir` fallback to the newest known copy is also untested.
8. LOW, `consistency`, test economy (focus item), the three tests the redundancy report calls contained. Each pins an assertion its named covering tests lack, so each should carry `//testtiming:keep <reason>` directly above its `func Test…` line, not be deleted:
   - `internal/orchengine/restart_test.go`, `TestWatcher_RefusedResumeStartsTheOrchFresh`: the covering test never drives the `RefusedResume` path, the `Fresh` call, or the "asked as a refused resume alone" assertion.
   - `internal/shuttleengine/attach_asking_test.go`, `TestDispositionCandidate_RelaunchingRecord`: the unit row "within the bound over an untracked strand reads `verdictError`" is covered by none of its named tests.
   - `internal/shuttleengine/claudeengine/worktree_test.go`, `TestNewForWorktree`: the zero `Worktree` from `NewFromConfig` is asserted nowhere else.

**Tool calls:** none. My context already held every piece I judged.
