```yaml
lens: fix-consistency
outcome: reported
```

No findings.

Task: check the fix change set for consistency with its surroundings (lens fix-consistency).

- **`DobbyRuns` doc comment:** it now says the landing commit is "empty when none is readable". That agrees with the `LandingCommit` field comment in `passes.go:49` ("empty when none is recorded") and with `dobbyengine/landnotice.go:162`, which renders an empty commit as "landed, commit unknown".
- **`doc.go` twin:** `beamcli/doc.go:133` describes `DobbyRuns` as reading the commit through `shedrun.ReadLandedCommit`. It says nothing about error handling, so the new warn-and-continue behavior doesn't contradict it.
- **Warn convention:** the new `logger.Warn` call uses the same `"battencli: … "` message prefix and `"slug"`/`"error"` keys as the sibling warns in `dobbyreads.go` (lines 79, 90, 98).
- **Test rows:** the new `TestLandNotices` rows and fields are documented by the table's comments, and the `TestEntryDates` doc comment was updated for the new `notify_on_land` step. The other tests keep their behavior descriptions accurate. The one-test-per-behavior rule leaves the new first-try test beside `TestFinalize_RecordLandingFailureKeepsDone` as the right place.

Calls made:
1. `grep` for `ReadLandedCommit` and `LandingCommit` in non-test Go and markdown files: my context lacked the other non-test readers and docs that describe the landing-commit read.
