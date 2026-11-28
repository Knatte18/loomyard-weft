```yaml
lens: fix-consistency
outcome: reported
```

Task: checking that the fix change set agrees with its surroundings (lens fix-consistency).

1. **LOW.** Changed text: the new commit's `Loosened-Check:` trailer, "over alternating runs at 1-min load 28 to 52".
   The load range does not match the runs it names.
   The six alternating runs started at loads 34.32, 35.71, 46.82, 44.77, 51.56 and 49.54, so the range is about 34 to 52.
   The 27.49 reading belongs to the first, separate three-row run (18.22 s), which was not part of the alternation.
   The row timings quoted (18.1 to 23.6 s against 12.5 to 14.2 s) do match the output.

2. **LOW.** Changed text: the `TestGateRedFirst_Dotnet` doc comment in `internal/gatecli/redfirstdotnet_integration_test.go`, "Each call builds the solution cold in a fresh checkout of the base."
   The two sentences after it, "So every row has its own hub and runs in parallel." and "For the same reason, one row holds both…", rested on the removed clause about cost.
   A cold build per call no longer says why rows run in parallel, or why two tests share one row, because nothing now says the build is costly.
   The review's proposed wording caused this.
   Saying that each call costs a cold build, without claiming its share of the row's time, would keep the inference and claim nothing unmeasured.

3. **LOW.** Sibling left as it was: `_lym/landing/summary.md`, Design decisions: "The dropped row asserted the `gateslot.RedFirstGreenExit` exit code over a real dotnet run."
   The new trailer says the dropped row also asserted "does not show the bug" in the envelope.
   The summary names only the exit code, and gives no three-row against two-row figure for keeping the drop.
   The fixer may treat this file as outside its write surface.

Nothing else disagrees:
- The added `tests` needle makes the summary's "a real passing test still has to read as not red" hold.
- The row count is unchanged, so the summary's Measurements table and the "each row adds tests of its own" line still agree.

Tool calls made: none.
