```yaml
lens: fix-affected-set
outcome: reported
```

No findings. The fixes cover every site in each finding's set.

- **F1:** `adoptGateTiers` has one call site, in `StartSuite` at `internal/adoptcli/deps.go:158`. The new `TestAdoptGateTiers` covers the function's return paths. The `StartSuite` line that assigns `Tiers` is still unpinned, because `startSuite` errors under `go test` before that line is reached. The review's own `fix` text does not ask for that test, so I am not counting it as a finding.
- **F2:** `checkFixtureNow` appears only in the `tick` helper, in `checkFixtureAfter` and in the `starting` mark of `TestRunCheckMarks`. Every `deps.Now()` call in `runCheck` (`internal/adoptcli/check.go:144`, `:170`, `:187`) goes through the advancing clock. Nothing else in the package depends on the old constant clock, and `check.go:308` uses the real `time.Now` outside tests.
- **F3:** `adoptFields` is the only place that builds the `message` and `suite_warning` fields for the adopt envelope. Elsewhere in `internal/`, `"message"` appears only in other modules' own envelopes.
- **F4:** The only other tests that touch the check mark are `checkmark_test.go` and the sites inside `TestBar_LineTwoShowsTheCheckMarkBeforeTheClock`. `checkmark_test.go` pins the tmux set-option arguments, not the mark's position on the line. So the new before-the-clock assertion has no twin to update.

Tool calls made:
1. A grep of `internal/adoptcli/*.go` for `checkFixtureNow`, `newCheckFixture`, `deps.Now` and `Now:`, plus a grep across `internal/` for `"suite_warning"`, `"message"`, `adoptFields` and `prime_recorded`. I made it to find every clock reader and envelope builder the F2 and F3 fixes had to cover, which my context did not list.
2. A grep of the reedengine and reedcli tests for `adopt check`, `SetCheckMark` and `@lym_check`. I made it to find other tests asserting the mark's placement that F4 would need to match.
