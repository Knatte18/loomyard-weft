```yaml
lens: fix-affected-set
outcome: reported
```

Task: run the fix-affected-set lens over the F1–F3 fixes.

Unchecked findings: none. I reran the search for each finding with a set premise.

- **F3, sites carrying the recovery text:** a repo-wide grep for the old text ("no longer reads running", "waiting process gone") and for `waiterlessRecovered` and `waiterlessRuns` finds no stale copy. The only users of the constant are `activity.go:86`, `activity.go:93` and `activity.go:125`, plus the test, which reads the constant. The `doc.go:426` sentence ("Info once it reads live again or no longer reads running") describes the episode and does not quote the message, so it is unaffected.
- **F2, sites that leave `ReadAgentActivity` without ending the episode:** the loop has four exits.
  - The `loadRunState` error exit is intentional: an unreadable record is a different episode, and the waiterless episode stays open until the record reads again.
  - The `!found` exit now recovers (`activity.go:86`).
  - The not-live exit goes through `noteWaiterlessRun`, which recovers when the record is not running with a recorded pid (`activity.go:125`).
  - The live exit recovers at `activity.go:93`.
  - Each exit that can end an episode now does, so the set is covered. Run directories skipped for `!entry.IsDir()` never opened an episode.
- **F1, test coverage:** the subtest now covers the three recovery paths: live again, terminal outcome (INFO count 2) and vanished record (INFO count 3). `run.json` is the only record file that `loadRunState` reads, and `runStateFileName` is the name `loadRunState` uses, so removing it exercises `!found`.

No findings.

Calls made:
1. `grep -rn` for the old message text, `waiterlessRecovered` and `waiterlessRuns` across the repo, to see every user of the changed text and episode set. My context held only the two files read earlier, not docs or other packages.
2. `sed -n 426,428p internal/shuttleengine/doc.go`, to check whether the doc quotes the old recovery message. The grep line was truncated and `doc.go` was not in my context.
