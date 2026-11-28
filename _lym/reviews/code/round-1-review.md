---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/run.go#runWithSlots
    summary: The printed wait is measured from the start of the slot wait to the tier command's Start, so it also counts work done inside the held slots before the command starts. That is the whole shared build on the first opted-in tier, and the lym prebuild of a --copies run. The help and package doc call that figure the slot wait.
    fix: "Stop counting the wait at the moment the slots are held. In runWithSlots, take a timestamp right after acquireSlot returns, or right after the no-slot branch is decided. Carry it into report next to waitStart, and store it on the RunRecord as a new omitempty time field, such as SlotsHeld with JSON key slots_held. RunRecord.SlotWaitSeconds then returns SlotsHeld minus WaitStart when SlotsHeld is set, and falls back to Start minus WaitStart for records that lack it. A tier that runs the shared build, or a --copies tier whose lym prebuild runs before the copies, then prints only its slot wait. Leave WaitStart and Start as they are, since callSpan reads them. In _lym/landing/summary.md, rewrite the Design decisions sentence saying the tier's record already holds the build's wait, so it says the build's time is on the build line and not in the tier's wait. Add a gatecli test row for a shared-build tier asserting that the printed wait leaves out the build's duration."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/run.go#report
    summary: On a tier that is rerun after an environment failure, the only summary line printed is the rerun's, and report sets its WaitStart to Start, so it shows wait 0.0s. The slot wait the call spent sits only on the first run's record, which prints no line. In this case the call's output does not show the slot wait apart from the run time, though the board entry asks for it.
    fix: "Keep the rerun's record WaitStart equal to its Start, so lym gate report does not count the wait twice. Render the rerun's summary line with the slot wait the first run's record holds. One way is for retryTier's recordFirst callback to keep the first record's SlotWaitSeconds on the returned tierResult. TierSummaryLine then takes the wait as an argument, or report renders the line from a record copy that carries it. Update the change description's sentence that a rerun shows wait 0.0s. Add a test row for a rerun tier asserting that the line's wait is the first run's slot wait."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-affected-set
    location: internal/gateslot/runlog.go:46
    summary: tierSummaryPattern and tierDiscoveredPattern now require the wait field. FailedTier runs in internal/landingshed/verifygate.go over verify logs written by the lym that a shell-run lym gate test --wait command finds on PATH, which need not be the reading binary. A line written by a binary on the other side of this change matches neither pattern, so the landing failure record misses the failed tier and the build flag. This contradicts the change description's same-binary claim, and with it the two-way door and the gate-only blast radius.
    fix: "Make the field optional in both patterns: replace ` wait \\S+s wall ` with `(?: wait \\S+s)? wall ` in tierSummaryPattern (runlog.go:46) and tierDiscoveredPattern (runlog.go:49). In internal/gateslot/runlog_test.go, add a row to the FailedTier table and one to the TierDiscovered table, each holding a literal line without the wait field, such as `gate test: tier1 diff discovered 3 passed 2 failed 1 skipped 0 wall 1.5s exit 1 log x`, and assert that each is still read. In _lym/landing/summary.md, replace the Design decisions paragraph claiming the writer and readers ship in the same binary with one saying the patterns accept the line with or without the wait field, because verify logs are written by the lym on PATH."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/run.go#report
    summary: No test pins the wait figure the gate prints for a real tier run. TestTierSummaryLine renders only hand-built records, so a wrong WaitStart or Start derivation in runWithSlots, report, reportCarried, retryTier, the shared build or --copies passes every test. F1 and F2 are such defects.
    fix: "Add a table-driven gatecli test at the lowest tier that drives the verb's in-process tier run, the way the existing integration subtests in internal/gatecli/test_integration_test.go do. Give it rows for a slotted run with a held-slot delay, a carried tier, a shared-build tier and a rerun. Each row asserts the wait value on the printed summary line, taken from a literal or from the injected delay, never from SlotWaitSeconds. Keep the F1 and F2 rows in this test instead of adding separate ones."
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/gateslot/startedlines_test.go:36
    summary: The OtherLinesAreSkipped row still spells a tier summary line in the old form, with no wait field, so it no longer shows a line the gate prints.
    fix: "Change the literal at internal/gateslot/startedlines_test.go:36 to `gate test: tier0 diff discovered 1 passed 1 failed 0 skipped 0 wait 0.0s wall 0.1s exit 0 log x`."
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/gateslot/runlog_test.go:268
    summary: The FailedTier and TierDiscovered assertions inside TestTierSummaryLine repeat coverage that testFailedTier (runlog_test.go:161) and TestTierDiscovered (runlog_test.go:205) already hold, since both build their lines through TierSummaryLine.
    fix: "Delete the FailedTier and TierDiscovered checks at internal/gateslot/runlog_test.go:268-273, and drop the doc comment's second line, which describes them. TestTierSummaryLine then pins only the rendered line."
---

### [BLOCKING] The printed wait counts the shared build and the --copies prebuild

**Location:** `internal/gatecli/run.go#runWithSlots`

**Issue:** `runWithSlots` takes `waitStart` before `acquireSlot` (run.go:678).
`report` sets `record.WaitStart = waitStart` (run.go:729), and the line's wait is `Start - WaitStart`.
On the first tier that opts into the shared build, the body runs `runBuild` before `runTierChecked` (run.go:640-644).
The tier's `Start` is taken only once `runTierCommands` begins (tierrun.go:130), so the printed wait holds the slot wait plus the whole build.
The build already has its own summary line and `wall` (build.go:72-79), so a caller who adds the figures counts the build twice.
Under `--copies`, `runTierCopies` prebuilds lym (copies.go:64-79) before any copy takes its `started`.
`mergeCopies` keeps the earliest copy's `Start` (copies.go:135-137), so the prebuild lands in wait as well.
A run without copies does the same prebuild inside `runInvocations`, after `started` (tierrun.go:559), so there it lands in wall.
The help line the diff adds, "shows its slot wait as wait", and the package doc (doc.go:373) both promise the slot wait alone.
The change description's "the tier's record already holds that wait" claims the build is handled.
I rate this BLOCKING, not the fork's MEDIUM: the figure the task adds is wrong on a configured path, and its documentation says otherwise.
The --copies case is the same defect with the same fix, so it is folded in here as one finding about the method.

### [MEDIUM] A rerun tier's line hides the slot wait

**Location:** `internal/gatecli/run.go#report`

**Issue:** `retryTier` writes the first run's record through `recordFirst` with `WaitStart = waitStart` (run.go:649, tierrun.go:522) and prints no line for it.
For the rerun's record, `report` overwrites `WaitStart` with `Start` (run.go:730-733), so the one line printed reads `wait 0.0s`.
The call can still have waited 300 s for its slots.
A rerun is the long-call case where the board entry's caller most needs the split.

### [MEDIUM] The summary-line patterns refuse a line written by a binary on the other side of the change

**Location:** `internal/gateslot/runlog.go:46`

**Issue:** `verifyGate` calls `gateslot.FailedTier` on `result.Log` (verifygate.go:99, verifygate.go:394).
That log comes from a verify command, which `Verify` runs through the shell as `lym gate test --wait` (verify.go:334), so it is written by whatever `lym` is on PATH.
After an install, or under a pinned hub binary that differs from PATH, the writer and reader may straddle this change.
A line from the old writer has no `wait`, and the new pattern requires one.
A line from the new writer has `skipped 0 wait`, and the old pattern requires `skipped \d+ wall`.
Either way `FailedTier` finds nothing, the failure record loses `Build`, and the failing-tier reason falls to the generic path.
The change description's "The writer and readers of the line ship in the same binary" does not hold for this reader.
Its `two-way` door and `gate` blast radius leave out landing's verify-failure reporting.
`judgeDotnetRedFirst` reads output captured in-process, so it is not affected.

### [MEDIUM] No test pins the printed wait on any real path

**Location:** `internal/gatecli/run.go#report`

**Issue:** `TestTierSummaryLine` renders records the test builds by hand.
None of the code that sets `WaitStart` and `Start` before the line is printed is under test, so F1 and F2 pass every existing test.
The integration subtests in `internal/gatecli/test_integration_test.go` assert no wait value.

### [LOW] Fixture still spells the old summary line

**Location:** `internal/gateslot/startedlines_test.go:36`

**Issue:** The literal reads `... skipped 0 wall 0.1s exit 0 log x`.
The row only checks that the line is skipped, so it passes, but it no longer shows the line the gate prints.

### [LOW] TestTierSummaryLine repeats the readers' round-trip coverage

**Location:** `internal/gateslot/runlog_test.go:268`

**Issue:** `testFailedTier` and `TestTierDiscovered` already pass `TierSummaryLine` output through `FailedTier` and `TierDiscovered`, so the new test's checks of the same readers add no coverage.

## Rejected

- code-scope NIT on `internal/gatecli/report.go:146,160`, which flags `SlotWaitSeconds` replacing the report's inline formula as unrequested scope: the helper has two real call sites, and sharing it keeps the report and the summary line on one figure. The change follows from adding the line's wait and alters no behaviour.

Focus entry 1: of the paths it names, the shared build and `--copies` are wrong (F1) and so is the rerun (F2). The blocked tier prints no summary line. The carried tier, empty scope and inherited or unslotted call show a wait of about 0, which is correct.
Focus entry 2: settled by F3.
Focus entry 3: `startedlines_test.go:36` is the one stale literal (F5). The other test lines are built through `TierSummaryLine`.
Focus entry 4: the dropped `--copies` paragraph loses no fact. The flag help at `internal/gatecli/cli.go:132` and the package doc at `internal/gatecli/doc.go:44` state all of it, and the byte cap that commit a786a6b6a names forced the cut. Not a defect.
Focus entry 5: settled by F4 and F6.
