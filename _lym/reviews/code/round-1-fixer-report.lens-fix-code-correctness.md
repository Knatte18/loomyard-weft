```yaml
lens: fix-code-correctness
outcome: reported
```

Lens fix-code-correctness: I found no BLOCKING or MEDIUM defects, and one LOW.

**Traces**
- **F5, `recordUsageLimit`:** the reset is 1h in the past, so `wake` = reset+1m is not after `now`, and `UsageLimitWakeAt` = `now`. The caller at `watcher.go:337-338` runs it only when a new turn end arrived (`lastTurnEnd != nil`). A standing limit therefore doesn't re-arm `now` on every tick, and a later tick sends the wake line and moves the wake to `now`+30m. A zero reset gives `now`+30m, and a future reset gives reset+1m, both as before.
- **F5 test:** `TestWatcher_UsageLimitReadAfterItsResetWakesTheOrchAtOnce` fails with the old code, which left the wake at `now`+30m and so sent nothing.
- **F6, `usageLimitResetsAt`:** `Feb 31, 2pm`, written 2026-10-28, gives Mar 3 2026. That is not after the write time, so it rolls to Mar 3 2027. Its day is not 31, so it returns the zero time. A valid date such as `Oct 30` passes the check, since `time.Date` returns day and month in the same zone. The new row fails without the check, because it would yield Mar 3 2027.
- **F7:** `stepInjectTTL` equals creel's default of 10m, so behaviour is unchanged. The fake creel records the TTL and no test asserts it, which is fine.
- **F4 tests:**
  - Render failure: `stencilkit.Remove` runs after the watcher is built, and `RenderUsageLimitWake` reads the stencil at render time. The wake defers to `now`+30m and nothing is sent, and the test would fail without the deferral branch.
  - Inject failure: `failNext("inject")` is consumed by the wake inject, because the colour inject happened in earlier ticks. `creelFailed` leaves `UsageLimitWakeAt` unchanged and the next tick sends, and the test would fail if the failure moved the wake.
- **F3:** The idle subtest sets and asserts all three fields. `ResetForFreshLaunch` must clear them for the test to pass.
- **F2:** The first row would fail if the copy dropped `UsageLimit`. The second row fails if the limit were copied from a turn end before the prompt offset.

**Findings**
- **LOW**, `internal/battenshed/notice_test.go`, `TestNotice_UsageLimitOfOneRunOutranksTheOthersAndStandsWhileItReadsLimited`: the limited run is always at index 1 and the other run is always unlimited. A regression to `stamp.usageLimited = reading.runs[0].UsageLimited` would make the standing-key check fail in that test. A regression that checks only the last run would still pass. The `AnotherRunNotLimited` case therefore pins the any-run loop only partly. Adding a row with the limited run at index 0 would close it. This is not an incorrect result, so I treat it as a minor coverage point.

**Tool calls**
1. `grep` of `recordUsageLimit`/`tickUsageLimit` in `watcher.go`: my context lacked the caller, and I needed it to check that the new "wake at once" value isn't re-armed on every tick.
