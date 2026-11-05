```yaml
lens: fix-affected-set
outcome: reported
```

Task: run the fix-affected-set lens on F1, the only finding.

F1's premise is the set of callers of the `readinessState` fallback. I searched for `readinessState(` and `sessionReadiness(` in `internal`. They have three call sites:
- `readinessState` is called once, from `sessionReadiness` (`readiness.go:149`).
- `sessionReadiness` is called from `send.go:239`, the start and send path, which the review says the two hook-state start rows already pin.
- `sessionReadiness` is also called from `session.go:266`, in `Runner.SessionIdle`. The three new rows in `session_test.go` drive this path.

The new rows cover the three cases the review's `fix` key named: a non-zero offset with a finished load turn, a non-zero offset with a stale unmatched turn start, and a zero offset with an empty events file. No caller or reading is left without a pinned row.

No findings.

Calls:
1. `grep -rn "readinessState(\|sessionReadiness(" internal --include=*.go`: to rerun F1's caller-set search, which my context lacked.
