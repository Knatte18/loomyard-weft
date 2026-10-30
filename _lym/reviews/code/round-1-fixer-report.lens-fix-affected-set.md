```yaml
lens: fix-affected-set
outcome: reported
```

Lens: fix-affected-set. I re-ran each set-premised finding's own search. Findings F1, F2, F7, F8 and F9 are coverage gaps, so they have no set to check.

**Covered, no gap**
- F3: `RespawnCapReached` has one non-test caller, `internal/battencli/dobbynotice.go:88`, and it already passes the new `statusModified` argument. The only other callers are the updated `internal/battenshed/innerrun_respawncap_test.go:92,95`.
- F4: no `RespawnCapped` and no dobbyengine `respawnCap` remain. The `respawnCap` uses left are all in `internal/battenshed`, where the constant belongs.
- F5: `runShedDuration` has two callers, the probe and the quiet window, and both get the non-positive fallback. The recipe sets 30, and `innerRunEntry` only validates non-negative. Its doc, `entries_batten.go:89`, says exactly that, so nothing is stale.
- F6: `resetText` and `resetStamp` are gone from `internal/battenshed`. The only remaining definitions and callers are in `internal/dobbyengine/runnotice.go:295,297,441-450`.

**Findings**

LOW — `internal/battenshed/doc.go:89`
- Summary: the doc line is now stale about `RespawnCapReached`.
- Text: "RespawnCapReached reads the respawn count for a reader outside the row, which names the cap in the driver-dead notice of a running child."
- Evidence: after F3 the function reports the capped state, not the count. It reads false when the status file was written after the last respawn, and it takes a `statusModified` argument. The doc line was not touched by the fix.

**Tool calls**
- One Bash call ran four greps: `RespawnCapReached`, `RespawnCapped` and `respawnCap`, `notice_quiet_min` and `defaultNoticeQuiet`, and `resetText` and `resetStamp`. My context held only the files the fixer read, not a repo-wide view of callers and docs.
