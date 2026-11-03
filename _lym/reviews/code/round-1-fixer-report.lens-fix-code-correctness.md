```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix-code-correctness lens over the F1, F3, F5 and F6 change set, running each added or changed branch step by step.

No findings. I made no further tool calls.

- **envkit stub:** `LandingDeps` now fills `DriftVerdict` and `nilLegal` no longer lists it, so `NilSeams(FullEnv)` stays clean. `wire` leaves `Env.Landing` nil, so `NilSeams(c.env)` does report `Landing.DriftVerdict` as nil, and the new `intentionallyNil` entry covers it.
- **`TestLandingDeps_DriftVerdict_RendersTheScopeRecord`:** a nil seam would panic, and a closure reading the wrong anchor would return the empty section, so the test fails either way. It reads the record at call time, so the empty answer before seeding and the rendered section after are both correct.
- **`TestWire_DescribeTableCarriesTheDriftVerdict`:** if the `DescribeTable` closure passed `""`, `drift_verdict` would hold the no-regression-check line, so the exact-string assertion fails. The closure and the seeded record use the same `location.AnchorPath()`, and the test passed at tier 1.
- **Finalize merge row:** `Finalize` sets `summary.Body = withSection(...)` and `bodyWithCredit` then composes the merge message, so the expected string is body, section, credit. If the section were left out of the body, the row would fail.
- **`TestWithSection`:** with the whitespace-only branch removed, `"" + "\n\n" + section` would differ from the section alone, so that row fails. The other two rows pin the empty-section and trimmed-body cases.
- **`ReentryRefreshesTheBodyWithTheRegressionCheckOnce`:** the record is found, not review-ended and at a different gated head, so the call reaches `api.Edit` with the section once. Exact `cmp.Diff` on the calls fails if the section is missing or doubled.
- **`AFailingRegressionCheckReadWhileBuildingTheSpecIsAnErrorWithNoSessionStart`:** the first read passes in `openPullRequest` and the second errors in `sessionSpec`, reached through `ensureSession`. If that error were ignored, a session would start and the starts count would be 1, not 0.
- **F6:** `captureAt("base")` makes `BaselineHead` and `ComparedHead` differ. Swapping the `recordCompare` arguments, or passing `told.BaselineHead` where HEAD belongs at `verify.go:545`, now fails the passing-compare, agent-Done and final-failing rows. The only changed signature is `requireCompareRecorded`, and tier 0 compiled and passed, so no caller was missed.

Tool calls: none.
