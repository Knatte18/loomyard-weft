```yaml
lens: affected-set
outcome: reported
```

My lens is affected-set: for each change a card makes, find everything outside the card that reads or asserts it, and check that some card covers it.

**Per change: what reads it, and which cards cover it**

- **Card 1, the fix-tries reader and writer move to `fixtries`.**
  - Production readers: `failedEnd` (fixloop.go:213, 279), `greenEnd` (green.go:35) and the `ResetTries` closure in `hubDepsAs` (deps.go:104).
  - Test readers: `TestGreenEnd`, `TestRunCheck` (including its read of `triesFileName`), `fixFixture.tries` and `fixFixture.setTries`.
  - Covered by card 1. Held.
- **Card 7, `OwnedElsewhere` re-signed and name comparisons moved onto the registry.**
  - Every `machinename.Name` reader in production code is either in card 7, 8 or 9, or only uses the name rather than comparing it:
    - Name only: `SlackGeometry`, `orchCLI.machineName` (remote-control session name), batten `specFor`, and `hubFixDeps` (fix task title).
  - Comparisons that card 7 lists: `fabriccli#foreignOwner`, `orchcli#beamOwnerElsewhere`, `adoptcli#subjectOwnedElsewhere` and `hubCancelDeps`.
  - Held.
- **Card 9, `ThisMachine(hubPath)`.** Callers: `PushRun`, `fabriccli` weft_verbs.go:258 and 284, `loomcli` landingdeps.go:170, and `machineLandings`. Covered by card 9. Held.
- **Card 12, `bolt-hold` entries set `Coverage.Held` and change `CoveredError` and `heldLine` wording.**
  - Readers: `DobbyRuns` → `RunView.Held` (card 20 changes its doc), whose only reader `recheckOwnerPause` is deleted by card 23.
  - Wording asserters: `TestCoveredError` and `TestWaitUncovered` (card 12).
  - `TestDobbyRuns_PairAtItsOwnAnchor` still passes, since its existing rows are unchanged.
  - Held.
- **Card 13, `ReadRun` fills `Held` and the held refusal is reworded.**
  - Readers of the verdict:
    - Dobby (dobbyreads.go:199): intended.
    - `loomcli` owner check (ownercheck.go:149): intended.
    - `batten run` (`runAside`, arm.go:638): intended.
    - `batten cancel` (`cancelAside` carries `Hold`): passes, as D5 requires.
    - Overview (overview.go:245): display only.
    - `tender resume` (`resumeAside`, runintents.go:31): **broken. No card covers it (F1).**
  - Wording asserters: `TestAct` and tenderengine `TestResume`, both in card 13. Held.
- **Card 20, the origin-owed marker retires.**
  - Production readers: `DobbyRuns` (dobbyreads.go:216), `refuseContinuePass` (arm.go:626), `driveRun` (drive.go:113), `battenCLI.originReached`, `planPass` and `passExited`.
  - Tests: `TestPlanPass`, `TestDrivePasses_ExitHandling`, `TestRunCmd_ResumeDispositions`, `TestBattenPreRun_ContinuePass`, `TestOwnerPreflight_GoneTaskWorktree`, `TestOwnerPreflight_TeardownResume` and originowed_test.go.
  - All covered by card 20. Held.
- **Card 23, the owner-pause seams go: `Deps.PauseOwned`, `Deps.LiftOwnerPause`, `InnerRunDeps.PauseOwned`, `prePushCheck.pauseOwned`, the hibernate `EndOwned` and the orch install.**
  - Covered:
    - `newLoopHarness`, `TestOwnedElsewhere`, `TestDrivePasses_ExitHandling`, `TestRun_ForeignOwner` (card 21).
    - `TestInnerRun_OwnerCheck`, `TestPrePushCheckGuard`.
    - `fakeWorld.endOwned`, `fakeWorld.deps`, `TestSequence`.
    - `startHarness.runWith`, `TestStart_ReconcilesTheHubFirst`, `envkit#nilLegal`.
  - **Missed (F2):** `newPassHarness`, `runAnAdoptHold`, battenshed `newLockHarness` and loomcli `stoppingPrePushCheck`.
- **Card 23, the dobby notice reword ("so it is paused here").** Its only reader is owned.go:78, asserted by `TestOwnedElsewhere`. Covered by card 23. Held.

**Findings**

- **F1. BLOCKING, class: decision.** Location: card 13 (`internal/pairteardown#runReader.read`), with card 15 (`internal/tenderengine#Resume`).
  - Summary: once card 13 fills `Held`, `lym tender resume <slug>` hits the act rule's generic held refusal before any lift. That makes card 15's resume rules under a standing hold unreachable:
    - the lift of the slug's own entry that writes a `bolt-hold` entry in its place;
    - the dedicated refusal "`<slug>` is held by an adopt on machine `<m>`".
  - D5's rule "lifting a slug's own entry under a standing hold writes the bolt-hold entry in its place" is therefore not delivered.
  - No card adds `Hold` to `resumeAside` or otherwise sets the hold aside for the resume's verdict.
  - Evidence:
    - `internal/tendercli/runintents.go:31`: `resumeAside` has no `Hold`.
    - `internal/tenderengine/resume.go:122-128`: `req.Verdict()` and its `Refusal` run before the lift.
    - `internal/runfsm/act.go:117`: `in.Held` gives `ReasonHeld`.
- **F2. MEDIUM, class: consistency.** Location: card 23 (Edit list).
  - Summary: removing the `PauseOwned`, `LiftOwnerPause` and `pauseOwned` seams misses four test helpers that set them, so those test binaries stop compiling:
    - `internal/dobbyengine#newPassHarness` (passes_test.go:229, 239);
    - `internal/dobbyengine#runAnAdoptHold` (handover_test.go:178);
    - `internal/battenshed#newLockHarness` (innerrun_parentlock_test.go:97);
    - `internal/loomcli#stoppingPrePushCheck` (prepush_test.go:392).
  - These are field removals, not re-signed members, so the gate's `caller-uncovered` check does not list them.
  - Evidence: the enclosing-function lookup from tool call 1; `internal/loomcli/prepush_test.go:392`.

**Tool calls**

1. Looked up the functions enclosing the affected test and caller lines. The manifest's greps gave line numbers without the enclosing names.
2. Grepped the `runfsm.Aside` uses, the `machinename.Name` readers in production code and the tests asserting `CoveredError` wording. The manifest lacked the act-rule asides and the full list of name readers.
3. Named the functions around the remaining name reads outside the cards, to tell comparisons from name-only uses.
4. Checked the `fixloop`, `orchCLI.machineName` and `specFor` name uses, to confirm none of them compares names.
5. Read how `tenderengine.Resume` and the batten overview use the verdict. The manifest did not include resume.go.
6. Found the other users of `StartDeps`, `BeamPair` and `PrimeOwner`, the loomcli tests that set `pauseOwned`, and the orchcli and hibernate tests that set the owner-pause seams. The manifest lacked these test files.
