```yaml
lens: fix-affected-set
outcome: reported
```

Task: the fix-affected-set lens, covering the sets under F1 (restart rows) and F2 (the run-shed count readers).

No findings.

- F2: the only non-test readers that turn the recipe count into a duration are the two `runShedDuration` calls at `internal/beamcli/dobbydeps.go:80` and `:84`; both go through `durationFromCount`. `internal/shedrecipe/entries_batten.go:118` and `:126` read the keys only to validate them and are tested in `entries_batten_test.go`. Nothing else duplicates the int, int64 and float64 switch.
- F1: `restartDobby` is used at `runnotice_test.go:355`, `:428` and `:459`. `APIRevives` is written only in `runnotice.go:333` and `:343`. The new row is the only streak-across-restart case.

Calls made:
1. `grep` over `internal`, `contracts` and `cmd` for the two keys, `runShedDuration`, `durationFromCount` and `case int64:`.
2. `grep` over `internal` for `restartDobby`, `APIRevives` and `RunNoticeState{`.
