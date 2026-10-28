```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, the test that pins it, or none.

**Per-behaviour list**

1. Reset-time parse (`usageLimitResetsAt`): pinned by `TestUsageLimitResetsAt` (`usagelimit_test.go`).
   - Rows cover time with and without minutes, month and day, rollover to the next day and the next year, 12am, the older "reset at" wording, an unknown zone, a missing clause, an hour out of range, and an unparseable timestamp.
   - No row covers a DST change. The behaviour is `time.Date` in the named zone, so this is not a gap for a finding.
   - There is no row for minute > 59. The regex `\d{2}` allows it and the code rejects it, so a break of that guard would go unnoticed. This is LOW.
2. The `newestTurnEndError` limit reading, `rate_limit` classification only: pinned by the "a usage-limit turn end reads the limit and its reset time" row in `TestTurnEndActivity`.
   - The existing `apiError` row, a plain API error with no `error` field, still asserts a zero `UsageLimit`. The comparison in `TestTurnEndActivity` now checks `Limited` and `ResetsAt`, so that row fails if a plain error reads as limited.
   - A limited entry in a sidechain, or one older than a normal turn end, reads as none. The `sidechainErr` row and the `normalTurnEnd` row cover this only for plain errors. This follows the unchanged scan logic and needs no new row.
3. `readRunActivity` copies `UsageLimit` only for an own turn end (`shuttleengine/activity.go`): none.
   - I saw no test of the `ownTurnEnd` gating for `UsageLimit`. A turn end before the prompt offset must not carry a limit into the run's reading. Whether an existing test pins the same gating for `APIError` is not visible in my context.
   - MEDIUM, design.
4. `Runner.UsageLimit` (`session.go`), including the engine-without-capability case and the `FindRun` error: none.
   - `fakeSession` in `watcher_test.go` stubs it, so no test exercises the real method.
   - It is a thin delegate, so this is LOW.
5. `battencli/wire.go` mapping of `reading.UsageLimit` onto `AgentActivity`: none.
   - LOW. A mis-copied field would not fail any test I can see.
6. Batten usage-limit notice: pinned by `TestNotice_UsageLimitNotifiesOncePerLimitWithoutARevive`.
   - One notice across repeated activity moves, no revive, the readable and unknown reset text, and the end of one limit followed by a new one are covered.
   - Not covered:
     - A usage-limit run takes precedence over an API error and a quiet notice. The test uses only a limited run.
     - Key retention across several runs, where one run stays limited while another does not. This is the "any run" logic in `keepEpisode`.
     - An API-error run on the same child still notices as before. The existing `TestNotice_APIErrorStall` still pins the plain case.
     - A usage-limit key staying when the activity is unreadable (`known` false).
   - MEDIUM, scope: the "any run" and precedence behaviours have no row.
   - The `resetStamp` key distinguishing two limits is covered only by the "later limit" step. If the reset time were unknown for both limits, the second limit would not get its own notice, which is the unknown-time case. This is LOW and I treat it as a design note.
7. Watcher records the limit, holds the orch, starts no cycle, and wakes at reset plus one minute or after 30 minutes with no reset time: pinned by `TestWatcher_UsageLimitHoldsTheOrchAndWakesItAfterTheReset`.
   - Both rows are covered. They also cover the repeat 30 minutes later, the clearing on a non-limit turn end, and the release.
   - The `usage` field is set above the hard threshold, so the "no cycle while limited" claim is exercised.
8. Watcher paths other than the happy path in `tickUsageLimit`: none.
   - Not covered:
     - A render failure, which warns and moves the wake on.
     - A creel failure through `creelFailed` that leaves the wake due.
     - `renewHold` returning not held.
     - A reset time already past, so the wake falls back to the retry.
     - The record surviving a restart, since the state is persisted.
   - LOW to MEDIUM. Only the render-failure and creel-failure rows pin behaviour the doc comment promises. I rate it MEDIUM, design.
9. `ResetForFreshLaunch` clears the three limit fields: none.
   - I saw no change to the existing fresh-launch test. MEDIUM, design.
10. Tolerant read of an old state record (`PATTERN-tolerant-records`): none.
    - The fields are plain `time.Time` with JSON tags. An existing test may cover tolerant reading generally.
    - LOW.
11. Wake stencil render, `RenderUsageLimitWake`: pinned by the two rows in `TestRenderPointers_OneLineNamingTheirPaths` (reset known and unknown) and the multi-line-override failure row.
    - The stencil's static prose is exempt.
12. `recordUsageLimit` keeps the first sight's `UsageLimitAt` across ticks, and a later turn end that is limited moves the wake: none directly. The happy-path test does not feed a second limited turn end.
    - LOW.

**Findings**

- MEDIUM, design. `internal/shuttleengine/activity.go` (`readRunActivity`, the `ownTurnEnd` branch). No test shows a turn end before the prompt offset leaves `UsageLimit` zero.
- MEDIUM, scope. `internal/battenshed/notice.go` (`judgeAgents`, `keepEpisode`). No row covers a usage-limit run winning over an API-error or quiet run, or the key standing while another run is not limited.
- MEDIUM, design. `internal/orchengine/usagelimit.go` (`tickUsageLimit`). The render-failure and creel-failure paths have no test.
- MEDIUM, design. `internal/orchengine/state.go` (`ResetForFreshLaunch`). The clearing of the new fields has no test.
- LOW, scope. `internal/shuttleengine/session.go`, `internal/battencli/wire.go`. The `Runner.UsageLimit` delegate and the wire mapping are untested.
- LOW, scope. `internal/shuttleengine/claudeengine/usagelimit_test.go`. No minute > 59 row.

**Tool calls:** none. My context held the diff and the tests.
