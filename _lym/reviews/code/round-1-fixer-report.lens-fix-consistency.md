```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-round change set (`reedcli`, `reedengine`, `proc`) for stale twins and cross-references (lens fix-consistency).

- LOW — `internal/reedcli/doc.go:7`. The package doc still says "Each cycle it reads the OOM kill count of every strand pane's cgroup." The fix moved the read out of the discovery cycle into a per-session watch goroutine with its own backed-off cadence, and the watch now counts only panes in a `lym-pane-` scope (`oomnotice.go`, `planPaneOOMNotices`). The doc contradicts both points. It should say each entered session's watch reads the count for its capped panes.
- LOW — `internal/reedcli/entersession_integration_test.go:20`. The test's doc comment says it "starts no watcher". `enterSession` now always starts the OOM watch and only skips the name-repair loop. The assertion was changed to `repairsNames`, but this sentence was not.
- LOW — `internal/reedcli/watchdog_integration_test.go:455`. The comment says "a watchdog:off worktree enters the known set but starts no watcher". It is stale in the same way, and the changed assertion at line 515 now checks `repairsNames`.

Checked and consistent:
- `PaneScopeUnitPrefix` has no leftover references to the old lowercase name.
- `watchedSession` has no remaining uses of `oomKills`, `location` or a nil `cancel`.
- `enterSession` has no caller left on the old signature.
- Step 7 in the `runWatchdogLoop` doc was replaced by the paragraph after step 6.
- `CapProcessMemory`'s doc in `proc/processscope.go` still covers the new "scope ended" cause under "never moves the process".

Calls made:
1. `grep -rniE` for nil-cancel, no-watcher, `paneScopeUnitPrefix`, OOM and `watchdog: off` wording across `.go`, `.md` and `.yaml`. My context did not show stale wording in files outside the changed set.
2. `sed -n` of `internal/reedcli/doc.go` lines 1–14 and `internal/proc/processscope.go` lines 22–36. Neither was in my context in its current form.
