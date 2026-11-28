```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set check of the fix for F1, F2 and F3.

Findings left unchecked: none.

no findings

- **F1:** the dropped row's two assertions were the exit `gateslot.RedFirstGreenExit` and the envelope text `does not show the bug`. The new `Loosened-Check:` trailer names both and quotes the three-row figure against the two-row one. The stand-in and Go-path tests that pin these are untouched.
- **F2:** the premise concerns the one row that holds a passing test, and that row now has the `tests` needle.
- **F3:** the search found no other copy of the time-share claim in `internal/gatecli` or `_lym/landing/summary.md`.

Outside my lens: the `tests` needle shows that `RadiusIsOne` was selected, not that `dotnet test` ran it. The review's proposed fix goes no further than that.

Tool calls:
1. `grep` for time-share claims in `internal/gatecli` and the summary: my context did not show whether other copies existed.
2. `grep` for tests of the green exit, its message and this test's name: my context did not list the sibling pins.
