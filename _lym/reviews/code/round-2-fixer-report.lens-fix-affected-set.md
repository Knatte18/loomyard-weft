```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- F1: `queueShellNudge()` has one call site, `wait.go:936`, the one the fix changed. No other site queues the nudge before `expiredTurnEnd`.
- F2: the grep found no other "both present" or "both output files" wording about Master's shell wait. The hits are unrelated: `runlevel.go:1` and `runlevel.go:768` say "both output files" for the Master spawn, and the rest are tests or other packages. The `runlevel.go:1` and `runlevel.go:768` hits pin a count of output files too. They sit outside the paragraph and the finding the review named, so I'm only noting them.

Tool calls used:
1. `grep queueShellNudge()` over `internal` Go files: to list every call site of the F1 set, which my context lacked.
2. `grep` for "both present" and related phrasing: to find twins of the F2 phrase, which my context lacked.
