---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/redfirstdotnet_integration_test.go#TestGateRedFirst_Dotnet
    summary: Dropping the green row leaves no real dotnet run in red-first's exported checkout that exits 0, and the Loosened-Check cause measures serial cost that the diff's own parallel rows removed, not what a third parallel row would cost.
    fix: "In the `tests` table of TestGateRedFirst_Dotnet, add back a third parallel row (for example `ATestThatPassesOnTheBaseExitsWithTheGreenCode`, testMethods holding only `public void RadiusIsOne() { Assert.That(new Circle().Radius, Is.EqualTo(1)); }`) with a per-row `wantExit` field set to `gateslot.RedFirstGreenExit` and wantContains `does not show the bug`; replace the loop's `code != 0` check with `code != tt.wantExit` and set wantExit 0 on the two red rows. If a measured three-row parallel run shows the row costs wall time, keep the drop instead, and land a commit whose `Loosened-Check:` trailer names the dropped exit and envelope-text assertions and quotes that three-row parallel figure against the two-row one."
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/redfirstdotnet_integration_test.go:98
    summary: Row OnlyTheTestThatFailsOnTheBaseIsNamedRed still passes if RadiusIsOne is never selected or run, so it does not show that a real passing test was judged not red, as the summary claims.
    fix: "Add the needle `\"tests\":[\"Fixture.Tests.CircleTests.AreaIsPi\",\"Fixture.Tests.CircleTests.RadiusIsOne\"]` to that row's wantContains, beside the red_tests needle."
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-scope
    location: internal/gatecli/redfirstdotnet_integration_test.go:27
    summary: The new doc-comment line saying the cold build is most of a row's time rests on no measured breakdown; the summary gives whole-row totals only.
    fix: "Change the line to `Each call builds the solution cold in a fresh checkout of the base.`, dropping `which is most of a row's time`, and start the next line `So every row has its own hub and runs in parallel.` unchanged; or add a per-phase timing of one row (restore, build, test run) to `_lym/landing/summary.md`'s Measurements section and keep the line."
---

### [MEDIUM] No real green red-first run remains, and the loosening's cause does not measure the drop

**Location:** `internal/gatecli/redfirstdotnet_integration_test.go#TestGateRedFirst_Dotnet`

**Issue:** The removed row was the only one whose real `dotnet test` tier exited 0 in red-first's exported checkout, which is the only path to `redFirstGreen` in `judgeDotnetRedFirst` (`internal/gatecli/redfirstdotnet.go:313-318`).
Both remaining rows hold a failing test, so a defect on the real path that turns an all-passing run into a non-zero exit is no longer caught there.
That path includes the checkout-rooted, `noCarry` run, the shared `--no-build` build in a non-git checkout, and the stray-file check.
`TestDotnetRedFirstCallRun_…/ATestThatPassesOnTheBaseExitsWithTheGreenCode` (`redfirstdotnet_test.go:177-178`) stands in for the tier run, and `TestGateTest_DotnetFixture` runs in a git repository with no checkout root, so neither holds this behaviour on the same path.
The `Loosened-Check:` trailer on f8de6d0f7 names the dropped row.
Its cause is "31.0 to 47.5 s for three such builds in series", but the same commit makes the rows parallel, so that figure no longer measures what the third row costs.
The summary's own table shows wall time set by the longest row (about 13.6 s).

### [LOW] The red row does not show that RadiusIsOne ran

**Location:** `internal/gatecli/redfirstdotnet_integration_test.go:98`

**Issue:** `red` is built from failed names only (`redfirstdotnet.go:321-327`), so if the selection dropped `RadiusIsOne`, `red_tests` would still be exactly `["…AreaIsPi"]` and the row would pass.
The envelope's `tests` key (`redfirst.go:227`) holds the selected labels, and no needle asserts it.
The summary's "a real passing test still has to read as not red" therefore holds only when the test was selected.

### [LOW] Unmeasured claim about where a row's time goes

**Location:** `internal/gatecli/redfirstdotnet_integration_test.go:27`

**Issue:** "which is most of a row's time" is stated as found, while the only figures are whole-row totals in the summary's Measurements table.
The board entry asks to find where the time goes, and no figure splits a row into hub copy, restore, build and test run.

## Rejected

- Shared state between parallel rows (focus 3): each row has its own hub, so its own gate-slot pool under its board dir (`internal/gateslot/pool.go:118`) and its own `lymtemp.MkdirTemp` checkout.
  `files` is only read after the loop starts.
  The MSBuild node and the NuGet global-packages folder were already shared with `TestGateTest_DotnetFixture` running beside it, and both support concurrent use, so they are no defect.
- Measurements (focus 5): the before and after runs are one each at comparable load (40.4 and 41.7), the table shows the row count changing from three to two, and the package figure agrees with the claim that `TestGateTest_DotnetFixture` now sets it.
  No defect beyond F1's point about the trailer.
- Merge danger (focus 6): the diff changes one test file, so a `two-way` door with blast radius gatecli is borne out.
  `//lym:only upkeep`, `tiergate.Require(t, 1)`, `t.Parallel` on test and rows, and the direct `*hubforge.Hub` use follow PATTERN-test-speed and PATTERN-test-isolation.
- Scope (focus 4): the board entry offers sharing a built fixture only as an example.
  The summary's reason for not sharing (each call exports its own fresh base checkout, `redfirstdotnet.go:90-96`) is borne out.
