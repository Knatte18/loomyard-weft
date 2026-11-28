```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the fix's change set (the new `tests` needle, the doc-comment edit and the commit's measured trailer).

**LOW: the new `"tests":[…]` needle in `redfirstdotnet_integration_test.go`, row `OnlyTheTestThatFailsOnTheBaseIsNamedRed`. It pins that `RadiusIsOne` was selected, not that it ran.**
- F2's summary says the row passes if `RadiusIsOne` is "never selected or run", and only the first half is closed.
- `labels := dotnetLabels(selectable)` (`redfirstdotnet.go:108`) is computed before, and independently of, `run.terms = filterTerms(group.tests)` (line 122).
- Concrete case: a `filterTerms` or tier-filter defect drops the `RadiusIsOne` term, so only `AreaIsPi` runs and fails. `red_tests` is `[AreaIsPi]`, `tests` still lists both, the exit is 0 and the row passes.
- The doc comment's "one row holds both a test that fails on the base and one that passes there" and the landing summary's "a real passing test still has to read as not red" therefore still rest on an unasserted run.
- The fixer-report should state this limit rather than claim the row shows the test ran.
- The commit body's "fails when the passing test is not selected" is accurate.

**LOW: the commit's `Loosened-Check:` trailer states "1-min load 28 to 52", which matches none of the recorded readings' lower bound.**
- The quoted ranges (18.1 to 23.6 s, 12.5 to 14.2 s) are exactly the six alternating runs.
- Those runs started at loads 34.32, 35.71, 46.82, 44.77, 51.56 and 49.54, so the range is about 34 to 52.
- The earlier single three-row run started at 27.49, which also does not round to 28.
- Rewriting history is not allowed, so correct the figure in the fixer-report.

Checked with no finding:
- The needle's JSON shape matches `redFirstVerdictExit`'s `"tests": labels`, and it did pass in a real run.
- With `RadiusIsOne` unselected, the array would be `["…AreaIsPi"]` and the needle would fail.
- The shortened doc-comment line and the "For the same reason" sentence after it still read correctly.
- The final diff has no leftover `wantExit` field or `gateslot` import.
- The quoted time ranges match the measured longest rows.

Tool calls made: none.
