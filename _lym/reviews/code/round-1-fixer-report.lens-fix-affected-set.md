```yaml
lens: fix-affected-set
outcome: reported
```

Task: check, through the affected-set lens, that the fixes cover every member of the sets the findings name.

Findings checked by rerunning their searches:
- **F1 (error-return sites of `mapPackages`)**: Every error return after jobs exist now adds the unmeasured jobs to `r.queued`. Those are the plan-loop `ctx.Err()` and `planPackage` errors, the `Limits` error, and the job-loop `ctx.Err()`, `readHead` and `measureJob` errors. The first `HeadSHA` read at `testmap.go:458` fails before any job exists and before `r.head` is set, so the record is now skipped there. `r.audit` errors stay uncounted, which matches the review's rejected item on the audit round.
- **F6 and F11 (sites that describe or read the exit record)**: I found no stale statement. The "writes the exit record" texts at `testmap.go:4`, `testmap.go:72`, `testmap.go:130` and `testcover/doc.go:58` stay true after the change. The other updated sites are `dirs.go:58`, `exitrecord.go:18` and `gatecli/doc.go:1153-1154`. The only consumers of `PlannedHead` are `exitRecord()` and the verb's output envelope. No dobby restart code in the repo reads it yet, so no consumer is left reading an empty value.

Findings left unchecked: F2, F3, F4, F5, F7, F8, F9 and F10. Their premises are single missing tests, not sets I could re-search. The one-line `grep` of `_test.go` files that F10 would need is the only gap, and the tests I did see cover it.

No findings.

Tool calls:
1. `grep` for `exit record|testmap-exit|TestMapExitFileName|PlannedHead|planned_head` over the non-test `.go` and `.md` files, for the F6 and F11 sites. My context had no repo-wide search.
2. `grep` of the `return` lines in `testmap.go`, for the F1 error-return set. It also checked `queued` in `testmapaudit.go`, which showed no matches.
