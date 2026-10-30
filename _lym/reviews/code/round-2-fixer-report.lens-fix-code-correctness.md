```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix for F1 and F2 through the correctness lens. No findings.

- `durationFromCount`: the extracted switch is the same as the old inline one. The recipe has only one Run-Shed row, so returning `fallback` at once changes nothing. Nil or absent, a string, zero and negative counts all reach `return fallback`. The `TestDurationFromCount` rows fail with the `> 0` guards or the fallback removed. The only callers are the two `runShedDuration` calls, and their signature is unchanged.
- F1 restart row (`TestRunNotice_OncePerEpisode`): `Modified` now differs from the saved `Since`; after the restart `runNotice` takes `Since` from the saved memory (`runnotice.go:187`), so no second notice goes out. A dobby that did not read `Since` back would send a second notice, so the row fails with the defect in place.
- F1 streak row (`ARestartBetweenThemKeepsTheCount`): after episode 2 the record holds `APIRevives=2`; the restarted dobby reads it back, episode 3 reaches the streak of 3 and one notice goes out. Without the read-back the count restarts at 1 and no notice goes out, so the row fails with the defect in place. The other two rows leave `restartAfter` at zero.

Tool calls (2):
1. Read `internal/dobbyengine/runnotice.go` lines 160–215 and 322–346 to trace `runNotice` and `apiErrorNoticeDue`. My context did not hold that file.
2. Grepped `runnotice.go` for `Since` and the revive fields, to find those line ranges.
