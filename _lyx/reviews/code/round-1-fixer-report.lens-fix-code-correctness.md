```yaml
lens: fix-code-correctness
outcome: reported
```

No findings of BLOCKING or MEDIUM severity; two LOW findings.

Lens fix-code-correctness: checked the code and tests the fix changed in `smoke_helpers_test.go`, `startup_test.go` and `overlay_test.go`.

**LOW — `overlay_test.go`, new subtest "runStdin hands a retry the same stdin"**
- The test passes with the defect in place. Its path goes through `execHook`, and `runStdin` (`overlay.go:126`) passes the captured `stdin` slice on every try.
- The defect the review names is a non-hook retry reusing a consumed reader. That lives at `overlay.go:134`, where `bytes.NewReader(stdin)` is built inside the attempt closure.
- The hook path cannot reach it, so the test only pins that the slice is forwarded.
- Dropping the per-attempt `NewReader` rebuild would not fail the test. The reviewer's proposed row has the same limit, so this is a gap in the proposal, not a regression.

**LOW — `smoke_helpers_test.go:101-102`, `providerlessShuttleConfig`**
- The new `strings.Replace(cfg, "start_retries: 1", "start_retries: 0", 1)` does nothing if the template line drifts, for example to `start_retries: 2`. The fixtures would then quietly retry again.
- The sibling `claudeConfigWithBinary` panics on drift. The existing `startup_timeout_s` replace is silent in the same way, so this matches the surrounding code.

**Checked and correct**
- **"Dies on every launch" row:** each attempt consumes `live` then `dead`. The last attempt's error names exactly one run directory, and the 5 s retry sleep happens once.
- **"Timed out" row:** with a 3600 s startup window and a 5 min run deadline the outcome is `OutcomeTimeout`, so `startRetryable` is false and there is one launch.
  - Dropping the `OutcomeDied` check makes `startRetryable` true there, since the strand was removed. The row then sees two launches and fails.
  - Without the 3600 s override the startup window expires through the tick cap, which classifies it as died and retries. That is why the first run of the row failed.
- **Removal-wait suffix assertions:** the removal waits are the last sleeps, because `start` returns right after teardown without a retry sleep. `1 s` then `2 s` matches `removeStartupStrand`.
- **`retryWaits` count:** it only counts 5 s sleeps, and the poll backoff never hits 5 s with this config. The gate run passed.
- **`awaitClock`:** it embeds `*fakeClock`, so it still satisfies `AwakeClock`.
- **Doc comment move:** `keyTimesReed` is restored with its own comment.

Further tool calls (1):
- Read `internal/reedengine/overlay.go:66-135` to confirm where `runStdin` builds its stdin reader. My context had only the test, not the production code.
