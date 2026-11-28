```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens on the `TestGateRedFirst_Dotnet` diff. For each behaviour it changes, I name the row that would fail if that behaviour broke.

**Per behaviour, the test that pins it:**

1. **A test that fails on the base is named red over a real dotnet run.** Pinned by `OnlyTheTestThatFailsOnTheBaseIsNamedRed`. Its exact `"red_tests":["Fixture.Tests.CircleTests.AreaIsPi"]` substring fails if AreaIsPi is missing.
2. **A test that passes on the base, run beside a red one, is not named red.** Only partly pinned, by the same row. It catches RadiusIsOne being judged red, but only when RadiusIsOne was selected. Nothing asserts that it was selected and run (finding 2).
3. **Two filter terms combine into one real `dotnet test --filter`.** Pinned by the same row. AreaIsPi would not run or be named red if the combined filter selected nothing.
4. **A test that reads a member the change adds counts as red, through a real compile error.** Pinned by `ATestThatReadsAMemberTheChangeAddsIsRedSinceItDoesNotBuildOnTheBase`, which checks `red_tests`, `"build_errors"` and `Perimeter`.
5. **Exit 0 and a valid JSON envelope on the last line.** Pinned by the shared exit check and `json.Valid` in both rows.
6. **Every changed test passes on a real dotnet run: exit `gateslot.RedFirstGreenExit`, "does not show the bug".** No real-run row pins this any more.
   - The verdict and the message are pinned with stand-in tiers by `TestDotnetRedFirstCallRun_RunsTheChangedTestMethodsInACheckoutOfTheBase/ATestThatPassesOnTheBaseExitsWithTheGreenCode` (`internal/gatecli/redfirstdotnet_test.go:177-178`).
   - Real vstest counts are pinned by `TestGateTest_DotnetFixture` (`internal/gatecli/dotnetfixture_integration_test.go:68`). That test runs in a git repo, not through red-first's exported checkout.
   - Nothing pins that a real all-passing run in red-first's checkout comes out of the tier run with exit 0 (finding 1).

**Findings:**

1. **MEDIUM, design.** Location: `internal/gatecli/redfirstdotnet_integration_test.go:92-99`, the row `OnlyTheTestThatFailsOnTheBaseIsNamedRed`.
   - **Summary:** No row now runs a real dotnet tier that exits 0 in red-first's checkout, so a real green run being misjudged is no longer caught.
   - **Why it matters:** Both remaining rows hold a failing test, so the real tier always exits non-zero. That exit hides anything on the real path that turns an all-passing run into a non-zero exit.
   - **Paths not checked in this review:** the checkout-rooted run in `internal/gatecli/run.go:101-102` with `noCarry` (`redfirstdotnet.go:123-124`), the shared `--no-build` build in an exported, non-git checkout, and the stray-file check (`tierrun.go`, `runTierChecked` → `checkStrayFiles`). A defect in any of them would make the result unjudged or exit 1 instead of `RedFirstGreenExit`.
   - **Why the other tests miss it:** the stand-in row at `redfirstdotnet_test.go:177` never runs those paths, and `TestGateTest_DotnetFixture` (`dotnetfixture_integration_test.go:58`) runs in a git repo with no `checkoutRoot`.
   - **Evidence:** the removed row in the diff at lines 136-143, and `judgeDotnetRedFirst` at `redfirstdotnet.go:313-318`, where only an exit of 0 reaches green.
2. **LOW, design.** Location: `internal/gatecli/redfirstdotnet_integration_test.go:98`.
   - **Summary:** The row holds even if RadiusIsOne was never selected or run, so the passing-beside-red behaviour it claims is only half pinned.
   - **Why:** If the change detection dropped RadiusIsOne, `red_tests` would still be exactly `["…AreaIsPi"]`. The envelope's `"tests"` key, built from `labels` in `redfirst.go:227`, is not asserted.
   - **Independent value to assert:** `"tests":["Fixture.Tests.CircleTests.AreaIsPi","Fixture.Tests.CircleTests.RadiusIsOne"]`, a literal taken from the order of the methods in the fixture file.

**Tool calls:**
1. Grepped `checkoutRoot`, `noCarry` and the gateslot tests that cover the summary line, to see which production paths a checkout-rooted tier run takes. The manifest did not hold `run.go`, `tiers.go` or the gateslot tests.
2. Read `internal/gatecli/run.go:85-130` and grepped `runTierChecked` for the stray-file check, to confirm the real green path passes through code no remaining row runs with exit 0. Neither was in the manifest.
