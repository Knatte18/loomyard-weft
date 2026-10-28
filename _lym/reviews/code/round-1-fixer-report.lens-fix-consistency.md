```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-consistency lens (twins and docs the changed text depends on).

**Findings**

1. **MEDIUM** — `internal/orchengine/doc.go:432-433` was left stale by F5.
   - The doc says the wake is "one minute past a reset time still ahead, else 30 minutes on".
   - `recordUsageLimit` now wakes at once for a reset that has already passed. It waits 30 minutes only when no reset time is stated.
   - The fixer updated the `recordUsageLimit` doc comment but not this module doc, which is what CLAUDE.md requires to land in the same commit.
   - Suggested wording: "one minute past the reset time, at once when that has passed, and 30 minutes on when the limit states no reset time".

2. **LOW** — the `doc.go` sentence at `internal/orchengine/doc.go:374` is about step injects, so it does not cover the wake inject.
   - It says "Every step inject carries `stepInjectTTL`". F7 makes the wake inject carry it too, so nothing contradicts it.
   - I found no wake-ttl claim to correct.

3. **LOW** — the reset-clause docs in `internal/shuttleengine/claudeengine/doc.go:156-158` do not mention the F6 behavior.
   - They say a text with no readable clause gives no reset time. They do not say that a clause naming a day the month lacks does too.
   - The `usageLimitResetsAt` doc comment was updated for this, so only the module doc lags.
   - This is optional.

4. **No finding** — `ResetForFreshLaunch` (`internal/orchengine/state.go:463`) and `doc.go:438` still agree with the F3 test.

5. **No finding** — the `usagelimit.go` header, the `tickUsageLimit` comment and the `State` field comments agree with the new wake logic.
   - `tickUsageLimit` still moves the wake `usageLimitRetry` on after sending.
   - `UsageLimitWakeAt` is documented as "when the watcher next types the wake line", which holds for the "at once" case.

**Calls made**
1. `grep` over `doc.go` and `.md` files for wake and limit wording, to find docs that restate the wake timing or ttl; my context held only the code files.
2. `sed -n 430,437p` of `internal/orchengine/doc.go` plus `sed -n 154,160p` of `internal/shuttleengine/claudeengine/doc.go` and a `grep` of `state.go`, to read the exact doc text and the `State` field comments; the doc files were never in my context.
