```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran the set searches for the findings whose premise is a set. I found one stale site.

**Findings**

- **LOW**, `internal/orchengine/doc.go:433`: the F5 fix left the package doc stale. Line 433 reads "one minute past a reset time still ahead, else 30 minutes on". After the fix, a reset time that has already passed wakes the orch at once, and only a missing reset time waits 30 minutes. The reviewer's `fix` key did not name `doc.go`, but the project's "Docs land in the same commit" rule covers it. The fixer updated the `recordUsageLimit` doc comment in `internal/orchengine/usagelimit.go` and not this copy.
- **No finding** on F7. The only `Inject` calls outside tests are at `watcher.go:632`, `watcher.go:793` and `usagelimit.go:68`, and all three pass `stepInjectTTL`.
- **No finding** on F6. The `time.Date` calls in `internal/shuttleengine` outside tests are all in `claudeengine/usagelimit.go`, and the no-month branch (lines 47 and 49) cannot name a day the month lacks. `claudeengine/doc.go:157` states no validity list, so it stays true.

**Left unchecked**

- F1, F2, F3 and F4 are single-site coverage gaps, not set premises, so I did not search them.
- I did not check that no other state-reset path besides `ResetForFreshLaunch` clears the three limit fields.
- I did not check `watcher.go` for the wake-at-once path. That path now sets `UsageLimitWakeAt` to `now`, which `tickUsageLimit` treats as due (`now.Before(...)` is false).

**Calls made**
1. `grep creel.Inject(` over `internal`, excluding tests: to rerun F7's set of inject sites, which my context lacked.
2. `grep time.Date(` in `internal/shuttleengine` and a `UsageLimit` file list: to rerun F6's set of date-building sites and to list the files that mention `UsageLimit`.
3. `grep` of `orchengine/doc.go` and `claudeengine/doc.go` for stale wake and reset wording: to find doc copies of the F5 and F6 behaviour, which my context lacked.
