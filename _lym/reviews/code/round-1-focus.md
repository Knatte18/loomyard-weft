---
round: 1
focus:
  - "internal/gatecli/redfirstdotnet_integration_test.go, row OnlyTheTestThatFailsOnTheBaseIsNamedRed: settle whether its assertions show that RadiusIsOne was selected, run on the base and judged not red, or whether they would also hold if that test never ran, and compare this with what the removed ATestThatPassesOnTheBaseExitsWithTheGreenCode row checked."
  - "The Loosened-Check trailer on f8de6d0f7 and the matching claims in _lym/landing/summary.md: settle whether the trailer names the dropped assertions with a measured cause, and whether the tests the summary says still hold the green exit and the real vstest counts do hold the same behaviour no looser (internal/gatecli/redfirstdotnet_test.go, the dotnet fixture test). Check every removed assertion, the envelope text included, not only the exit code."
  - "The parallel rows: settle whether concurrent red-first calls and concurrent dotnet restore and build runs share any state that can make a row fail or wait (the files map, gate slots or locks, the checkout export location, process environment, the MSBuild or compiler server, NuGet caches), reading the red-first production path and not only the test."
  - "Scope against the board entry, which asks to find where TestGateRedFirst_Dotnet spends its time and cut it: settle whether the diff does that, and whether the summary's and doc comment's statements about where the time goes and why a built fixture is not shared are borne out by the code."
  - "The Measurements section of _lym/landing/summary.md and the figures in the commit message: settle whether each improvement claim has comparable before and after figures from the current code, names the set-up differences, and agrees with itself and with the rows the diff leaves."
  - "The summary's Merge danger call and the repository's constraints document (testing rules for parallel tests, hub fixtures, upkeep markers and comments): settle whether the door, the blast radius, the changed doc comment and the table test comply."
---

Seed focus from the decision record, which holds the board entry alone, and the one-file test diff over the review range.
